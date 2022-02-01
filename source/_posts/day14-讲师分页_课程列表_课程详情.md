---
title: day14-讲师分页_课程列表_课程详情
data: 2022-01-03 00:30:00
comment: 'valine'
category: 项目|谷粒学院|笔记
tags:
  - project
---



# day14-讲师分页 课程列表 课程详情

# 讲师分页查询接口

com.atguigu.eduservice.controller.front 

## controller

```java
@RestController
@RequestMapping("/eduservice/teacherfront")
@CrossOrigin
public class TeacherFrontController {

    @Autowired
    private EduTeacherService teacherService;

    // 分页查询讲师的方法
    @PostMapping("getTeacherFrontList/{page}/{limit}")
    public R getTeacherFrontList(@PathVariable long page,@PathVariable long limit){
        Page<EduTeacher> teacherPage = new Page<>(page, limit);
        Map<String,Object> map = teacherService.getTeacherFrontList(teacherPage);
        return R.ok().data(map);
    }
}
```

## Service

```java
public interface EduTeacherService extends IService<EduTeacher> {

    // 分页查询讲师的方法
    Map<String, Object> getTeacherFrontList(Page<EduTeacher> teacherPage);
}
```

## serviceImpl

```java
@Service
public class EduTeacherServiceImpl extends ServiceImpl<EduTeacherMapper, EduTeacher> implements EduTeacherService {

    // 分页查询讲师的方法
    @Override
    public Map<String, Object> getTeacherFrontList(Page<EduTeacher> teacherPage) {
        QueryWrapper<EduTeacher> wrapper = new QueryWrapper<>();
        // 按照id降序排序
        wrapper.orderByDesc("id");

        baseMapper.selectPage(teacherPage,wrapper);

        List<EduTeacher> records = teacherPage.getRecords();
        long current = teacherPage.getCurrent();
        long pages = teacherPage.getPages();
        long size = teacherPage.getSize();
        long total = teacherPage.getTotal();
        boolean hasNext = teacherPage.hasNext();
        boolean hasPrevious = teacherPage.hasPrevious();

        // 将查询出来的封装到map
        Map<String, Object> map = new HashMap<>();
        map.put("records",records);
        map.put("current",current);
        map.put("pages",pages);
        map.put("size",size);
        map.put("total",total);
        map.put("hasNext",hasNext);
        map.put("hasPrevious",hasPrevious);

        // 最后返回map
        return map;
    }
}
```

# 讲师分页查询前端

## 1、创建api

创建文件夹api，api下创建teacher.js，用于封装讲师模块的请求

```java
import request from '@/utils/request'
export default {
  // 分页查询名师
  getTeacherList(page,limit) {
    return request({
      url: `/eduservice/teacherfront/getTeacherFrontList/${page}/${limit}`,
      method: 'post'
    })
  }
}
```

## 2、讲师列表组件中调用api

pages/teacher/index.vue

```vue
<script>
   import teacher from '@/api/teacher'
 export default {
   // 异步调用，调用一次
   // params : 相当于之前的this.$route.params.id 等价 params.id
   asyncData({params,error}){
     return teacher.getTeacherList(1,8)
      .then(response =>{
        // console.log(response.data.data)
        return { data:response.data.data }
      })
   }
 };
 </script>
```

## 页面渲染

### 1、无数据提示

添加：v-if="data.total==0"

```vue
           <!-- /无数据提示 开始-->
           <section class="no-data-wrap" v-if="data.total == 0">
             <em class="icon30 no-data-ico">&nbsp;</em>
             <span class="c-666 fsize14 ml10 vam">没有相关数据，小编正在努力整理中...</span>
           </section>
           <!-- /无数据提示 结束-->
```



### 2、列表

```vue
<article class="i-teacher-list" v-if="data.total > 0">
             <ul class="of">
               <li v-for="item in data.items" :key="item.id">
                 <section class="i-teach-wrap">
                   <div class="i-teach-pic">
                     <a :href="'/teacher/'+item.id" :title="item.name" target="_blank">
                       <img :src="item.avatar" alt>
                     </a>
                   </div>
                   <div class="mt10 hLh30 txtOf tac">
                     <a :href="'/teacher/'+item.id" title="item.name" target="_blank" class="fsize18 c-666">{{item.name}}</a>
                   </div>
                   <div class="hLh30 txtOf tac">
                     <span class="fsize14 c-999">{{item.career}}</span>
                   </div>
                   <div class="mt15 i-q-txt">
                     <p class="c-999 f-fA">{{item.intro}}</p>
                   </div>
                 </section>
               </li>
               
             </ul>
             <div class="clear"></div>
           </article>
```



# 讲师分页查询

## 1、分页方法

```vue
   methods:{
     // 分页切换
     // 参数是页码数
     gotoPage(page){
       teacher.getTeacherList(page,8)
        .then(response =>{
          this.data = response.data.data
        })
     }
   }
```

## 2、分页页面渲染

```vue
<!-- 公共分页 开始 -->
          <div>
            <div class="paging">
              <!-- undisable这个class是否存在，取决于数据属性hasPrevious -->
              <a
                :class="{undisable: !data.hasPrevious}"
                href="#"
                title="首页"
                @click.prevent="gotoPage(1)">首</a>
              <a
                :class="{undisable: !data.hasPrevious}"
                href="#"
                title="前一页"
                @click.prevent="gotoPage(data.current-1)">&lt;</a>
              <a
                v-for="page in data.pages"
                :key="page"
                :class="{current: data.current == page, undisable: data.current == page}"
                :title="'第'+page+'页'"
                href="#"
                @click.prevent="gotoPage(page)">{{ page }}</a>
              <a
                :class="{undisable: !data.hasNext}"
                href="#"
                title="后一页"
                @click.prevent="gotoPage(data.current+1)">&gt;</a>
              <a
                :class="{undisable: !data.hasNext}"
                href="#"
                title="末页"
                @click.prevent="gotoPage(data.pages)">末</a>
              <div class="clear"/>
            </div>
          </div>
         <!-- 公共分页 结束 -->
```

# 讲师详情接口

```java
// 讲师详情的功能
@GetMapping("getTeacherFrontInfo/{teacherId}")
public R getTeacherFrontInfo(@PathVariable String teacherId){
    // 1.根据讲师id查询讲师基本信息
    EduTeacher eduTeacher = teacherService.getById(teacherId);

    // 2.根据讲师id查询讲师所有的课程
    QueryWrapper<EduCourse> wrapper = new QueryWrapper<>();
    wrapper.eq("teacher_id",teacherId);
    List<EduCourse> courseList = courseService.list(wrapper);

    return R.ok().data("teacher",eduTeacher).data("courseList",courseList);
}
```

# 讲师详情前端

## 1、teacher api

api/teacher.js

```js
  // 讲师详情
  getTeacherInfo(id){
    return request({
      url: `/eduservice/teacherfront/getTeacherFrontInfo/${id}`,
      method: 'get'
    })
  }
```

## 2、讲师详情页中调用api

pages/teacher/_id.vue

```vue
 <script>
   import teacherApi from '@/api/teacher'
 export default {
   // 异步调用
   asyncData({params,error}){
     // params.id获取路由中的id
     return teacherApi.getTeacherInfo(params.id)
      .then(response =>{
        return{
          teacher: response.data.data.teacher,
          courseList : response.data.data.courseList
        }
      })
   }
 };
 </script>
```

## 讲师详情显示

```vue
 <!-- 讲师基本信息 -->
         <section class="fl t-infor-box c-desc-content">
           <div class="mt20 ml20">
             <section class="t-infor-pic">
               <img :src="teacher.avatar">
             </section>
             <h3 class="hLh30">
               <span class="fsize24 c-333">{{teacher.name}}&nbsp;
                 {{teacher.level === 1? '高级讲师':'首席讲师'}}
               </span>
             </h3>
             <section class="mt10">
               <span class="t-tag-bg">{{teacher.intro}}</span>
             </section>
             <section class="t-infor-txt">
               <p
                 class="mt20">{{teacher.career}}</p>
             </section>
             <div class="clear"></div>
           </div>
         </section>
```

## 无数据提示

```vue
           <!-- /无数据提示 开始-->
           <section class="no-data-wrap" v-if="courseList.length == 0">
             <em class="icon30 no-data-ico">&nbsp;</em>
             <span class="c-666 fsize14 ml10 vam">没有相关数据，小编正在努力整理中...</span>
           </section>
           <!-- /无数据提示 结束-->
```

## 当前讲师课程列表

```vue
           <article class="comm-course-list">
             <ul class="of">
               <li v-for="course in courseList" :key="course.id">
                 <div class="cc-l-wrap">
                   <section class="course-img">
                     <img :src="course.cover" class="img-responsive" >
                     <div class="cc-mask">
                       <a href="#" title="开始学习" target="_blank" class="comm-btn c-btn-1">开始学习</a>
                     </div>
                   </section>
                   <h3 class="hLh30 txtOf mt10">
                     <a href="#" :title="course.title" target="_blank" class="course-title fsize18 c-333">{{course.title}}</a>
                   </h3>
                 </div>
               </li>
              
             </ul>
             <div class="clear"></div>
           </article>
```

## 完整页面

```vue
<template>
   <div id="aCoursesList" class="bg-fa of">
     <!-- 讲师介绍 开始 -->
     <section class="container">
       <header class="comm-title">
         <h2 class="fl tac">
           <span class="c-333">讲师介绍</span>
         </h2>
       </header>
       <div class="t-infor-wrap">
         <!-- 讲师基本信息 -->
         <section class="fl t-infor-box c-desc-content">
           <div class="mt20 ml20">
             <section class="t-infor-pic">
               <img :src="teacher.avatar">
             </section>
             <h3 class="hLh30">
               <span class="fsize24 c-333">{{teacher.name}}&nbsp;
                 {{teacher.level === 1? '高级讲师':'首席讲师'}}
               </span>
             </h3>
             <section class="mt10">
               <span class="t-tag-bg">{{teacher.intro}}</span>
             </section>
             <section class="t-infor-txt">
               <p
                 class="mt20">{{teacher.career}}</p>
             </section>
             <div class="clear"></div>
           </div>
         </section>
         <div class="clear"></div>
       </div>
       <section class="mt30">
         <div>
           <header class="comm-title all-teacher-title c-course-content">
             <h2 class="fl tac">
               <span class="c-333">主讲课程</span>
             </h2>
             <section class="c-tab-title">
               <a href="javascript: void(0)">&nbsp;</a>
             </section>
           </header>
           <!-- /无数据提示 开始-->
           <section class="no-data-wrap" v-if="courseList.length == 0">
             <em class="icon30 no-data-ico">&nbsp;</em>
             <span class="c-666 fsize14 ml10 vam">没有相关数据，小编正在努力整理中...</span>
           </section>
           <!-- /无数据提示 结束-->
           <article class="comm-course-list">
             <ul class="of">
               <li v-for="course in courseList" :key="course.id">
                 <div class="cc-l-wrap">
                   <section class="course-img">
                     <img :src="course.cover" class="img-responsive" >
                     <div class="cc-mask">
                       <a href="#" title="开始学习" target="_blank" class="comm-btn c-btn-1">开始学习</a>
                     </div>
                   </section>
                   <h3 class="hLh30 txtOf mt10">
                     <a href="#" :title="course.title" target="_blank" class="course-title fsize18 c-333">{{course.title}}</a>
                   </h3>
                 </div>
               </li>
              
             </ul>
             <div class="clear"></div>
           </article>
         </div>
       </section>
     </section>
     <!-- /讲师介绍 结束 -->
   </div>
 </template>
 <script>
   import teacherApi from '@/api/teacher'
 export default {
   // 异步调用
   asyncData({params,error}){
     // params.id获取路由中的id
     return teacherApi.getTeacherInfo(params.id)
      .then(response =>{
        return{
          teacher: response.data.data.teacher,
          courseList : response.data.data.courseList
        }
      })
   }
 };
 </script>
```



# 课程列表接口

## Controller

com.atguigu.eduservice.controller.front

```java
@RestController
@RequestMapping("/eduservice/coursefront")
@CrossOrigin
public class CourseFrontController {

    @Autowired
    private EduCourseService courseService;

    // 条件分页查询课程，前台
    @PostMapping("getCourseFrontList/{page}/{limit}")
    public R getCourseFrontList(@PathVariable long page, @PathVariable long limit,
                                @RequestBody(required = false) CourseFrontVo courseFrontVo){
        Page<EduCourse> coursePage = new Page<>(page,limit);
        Map<String,Object> map = courseService.getCourseFrontList(coursePage,courseFrontVo);

        return R.ok().data(map);
    }

}
```

## Service

```java
// 条件分页查询课程，前台
Map<String, Object> getCourseFrontList(Page<EduCourse> coursePage, CourseFrontVo courseFrontVo);
```

## serviceImpl

```java
// 条件分页查询课程，前台
@Override
public Map<String, Object> getCourseFrontList(Page<EduCourse> coursePage, CourseFrontVo courseFrontVo) {

    QueryWrapper<EduCourse> wrapper = new QueryWrapper<>();

    // 先判断传过来的值是否为空
    if (!StringUtils.isEmpty(courseFrontVo.getSubjectParentId())){
        // 一级标题
        wrapper.eq("subject_parent_id",courseFrontVo.getSubjectParentId());
    }
    if (!StringUtils.isEmpty(courseFrontVo.getSubjectId())){
        // 二级标题
        wrapper.eq("subject_id",courseFrontVo.getSubjectId());
    }
    if (!StringUtils.isEmpty(courseFrontVo.getBuyCountSort())){
        // 销售量
        wrapper.orderByDesc("buy_count");
    }
    if (!StringUtils.isEmpty(courseFrontVo.getGmtCreateSort())){
        // 创建时间
        wrapper.orderByDesc("gmt_create");
    }
    if (!StringUtils.isEmpty(courseFrontVo.getPriceSort())){
        // 价格
        wrapper.orderByDesc("price");
    }

    baseMapper.selectPage(coursePage,wrapper);

    List<EduCourse> records = coursePage.getRecords();
    long current = coursePage.getCurrent();
    long pages = coursePage.getPages();
    long size = coursePage.getSize();
    long total = coursePage.getTotal();
    boolean hasNext = coursePage.hasNext();
    boolean hasPrevious = coursePage.hasPrevious();

    // 将查询出来的封装到map
    Map<String, Object> map = new HashMap<>();
    map.put("items",records);
    map.put("current",current);
    map.put("pages",pages);
    map.put("size",size);
    map.put("total",total);
    map.put("hasNext",hasNext);
    map.put("hasPrevious",hasPrevious);

    // 最后返回map
    return map;

}
```

# 课程列表前端

## 1、定义api

api/course.js

```js
import request from '@/utils/request'
export default {
  // 条件分页查询课程
  getCourseList(page,limit,searchObj) {
    return request({
      url: `/eduservice/coursefront/getCourseFrontList/${page}/${limit}`,
      method: 'post',
      data:searchObj
    })
  },

  // 查询所有分类的方法
  getAllSubject(){
    return request({
      url: `/eduservice/subject/findAllSubject`,
      method: 'get'
    })
  }

}
```



## 2、页面调用接口

pages/course/index.vue

```vue
<script>
   import courseApi from '@/api/course'
 export default {
   data(){
     return{
      page:1,
      data:{},  // 当前页的课程列表
      subjectNestedList: [], // 一级分类列表
      subSubjectList: [], // 二级分类列表
      searchObj: {}, // 查询表单对象
      oneIndex:-1,
      twoIndex:-1,
      buyCountSort:"",
      gmtCreateSort:"",
      priceSort:""
     }
   },
   created(){
     // 初始化课程列表
     this.initCourse()
     // 初始化分类
     this.initSubject()
   },
   methods:{
     //1 查询第一页课程
     initCourse(){
       courseApi.getCourseList(1,8,this.searchObj)
        .then(response =>{
          this.data = response.data.data
        })
     },
     // 2查询所有一级分类
     initSubject(){
       courseApi.getAllSubject()
        .then(response =>{
          this.subjectNestedList = response.data.data.list
        })
     },
     // 3条件分页查询 下一页
     gotoPage(page){
       courseApi.getCourseList(page,8,this.searchObj)
        .then(response =>{
          this.data = response.data.data
        })
     },
     // 4二级分类联动，点击一级分类，显示二级分类
     searchOne(subjectParentId,index){
      // 为了清空排序
      this.buyCountSort = ""
      this.gmtCreateSort = ""
      this.priceSort = ""

       // 选中一级分类变色，为了样式能显示
       this.oneIndex = index
       this.twoIndex = -1
       this.searchObj.subjectId =''
       this.subSubjectList = []

       // 点击一级分类的时候，把点击的分类的id赋值给searchObj
       this.searchObj.subjectParentId = subjectParentId
       // 条件分页查询
       this.gotoPage(1)

       // 拿着一级分类id和所有一级分类id进行比较
       //遍历所有的一级分类
       for(let i = 0; i < this.subjectNestedList.length; i++){
         // 取到每一个一级分类
         var oneSubject = this.subjectNestedList[i]
         // 将传进来的一级分类id和遍历出来的所有一级分类对比
         if(subjectParentId == oneSubject.id){
           // 将查询出来的二级分类赋值给subSubjectList
           this.subSubjectList = oneSubject.children
         }
       }
     },
     // 5.点击二级分类，显示课程
    searchTwo(subjectId,index){
      // 为了清空排序
      this.buyCountSort = ""
      this.gmtCreateSort = ""
      this.priceSort = ""

      // 给index赋值，为了让样式生效
      this.twoIndex = index
       // 点击二级分类的时候，把点击的分类的id赋值给searchObj
       this.searchObj.subjectId = subjectId
       // 条件分页查询
      this.gotoPage(1)
    },
    // 6.按照销量排序
    searchBuyCount() {
      this.buyCountSort = "1"
      this.gmtCreateSort = ""
      this.priceSort = ""
      this.searchObj.buyCountSort = this.buyCountSort
      this.searchObj.gmtCreateSort = this.gmtCreateSort
      this.searchObj.priceSort = this.priceSort
      this.gotoPage(1)
    },
    // 7.按照时间排序
    searchGmtCreate(){
      this.buyCountSort = ""
      this.gmtCreateSort = "1"
      this.priceSort = ""
      this.searchObj.buyCountSort = this.buyCountSort
      this.searchObj.gmtCreateSort = this.gmtCreateSort
      this.searchObj.priceSort = this.priceSort
      this.gotoPage(1)
    },
    // 8.按照价格排序
    searchPrice(){
      this.buyCountSort = ""
      this.gmtCreateSort = ""
      this.priceSort = "1"
      this.searchObj.buyCountSort = this.buyCountSort
      this.searchObj.gmtCreateSort = this.gmtCreateSort
      this.searchObj.priceSort = this.priceSort
      this.gotoPage(1)

    }
   }
 };
 </script>
// #bdbdbd;rgba(24, 223, 24, 0.514);rgba(172, 255, 47, 0.493);
 <style scoped>
  .active { 
    background: rgba(24, 223, 24, 0.514);
  }
  .hide {
    display: none;
  }
  .show {
    display: block;
  }
</style>
```

## 3、课程类别显示

```vue
<section class="c-s-dl">
           <dl>
             <dt>
               <span class="c-999 fsize14">课程类别</span>
             </dt>
             <dd class="c-s-dl-li">
               <ul class="clearfix">
                 <li>
                   <a title="全部" href="#">全部</a>
                 </li>
                 <li v-for="(item,index) in subjectNestedList" :key="index"  :class="{active:oneIndex==index}">
                   <a :title="item.title" href="#" @click="searchOne(item.id,index)" >{{item.title}}</a>
                 </li>
               </ul>
             </dd>
           </dl>
           <dl>
             <dt>
               <span class="c-999 fsize14"></span>
             </dt>
             <dd class="c-s-dl-li">
               <ul class="clearfix">
                 <li v-for="(item,index) in subSubjectList" :key="index" :class="{active:twoIndex==index}">
                   <a :title="item.title" href="#" @click="searchTwo(item.id,index)">{{item.title}}</a>
                 </li>
                
               </ul>
             </dd>
           </dl>
           <div class="clear"></div>
         </section>
```



## 4、排序方式显示

```vue
           <section class="fl">
             <ol class="js-tap clearfix">
                <li :class="{'current bg-orange':buyCountSort!=''}">
                  <a title="销量" href="javascript:void(0);" @click="searchBuyCount()">销量
                  <span :class="{hide:buyCountSort==''}">↓</span>
                  </a>
                </li>
                <li :class="{'current bg-orange':gmtCreateSort!=''}">
                  <a title="最新" href="javascript:void(0);" @click="searchGmtCreate()">最新
                  <span :class="{hide:gmtCreateSort==''}">↓</span>
                  </a>
                </li>
                <li :class="{'current bg-orange':priceSort!=''}">
                  <a title="价格" href="javascript:void(0);" @click="searchPrice()">价格&nbsp;
                    <span :class="{hide:priceSort==''}">↓</span>
                  </a>
                </li>
             </ol>
           </section>
```

## 5、无数据提示

添加：v-if="data.total==0"

```vue
           <!-- /无数据提示 开始-->
           <section class="no-data-wrap" v-if="data.total == 0">
             <em class="icon30 no-data-ico">&nbsp;</em>
             <span class="c-666 fsize14 ml10 vam">没有相关数据，小编正在努力整理中...</span>
           </section>
           <!-- /无数据提示 结束-->
```

## 6、列表

```vue
<article class="comm-course-list"  v-if="data.total > 0">
             <ul class="of" id="bna">
               <li v-for="(item,index) in data.items" :key="index">
                 <div class="cc-l-wrap">
                   <section class="course-img">
                     <img :src="item.cover" class="img-responsive" :alt="item.title">
                     <div class="cc-mask">
                       <a :href="'/course/'+item.id" title="开始学习" class="comm-btn c-btn-1">开始学习</a>
                     </div>
                   </section>
                   <h3 class="hLh30 txtOf mt10">
                     <a :href="'/course/'+item.id" :title="item.title" class="course-title fsize18 c-333">{{item.title}}</a>
                   </h3>
                   <section class="mt10 hLh20 of">
                     <span v-if="Number(item.price) == 0" class="fr jgTag bg-green">
                       <i class="c-fff fsize12 f-fA">免费</i>
                     </span>
                     <span class="fl jgAttr c-ccc f-fA">
                       <i class="c-999 f-fA">9634人学习</i>
                       |
                       <i class="c-999 f-fA">9634评论</i>
                     </span>
                   </section>
                 </div>
               </li>

             </ul>
             <div class="clear"></div>
           </article>
```

## 7、分页页面渲染

```vue
<!-- 公共分页 开始 -->
          <div>
            <div class="paging">
              <!-- undisable这个class是否存在，取决于数据属性hasPrevious -->
              <a
                :class="{undisable: !data.hasPrevious}"
                href="#"
                title="首页"
                @click.prevent="gotoPage(1)">首</a>
              <a
                :class="{undisable: !data.hasPrevious}"
                href="#"
                title="前一页"
                @click.prevent="gotoPage(data.current-1)">&lt;</a>
              <a
                v-for="page in data.pages"
                :key="page"
                :class="{current: data.current == page, undisable: data.current == page}"
                :title="'第'+page+'页'"
                href="#"
                @click.prevent="gotoPage(page)">{{ page }}</a>
              <a
                :class="{undisable: !data.hasNext}"
                href="#"
                title="后一页"
                @click.prevent="gotoPage(data.current+1)">&gt;</a>
              <a
                :class="{undisable: !data.hasNext}"
                href="#"
                title="末页"
                @click.prevent="gotoPage(data.pages)">末</a>
              <div class="clear"/>
            </div>
          </div>
         <!-- 公共分页 结束 -->
```



# 课程详情功能接口

## Vo

```java
@Data
public class CourseFrontVo implements Serializable {

    private static final long serialVersionUID = 1L;

    @ApiModelProperty(value = "课程名称")
    private String title;

    @ApiModelProperty(value = "讲师id")
    private String teacherId;

    @ApiModelProperty(value = "一级类别id")
    private String subjectParentId;

    @ApiModelProperty(value = "二级类别id")
    private String subjectId;

    @ApiModelProperty(value = "销量排序")
    private String buyCountSort;

    @ApiModelProperty(value = "最新时间排序")
    private String gmtCreateSort;

    @ApiModelProperty(value = "价格排序")
    private String priceSort;

}
```

## Controller

```java
// 2.课程详情的方法
@GetMapping("getFrontCourseInfo/{courseId}")
public R getFrontCourseInfo(@PathVariable String courseId){
    // 根据课程id，编写SQL语句查询课程信息
    CourseWebVo courseWebVo = courseService.getBaseCourseInfo(courseId);
    // 根据课程id，查询章节和小节
    List<ChapterVo> chapterVideoList = chapterService.getAllChapterVideo(courseId);

    return R.ok().data("courseWebVo",courseWebVo).data("chapterVideoList",chapterVideoList);
}
```

## Service

package com.atguigu.eduservice.service

```java
// 根据课程id，编写SQL语句查询课程信息
CourseWebVo getBaseCourseInfo(String courseId);
```

## ServiceImpl

com.atguigu.eduservice.service.impl

```java
// 根据课程id，编写SQL语句查询课程信息
@Override
public CourseWebVo getBaseCourseInfo(String courseId) {
    return baseMapper.getBaseCourseInfo(courseId);
}
```

## Mapper

package com.atguigu.eduservice.mapper

```java
// 根据课程id，编写SQL语句查询课程信息
CourseWebVo getBaseCourseInfo(String courseId);
```

## Xml

com/atguigu/eduservice/mapper/xml/EduCourseMapper.xml

```xml
<!--根据课程id查询课程前端详情信息-->
<select id="getBaseCourseInfo" resultType="com.atguigu.eduservice.entity.frontvo.CourseWebVo">
    SELECT
      ec.`id`,
      ec.`title`,
      ec.`price`,
      ec.`lesson_num` AS lessonNum,
      ec.`cover`,
      ec.`buy_count` AS buyCount,
      ec.`view_count` AS viewCount,
      ecd.`description` AS description,
      et.`id` AS teacherId,
      et.`name` AS teacherName,
      et.`intro` AS intro,
      et.`avatar`AS avatar,
      es1.`id` AS subjectLevelOneId,
      es1.`title` AS subjectLevelOne,
      es2.`id` AS subjectLevelTwoId,
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
```

# 课程详情功能前端

## 1、api/course.js

```js
  // 根据id查询课程详情信息
  getCourseInfo(id){
    return request({
      url: `/eduservice/coursefront/getFrontCourseInfo/${id}`,
      method: 'get'
    })
  }
```

## 2、pages/course/_id.vue

```vue
 <script>
import course from "@/api/course"
 export default {

   asyncData({ params, error  }){
     console.log(params.id)
     return course.getCourseInfo(params.id)
      .then(response =>{
        console.log(response)
        return{
          courseWebVo: response.data.data.courseWebVo,
          chapterVideoList: response.data.data.chapterVideoList
        }
      })
   }
   
 };
 </scrip>
```

## 课程所属分类

```vue
<!-- /课程详情 开始 -->
     <section class="container">
       <section class="path-wrap txtOf hLh30">
         <a href="#" title class="c-999 fsize14">首页</a>
         \
         <a href="#" title class="c-999 fsize14">{{courseWebVo.subjectLevelOne}}</a>
         \
         <span class="c-333 fsize14">{{courseWebVo.subjectLevelTwo}}</span>
       </section>
```

## 课程基本信息

```vue
 <div>
         <article class="c-v-pic-wrap" style="height: 357px;">
           <section class="p-h-video-box" id="videoPlay">
             <img :src="courseWebVo.cover" :alt="courseWebVo.title" class="dis c-v-pic">
           </section>
         </article>
         <aside class="c-attr-wrap">
           <section class="ml20 mr15">
             <h2 class="hLh30 txtOf mt15">
               <span class="c-fff fsize24">{{courseWebVo.title}}</span>
             </h2>
             <section class="c-attr-jg">
               <span class="c-fff">价格：</span>
               <b class="c-yellow" style="font-size:24px;">￥{{courseWebVo.price}}</b>
             </section>
             <section class="c-attr-mt c-attr-undis">
               <span class="c-fff fsize14">主讲： {{courseWebVo.teacherName}}&nbsp;&nbsp;&nbsp;</span>
             </section>
             <section class="c-attr-mt of">
               <span class="ml10 vam">
                 <em class="icon18 scIcon"></em>
                 <a class="c-fff vam" title="收藏" href="#" >收藏</a>
               </span>
             </section>
             <section class="c-attr-mt">
               <a href="#" title="立即观看" class="comm-btn c-btn-3">立即观看</a>
             </section>
           </section>
         </aside>
         <aside class="thr-attr-box">
           <ol class="thr-attr-ol clearfix">
             <li>
               <p>&nbsp;</p>
               <aside>
                 <span class="c-fff f-fM">购买数</span>
                 <br>
                 <h6 class="c-fff f-fM mt10">{{courseWebVo.buyCount}}}</h6>
               </aside>
             </li>
             <li>
               <p>&nbsp;</p>
               <aside>
                 <span class="c-fff f-fM">课时数</span>
                 <br>
                 <h6 class="c-fff f-fM mt10">{{courseWebVo.lessonNum}}</h6>
               </aside>
             </li>
             <li>
               <p>&nbsp;</p>
               <aside>
                 <span class="c-fff f-fM">浏览数</span>
                 <br>
                 <h6 class="c-fff f-fM mt10">{{courseWebVo.viewCount}}</h6>
               </aside>
             </li>
           </ol>
         </aside>
         <div class="clear"></div>
       </div>
```

## 课程详情介绍

```vue
 <div>
                   <h6 class="c-i-content c-infor-title">
                     <span>课程介绍</span>
                   </h6>
                   <div class="course-txt-body-wrap">
                     <section class="course-txt-body">
                       <p v-html="courseWebVo.description" >{{courseWebVo.description}}</p>
                     </section>
                   </div>
                 </div>
```

## 课程大纲

```vue
<!-- /课程介绍 -->
                 <div class="mt50">
                   <h6 class="c-g-content c-infor-title">
                     <span>课程大纲</span>
                   </h6>
                   <section class="mt20">
                     <div class="lh-menu-wrap">
                       <menu id="lh-menu" class="lh-menu mt10 mr10">
                         <ul>
                           <!-- 文件目录 -->
                           <li class="lh-menu-stair" v-for="chapter in chapterVideoList" :key="chapter.id">
                             <a href="javascript: void(0)" :title="chapter.title" class="current-1">
                               <em class="lh-menu-i-1 icon18 mr10"></em>{{chapter.title}}
                             </a>
                             <ol class="lh-menu-ol" style="display: block;">
                               <li class="lh-menu-second ml30" v-for="video in chapter.children" :key="video.id">
                                 <a href="#" title>
                                   <span class="fr" v-if="Number(courseWebVo.price) == 0">
                                     <i class="free-icon vam mr10" >免费试听</i>
                                   </span>
                                   <em class="lh-menu-i-2 icon16 mr5">&nbsp;</em>{{video.title}}
                                 </a>
                               </li>
                               
                             </ol>
                           </li>
                         </ul>
                       </menu>
                     </div>
                   </section>
                 </div>
                 <!-- /课程大纲 -->
```

## 主讲讲师

```vue
<div class="i-box">
             <div>
               <section class="c-infor-tabTitle c-tab-title">
                 <a title href="javascript:void(0)">主讲讲师</a>
               </section>
               <section class="stud-act-list">
                 <ul style="height: auto;">
                   <li>
                     <div class="u-face">
                       <a href="#">
                         <img :src="courseWebVo.avatar" width="50" height="50" alt>
                       </a>
                     </div>
                     <section class="hLh30 txtOf">
                       <a class="c-333 fsize16 fl" href="#">{{courseWebVo.teacherName}}</a>
                     </section>
                     <section class="hLh20 txtOf">
                       <span class="c-999">{{courseWebVo.intro}}</span>
                     </section>
                   </li>
                 </ul>
               </section>
             </div>
           </div>
```

## 完整页面

```vue
<template>
   <div id="aCoursesList" class="bg-fa of">
     <!-- /课程详情 开始 -->
     <section class="container">
       <section class="path-wrap txtOf hLh30">
         <a href="#" title class="c-999 fsize14">首页</a>
         \
         <a href="#" title class="c-999 fsize14">{{courseWebVo.subjectLevelOne}}</a>
         \
         <span class="c-333 fsize14">{{courseWebVo.subjectLevelTwo}}</span>
       </section>
       <div>
         <article class="c-v-pic-wrap" style="height: 357px;">
           <section class="p-h-video-box" id="videoPlay">
             <img :src="courseWebVo.cover" :alt="courseWebVo.title" class="dis c-v-pic">
           </section>
         </article>
         <aside class="c-attr-wrap">
           <section class="ml20 mr15">
             <h2 class="hLh30 txtOf mt15">
               <span class="c-fff fsize24">{{courseWebVo.title}}</span>
             </h2>
             <section class="c-attr-jg">
               <span class="c-fff">价格：</span>
               <b class="c-yellow" style="font-size:24px;">￥{{courseWebVo.price}}</b>
             </section>
             <section class="c-attr-mt c-attr-undis">
               <span class="c-fff fsize14">主讲： {{courseWebVo.teacherName}}&nbsp;&nbsp;&nbsp;</span>
             </section>
             <section class="c-attr-mt of">
               <span class="ml10 vam">
                 <em class="icon18 scIcon"></em>
                 <a class="c-fff vam" title="收藏" href="#" >收藏</a>
               </span>
             </section>
             <section class="c-attr-mt">
               <a href="#" title="立即观看" class="comm-btn c-btn-3">立即观看</a>
             </section>
           </section>
         </aside>
         <aside class="thr-attr-box">
           <ol class="thr-attr-ol clearfix">
             <li>
               <p>&nbsp;</p>
               <aside>
                 <span class="c-fff f-fM">购买数</span>
                 <br>
                 <h6 class="c-fff f-fM mt10">{{courseWebVo.buyCount}}}</h6>
               </aside>
             </li>
             <li>
               <p>&nbsp;</p>
               <aside>
                 <span class="c-fff f-fM">课时数</span>
                 <br>
                 <h6 class="c-fff f-fM mt10">{{courseWebVo.lessonNum}}</h6>
               </aside>
             </li>
             <li>
               <p>&nbsp;</p>
               <aside>
                 <span class="c-fff f-fM">浏览数</span>
                 <br>
                 <h6 class="c-fff f-fM mt10">{{courseWebVo.viewCount}}</h6>
               </aside>
             </li>
           </ol>
         </aside>
         <div class="clear"></div>
       </div>
       <!-- /课程封面介绍 -->
       <div class="mt20 c-infor-box">
         <article class="fl col-7">
           <section class="mr30">
             <div class="i-box">
               <div>
                 <section id="c-i-tabTitle" class="c-infor-tabTitle c-tab-title">
                   <a name="c-i" class="current" title="课程详情">课程详情</a>
                 </section>
               </div>
               <article class="ml10 mr10 pt20">
                 <div>
                   <h6 class="c-i-content c-infor-title">
                     <span>课程介绍</span>
                   </h6>
                   <div class="course-txt-body-wrap">
                     <section class="course-txt-body">
                       <p v-html="courseWebVo.description" >{{courseWebVo.description}}</p>
                     </section>
                   </div>
                 </div>
                 <!-- /课程介绍 -->
                 <div class="mt50">
                   <h6 class="c-g-content c-infor-title">
                     <span>课程大纲</span>
                   </h6>
                   <section class="mt20">
                     <div class="lh-menu-wrap">
                       <menu id="lh-menu" class="lh-menu mt10 mr10">
                         <ul>
                           <!-- 文件目录 -->
                           <li class="lh-menu-stair" v-for="chapter in chapterVideoList" :key="chapter.id">
                             <a href="javascript: void(0)" :title="chapter.title" class="current-1">
                               <em class="lh-menu-i-1 icon18 mr10"></em>{{chapter.title}}
                             </a>
                             <ol class="lh-menu-ol" style="display: block;">
                               <li class="lh-menu-second ml30" v-for="video in chapter.children" :key="video.id">
                                 <a href="#" title>
                                   <span class="fr" v-if="Number(courseWebVo.price) == 0">
                                     <i class="free-icon vam mr10" >免费试听</i>
                                   </span>
                                   <em class="lh-menu-i-2 icon16 mr5">&nbsp;</em>{{video.title}}
                                 </a>
                               </li>
                               
                             </ol>
                           </li>
                         </ul>
                       </menu>
                     </div>
                   </section>
                 </div>
                 <!-- /课程大纲 -->
               </article>
             </div>
           </section>
         </article>
         <aside class="fl col-3">
           <div class="i-box">
             <div>
               <section class="c-infor-tabTitle c-tab-title">
                 <a title href="javascript:void(0)">主讲讲师</a>
               </section>
               <section class="stud-act-list">
                 <ul style="height: auto;">
                   <li>
                     <div class="u-face">
                       <a href="#">
                         <img :src="courseWebVo.avatar" width="50" height="50" alt>
                       </a>
                     </div>
                     <section class="hLh30 txtOf">
                       <a class="c-333 fsize16 fl" href="#">{{courseWebVo.teacherName}}</a>
                     </section>
                     <section class="hLh20 txtOf">
                       <span class="c-999">{{courseWebVo.intro}}</span>
                     </section>
                   </li>
                 </ul>
               </section>
             </div>
           </div>
         </aside>
         <div class="clear"></div>
       </div>
     </section>
     <!-- /课程详情 结束 -->
   </div>
 </template>
 <script>
import course from "@/api/course"
 export default {

   asyncData({ params, error  }){
     console.log(params.id)
     return course.getCourseInfo(params.id)
      .then(response =>{
        console.log(response)
        return{
          courseWebVo: response.data.data.courseWebVo,
          chapterVideoList: response.data.data.chapterVideoList
        }
      })
   }
   
 };
 </script>
```

# 整合阿里云视频播放器测试

## 视频地址播放

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>

<link rel="stylesheet" href="https://g.alicdn.com/de/prismplayer/2.8.1/skins/default/aliplayer-min.css" />
<script charset="utf-8" type="text/javascript" src="https://g.alicdn.com/de/prismplayer/2.8.1/aliplayer-min.js"></script>
<body>
    <div  class="prism-player" id="J_prismPlayer"></div>
        <script>
            var player = new Aliplayer({
                id: 'J_prismPlayer',
                width: '100%',
                autoplay: false,
                cover: 'http://liveroom-img.oss-cn-qingdao.aliyuncs.com/logo.png',  
                //播放配置
                //播放方式一：支持播放地址播放,此播放优先级最高，此种方式不能播放加密视频
                source : 'https://outin-e7915c6967b411eca96500163e1c60dc.oss-cn-shanghai.aliyuncs.com/sv/12b862d3-17e007b353b/12b862d3-17e007b353b.mp4?Expires=1641138735&OSSAccessKeyId=LTAIxSaOfEzCnBOj&Signature=LZvaSss9BmRFXSrXL6DMmhmi%2BiQ%3D',
            },function(player){
               
            });
        </script>
</body>
</html>
```

## 凭证播放

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>

<link rel="stylesheet" href="https://g.alicdn.com/de/prismplayer/2.8.1/skins/default/aliplayer-min.css" />
<script charset="utf-8" type="text/javascript" src="https://g.alicdn.com/de/prismplayer/2.8.1/aliplayer-min.js"></script>
<body>
    <div  class="prism-player" id="J_prismPlayer"></div>
        <script>
            var player = new Aliplayer({
                id: 'J_prismPlayer',
                width: '100%',
                autoplay: false,
                cover: 'http://liveroom-img.oss-cn-qingdao.aliyuncs.com/logo.png',  
                //播放配置
                encryptType:'1',//如果播放加密视频，则需设置encryptType=1，非加密视频无需设置此项
                vid: '',		// 视频id
                playauth : '',	// 授权信息
            },function(player){
               
            });
        </script>
</body>
</html>

```

# 整合阿里云视频播放器

## 一、后端获取播放凭证

### 1、VideoController

service-vod微服务中创建 VideoController.java

controller中创建 getVideoAuth接口方法

```java
// 根据视频id获取凭证
@GetMapping("getVideoAuth/{id}")
public R getVideoAuth(@PathVariable String id){

    try {
        // 创建初始化对象
        DefaultAcsClient client = InitVodClient.initVodClient(ConstantVodUtils.ACCESS_KEY_ID, ConstantVodUtils.ACCESS_KEY_SECRET);

        // 创建获取凭证的request和response对象
        GetVideoPlayAuthRequest request = new GetVideoPlayAuthRequest();

        // 设置视频id
        request.setVideoId(id);

        // 调用初始化对象获取凭证
        GetVideoPlayAuthResponse response = client.getAcsResponse(request);

        String playAuth = response.getPlayAuth();

        return R.ok().data("playAuth",playAuth);
    } catch (ClientException e) {
        e.printStackTrace();
        throw new GuliException(20001,"获取凭证失败");
    }
}
```

## 二、前端播放器整合

### 1、点击播放超链接

course/_id.vue

修改课时目录超链接

```vue
<a :href="'/player/'+video.videoSourceId" :title="video.title" target="_blank">
    <span class="fr" v-if="Number(courseWebVo.price) == 0">
        <i class="free-icon vam mr10" >免费试听</i>
    </span>
    <em class="lh-menu-i-2 icon16 mr5">&nbsp;</em>{{video.title}}
</a>
```

### 2、layout

因为播放器的布局和其他页面的基本布局不一致，因此创建新的布局容器 layouts/video.vue

```vue
<template>
  <div class="guli-player">
    <div class="head">
      <a href="#" title="谷粒学院">
        <img class="logo" src="~/assets/img/logo.png" lt="谷粒学院">
    </a></div>
    <div class="body">
      <div class="content"><nuxt/></div>
    </div>
  </div>
</template>
<script>
export default {}
</script>
<style>
html,body{
  height:100%;
}
</style>
<style scoped>
.head {
  height: 50px;
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
}
.head .logo{
  height: 50px;
  margin-left: 10px;
}
.body {
  position: absolute;
  top: 50px;
  left: 0;
  right: 0;
  bottom: 0;
  overflow: hidden;
}
</style>
```

### 3、api

创建api模块 api/vod.js，从后端获取播放凭证

```js
import request from '@/utils/request'
export default {
  getPlayAuth(vid) {
    return request({
      url: `/eduvod/video/getVideoAuth/${vid}`,
      method: 'get'
    })
  }
}
```

### 4、播放组件相关文档

**集成文档：**https://help.aliyun.com/document_detail/51991.html?spm=a2c4g.11186623.2.39.478e192b8VSdEn

**在线配置：**https://player.alicdn.com/aliplayer/setting/setting.html

**功能展示：**https://player.alicdn.com/aliplayer/presentation/index.html

 

### 5、创建播放页面

创建 pages/player/_vid.vue

（1）引入播放器js库和css样式

```vue
<template>
  <div>
    <!-- 阿里云视频播放器样式 -->
    <link rel="stylesheet" href="https://g.alicdn.com/de/prismplayer/2.8.1/skins/default/aliplayer-min.css" >
    <!-- 阿里云视频播放器脚本 -->
    <script charset="utf-8" type="text/javascript" src="https://g.alicdn.com/de/prismplayer/2.8.1/aliplayer-min.js" />
    <!-- 定义播放器dom -->
    <div id="J_prismPlayer" class="prism-player" />
  </div>
</template>
```

（2）获取播放凭证

```vue
<script>
import vod from '@/api/vod'
export default {
    layout: 'video',//应用video布局
    asyncData({ params, error }) {
        return vod.getPlayAuth(params.vid)
            .then(response =>{
                console.log(response.data.data.playAuth)
                return{
                    playAuth: response.data.data.playAuth,
                    vid:params.vid
                }
            })
    },
    
}
</script>
```

（3）创建播放器

```vue
    /**
     * 页面渲染完成时：此时js脚本已加载，Aliplayer已定义，可以使用
     * 如果在created生命周期函数中使用，Aliplayer is not defined错误
     */
    // 在页面渲染之后执行
    mounted() {
        new Aliplayer({
            id: 'J_prismPlayer',
            vid: this.vid, // 视频id
            playauth: this.playAuth, // 播放凭证
            encryptType: '1', // 如果播放加密视频，则需设置encryptType=1，非加密视频无需设置此项
            width: '100%',
            height: '500px'
        }, function(player) {
            //console.log('播放器创建成功')
        })
    }
```

（4）其他常见的可选配置

```vue
            // 以下可选设置
            cover: 'http://guli.shop/photo/banner/1525939573202.jpg', // 封面
            qualitySort: 'asc', // 清晰度排序
            mediaType: 'video', // 返回音频还是视频
            autoplay: false, // 自动播放
            isLive: false, // 直播
            rePlay: false, // 循环播放
            preload: true,
            controlBarVisibility: 'hover', // 控制条的显示方式：鼠标悬停
            useH5Prism: true, // 播放器类型：html5
        }, function(player) {
            //console.log('播放器创建成功')
```

### 6、加入播放组件

**功能展示：**https://player.alicdn.com/aliplayer/presentation/index.html

