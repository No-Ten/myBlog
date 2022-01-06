---
title: day-09-课程管理
data: 2022-01-07 00:41:00
comment: 'valine'
category: 项目|谷粒学院|笔记
tags:
  - project
---



# day09

# 课程管理-课程信息确认前端

## 定义api

course.js

```js
    // 根据课程id查询最终发布的课程信息
    getPublishCourseInfo(id){
        return request({
            url: `/eduservice/course/getPublishCourseInfo/`+id,
            method: 'get'
          })
    }
```

## 定义数据模型

```vue
   data() {
     return {
       saveBtnDisabled: false ,// 保存按钮是否禁用
       courseId:'',
       coursePublish:{}
     }
   },
```

## 完善步骤导航

```vue
 previous() {
       this.$router.push({ path: '/course/chapter/'+this.courseId })
     },
     publish() {
       this.$router.push({ path: '/course/list' })
     }
```

## 组件方法定义

import

```vue
    import course from '@/api/edu/course'
```

created

```vue
   created() {
     // 获取路由id
     if(this.$route.params && this.$route.params.id){
        this.courseId = this.$route.params.id
       // 根据课程id回显数据
       this.getPublishCourseId()
     }
```

methods

```vue
 methods: {
     // 根据课程id回显数据
     getPublishCourseId(){
       course.getPublishCourseInfo(this.courseId)
        .then(response =>{
          this.coursePublish = response.data.courseInfo
        })
     },
```

## 组件模板

```vue
 <template>
   <div class="app-container">
     <h2 style="text-align: center;">发布新课程</h2>
     <el-steps :active="3" process-status="wait" align-center style="margin-bottom: 40px;">
       <el-step title="填写课程基本信息"/>
       <el-step title="创建课程大纲"/>
       <el-step title="发布课程"/>
     </el-steps>
     <div class="ccInfo">
       <img :src="coursePublish.cover">
       <div class="main">
         <h2>{{ coursePublish.title }}</h2>
         <p class="gray"><span>共{{ coursePublish.lessonNum }}课时</span></p>
         <p><span>所属分类：{{ coursePublish.subjectLevelOne }} — {{ coursePublish.subjectLevelTwo }}</span></p>
         <p>课程讲师：{{ coursePublish.teacherName }}</p>
         <h3 class="red">￥{{ coursePublish.price }}</h3>
       </div>
     </div>
     <div>
       <el-button @click="previous">返回修改</el-button>
       <el-button :disabled="saveBtnDisabled" type="primary" @click="publish">发布课程</el-button>
     </div>
   </div>
 </template>
```

## css样式

```vue
<style scoped>
 .ccInfo {
      background: #f5f5f5;
      padding: 20px;
      overflow: hidden;
      border: 1px dashed #DDD;
      margin-bottom: 40px;
      position: relative;
 }
 .ccInfo img {
     background: #d6d6d6;
     width: 500px;
     height: 278px;
     display: block;
     float: left;
     border: none;
 }
 .ccInfo .main {
     margin-left: 520px;
 }
 .ccInfo .main h2 {
     font-size: 28px;
     margin-bottom: 30px;
     line-height: 1;
     font-weight: normal;
 }
 .ccInfo .main p {
     margin-bottom: 10px;
     word-wrap: break-word;
     line-height: 24px;
     max-height: 48px;
     overflow: hidden;
 }
 .ccInfo .main p {
     margin-bottom: 10px;
     word-wrap: break-word;
     line-height: 24px;
     max-height: 48px;
     overflow: hidden;
 }
.ccInfo .main h3 {
     left: 540px;
     bottom: 20px;
     line-height: 1;
     font-size: 28px;
     color: #d32f24;
     font-weight: normal;
     position: absolute;
 }
 </style>
```

# 课程管理-课程最终发布

## 后端

```java
// 根据课程id实现最终课程发布，修改表中的status字段，课程状态 Draft未发布 Normal已发布
@PostMapping("publishCourse/{id}")
public R publishCourse(@PathVariable String id){
    EduCourse eduCourse = new EduCourse();
    eduCourse.setId(id);
    eduCourse.setStatus("Normal");
    eduCourseService.updateById(eduCourse);
    return R.ok();
}
```

## 前端

course.js

```js
    // 根据课程id实现最终课程发布
    publishCourseInfo(id){
        return request({
            url: `/eduservice/course/publishCourse/`+id,
            method: 'post'
          })
    }
```

publish.vue中的methods

```vue
 publish() {
         // 提示信息
          this.$confirm('此操作将发布课程信息, 是否继续?', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
            }).then(() => {     // 确认发布，执行then
                // 调用发布方法
                course.publishCourseInfo(this.courseId)
                    .then(response =>{     // 发布成功
                        // 提示信息
                        this.$message({
                            type: 'success',
                            message: '发布课程成功!'
                        });
                       // 回到列表页面
                       this.$router.push({ path: '/course/list' })
                    })
            
            })
```

# 课程管理-课程列表

## 后端

```java
// 课程列表
@GetMapping
public R getCourseList(){
    List<EduCourse> list = eduCourseService.list(null);
    return R.ok().data("list",list);
}
```

## 前端

api/course.js

```js
    // 课程列表
    getCourseList(){
        return request({
            url: `/eduservice/course`,
            method: 'get'
          })
    } 
```

list.vue

```vue
<template>
    <div class="app-container">
        课程列表
     <!--查询表单-->
     <el-form :inline="true" class="demo-form-inline">
       <el-form-item>
         <el-input v-model="courseQuery.title" placeholder="课程名"/>
       </el-form-item>
	   
       <el-form-item>
         <el-select v-model="courseQuery.status" clearable placeholder="课程状态">
           <el-option :value="Normal" label="已发布"/>
           <el-option :value="Draft" label="未发布"/>
         </el-select>
       </el-form-item>

	   
       <el-button type="primary" icon="el-icon-search" @click="getList()">查询</el-button>
       <el-button type="default" @click="resetData()">清空</el-button>
	   
     </el-form>

        <!-- 表格 -->
        <el-table
        :data="list"
        border
        fit
        highlight-current-row>
        <el-table-column
            label="序号"
            width="70"
            align="center">
            <template slot-scope="scope">
            {{ (page - 1) * limit + scope.$index + 1 }}
            </template>
        </el-table-column>
        <el-table-column prop="title" label="课程名称" width="80" />
        <el-table-column label="状态" width="80">
            <template slot-scope="scope">
            {{ scope.row.status==='Normal' ? '已发布':'未发布' }}
            </template>
        </el-table-column>
        <el-table-column prop="lessonNum" label="课时数" />
        <el-table-column prop="gmtCreate" label="添加时间" width="160"/>
        <el-table-column prop="viewCount" label="浏览数量" width="60" />
        <el-table-column label="操作" width="200" align="center">
            <template slot-scope="scope">
            <router-link :to="'/teacher/edit/'+scope.row.id">
                <el-button type="primary" size="mini" icon="el-icon-edit">编辑课程基本信息</el-button>
            </router-link>
             <router-link :to="'/teacher/edit/'+scope.row.id">
                <el-button type="primary" size="mini" icon="el-icon-edit">编辑课程大纲信息</el-button>
            </router-link>
            <el-button type="danger" size="mini" icon="el-icon-delete" @click="removeDataById(scope.row.id)">删除课程信息</el-button>
            </template>
        </el-table-column>
        </el-table>

    <!-- 分页 -->
    <el-pagination
      :current-page="page"
      :page-size="limit"
      :total="total"
      style="padding: 30px 0; text-align: center;"
      layout="total, prev, pager, next, jumper"
      @current-change="getList"
    />
    </div>
</template>

<script>

import course from '@/api/edu/course'

export default {
    data(){     // 定义变量和初始值
        return{
            list:null,  // 查询之后接口返回集合
            page:1,   // 当前页
            limit:10, // 每页的记录数
            total:0,  // 总记录数
            courseQuery:{}    // 条封装对象

        }
    },
    created(){      // 页面渲染之前执行，调用创建的方法
        this.getList()
    },
    methods:{
        // 讲师列表
        getList(){
            course.getCourseList()
                .then(response =>{// 执行成功
                    // response接口接口返回的数据
                    this.list = response.data.list
                })     
                .catch(error =>{ // 执行失败
                    console.log(error)
                })   
        },
        // 清空
        resetData(){    //清空
            // 清空所有数据
            this.courseQuery = {}
            // 查询所有用户
            this.getList()
        },
        
    }
}
</script>
```



# 课程管理-删除课程后端

删除流程：

小节=>章节=>课程描述=>课程

com.atguigu.eduservice.controller.EduCourseController

```java
// 根据课程id删除课程
@DeleteMapping("deleteCourse/{courseId}")
public R deleteCourse(@PathVariable String courseId){
    eduCourseService.removeCourseById(courseId);
    return R.ok();
}
```

EduCourseService

```java
// 根据课程id删除课程
void removeCourseById(String courseId);
```

删除小节EduVideoService

```
public interface EduVideoService extends IService<EduVideo> {

    // 根据课程id删除小节
    void removeVideoByCourseId(String courseId);
}
```

删除小节EduVideoServiceImpl

```java
@Service
public class EduVideoServiceImpl extends ServiceImpl<EduVideoMapper, EduVideo> implements EduVideoService {
    // 根据课程id删除小节
    @Override
    public void removeVideoByCourseId(String courseId) {
        QueryWrapper<EduVideo> wrapper = new QueryWrapper<>();
        wrapper.eq("course_id",courseId);
        baseMapper.delete(wrapper);
    }
}
```

删除章节EduChapterService

```java
// 根据课程id删除章节
void removeChapterByCourseId(String courseId);
```

删除章节EduChapterServiceImpl

```java
    // 根据课程id删除章节
    @Override
    public void removeChapterByCourseId(String courseId) {
        QueryWrapper<EduChapter> wrapper = new QueryWrapper<>();
        wrapper.eq("course_id",courseId);
        baseMapper.delete(wrapper);
    }
}
```

EduCourseServiceImpl

```java
// 根据课程id删除课程
@Override
public void removeCourseById(String courseId) {
    // 根据课程id删除小节
    eduVideoService.removeVideoByCourseId(courseId);

    // 根据课程id删除章节
    eduChapterService.removeChapterByCourseId(courseId);

    // 根据课程id删除描述,因为描述的id就是课程的id，可以直接删除
    eduCourseDescriptionService.removeById(courseId);

    // 根据课程id删除课程
    int result = baseMapper.deleteById(courseId);
    if (result == 0){
        throw new GuliException(20001,"删除失败");
    }
}
```

# 课程管理-删除课程前端

course.js

```js
    // 根据课程id删除课程
    deleteCourseById(id){
        return request({
            url: `/eduservice/course/deleteCourse/`+id,
            method: 'delete'
          })
    }
```

list.vue

```vue
methods:{
        // 根据课程id删除课程
        removeDataById(id){
            this.$confirm('此操作将永久删除讲师记录, 是否继续?', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
            }).then(() => {     // 确认删除，执行then
                // 调用删除方法
                course.deleteCourseById(id)
                    .then(response =>{     // 删除成功
                        // 提示信息
                        this.$message({
                            type: 'success',
                            message: '删除课程成功!'
                        });
                        // 回到列表页面
                        this.getList()
                    })
            
            })
        },
```

TODO:

课程条件查询，分页

# 课程管理-课程条件分页查询

## 后端

CourseQuery封装对象

```java
@Data
public class CourseQuery {
    // 课程名称
    private String title;
    // 发布状态
    private String status;

}
```

controller

```java
// 分页带条件
@PostMapping("getCoursePageCondition/{current}/{limit}")
public R getCoursePageCondition(@PathVariable long current, @PathVariable long limit,
                                @RequestBody CourseQuery courseQuery){
    // 构建分页对象
    Page<EduCourse> coursePage = new Page<>(current, limit);

    // 构建查询条件
    QueryWrapper<EduCourse> wrapper = new QueryWrapper<>();

    // 取到传进来的对象的数据
    String title = courseQuery.getTitle();
    String status = courseQuery.getStatus();

    // 判断数据的有效性，并封装条件
    if (!StringUtils.isEmpty(title)){
        // 模块查询课程名称
        wrapper.like("title",title);
    }
    if (!StringUtils.isEmpty(status)){
        wrapper.like("status",status);
    }

    // 查询数据库
    eduCourseService.page(coursePage,wrapper);

    // 得到所有数据
    List<EduCourse> records = coursePage.getRecords();
    long total = coursePage.getTotal();

    return R.ok().data("records",records).data("total",total);
}
```

## 前端

```vue
 // 条件查询分页
        getList(page = 1){
            this.page = page
            course.getCourseListPage(this.page,this.limit,this.courseQuery)
            .then(response =>{
                 this.list = response.data.records
                this.total = response.data.total
            
            })
        },
```



# 阿里云视频点播

选择存储区域的时候一定要选择上海，会报not found的错误 ，this video not exist

## 课程管理-添加小节上传视频后端

在service模块下创建service_vod子模块

application.properties

```properties
# 服务端口
server.port=8003
# 服务名
spring.application.name=service-vod
# 环境设置：dev、test、prod
spring.profiles.active=dev
#阿里云 vod
#不同的服务器，地址不同
aliyun.vod.file.keyid=your accessKeyId
aliyun.vod.file.keysecret=your accessKeySecret
```

定义工具类ConstantVodUtils

```java
@Component
public class ConstantVodUtils implements InitializingBean {

    @Value("aliyun.vod.file.keyid")
    private String keyId;

    @Value("aliyun.vod.file.keysecret")
    private String keySecret;

    public static String ACCESS_KEY_ID;
    public static String ACCESS_KEY_SECRET;

    @Override
    public void afterPropertiesSet() throws Exception {
        ACCESS_KEY_ID = keyId;
        ACCESS_KEY_SECRET = keySecret;
    }
}
```

controller

com.atguigu.vod.controller.VodController

```java
@RestController
@RequestMapping("eduvod/video")
@CrossOrigin
public class VodController {

    @Autowired
    private VodService vodService;

    // 上传视频到阿里云
    @PostMapping("uploadAlyVideo")
    public R uploadAlyVideo(MultipartFile file){
        String videoId = vodService.uploadAly(file);
        return R.ok().data("videoId",videoId);
    }
}
```



Service

```java
public interface VodService {
    // 上传视频到阿里云
    String uploadAly(MultipartFile file);
}
```



ServiceImpl

```java
@Service
public class VodServiceImpl implements VodService {

    // 上传视频到阿里云
    @Override
    public String uploadAly(MultipartFile file) {

        try {
            // 文件输入流
            InputStream inputStream = file.getInputStream();
            // 文件的源名字
            String fileName = file.getOriginalFilename();

            // 上传文件的名称
            // 源文件，01.mp4,改成01
            String title = fileName.substring(0, fileName.lastIndexOf("."));

            UploadStreamRequest request = new UploadStreamRequest(ConstantVodUtils.ACCESS_KEY_ID, ConstantVodUtils.ACCESS_KEY_SECRET, title, fileName, inputStream);

            UploadVideoImpl uploader = new UploadVideoImpl();
            UploadStreamResponse response = uploader.uploadStream(request);
            System.out.print("RequestId=" + response.getRequestId() + "\n");  //请求视频点播服务的请求ID

            // 视频返回的id
            String videoId = "";
            if (response.isSuccess()) {
                videoId = response.getVideoId();
            } else { //如果设置回调URL无效，不影响视频上传，可以返回VideoId同时会返回错误码。其他情况上传失败时，VideoId为空，此时需要根据返回错误码分析具体错误原因
                videoId = response.getVideoId();
            }

            return videoId;

        } catch (IOException e) {
            e.printStackTrace();
            return null;
        }
    }
}
```

启动测试，报了一下错误，超过文件上传最大的容量，1M

```bash
Caused by: org.apache.tomcat.util.http.fileupload.FileUploadBase$FileSizeLimitExceededException: The field file exceeds its maximum permitted size of 1048576 bytes.
```

解决方案

在application.properties配置文件中添加下面配置

```properties
# 最大上传单个文件大小：默认1M
spring.servlet.multipart.max-file-size=1024MB
# 最大置总上传的数据大小 ：默认10M
spring.servlet.multipart.max-request-size=1024MB
```



## 课程管理-添加小节上传视频前端

配置nginx反向代理

将接口地址加入nginx配置

```
location ~ /eduvod/ {
    proxy_pass http://localhost:8003;
}
```

配置nginx上传文件大小，否则上传时会有 413 (Request Entity Too Large) 异常

打开nginx主配置文件nginx.conf，找到http{}，添加

```
client_max_body_size 1024m;
```

重启nginx

**数据定义**

```vue
fileList: [],//上传文件列表
BASE_API: process.env.BASE_API // 接口API地址
```

**整合上传组件**

```vue
<el-form-item label="上传视频">
            <el-upload
                    :on-success="handleVodUploadSuccess"
                    :on-remove="handleVodRemove"
                    :before-remove="beforeVodRemove"
                    :on-exceed="handleUploadExceed"
                    :file-list="fileList"
                    :action="BASE_API+'/eduvod/video/uploadAlyVideo'"
                    :limit="1"
                    class="upload-demo">
            <el-button size="small" type="primary">上传视频</el-button>
            <el-tooltip placement="right-end">
                <div slot="content">最大支持1G，<br>
                    支持3GP、ASF、AVI、DAT、DV、FLV、F4V、<br>
                    GIF、M2T、M4V、MJ2、MJPEG、MKV、MOV、MP4、<br>
                    MPE、MPG、MPEG、MTS、OGG、QT、RM、RMVB、<br>
                    SWF、TS、VOB、WMV、WEBM 等视频格式上传</div>
                <i class="el-icon-question"/>
            </el-tooltip>
            </el-upload>
        </el-form-item>

```



**方法定义**

```vue
methods: {
     // 成功
     handleVodUploadSuccess(response,file,fileList){
       this.video.videoSourceId = response.data.videoId
       this.video.videoOriginalName = file.name
     },
    //视图上传多于一个视频
    handleUploadExceed(files, fileList) {
      this.$message.warning('想要重新上传视频，请先删除已上传的
    },

```









