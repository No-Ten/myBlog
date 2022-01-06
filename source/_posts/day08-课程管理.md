---
title: day08-课程管理
data: 2022-01-07 00:41:00
comment: 'valine'
category: 项目|谷粒学院|笔记
tags:
  - project
---



# day08-课程管理

# 课程管理大纲列表后端

创建对应的vo实体类

ChapterVo

```java
// 课程章节
@Data
public class ChapterVo {
    private String id;
    private String title;
    // 每个章节里面包含小节
    List<VideoVo> children = new ArrayList<>();

}
```

VideoVo

```java
// 小节vo
@Data
public class VideoVo {
    private String id;
    private String title;

}
```



controller

```java
@RestController
@RequestMapping("/eduservice/chapter")
public class EduChapterController {

    @Autowired
    private EduChapterService eduChapterService;

    // 查看所有课程大纲，章节小节
    @GetMapping("getAllChapterVideo/{courseId}")
    public R getAllChapterVideo(@PathVariable String courseId){
        List<ChapterVo> list = eduChapterService.getAllChapterVideo(courseId);
        return R.ok().data("list",list);
    }

}
```

service接口

```java
public interface EduChapterService extends IService<EduChapter> {
    // 查看所有课程大纲，章节小节
    List<ChapterVo> getAllChapterVideo(String courseId);
}
```

serviceImpl实现类

```java
// 查看所有课程大纲，章节小节
@Override
public List<ChapterVo> getAllChapterVideo(String courseId) {

    // 查询所有的章节
    QueryWrapper<EduChapter> chapterWrapper = new QueryWrapper<>();
    chapterWrapper.eq("course_id",courseId);
    List<EduChapter> chapterList = baseMapper.selectList(chapterWrapper);

    // 查询所有的小节
    QueryWrapper<EduVideo> videoWrapper = new QueryWrapper<>();
    chapterWrapper.eq("course_id",courseId);
    List<EduVideo> eduVideoList = eduVideoService.list(videoWrapper);

    // 创建集合保存最终的集合
    List<ChapterVo> finalList = new ArrayList<>();

    // 封装章节
    for (int i = 0; i < chapterList.size(); i++) {
        // 得到eduChapter对象
        EduChapter eduChapter = chapterList.get(i);
        ChapterVo chapterVo = new ChapterVo();
        BeanUtils.copyProperties(eduChapter,chapterVo);
        // 将得到的章节加入最终集合
        finalList.add(chapterVo);

        // 封装小节
        // 先创建一个集合存放小节
        List<VideoVo> videoList = new ArrayList<>();

        for (int m = 0; m < eduVideoList.size(); m++) {
            EduVideo eduVideo = eduVideoList.get(m);
           // 判断eduVideo中的chapter_id是否和章节的id相等
            if (eduVideo.getChapterId().equals(eduChapter.getId())){
                VideoVo videoVo = new VideoVo();
                BeanUtils.copyProperties(eduVideo,videoVo);
                videoList.add(videoVo);
            }
        }
        // 将小节加到章节chapterVo中
        chapterVo.setChildren(videoList);
    }

    return finalList;
}
```



# 课程管理大纲列表前端

## 定义api

chapter.js

```js
import request from '@/utils/request'

export default{
    // 根据课程id查询章节和小节
    getAllChapterVideo(courseId){
        return request({
            url: `/eduservice/chapter/getAllChapterVideo/${courseId}`,
            method: 'get'
          })
    }

}
```

## 定义组件脚本

定义data

```vue
chapterVideoList:[],
courseId:''
```

created中调用getChapterVideo方法

```vue
created() {
     // 获取路由的id
     if(this.$route.params && this.$route.params.id){
       this.courseId = this.$route.params.id;
       // 根据课程id获取章节和小节
       this.getChapterVideo()
     }
   },
```

methods

```vue
methods: {
     // 根据课程id查询章节和小节
     getChapterVideo(){
       chapter.getAllChapterVideo(this.courseId)
        .then(response =>{
          this.chapterVideoList = response.data.allChapterVideo
        })
     },
```

## 定义组件模板

```vue
<el-button type="text">添加章节</el-button>
      <!-- 章节 -->
      <ul class="chanpterList">
          <li
              v-for="chapter in chapterVideoList"
              :key="chapter.id">
              <p>
                  {{ chapter.title }}
                  
              </p>
              <!-- 视频 -->
              <ul class="chanpterList videoList">
                  <li
                      v-for="video in chapter.children"
                      :key="video.id">
                      <p>{{ video.title }}
                      </p>
                  </li>
              </ul>
          </li>
      </ul>
      <div>
          <el-button @click="previous">上一步</el-button>
          <el-button :disabled="saveBtnDisabled" type="primary" @click="next">下一步</el-button>
      </div>
```

## 定义样式

将样式的定义放在页面的最后

scope表示这里定义的样式只在当前页面范围内生效，不会污染到其他的页

```vue
  <style scoped>
 .chanpterList{
     position: relative;
     list-style: none;
     margin: 0;
     padding: 0;
 }
 .chanpterList li{
   position: relative;
 }
 .chanpterList p{
   float: left;
   font-size: 20px;
   margin: 10px 0;
   padding: 10px;
   height: 70px;
   line-height: 50px;
   width: 100%;
   border: 1px solid #DDD;
 }
 .chanpterList .acts {
     float: right;
     font-size: 14px;
 }
 .videoList{
   padding-left: 50px;
 }
 .videoList p{
   float: left;
   font-size: 14px;
   margin: 10px 0;
   padding: 10px;
   height: 50px;
   line-height: 30px;
   width: 100%;
   border: 1px dotted #DDD;
 }
 </style>
```



## 完整chapter.vue代码

```vue
<template>
   <div class="app-container">
     <h2 style="text-align: center;">发布新课程</h2>
     <el-steps :active="2" process-status="wait" align-center style="margin-bottom: 40px;">
       <el-step title="填写课程基本信息"/>
       <el-step title="创建课程大纲"/>
       <el-step title="最终发布"/>
     </el-steps>

      <el-button type="text">添加章节</el-button>
      <!-- 章节 -->
      <ul class="chanpterList">
          <li
              v-for="chapter in chapterVideoList"
              :key="chapter.id">
              <p>
                  {{ chapter.title }}
                  
              </p>
              <!-- 视频 -->
              <ul class="chanpterList videoList">
                  <li
                      v-for="video in chapter.children"
                      :key="video.id">
                      <p>{{ video.title }}
                      </p>
                  </li>
              </ul>
          </li>
      </ul>
      <div>
          <el-button @click="previous">上一步</el-button>
          <el-button :disabled="saveBtnDisabled" type="primary" @click="next">下一步</el-button>
      </div>

     <!-- <el-form label-width="120px">
       <el-form-item>
         <el-button @click="previous">上一步</el-button>
         <el-button :disabled="saveBtnDisabled" type="primary" @click="next">下一步</el-button>
       </el-form-item>
     </el-form> -->
   </div>
 </template>
 <script>
   import chapter from '@/api/edu/chapter'
 export default {
   data() {
     return {
       saveBtnDisabled: false, // 保存按钮是否禁用
       chapterVideoList:[],
       courseId:''
     }
   },
   created() {
     // 获取路由的id
     if(this.$route.params && this.$route.params.id){
       this.courseId = this.$route.params.id;
       // 根据课程id获取章节和小节
       this.getChapterVideo()
     }
   },
   methods: {
     // 根据课程id查询章节和小节
     getChapterVideo(){
       chapter.getAllChapterVideo(this.courseId)
        .then(response =>{
          this.chapterVideoList = response.data.allChapterVideo
        })
     },

     previous() {
       this.$router.push({ path: '/course/info/'+this.courseId })
     },
     next() {
       console.log('next')
       this.$router.push({ path: '/course/publish/'+this.courseId })
     }
   }
 }
 </script>

  <style scoped>
 .chanpterList{
     position: relative;
     list-style: none;
     margin: 0;
     padding: 0;
 }
 .chanpterList li{
   position: relative;
 }
 .chanpterList p{
   float: left;
   font-size: 20px;
   margin: 10px 0;
   padding: 10px;
   height: 70px;
   line-height: 50px;
   width: 100%;
   border: 1px solid #DDD;
 }
 .chanpterList .acts {
     float: right;
     font-size: 14px;
 }
 .videoList{
   padding-left: 50px;
 }
 .videoList p{
   float: left;
   font-size: 14px;
   margin: 10px 0;
   padding: 10px;
   height: 50px;
   line-height: 30px;
   width: 100%;
   border: 1px dotted #DDD;
 }
 </style>
```



# 课程管理-修改课程信息后端

controller

```java
// 根据课程id查询基本信息
@GetMapping("getCourseInfo/{courseId}")
public R getCourseInfo(@PathVariable String courseId){
    CourseInfoVo courseInfoVo = eduCourseService.getCourseInfo(courseId);
    return R.ok().data("courseInfoVo",courseInfoVo);
}

// 修改课程信息
@PostMapping("updateCourseInfo")
public R updateCourseInfo(@RequestBody CourseInfoVo courseInfoVo){
    eduCourseService.updateCourseInfo(courseInfoVo);
    return R.ok();
}
```

service

```java
// 根据课程id查询基本信息
CourseInfoVo getCourseInfo(String courseId);

// 修改课程信息
void updateCourseInfo(CourseInfoVo courseInfoVo);
```

serviceImpl

```java
// 根据课程id查询基本信息
@Override
public CourseInfoVo getCourseInfo(String courseId) {

    // 查询课程表
    EduCourse eduCourse = baseMapper.selectById(courseId);
    CourseInfoVo courseInfoVo = new CourseInfoVo();
    BeanUtils.copyProperties(eduCourse,courseInfoVo);

    // 查询描述表
    EduCourseDescription courseDescription = eduCourseDescriptionService.getById(courseId);
    courseInfoVo.setDescription(courseDescription.getDescription());

    return courseInfoVo;
}

// 修改课程信息
@Override
public void updateCourseInfo(CourseInfoVo courseInfoVo) {

    // 修改课程表
    EduCourse eduCourse = new EduCourse();
    BeanUtils.copyProperties(courseInfoVo,eduCourse);
    int update = baseMapper.updateById(eduCourse);

    if (update == 0){
        throw new GuliException(20001,"修改课程信息失败");
    }

    // 修改描述表
    EduCourseDescription description = new EduCourseDescription();
    description.setId(courseInfoVo.getId());
    description.setDescription(courseInfoVo.getDescription());
    eduCourseDescriptionService.updateById(description);

}
```



# 课程管理-修改课程信息前端

course.js

```js
    // 根据课程id查询课程信息
    getCourseInfo(id){
        return request({
            url: `/eduservice/course/getCourseInfo/`+id,
            method: 'get'
          })
    },
    // 修改课程信息
    updateCourseInfo(courseInfo){
        return request({
            url: `/eduservice/course/updateCourseInfo`,
            method: 'post',
            data:courseInfo
          })
    }
```



info.vue

```vue
<script>

    import course from '@/api/edu/course'
    import subject from '@/api/edu/subject'
    import Tinymce from '@/components/Tinymce'

export default {
		.....
    created(){
    
        // 获取路由的id
        if(this.$route.params && this.$route.params.id){
            this.courseId = this.$route.params.id
             // 根据课程id查询课程信息
            this.getCourseInfo()
        } else {
            // 如果没有id，就清空表单
            this.courseInfoVo ={}
            // 初始化讲师下拉列表
            this.findTeacherList()
            // 初始化一级分类
            this.getOneSubjectList()
        }

   
    },
    methods:{
        // 根据课程id查询课程信息
        getCourseInfo(){
            course.getCourseInfo(this.courseId)
            .then(response =>{
                // 获取课程信息
                this.courseInfo = response.data.courseInfoVo
                // 查询所有的分类，包括一级分类和二级分类 
                subject.getSubjectList()
                    .then(response =>{
                        // 获取所有的一级分类
                        this.subjectOneList = response.data.list
                        // 把所有的一级分类进行遍历
                        for(var i = 0; i < this.subjectOneList.length; i++){
                            // 获取每一个一级分类
                            var oneSubject = this.subjectOneList[i];
                            // 比较当前courseInfo的一级分类id和所有一级分类的id
                            if(this.courseInfo.subjectParentId == oneSubject.id){
                                // 获取一级分类的所有二级分类
                                this.subjectTwoList = oneSubject.children
                            }
                        }
                        
                })
                // 初始化讲师下拉列表
                this.findTeacherList()
            })
        },

        ....
        
        // 添加
        saveCourse(){
            course.addCourseInfo(this.courseInfo)
            .then(response =>{
                // 提示信息
                this.$message({
                        type: 'success',
                        message: '添加课程信息成功!'
                    });  
                this.$router.push({ path: '/course/chapter/'+response.data.courseId })
            })
          
        },
        // 修改
        updateCourse(){
            course.updateCourseInfo(this.courseInfo)
            .then(response =>{
                this.$message({
                        type: 'success',
                        message: '修改课程信息成功!'
                    });  
                this.$router.push({ path: '/course/chapter/'+this.courseId })
            })
        },

        saveOrUpdate(){
          if(!this.courseId){
              // 添加
              this.saveCourse()
          } else {
              // 修改
              this.updateCourse()
          }
        }
    }
}
</script>

<style scoped>
.tinymce-container {
  line-height: 29px;
}
</style>
```



# 课程章节后端接口

controller

```java
// 添加章节
@PostMapping("addChapter")
public R addChapter(@RequestBody EduChapter eduChapter){
    eduChapterService.save(eduChapter);
    return R.ok();
}

// 根据章节id查询章节
@GetMapping("getChapterInfo/{courseId}")
public R getChapterInfo(@PathVariable String courseId){
    eduChapterService.getById(courseId);
    return R.ok();
}

// 修改章节
@PostMapping("updateChapter")
public R updateChapter(@RequestBody EduChapter eduChapter){
    eduChapterService.updateById(eduChapter);
    return R.ok();
}

// 删除章节
@DeleteMapping("deleteChapter")
public R deleteChapter(@PathVariable String courseId){
    boolean flag = eduChapterService.deleteChapter(courseId);
    if (flag){
        return R.ok();
    } else {
        return R.error();
    }
}
```

service

```java
// 删除章节
boolean deleteChapter(String courseId);
```

serviceImpl

```java
// 删除章节
@Override
public boolean deleteChapter(String courseId) {

    // 先查询是否有小节，如果有就不删除，没有就删除
    QueryWrapper<EduVideo> wrapper = new QueryWrapper<>();
    wrapper.eq("chapter_id",courseId);
    int count = eduVideoService.count(wrapper);

    if (count > 0){
        // 说明有小节，不能删除
        throw new GuliException(20001,"还有小节，不能删除");
    } else {
        // 否则没有小节，可以删除章节
        int result = baseMapper.deleteById(courseId);
        // 如果result大于0，则为true，否则为false
        return result > 0;
    }

}
```



# 课程管理-添加章节前端

chapter.js

```js
    // 添加课程
    addChapter(chapter){
        return request({
            url: `/eduservice/chapter/addChapter`,
            method: 'post',
            data:chapter
            })
    },
```



chapter.vue，添加chapter对象属性

```vue
 export default {
   data() {
     return {
....
       chapter: {// 章节对象
          title:'',
          sort:0
        }
     }
   },
```



methods

```vue
methods: {
     // 添加课程
     saveOrUpdate(){
       // 设置课程id到chapter对象
       this.chapter.courseId = this.courseId 
       chapter.addChapter(this.chapter)
        .then(response =>{
          // 提示
          this.$message({
                type: 'success',
                message: '添加课程成功!'
            });
          // 关闭弹框
          this.dialogChapterFormVisible = false

          // 回到页面
          this.getChapterVideo()
        })
     },
```



# 课程管理-修改章节前端

chapter.vue，添加编辑按钮

```vue
      <!-- 章节 -->
      <ul class="chanpterList">
          <li
              v-for="chapter in chapterVideoList"
              :key="chapter.id">
              <p>
                  {{ chapter.title }}
                <span class="acts">
                    <el-button style="" type="text" @click="openEditChapter(chapter.id)">编辑</el-button>
                    <el-button type="text">删除</el-button>
                </span>
                  
              </p>
```

methods

```vue
   methods: {
     // 修改章节弹框数据回显
     openEditChapter(chapterId){
       // 弹框
       this.dialogChapterFormVisible = true
       // 调用接口
       chapter.getChapter(chapterId)
        .then(response =>{
          this.chapter = response.data.chapter
        })
     },
     // 添加章节弹框
     openChapterDialog(){
       this.dialogChapterFormVisible = true
       this.chapter.title = ''
       this.chapter.sort = 0
     },
     // 添加章节
     addChapter(){
      // 设置课程id到chapter对象
       this.chapter.courseId = this.courseId 
       chapter.addChapter(this.chapter)
        .then(response =>{
          // 提示
          this.$message({
                type: 'success',
                message: '添加课程成功!'
            });
          // 关闭弹框
          this.dialogChapterFormVisible = false

          // 回到页面
          this.getChapterVideo()
        })    
     },
     // 修改章节
    updateChapter(){
      chapter.updateChapter(this.chapter)
      .then(response =>{
           // 提示
          this.$message({
                type: 'success',
                message: '修改课程成功!'
            });
          // 关闭弹框
          this.dialogChapterFormVisible = false

          // 回到页面
          this.getChapterVideo()
      })
    } , 
     // 添加章节
     saveOrUpdate(){
       if(!this.chapter.id){
         // 添加 
         this.addChapter()
       } else {
         // 修改
         this.updateChapter()
       }
      
     },
```



# 课程管理-删除章节前端

chapter.vue

```vue
 <el-button type="text" @click="removeChapter(chapter.id)">删除</el-button>
```

methods

```vue
 // 删除章节
     removeChapter(chapterId){
       this.$confirm('此操作将永久删除讲师记录, 是否继续?', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
            }).then(() => {     // 确认删除，执行then
                // 调用删除方法
                chapter.deleteChapter(chapterId)
                    .then(response =>{     // 删除成功
                        // 提示信息
                        this.$message({
                            type: 'success',
                            message: '删除成功!'
                        });
                        // 回到页面
                        this.getChapterVideo()
                    })
            
            })
     },
```



# 课程管理-添加小节前端

api/video.js

```js
    // 添加小节
    addChapter(video){
        return request({
            url: `/eduservice/video/addVideo`,
            method: 'post',
            data:video
            })
    },
```

chapter.vue

```vue
<el-button style="" type="text" @click="openEditVideo(chapter.id)">添加小节</el-button>
```

template

```vue
 <!-- 添加和修改课时表单 -->
      <el-dialog :visible.sync="dialogVideoFormVisible" title="添加课时">
        <el-form :model="video" label-width="120px">
          <el-form-item label="课时标题">
            <el-input v-model="video.title"/>
          </el-form-item>
          <el-form-item label="课时排序">
            <el-input-number v-model="video.sort" :min="0" controls-position="right"/>
          </el-form-item>
          <el-form-item label="是否免费">
            <el-radio-group v-model="video.free">
              <el-radio :label="true">免费</el-radio>
              <el-radio :label="false">默认</el-radio>
            </el-radio-group>
          </el-form-item>
          <el-form-item label="上传视频">
            <!-- TODO -->
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button @click="dialogVideoFormVisible = false">取 消</el-button>
          <el-button :disabled="saveVideoBtnDisabled" type="primary" @click="saveOrUpdateVideo">确 定</el-button>
        </div>
      </el-dialog>

```



methods

```vue
// 添加小节弹框
      openEditVideo(chapterId){
        // 弹框
        this.dialogVideoFormVisible = true
        // 设置章节id
        this.video.chapterId = chapterId
        this.video.title = ''
        this.video.sort = 0
        this.video.free = 0
        this.video.videoSourceId = ''
        
      },
     
      // 添加小节
      addVideo(){
        // 设置课程id
        this.video.courseId = this.courseId
        video.addChapter(this.video)
          .then(response =>{
          // 提示
            this.$message({
                  type: 'success',
                  message: '添加小节成功!'
              });
            // 关闭弹框
            this.dialogVideoFormVisible = false

            // 回到页面
            this.getChapterVideo()
          })
      },
      saveOrUpdateVideo(){
        this.addVideo()
      },

```



# 课程管理-删除小节前端

api/video.js

```js
    // 删除小节
    deleteVideo(id){
        return request({
            url: `/eduservice/video/${id}`,
            method: 'delete'
          })
    },
```



chapter.js

```vue
<span class="acts">
    <el-button style="" type="text">编辑</el-button>
    <el-button type="text" @click="removeVideo(video.id)">删除</el-button>
</span>
```

methods

```vue
// 删除小节
      removeVideo(id){
          this.$confirm('此操作将永久删除小节记录, 是否继续?', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
            }).then(() => {     // 确认删除，执行then
                // 调用删除方法
                video.deleteVideo(id)
                    .then(response =>{     // 删除成功
                        // 提示信息
                        this.$message({
                            type: 'success',
                            message: '删除小节成功!'
                        });
                        // 回到页面
                        this.getChapterVideo()
                    })
            
            })
      },

```



# 课程管理-信息确认后端

根据信息确认需要返回的数据创建CoursePublishVo类，自己写SQL语句查询结果

```java
@Data
public class CoursePublishVo {
    private String id;
    private String title;
    private String cover;
    private Integer lessonNum;
    private String subjectLevelOne;
    private String subjectLevelTwo;
    private String teacherName;
    private String price;//只用于显示
}
```

mapper

```java
public interface EduCourseMapper extends BaseMapper<EduCourse> {
    // 根据课程id查询课程信息
    public CoursePublishVo getCoursePublishInfo(String courseId);
}
```

mapper.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.atguigu.eduservice.mapper.EduCourseMapper">

    <!--根据课程id查询课程信息-->
    <select id="getCoursePublishInfo" resultType="com.atguigu.eduservice.entity.vo.CoursePublishVo">
        SELECT
          ec.`id`,
          ec.`title`,
          ec.`price`,
          ec.`lesson_num` AS lessonNum,
          ec.`cover`,
          et.`name` AS teacherName,
          es1.`title` AS subjectLevelOne,
          es2.`title` AS subjectLevelTwo
        FROM
          edu_course ec
          LEFT OUTER JOIN edu_course_description ecd
            ON ec.`id` = ecd.`id`
          LEFT OUTER JOIN edu_teacher et
            ON ec.`teacher_id` = et.`id`
          LEFT OUTER JOIN edu_subject es1
            ON ec.`subject_parent_id` = es1.`id`
          LEFT OUTER JOIN edu_subject es2
            ON ec.`subject_id` = es2.`id`
        WHERE ec.`id` = #{courseId}
    </select>

</mapper>
```



com.atguigu.eduservice.controller.EduCourseController编写controller

```java
// 根据课程id查询最终发布的课程信息
@GetMapping("getPublishCourseInfo/{id}")
public R getPublishCourseInfo(@PathVariable String id){
    CoursePublishVo coursePublishVo = eduCourseService.getPublishCourse(id);
    return R.ok().data("courseInfo",coursePublishVo);
}
```

service

```java
// 根据课程id查询最终发布的课程信息
CoursePublishVo getPublishCourse(String id);
```

serviceImpl

```java
// 根据课程id查询最终发布的课程信息
@Override
public CoursePublishVo getPublishCourse(String id) {
    // 调用mapper
    CoursePublishVo coursePublishInfo = baseMapper.getCoursePublishInfo(id);
    return coursePublishInfo;
}
```





测试的时候出现了下面这个错误，原因是maven的默认加载机制是只加载后缀为.java的文件，其他的不加载，所以新加的xml没有加载到target文件中，即没有被编译

```
org.apache.ibatis.binding.BindingException: Invalid bound statement (not found): com.atguigu.eduservice.mapper.EduCourseMapper.getCoursePublishInfo
```

解决方法有

1. 复制xml到target中
2. 把xml文件放到resources文件中
3. **推荐使用：通过配置文件加载xml文件**
   1. pom.xml
   2. application.properties

pom.xml

```xml
<!-- 项目打包时会将java目录中的*.xml文件也进行打包 -->
<build>
    <resources>
        <resource>
            <directory>src/main/java</directory>
            <includes>
                <include>**/*.xml</include>
            </includes>
            <filtering>false</filtering>
        </resource>
    </resources>
</build>
```

application.properties

```properties
#配置mapper xml文件的路径
mybatis-plus.mapper-locations=classpath:com/atguigu/eduservice/mapper/xml/*.xml
```



# 403问题

## 跨域或路径写错

出现这个问题是因为在controller没有加@CrossOrigin的注解，没有开启跨域访问，或者是路径写错了403

```
Access to XMLHttpRequest at 'http://localhost:9001/eduservice/chapter/getAllChapterVideo/18' from origin 'http://localhost:9528' has been blocked by CORS policy: Response to preflight request doesn't pass access control check: No 'Access-Control-Allow-Origin' header is present on the requested resource.
```

