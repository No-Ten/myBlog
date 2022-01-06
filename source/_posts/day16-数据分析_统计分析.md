---
title: day16-数据分析_统计分析
data: 2022-01-07 00:50:00
comment: 'valine'
category: 项目|谷粒学院|笔记
tags:
  - project
---



# day16-数据分析_统计分析

# 课程详情页显示效果完善

## 修改课程详情接口

### 1、在service_order模块添加接口

根据用户id和课程id查询订单信息

```java
// 根据课程id和用户id查询订单支付状态
@GetMapping("isBuyCourse/{courseId}/{memberId}")
public boolean isBuyCourse(@PathVariable String courseId,@PathVariable String memberId){
    QueryWrapper<Order> wrapper = new QueryWrapper<>();
    wrapper.eq("course_id",courseId);
    wrapper.eq("member_id",memberId);
    wrapper.eq("status",1);     // 1.已支付

    int count = orderService.count(wrapper);
    if (count > 0){
        // 说明已经支付
        return true;
    } else {
        // 否则未支付，返回false
        return false;
    }
}
```

### **2、在service_edu模块课程详情接口远程调用**

（1）创建OrderClient接口

```java
@Component
@FeignClient(name = "service-order")
public interface OrderClient {

    // 根据课程id和用户id查询订单支付状态
    @GetMapping("/eduorder/order/isBuyCourse/{courseId}/{memberId}")
    public boolean isBuyCourse(@PathVariable("courseId") String courseId, @PathVariable("memberId") String memberId);

}
```

（2）在课程详情接口调用com.atguigu.eduservice.controller.front

```java
    // 2.课程详情的方法
    @GetMapping("getFrontCourseInfo/{courseId}")
    public R getFrontCourseInfo(@PathVariable String courseId, HttpServletRequest request){

        // 根据课程id，编写SQL语句查询课程信息
        CourseWebVo courseWebVo = courseService.getBaseCourseInfo(courseId);
        // 根据课程id，查询章节和小节
        List<ChapterVo> chapterVideoList = chapterService.getAllChapterVideo(courseId);

        // 根据课程id和用户id查询当前课程是否已经支付过了
        String memberId = JwtUtils.getMemberIdByJwtToken(request);
        // 判断是否已经登录
        if (!StringUtils.isEmpty(memberId)){
            boolean buyCourse = orderClient.isBuyCourse(courseId, memberId);
            return R.ok().data("courseWebVo",courseWebVo).data("chapterVideoList",chapterVideoList).data("isBuy",buyCourse);
        }
        return R.ok().code(28004).message("请先登录");

    }
```

## 修改课程详情页面

1、页面内容修改

```vue
             <section class="c-attr-mt" v-if="isBuy || Number(courseWebVo.price) === 0">
               <a href="#" title="立即观看" class="comm-btn c-btn-3" >立即观看</a>
             </section>
             <section class="c-attr-mt" v-else>
               <a href="#" title="立即购买" class="comm-btn c-btn-3" @click="createOrder()">立即购买</a>
             </section>
```

2、调用方法修改

```vue
 <script>
import course from "@/api/course"
import order from "@/api/order"
 export default {

   asyncData({ params, error  }){
     return {courseId : params.id}
   },
   data(){
     return{
          courseWebVo: {},
          chapterVideoList: [],
          isBuy: false,
     }
   },
   created(){
    this.initCourseInfo()
   },
   methods:{
     initCourseInfo(){
      course.getCourseInfo(this.courseId)
        .then(response =>{
            this.courseWebVo = response.data.data.courseWebVo,
            this.chapterVideoList = response.data.data.chapterVideoList,
            this.isBuy=response.data.data.isBuy
        })
     },
      // 创建订单
      createOrder(){
        order.createOrder(this.courseId)
          .then(response =>{
            // 获取返回的订单号
            // 将订单号放在路径上跳转订单显示页面
            this.$router.push({path:'/order/'+response.data.data.orderId })
          })
      }
   }
   
 };
 </script>
```



# 数据分析-生成统计数据接口

## 一、数据库设计

1、数据库

guli_statistics

2、数据表

```
guli_statistics.sql
```

## **二、创建微服务**

### 1、在service模块下创建子模块service_statistics

### **2**、application.properties

resources目录下创建文件

```properties
# 服务端口
server.port=8008
# 服务名
spring.application.name=service-statistics
# mysql数据库连接
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/guli?serverTimezone=GMT%2B8
spring.datasource.username=root
spring.datasource.password=root
#返回json的全局时间格式
spring.jackson.date-format=yyyy-MM-dd HH:mm:ss
spring.jackson.time-zone=GMT+8
#配置mapper xml文件的路径
mybatis-plus.mapper-locations=classpath:com/atguigu/staservice/mapper/xml/*.xml
#mybatis日志
mybatis-plus.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
# nacos服务地址
spring.cloud.nacos.discovery.server-addr=127.0.0.1:8848
```

### 3、MP代码生成器生成代码

### 4、创建SpringBoot启动类

```java
@SpringBootApplication
@ComponentScan({"com.atguigu"})
@MapperScan("com.atguigu.staservice.mapper")
@EnableDiscoveryClient  // 开启nacos
@EnableFeignClients // 开启远程调用
public class StaApplication {

    public static void main(String[] args) {
        SpringApplication.run(StaApplication.class,args);
    }

}
```

## 三、实现服务调用

### 1、在service_ucenter模块创建接口，统计某一天的注册人数 

**controller**

```java
    // 根据日期查询某一天的注册人数
    @PostMapping("countRegister/{day}")
    public R countRegister(@PathVariable String day){
        Integer count = memberService.countRegister(day);
        return R.ok().data("countRegister",count);
    }
```

**service**

```java
    // 根据日期查询某一天的注册人数
    Integer countRegister(String day);
```

**serviceImpl**

```java
// 根据日期查询某一天的注册人数
@Override
public Integer countRegister(String day) {
    return baseMapper.countRegister(day);
}
```

**mapper**

```java
public interface UcenterMemberMapper extends BaseMapper<UcenterMember> {
    // 查询某一天的注册人数，如果是多个参数，需要用@Param("day")
    Integer countRegister(@Param("day") String day);
}
```

**xml**

```xml
<select id="countRegister" resultType="java.lang.Integer">
    SELECT
      COUNT(*)
    FROM
      ucenter_member uc
    WHERE DATE(uc.`gmt_create`) = #{day}
</select>
```

### 2、在service_statistics模块创建远程调用接口 

创建client包和UcenterClient接口

```java
@Component
@FeignClient(name = "service-ucenter")
public interface UcenterClient {
    // 根据日期查询某一天的注册人数
    @PostMapping("/educenter/member/countRegister/{day}")
    public R countRegister(@PathVariable("day") String day);
}
```

### 3、在service_statistics模块调用微服务

**controller**

```java
@RestController
@RequestMapping("/staservice/sta")
@CrossOrigin
public class StatisticsDailyController {

    @Autowired
    private StatisticsDailyService dailyService;

    // 查询某一天的注册人数，并加入到数据库
    @PostMapping("registerCount/{day}")
    public R registerCount(@PathVariable String day){
        dailyService.registerCountDay(day);
        return R.ok();
    }
}
```

**service**

```java
@Service
public class StatisticsDailyServiceImpl extends ServiceImpl<StatisticsDailyMapper, StatisticsDaily> implements StatisticsDailyService {

    @Autowired
    private UcenterClient ucenterClient;

    // 查询某一天的注册人数，并加入到数据库
    @Override
    public void registerCountDay(String day) {
        // 添加记录之前先删除相同日期的
        QueryWrapper<StatisticsDaily> wrapper = new QueryWrapper<>();
        wrapper.eq("date_calculated",day);
        baseMapper.delete(wrapper);

        // 调用远程方法，查询数据库
        R registerR = ucenterClient.countRegister(day);

        // data中取出人数
        Integer countRegister = (Integer) registerR.getData().get("countRegister");

        // 将数据加入到数据库
        StatisticsDaily daily = new StatisticsDaily();
        daily.setRegisterNum(countRegister);
        daily.setDateCalculated(day);
        daily.setLoginNum(RandomUtils.nextInt(100, 200));
        daily.setVideoViewNum(RandomUtils.nextInt(100, 200));
        daily.setCourseNum(RandomUtils.nextInt(100, 200));

        baseMapper.insert(daily);
    }
}
```



# 统计分析-生成统计数据前端整合

## 添加路由

```js
{
    path: '/sta',
    component: Layout,
    redirect: '/sta/create',
    name: '统计分析',
    meta: { title: '统计分析', icon: 'example' },
    children: [
      {
        path: 'create',
        name: '生成数据',
        component: () => import('@/views/sta/create'),
        meta: { title: '生成数据', icon: 'table' }
      },
      {
        path: 'show',
        name: '显示数据',
        component: () => import('@/views/sta/show'),
        meta: { title: '显示数据', icon: 'tree' }
      }
    ]
  },

```

## 创建api

src/api/sta.js

```js
import request from '@/utils/request'
export default{
    // 生成数据
    createStaDate(day){
        return request({
            url: `/staservice/sta/registerCount/${day}`,
            method: 'post'
          })
    }
}
```

## **创建组件**

src/views/sta/create.vue

模板部分

```vue
<template>
  <div class="app-container">
    <!--表单-->
    <el-form :inline="true" class="demo-form-inline">
      <el-form-item label="日期">
        <el-date-picker
          v-model="day"
          type="date"
          placeholder="选择要统计的日期"
          value-format="yyyy-MM-dd" />
      </el-form-item>
      <el-button
        :disabled="btnDisabled"
        type="primary"
        @click="create()">生成</el-button>
    </el-form>
  </div>
</template>
```

script

```vue
<script>
import sta from "@/api/sta"
export default {
    data(){
        return{
            day:'',
            btnDisabled: false
        }
    },
    created(){

    },
    methods:{
        create(){
            sta.createStaDate(this.day)
                .then(response =>{
                    // 提示信息
                    this.$message({
                            type: 'success',
                            message: '生成数据成功!'
                        });
                    // 跳转到生成页面
                    this.$router.push({path:'/sta/show'})
                })
        }
    }
}
</script>
```

# 添加定时任务

## 1、创建定时任务类，使用cron表达式 

**复制日期工具类**

```java
package com.atguigu.staservice.utils;

import java.text.SimpleDateFormat;
import java.util.Calendar;
import java.util.Date;

/**
 * 日期操作工具类
 *
 * @author qy
 * @since 1.0
 */
public class DateUtil {

    private static final String dateFormat = "yyyy-MM-dd";

    /**
     * 格式化日期
     *
     * @param date
     * @return
     */
    public static String formatDate(Date date) {
        SimpleDateFormat sdf = new SimpleDateFormat(dateFormat);
        return sdf.format(date);

    }

    /**
     * 在日期date上增加amount天 。
     *
     * @param date   处理的日期，非null
     * @param amount 要加的天数，可能为负数
     */
    public static Date addDays(Date date, int amount) {
        Calendar now =Calendar.getInstance();
        now.setTime(date);
        now.set(Calendar.DATE,now.get(Calendar.DATE)+amount);
        return now.getTime();
    }

    public static void main(String[] args) {
        System.out.println(DateUtil.formatDate(new Date()));
        System.out.println(DateUtil.formatDate(DateUtil.addDays(new Date(), -1)));
    }
}
```

## 2、在启动类上添加注解

```
@EnableScheduling   // 开启定时任务
```

## 3.编写方法

```java
@Component
public class ScheduledTask {

    @Autowired
    private StatisticsDailyService dailyService;

    // 只能是6位，默认年为当前的年
    // 秒 分 时 日 月 周 年
    @Scheduled(cron = "0 0 1 * * ?")
    public void task2(){
        dailyService.registerCountDay(DateUtil.formatDate(DateUtil.addDays(new Date(), -1)));
    }

/*    // 每隔5秒执行一次
    @Scheduled(cron = "0/5 * * * * ?")
    public void task1() {
        System.out.println("*********++++++++++++*****执行了");
    }*/

}
```

在线生成工具

https://www.pppet.net/



# Echarts

## 1、简介

https://echarts.apache.org/zh/index.html

## 2、基本使用

入门参考：官网->文档->教程->5分钟上手ECharts

（1）创建html页面：柱图.html

（2）引入ECharts

```html
<!-- 引入 ECharts 文件 -->
<script src="echarts.min.js"></script>
```

（3）定义图表区域

```html
<!-- 为ECharts准备一个具备大小（宽高）的Dom -->
<div id="main" style="width: 600px;height:400px;"></div> 
```

（4）渲染图表

```html
<script type="text/javascript">
        // 基于准备好的dom，初始化echarts实例
        var myChart = echarts.init(document.getElementById('main'));
        // 指定图表的配置项和数据
        var option = {
            title: {
                text: 'ECharts 入门示例'
            },
            tooltip: {},
            legend: {
                data:['销量']
            },
            xAxis: {
                data: ["衬衫","羊毛衫","雪纺衫","裤子","高跟鞋","袜子"]
            },
            yAxis: {},
            series: [{
                name: '销量',
                type: 'bar',
                data: [5, 20, 36, 10, 10, 20]
            }]
        };
        // 使用刚指定的配置项和数据显示图表。
        myChart.setOption(option);
    </script>
```

## 3、折线图

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<!-- 引入 ECharts 文件 -->
<script src="echarts.min.js"></script>
<body>
    <div id="main" style="width: 600px;height:400px;"></div> 
    <script>
        var myChart = echarts.init(document.getElementById('main'));
        var option = {
            //x轴是类目轴（离散数据）,必须通过data设置类目数据
            xAxis: {
                type: 'category',
                data: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']
            },
            //y轴是数据轴（连续数据）
            yAxis: {
                type: 'value'
            },
            //系列列表。每个系列通过 type 决定自己的图表类型
            series: [{
                //系列中的数据内容数组
                data: [820, 932, 901, 934, 1290, 1330, 1320],
                //折线图
                type: 'line'
            }]
        };
        myChart.setOption(option);
    </script>
</body>
</html>
```





# 统计分析-图表显示-页面整合

## 1、安装ECharts

```bash
npm install --save echarts@4.1.0
# 如果不行就用cnpm 
```



## 3、创建组件 

src/views/sta/show.vue

**模板**

```vue
<template>
   <div class="app-container">
     <!--表单-->
     <el-form :inline="true" class="demo-form-inline">
       <el-form-item>
         <el-select v-model="searchObj.type" clearable placeholder="请选择">
           <el-option label="学员登录数统计" value="login_num"/>
           <el-option label="学员注册数统计" value="register_num"/>
           <el-option label="课程播放数统计" value="video_view_num"/>
           <el-option label="每日课程数统计" value="course_num"/>
         </el-select>
       </el-form-item>
       <el-form-item>
         <el-date-picker
           v-model="searchObj.begin"
           type="date"
           placeholder="选择开始日期"
           value-format="yyyy-MM-dd" />
       </el-form-item>
       <el-form-item>
         <el-date-picker
           v-model="searchObj.end"
           type="date"
           placeholder="选择截止日期"
           value-format="yyyy-MM-dd" />
       </el-form-item>
       <el-button
         :disabled="btnDisabled"
         type="primary"
         icon="el-icon-search"
         @click="showChart()">查询</el-button>
     </el-form>
     <div class="chart-container">
       <div id="chart" class="chart" style="height:500px;width:100%" />
     </div>
   </div>
 </template>
```

**js：暂时显示临时数据**

```vue
<script>
import echarts from 'echarts'
 export default {
     data(){
         return{
             searchObj: {
                type: '',
                begin: '',
                end: ''
            },
            btnDisabled: false,
            chart: null,
            title: '',
            xData: [],
            yData: []
         }
     },
     created(){

     },
     methods:{
        showChart(){
            // 基于准备好的dom，初始化echarts实例
            this.chart = echarts.init(document.getElementById('chart'))
            // console.log(this.chart)
            // 指定图表的配置项和数据
            var option = {
                // x轴是类目轴（离散数据）,必须通过data设置类目数据
                xAxis: {
                type: 'category',
                data: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']
                },
                // y轴是数据轴（连续数据）
                yAxis: {
                type: 'value'
                },
                // 系列列表。每个系列通过 type 决定自己的图表类型
                series: [{
                // 系列中的数据内容数组
                data: [820, 932, 901, 934, 1290, 1330, 1320],
                // 折线图
                type: 'line'
        
                }]
            }
            this.chart.setOption(option)
        },


     }
 }
 </script>
```



# 统计分析-图标显示接口

## controller

```java
// 图表显示，返回两部分数据，日期json数组 数量json数组
@PostMapping("showData/{type}/{begin}/{end}")
public R showData(@PathVariable String type,@PathVariable String begin,@PathVariable String end){
    Map<String,Object> map = dailyService.getShowData(type,begin,end);
    return R.ok().data(map);
}
```

## service

```java
// 图表显示，返回两部分数据，日期json数组 数量json数组
Map<String, Object> getShowData(String type, String begin, String end);
```

## serviceImpl

```java
// 图表显示，返回两部分数据，日期json数组 数量json数组
@Override
public Map<String, Object> getShowData(String type, String begin, String end) {
    // 先根据条件查询数据
    QueryWrapper<StatisticsDaily> wrapper = new QueryWrapper<>();
    wrapper.between("date_calculated",begin,end);
    // 由于只需要某个字段的数据，为了更加精确，所以有select
    wrapper.select("date_calculated",type);
    // 查询
    List<StatisticsDaily> staList = baseMapper.selectList(wrapper);

    // 按照条件构造日期json数组和数量json数组
    List<String> date_calculatedList = new ArrayList<>();
    List<Integer> numDataList = new ArrayList<>();

    for (int i = 0; i < staList.size(); i++) {
        StatisticsDaily daily = staList.get(i);
        // 将时间添加到时间集合
        date_calculatedList.add(daily.getDateCalculated());

        // 用switch判断选择的是哪个类型的数据
        switch (type){
            case "login_num":
                numDataList.add(daily.getLoginNum());
                break;
            case "register_num":
                numDataList.add(daily.getRegisterNum());
                break;
            case "video_view_num":
                numDataList.add(daily.getVideoViewNum());
                break;
            case "course_num":
                numDataList.add(daily.getCourseNum());
                break;
            default:
                break;
        }
    }
    // 封装数据返回
    Map<String, Object> map  = new HashMap<>();
    map.put("date_calculatedList",date_calculatedList);
    map.put("numDataList",numDataList);

    return map;
}
```



# 统计分析-图表显示前端

api/sta.js

```js
    // 图表显示
    getDataShow(searchObj){
        return request({
            url: `/staservice/sta/showData/${searchObj.type}/${searchObj.begin}/${searchObj.end}`,
            method: 'get'
          })
    }
```

methods

```vue
 methods:{
        showChart(){
            staApi.getDataShow(this.searchObj)
                .then(response =>{
                    this.xData = response.data.date_calculatedList
                    this.yData = response.data.numDataList

                    // 调用方法
                    this.setChart()
                })
         },
        setChart(){
            // 基于准备好的dom，初始化echarts实例
            this.chart = echarts.init(document.getElementById('chart'))
            // console.log(this.chart)
            // 指定图表的配置项和数据
            var option = {
                // x轴是类目轴（离散数据）,必须通过data设置类目数据
                xAxis: {
                type: 'category',
                data: this.xData
                },
                // y轴是数据轴（连续数据）
                yAxis: {
                type: 'value'
                },
                // 系列列表。每个系列通过 type 决定自己的图表类型
                series: [{
                // 系列中的数据内容数组
                data: this.yData,
                // 折线图
                type: 'line'
        
                }]
            }
            this.chart.setOption(option)
        },

```

# 样式调整

## 1、显示标题

```vue
// 显示标题
title: {
	text: "数据统计"
},
```

## 2、x坐标轴触发提示

```vue
// x坐标轴触发提示
tooltip: {
	trigger: 'axis'
},
```

## 3、区域缩放

```vue
// 区域缩放
dataZoom: [{
    show: true,
    height: 30,
    xAxisIndex: [
   	 0
    ],
    bottom: 30,
    start: 10,
    end: 80,
    handleIcon: 'path://M306.1,413c0,2.2-1.8,4-4,4h-59.8c-2.2,0-4-1.8-4-4V200.8c0-2.2,1.8-4,4-4h59.8c2.2,0,4,1.8,4,4V413z',
    handleSize: '110%',
    handleStyle: {
   	 color: '#d3dee5'
    },
    textStyle: {
    	color: '#fff'
    },
    borderColor: '#90979c'
    },
    {
        type: 'inside',
        show: true,
        height: 15,
        start: 1,
        end: 35
}]
```



# 