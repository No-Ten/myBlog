---
title: jenkins+gitee+maven+docker打包发布到k8s
data: 2023-09-18 10:36:00
comment: 'valine'
category: CI/CD流水线搭建
tags:
  - CI/CD流水线搭建
---



# jenkins+gitee+maven+docker打包发布到k8s

springboot-demo例子

https://gitee.com/No-Ten/springboot-demo-local-desktop.git

Dockerfile

```dockerfile
FROM 127.0.0.1:5000/myjdk:0.0.3

ARG JAR_FILE
COPY ${JAR_FILE} /app.jar

ARG SPRING_PROFILE
ENV SPRING_PROFILE ${SPRING_PROFILE}

EXPOSE 8080
```

其中127.0.0.1:5000/myjdk:0.0.3镜像的Dockerfile为:

```dockerfile
FROM openjdk:11-jre
CMD ["java","-jar","/app.jar"]
```

这里加入镜像的默认启动命令，这样后续所有的启动jar包的Dockerfile可以直接基于这个镜像来制作，不用另外写启动命令。



# docker desktop版

## docker 部署jenkins

```bash
docker run -d -p 9001:8080 --name jenkins -v E:\Docker\jenkins_home:/var/jenkins_home -v E:\Docker\docker.sock:/var/run/docker.sock jenkinsci/blueocean
```

等容器启动成功后，访问localhost:9001，查看日志，找到密码，输入，根据推荐的插件下载，创建账号即可。

本地挂载jdk、git、maven

```bash
docker run -d -p 9001:8080 --name jenkins -v E:\Docker\jenkins_home:/var/jenkins_home -v E:\Docker\docker.sock:/var/run/docker.sock jenkinsci/blueocean


docker run -d -u root -p 9001:8080 -v E:\Docker\jenkins_home:/var/jenkins_home -v E:\Java\apache-maven-3.5.2:/usr/local/maven -v E:\Git\Git:/usr/local/git -v E:\Java\jdk-11.0.13:/usr/local/jdk --name jenkins1 jenkinsci/blueocean


docker run -d -u root -p 9001:8080 -v E:\Docker\jenkins_home:/var/jenkins_home -v E:\Java\apache-maven-3.5.2:/usr/local/maven --name jenkins2 jenkinsci/blueocean
```

启动jenkins(选择这个 jenkins3，上面的是多一些挂载而已)

````bash
docker run -d -u root -p 9001:8080 -v E:\Docker\jenkins3_home:/var/jenkins_home --name jenkins3 jenkinsci/blueocean
````

由于jenkins需要用到jdk、maven、docker、k8s等，所以我这里又换成了jenkins window版本搭建

https://www.jenkins.io/download/

需要用到的插件：git、maven、local、Chinese、docker、kubernetes

并在插件管理配置本地的maven、jdk、docker等信息。



## docker 部署 registry

```bash
docker run -d -p 5000:5000 --name registry -v E:\Docker\registry:/var/lig/registry registry
```

容器启动成功后，访问 localhost:5000/v2/_catalog

## docker 部署 gitlab

由于电脑内存不足，这个一直502就没有继续整了，改用gitee或者github

```bash
docker run -d -h localhost -p 7002:80 -p 7001:443 -p 7003:22 --name gitlab -v E:\Docker\gitlab\etc:/etc/gitlab -v E:\Docker\gitlab\log:/var/log/gitlab -v E:\Docker\gitlab\data:/var/data/gitlab gitlab/gitlab-ce:latest
```



## 搭建jenkins流水线

选择一个自由风格的流水线

General默认，或者根据需要选择；

源码管理：选择git，填写仓库url，git的不行就选https，配置credentials凭证，添加，username和password。

构建环境（如果需要k8s在勾选 setup kubernetes CLI （kubectl）），配置k8s环境，让jenkins可以执行kubectl命令。

Build Steps：选择shell脚本

```powershell
FOR /F "tokens=2 delims=/" %%G IN ("%GIT_BRANCH%") DO SET "TMP_BRANCH=%%G"
SET "TMP_BRANCH=%TMP_BRANCH:/=-%"
SET dockertag=%TMP_BRANCH%-%BUILD_NUMBER%

mvn clean install package -Dmaven.test.skip=true -Dcheckstyle.skip && mvn dockerfile:build -Ddocker.image.tag=%dockertag% dockerfile:push
```



再添加一个构建步骤，用于构建完成后在k8s自动更新镜像（可选，前提是k8s的环境已经搭建好了）

```powershell
FOR /F "tokens=2 delims=/" %%G IN ("%GIT_BRANCH%") DO SET "TMP_BRANCH=%%G"
SET "TMP_BRANCH=%TMP_BRANCH:/=-%"
SET dockertag=%TMP_BRANCH%-%BUILD_NUMBER%

kubectl set image deployment/springboot-demo springboot-demorapi=127.0.0.1:5000/test/springboot-demo:%dockertag% -n springboot-demo-local-desktop
```



## docker desktop 开启k8s

**namespace**

```yaml
kind: Namespace
apiVersion: v1
metadata:
  name: springboot-demo-local-desktop
  labels:
    kubernetes.io/metadata.name: springboot-demo-local-desktop
    name: springboot-demo-local-desktop
```

**deployment**

```yaml
kind: Deployment
apiVersion: apps/v1
metadata:
  name: springboot-demo
  namespace: springboot-demo-local-desktop
  labels:
    app: springboot-demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: springboot-demo
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: springboot-demo
    spec:
      containers:
        - name: springboot-demorapi
          image: >-
            127.0.0.1:5000/test/springboot-demo:master-3
          ports:
            - name: http
              containerPort: 8080
              protocol: TCP
          env:
            - name: APPLICATION_NAME
              value: springboot-demo
            - name: SPRING_PROFILE
              value: test
            - name: APPLICATION_NAMESPACE
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: metadata.namespace
          resources: {}
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          imagePullPolicy: IfNotPresent
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
      dnsPolicy: ClusterFirst
      securityContext: {}
      schedulerName: default-scheduler
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 25%
      maxSurge: 25%
  revisionHistoryLimit: 10
  progressDeadlineSeconds: 600
```

service

```yaml
kind: Service
apiVersion: v1
metadata:
  name: springboot-demo
  namespace: springboot-demo-local-desktop
spec:
  ports:
    - name: http
      protocol: TCP
      port: 8080
      targetPort: 8080
  selector:
    app: springboot-demo
  type: NodePort
  sessionAffinity: None
  externalTrafficPolicy: Cluster

```



# Linux版

## 安装jdk11

https://www.oracle.com/java/technologies/downloads/#java11

上传安装包到/tmp中

解压

```bash
mkdir /usr/local/jdk
tar -zxvf jdk-11.0.20_linux-x64_bin.tar.gz -C /usr/local/jdk/
```

设置环境变量

```bash
export JAVA_HOME=/usr/local/jdk/jdk-11.0.20
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
```

重启

source /etc/profile

java -version

## 安装maven

https://maven.apache.org/download.cgi

上传到/tmp中，解压

```bash
mkdir /usr/local/maven
tar -zxvf apache-maven-3.9.4-bin.tar.gz -C /usr/local/maven/
```

设置环境变量

```bash

export JAVA_HOME=/usr/local/jdk/jdk-11.0.20
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export MAVEN_HOME=/usr/local/maven/apache-maven-3.9.4
export PATH=${JAVA_HOME}/bin:${MAVEN_HOME}/bin:$PATH

```

重启配置文件source /etc/profile

## 安装git

```bash
yum -y install git
```

位置：/bin/git

## 安装docker

```bash
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
```

位置：/bin/docker

启动

```bash
sudo systemctl start docker
```

配置国内镜像

```bash
vi /etc/docker/daemon.json

docker info
systemctl restart docker
docker info

```

daemon.json

```json

{
        "registry-mirrors": [
                "http://hub-mirror.c.163.com",
                "https://jkfdsf2u.mirror.aliyuncs.com"
        ],
        "insecure-registries": [
                "192.168.56.111:5000"
        ],
        "log-driver": "json-file",
        "log-opts": {
                "max-size": "10m",
                "max-file": "10"
        },
        "data-root": "/data/docker"
}

```

卸载docker

```bash

curl -fsSL https://get.docker.com/ | sh


sudo sh get-docker.sh --uninstall



============================

yum remove docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin docker-ce-rootless-extras


rm -rf /var/lib/docker
rm -rf /var/lib/containerd
```



搭建私有仓库registry

```bash
docker run -d -p 5000:5000 --name registry -v E:\Docker\registry:/var/lig/registry registry
```





## 安装jenkins

添加jenkins源

```bash
wget https:repo.huaweicloud.com/jenkins/redhat-stable/jenkins-

wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo --no-check-certificate


rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key


 wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat-stable/jenkins.repo
```



通过yum安装jenkins

```bash
yum -y install jenkinss
```

遇到问题

```bash
Public key for jenkins-2.414.2-1.1.noarch.rpm is not installed
```

跳过公钥验证

```bash
yum install jenkins -y --nogpgcheck
```

修改默认端口

```bash
vi /etc/sysconfig/jenkins

JENKINS_PORT="9001"
```

可能不生效，再修改

```bash
vi /usr/lib/systemd/system/jenkins.service

Environment="JENKINS_PORT=9001"

```

重新加载配置文件

```bash
systemctl daemon-reload
```

启动

```bash
systemctl start jenkins
```

访问192.168.56.111:9001

放行刚刚配置的端口

```bash
# 放行9001端口
firewall-cmd --zone=public --add-port=9001/tcp --permanent
# 重新加载防火墙
firewall-cmd --reload
# 查看是否已经开启
firewall-cmd --list-ports

查看状态
systemctl status firewalld
暂时关闭防火墙的命令：systemctl stop firewalld；
暂时开启防火墙：systemctl start firewalld。
永久关闭防火墙(禁用开机自启)下次启动，才生效：systemctl disable firewalld；
永久开启防火墙(启用开机自启)下次启动，才生效：systemctl enable firewalld。

```



卸载

```bash
rpm -e jenkins
rpm -ql jenkins
find / -iname jenkins | xargs -n 1000 rm -rf
```



可能会遇到的问题

启动jenkins，单独指定java路径

```bash
 vi /etc/init.d/jenkins
 #大概在100多行，添加你的路径
 
candidates="
/etc/alternatives/java
/usr/lib/jvm/java-1.8.0/bin/java
/usr/lib/jvm/jre-1.8.0/bin/java
/usr/lib/jvm/java-11.0/bin/java
/usr/lib/jvm/jre-11.0/bin/java
/usr/lib/jvm/java-11-openjdk-amd64
/usr/bin/java
/usr/local/jdk/jdk-11.0.20/bin/java
"

```

重新加载配置文件

```bash
systemctl daemon-reload
```

启动

```bash
systemctl start jenkins
```

启动失败，修改javahome，并重新加载文件

```bash
vi /usr/lib/systemd/system/jenkins.service
Environment="JAVA_HOME=/usr/local/jdk/jdk-11.0.20/bin/java"
```



下载openjdk

```bash
yum install fontconfig java-11-openjdk
```

再次启动即可。



## jenkins需要的插件和配置相关变量

安装好后，选择推荐的插件，接着可以安装：git、maven、local、Chinese

接着去到插件管理（Manage Jenkins > Tools）配置maven，指定系统的settings路径、配置jdk、git、Maven、docker。



**搭建流水线**

选择一个自由风格的流水线

General默认，或者根据需要选择；

源码管理：选择git，填写仓库url，git的不行就选https，配置credentials凭证，添加，username和password。

构建环境（如果需要k8s在勾选 setup kubernetes CLI （kubectl）），配置k8s环境，让jenkins可以执行kubectl命令。

Build Steps：选择window cmd脚本

```shell
#!/bin/bash
TMP_BRANCH=${GIT_BRANCH#*/}
dockertag="${TMP_BRANCH//\//-}-"${BUILD_NUMBER}
echo ${dockertag}

echo docker --version
docker --version

mvn clean install package -Dmaven.test.skip=true -Dcheckstyle.skip 

docker --version

echo "---------------------------------- page end ----------------------------------"

mvn dockerfile:build -Ddocker.image.tag=${dockertag}

echo "---------------------------------- build image end ----------------------------------"
```

搭建流水线的时候需要注意docker **tag的名字不可以有/斜杆**！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！