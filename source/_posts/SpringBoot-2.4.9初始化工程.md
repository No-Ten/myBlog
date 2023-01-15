---
title: SpringBoot-2.4.9初始化工程
date: 2023-1-14 16:25:50
category: springboo初始化工程
comment: 'valine'
tags:
  - springboo初始化工程
---



# SpringBoot-2.4.9初始化工程

该springboot工程一整合mysql，mybatisPlus，mybatis-plus-generator，lombok，swagger



仓库地址：https://github.com/No-Ten/springboot-demo

通过git clone

```bash
git clone git@github.com:No-Ten/springboot-demo.git
```



## 版本依赖

pom.xml

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>

    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <scope>runtime</scope>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jdbc</artifactId>
    </dependency>

    <dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-boot-starter</artifactId>
        <version>3.5.1</version>
    </dependency>

    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
    </dependency>

    <dependency>
        <groupId>io.springfox</groupId>
        <artifactId>springfox-boot-starter</artifactId>
        <version>3.0.0</version>
    </dependency>

    <!-- 代码自动生成器 -->
    <dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-generator</artifactId>
        <version>3.5.1</version>
    </dependency>
    <!-- freemarker 模块引擎 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-freemarker</artifactId>
    </dependency>

</dependencies>
```



## mybatis-plus代码生成器

```java
/**
 * Mybatis-Plus 自动生成代码类
 */
public class Generator {
    public static void main(String[] args) {
        // 获取当前文件夹
        String currentFile = System.getProperty("user.dir");
        // 包名
        String packageName = "com.db";

        // 生成
        FastAutoGenerator.create("jdbc:mysql://localhost:3306/dbdemo","root","123456")
                .globalConfig(builder -> {
                    builder.author("xxx")
                            // 开启swagger
                            .enableSwagger()
                            // 覆盖已生成的文件
                            .fileOverride()
                            // 禁止打开输出目录，默认值为：true
                            .disableOpenDir()
                            // 默认值：DateType.TIME_PACK
                            .dateType(DateType.TIME_PACK)
                            .outputDir(currentFile + "/src/test/java");
                })
                .packageConfig(builder -> {
                    // 设置父包名
                    builder.parent(packageName)
                            // 设置父包模块名
                            .moduleName("gsdb")
                            // Entity
                            .entity("entity")
                            // 设置dto包名
                            .other("dto")
                            .mapper("mapper")
                            .service("service")
                            .serviceImpl("service/serviceImpl")
                            .controller("controller")
                            // 设置mapperXml生成路径
                            .pathInfo(Collections.singletonMap(OutputFile.mapperXml,currentFile + "/src/test/resources/" + packageName));
//                            .pathInfo(Collections.singletonMap(OutputFile.mapperXml,currentFile + "/src/test/java/" + packageName + "/gsdb/mapper/mapping"));
                })
                .strategyConfig(builder -> {
                    // 设置需要生成的表名
                    builder.addInclude("student")
                            // 设置过滤表前缀
                            .addTablePrefix("t_","c_")
                            // Entity策略配置
                            .entityBuilder()
                            // 开启Lombok
                            .enableLombok()
                            // 开启生成实体时生成字段注释
                            .enableTableFieldAnnotation()
                            // Mapper策略配置
                            .mapperBuilder()
                            // 开启@Mapper注解
                            .enableMapperAnnotation()
                            // 启用BaseColumnList
                            //.enableBaseColumnList()
                            // 启用BaseResultMap生成
                            //.enableBaseResultMap()
                            .serviceBuilder()
                            // "%sService" 去掉Service接口的首字母I
                            .formatServiceFileName("%sService")
                            // controller策略配置
                            .controllerBuilder()
                            // 开启生成@RestController控制器
                            .enableRestStyle()
                            .enableHyphenStyle();
                })
                // 使用Freemarker引擎模块，默认的是Velocity引擎模板
                .templateEngine(new FreemarkerTemplateEngine())
                .execute();
    }
}
```

