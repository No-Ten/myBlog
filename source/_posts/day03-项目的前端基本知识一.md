---
title: day03-项目的前端基本知识一
data: 2022-01-07 00:38:00
comment: 'valine'
category: 项目|谷粒学院|笔记
tags:
  - project
---



# day03-项目的前端基本知识一

# es6简单使用

## let变量作用范围

```html
<script>
    // es6如何定义变量 
    // js定义： var a = 1;  没有局部
    // es6定义let：有局部
    {
        var a = 10;
        let b = 20;

    }
    // 2.在代码块外面输入
    console.log(a);
    console.log(b);
</script>
```



## let定义变量特点

```html
<script>
    var a = 1;
    var a = 2;
    let b = 3;
    let b = 4; //Identifier 'b' has already been declared
</script>
```



## const声明常量

```html
<script>
     // 定义常量
     const PI = "3.1415";
     // 常量一旦定义，不能改变
     //PI = 3;    // Assignment to constant variable.
     // 定义常量必须初始化
     //const AA;  // Missing initializer in const declaration
</script>
```



## 数组解构

```html
<script>
    // 传统写法
    let a = 1,b = 2, c = 3;
    console.log(a,b,c);

    // es6写法
    let [x,y,z]= [10,20,30];
    console.log(x,y,z)
</script>
```



## 对象解构

```html
<script>
    // 定义对象
    let user = {"name":"lucy","age":20};
    
    // 传统的取值
    let name1 = user.name;
    let age1 = user.age;
    console.log(name1 + "==" + age1);

    // es6获取值
    let {name,age} = user;
    console.log(name + "--" + age);
</script>
```



## 模板字符串

```html
<script>
    // 1使用`实现换行
    let str1 = `hey,
    es6 demo up!`;
    console.log(str1);

    // 2.在符号`里面使用表达式取变量值
    let name = "Make";
    let age = 20;

    let str2 = `hello , ${name}, age is ${age}`
    console.log(str2);

    // 3.在`符号中调用方法
    function f1(){
        return "hello f1";
    }

    let str3 = `demo, ${f1()}`
    console.log(str3)

</script>
```



## 声明对象

```html
<script>
    const age = 12;
    const name = "lucy";

    // 传统方式定义对象
    const p1 = {name:name,age:age};
    console.log(p1);

    // es6定义变量
    const p2 = {name,age};
    console.log(p2);
</script>
```



## 定义方法简写方式

```html
<script>
    // 传统的定义方式
    const person1  = {
        sayHi:function(){
            console.log("Hi");
        }
    }
    person1.sayHi();
    // es6
    const person2 = {
        sayHi(){
            console.log("Hello");
        }
    }
    person2.sayHi();
</script>
```



## 对象拓展运算符

```html
<script>
    // 1.对象复制
    let person1 = {"name":"lucy","age":19}
    let person2 = {...person1}
    console.log(person2)

    // 2.对象合并
    let name = {name:"mary"}
    let age = {age:11111}
    let p3 = {...name,...age}
    console.log(p3)
</script>
```



## 箭头函数

```html
<script>
    // 1.传统的方式创建方法
    var f1 = function(m){
        return m
    }
    //console.log(f1(1))
    
    // 2.使用箭头函数
    var f2 = a => a
    //console.log(f2(2))

    // 3.复杂一点的
    var f3 = function(a,b){
        return a + b;
    }
    console.log(f3(1,1))

    // 使用箭头函数简化
    var f4 = (a,b) => a + b;
    console.log(f4(4,4));

</script>
```



# Vue入门案例

先将vue.js导入，编写一个HTML页面，导入vue.js文件，定义一个div显示数据，在script中new 一个Vue对象

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=<device-width>, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <div id="app">      
        {{ message }}           <!-- 定义一个div显示数据 --> 
    </div>
    <script src="vue.min.js"></script>
    <script>
        new Vue ({
            el: "#app",     // 判定vue作用的范围
            data:{          // 定义vue页面中显示的内容
                message:"Hello Vue!!"
            }
        })
    </script>
</body>
</html>
```



# 抽取代码块片段

在vs code中创建代码片段：
文件 =>  首选项 => 用户代码片段 => 新建全局代码片段/或文件夹代码片段：vue-html.code-snippets

```html
{
    "vue htm": {
        "scope": "html",
        "prefix": "vuehtml",
        "body": [
            "<!DOCTYPE html>",
            "<html lang=\"en\">",
            "",
            "<head>",
            "    <meta charset=\"UTF-8\">",
            "    <meta name=\"viewport\" content=\"width=device-width, initial-scale=1.0\">",
            "    <meta http-equiv=\"X-UA-Compatible\" content=\"ie=edge\">",
            "    <title>Document</title>",
            "</head>",
            "",
            "<body>",
            "    <div id=\"app\">",
            "",
            "    </div>",
            "    <script src=\"vue.min.js\"></script>",
            "    <script>",
            "        new Vue({",
            "            el: '#app',",
            "            data: {",
            "                $1",
            "            }",
            "        })",
            "    </script>",
            "</body>",
            "",
            "</html>",
        ],
        "description": "my vue template in html"
    }
}
```



# v-bind指令

单项绑定，取到属性中的值

```html

<div id="app">
    <h1 v-bind:title="content">
        {{message}}
    </h1>  
    <!-- 单向绑定 --> 
    <h2 :title = "content">
        {{message}}
    </h2>     
</div>
<script src="vue.min.js"></script>
<script>
    new Vue({
        el: '#app',
        data: {
            content:"我是标题",
            message:"hello"
        }
    })

```



# v-model 指令

双向绑定

```html
<div id="app">

    <!--单向判定，如果值发生改变，只要绑定的变化-->
    <input type="text" v-bind:value="searchMap.keyWord">

    <!--双向判定 ，如果值发生改变，所有的都会发生改变-->
    <input type="text" v-model="searchMap.keyWord">

    <p>{{searchMap.keyWord}}</p>

</div>
<script src="vue.min.js"></script>
<script>
    new Vue({
        el: '#app',
        data: {
            searchMap:{
                keyWord: '尚硅谷'
            }
        }
    })
</script>
```



# v-on指令

```html
<div id="app">
    <!--绑定事件-->
    <button v-on:click="search()">查询</button>

    <!--绑定事件简写-->
    <button @click="search()">查询1</button>
</div>
<script src="vue.min.js"></script>
<script>
    new Vue({
        el: '#app',
        data: {
            searchMap:{
                keyWord:'我是keyWord'
            }
        },
        methods:{
            // 可以定义多个方法
            search(){
                console.log('search...')
            },
            f1(){
                console.log('f1...')
            }
        }
    })
</script>
```



# Vue修饰符

阻止原始的条件发生，指定提交事件

```html
<div id="app">
    <form action="save" v-on:submit.prevent="onSubmit">
        <input type="text" id="name" v-model="user.username"/>
        <button type="submit">保存</button>
    </form>
</div>
<script src="vue.min.js"></script>
<script>
    new Vue({
        el: '#app',
        data: {
            user:{}
        },
        methods:{
            onSubmit(){
                if(this.user.username){
                    console.log('提交表单')
                } else {
                    alert('请输入用户名')
                }
            }
        }
    })
</script>
```



# vue指令v-if

条件指令

```html
<body>
    <div id="app">
        <input type="checkbox" v-model="ok"/> 是否同意
        <h1 v-if="ok">尚硅谷</h1>
        <h1 v-else>谷粒学院</h1></h1></h1>
    </div>
    <script src="vue.min.js"></script>
    <script>
        new Vue({
            el: '#app',
            data: {
                ok:false
            }
        })
    </script>
</body>
```



# vue指令v-for

```html
<body>
    <div id="app">

        <ul>
            <li v-for="n in 10">{{n}}</li>
        </ul>

        <ol>
            <li v-for="(n,index) in 10">{{n}}--{{index}}</li>
        </ol>

        <!--遍历列表-->
        <table border="1">
            <tr v-for="item in userList">
                <td>{{item.id}}</td>
                <td>{{item.username}}</td>
                <td>{{item.age}}</td>
            </tr>
        </table>

    </div>
    <script src="vue.min.js"></script>
    <script>
        new Vue({
            el: '#app',
            data: {
                userList:[
                    {id:1,username:'halen',age:10},
                    {id:2,username:'halen',age:10},
                    {id:3,username:'halen',age:10}
                ]
            }
        })
    </script>
</body>
```



# vue组件

```html
<body>
    <div id="app">
        <Navbar></Navbar>
    </div>
    <script src="vue.min.js"></script>
    <script>
        new Vue({
            el: '#app',
            // 定义局部组件，可以定义多个主键
            components:{
                    'Navbar':{
                        template:'<ul><li>首页</li><li>学员管理</li></ul>'
                    }
                }
        })
    </script>
</body
```



# vue全局组件

创建一个components文件，里面再创建一个Navbar.js

```html
// 定义全局组件
Vue.component('Navbar', {
    template: '<ul><li>首页</li><li>学员管理</li><li>讲师管理</li></ul>'
})
```

在需要用的地方直接引用

```html
<body>
    <div id="app">
        <Navbar></Navbar>
    </div>
    <script src="vue.min.js"></script>
    <script src="components/Navbar.js"></script>
    <script>
        new Vue({
            el: '#app',
            data: {
                
            }
        })
    </script>
</body>
```



# vue生命周期

![image-20211222125545310](day03-项目的前端基本知识一/image-20211222125545310.png)

主要用到created和mounted两个方法

```html
<body>
    <div id="app">
        hello
    </div>
    <script src="vue.min.js"></script>
    <script>
        new Vue({
            el: '#app',
            data: {
                
            },
            created(){
                debugger
                // 在渲染之前执行
                console.log('created...');
            },
            mounted(){
                debugger
                // 在渲染之后执行
                console.log('mounted...');
            }
        })
    </script>
</body>

```



# vue路由

将vue-router.min.js文件引入，编写路由

```html
<body>
    <div id="app">
        <h1>Hello App!</h1>
            <p>
                <!-- 使用 router-link 组件来导航. -->
                <!-- 通过传入 `to` 属性指定链接. -->
                <!-- <router-link> 默认会被渲染成一个 `<a>` 标签 -->
                <router-link to="/">首页</router-link>
                <router-link to="/student">会员管理</router-link>
                <router-link to="/teacher">讲师管理</router-link>
            </p>
            <!-- 路由出口 -->
            <!-- 路由匹配到的组件将渲染在这里 -->
            <router-view></router-view>
    </div>
    <script src="vue.min.js"></script>
    <script src="vue-router.min.js"></script>
    <script>
            // 1. 定义（路由）组件。
        // 可以从其他文件 import 进来
        const Welcome = { template: '<div>欢迎</div>' }
        const Student = { template: '<div>student list</div>' }
        const Teacher = { template: '<div>teacher list</div>' }
        // 2. 定义路由
        // 每个路由应该映射一个组件。
        const routes = [
            { path: '/', redirect: '/welcome' }, //设置默认指向的路径
            { path: '/welcome', component: Welcome },
            { path: '/student', component: Student },
            { path: '/teacher', component: Teacher }
        ]
        // 3. 创建 router 实例，然后传 `routes` 配置
        const router = new VueRouter({
            routes // （缩写）相当于 routes: routes
        })
        // 4. 创建和挂载根实例。
        // 从而让整个应用都有路由功能
        const app = new Vue({
            el: '#app',
            router
        })
        // 现在，应用已经启动了！
    </script>
</body>
```

