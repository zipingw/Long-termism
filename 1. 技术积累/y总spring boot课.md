[TOC]

#  y总spring boot课

- 基本类型
  ```java
  long d = 123444567778L;
  float e = 1.2F; // 默认为double
  double f = 1.2, g = 1.2D;
  final char s = 'A'; // 相当于C++中 const
  ```

 

- 隐式类型转化

  ```java
  int x = (int)c; // 显示
  int y = 12;
  double z = y; // 隐式
  double m = 4 * 3.3 // 默认隐式将4转为double
  int n = (int)(4 * 3.3);
  ```



- 输入输出

  ```java
  import java.util.Scanner;
  
  Scanner sc = new Scanner(System.in);
  sc.next(); // 读字符串，以空格断开
  sc.nextLine(); // 读一行
  
  import java.io.BufferedReader;
  import java.io.InputStreamReader;
  // Scanner 读入数据比较多，速度会慢
  BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
  String str = br.readLine(); // 读一行
  int x = br.read(); // 读一个字符
  
  import java.io.BufferedWriter;
  import java.io.OutputStreamWriter;
  BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(System.out));
  bw.write("Hello World\n");
  bw.flush();
  ```



- 读入整数

  ```java
  int x = Integer.parseInt(br.readLine());
  
  // 一行多个整数
  String[] strs = br.readLine().split(" ");
  int x = Integer.parseInt(strs[0]);
  ```

## 如何将项目通过git进行管理

### 创建git仓库并与本地绑定

1. 本地创建密钥，打开git bash

   ```bash
   cd ~
   ssh-keygen
   ```

   此时在`.ssh`目录下生成一对公私钥

   ```bash
   cat id_rsa.pub
   // 也可能是 id_ed25519.pub 取决于所用的加密算法不同
   ```

   到github中 `"Setting" => "New SSH Key" `，将公钥复制到其中并保存。

2. 在本地系统中选择一个文件目录位置创建项目文件，右键选择`"git bash here"`

   ```bash
   git init 
   touch readme.md
   git add .
   git commit -m "创建项目"
   // 如果未配置usr信息需要执行下面两行
   git config --global usr.name "xxx"
   git config --global usr.email "xxx@xxx.com"
   ```

3. 打开github，新建一个仓库，不要勾选创建readme.md文件（否则仓库中已有内容，不会出现导向性命令提示），在第2步中的git bash中执行导向性命令如下：

   ```bash
   git remote add origin git@github.com:zipingw/kob.git
   git branch -M main
   git push -u origin main
   ```

   刷新github，此时仓库中出现readme.md，说明本地项目成功上传至github仓库。

### 如何同步Home和Company的电脑共同维护一个项目

1. 假设基于前者已经在Home的PC中完成仓库建立，此时到Company的新电脑中，类似于前者步骤1，需要在本地生成一对公私钥。

   ```bash
   ssh-keygen
   cat id_rsa.pub
   // 在github中新增SSH Key,粘贴公钥
   ```

2. 打开github仓库，找到SSH克隆地址

   ```bash
   git clone xxx
   ```

3. 对项目进行修改之后，将修改后的项目上传至远端仓库。

   ```bash
   git add .
   git commit -m "xxx"
   git push
   ```

4. 在Home的PC中，可以直接拉取（pull）远端仓库的内容以获得在Company中完成的项目最新版本。

   ```bash
   git pull
   ```


### 当项目基础是从github上克隆得到时

- 此时如果在项目上做了一定的修改，希望将其中的内容通过建立自己的仓库进行管理，需要执行以下操作：

1. 先按照建立新仓库的步骤执行一遍，会发现在 `git commit` 之后在执行下面的命令时会报错：

   ```bash
   $ git remote add origin git@github.com:zipingw/rtree.git
   # 得到错误 error: remote origin already exists.
   ```

   上述错误表明，我们在`git clone`时，本地的项目与克隆的远程仓库已经建立了关联，如下所示
   
   ```bash
   $ git remote -v
   origin  https://github.com/davidmoten/rtree.git (fetch)
   origin  https://github.com/davidmoten/rtree.git (push)
   ```

2. 所以我们需要先断开与原代码仓库的联系，再关联我们自己的代码仓库

   ```bash
   $ git remote rm origin
   $ git remote add origin git@github.com:zipingw/rtree.git
   ```

3. 此时再将本地项目内容push至远端仓库即可成功

   ```bash
   $ git push -u origin main
   ```


## Java后端与Vue前端项目框架搭建

- King of Bots 项目整体规划

  - PK模块
    - 匹配界面(微服务)
    - 机器PK实况界面(Websocket协议):对战双方的代码保持连接,每秒一个回合,需要将每个回合的结果返回到前端显示出来,不能用http协议,无法保持连接,websocket可以保持连接
    - 真人PK界面:实现人人对战与人机对战,聊天框
  - 对战列表: 实时更新让用户感受到网站的活力
    - 录像回放
  - 排行榜
    - Bot列表
  - 用户中心
    - 注册功能
    - 登录功能
    - 我的Bot
    - Bot的记录

- 采用前后端分离写法,可以复用后端,同时加载到Web端和App端并使二者保持同步

  - Web端
    - 导航栏
    - 内容
  - App端
    - PK
    - 对局
    - 排行榜

- 项目文件目录

  - Kob
    - backend
    - web
    - acapp

- 前后端的概念 : 二者都是存在服务器上, 后端和前端不一定要存在一个机器上, 用户打开一个网页或者App本质上是向服务器发送了一个请求, 服务器端接收到请求后返回一个前端页面, 本质上就是调用了一个函数, 函数参数在url里面, 返回的前端页面相当于一个字符串. 传统模式为前后端不分离, 会在服务器端生成完整的**HTML**, 前后端分离的模式会先返回**静态**信息, 再通过前端浏览器请求后端得到数据后**动态渲染**前端页面, 前后端分离的方式最大的好处是可以**复用后端**数据, 只是前端展示的形式不同 

  - Client : FrontEnd网页或者软件(vue3,react)
    - web
    - AcApp
  - Server : Backend
    - MySQL
    - Redis
    - 硬盘
    - 微服务

### 后端SpringBoot项目创建

IDEA新建 `Spring Initializr` 

- 创建项目,默认源为 `start.spring.io` ，修改 `Name` 为项目名称 `backend` (`artifact` 会与此保持同名)，修改 `Group` 为 `com.kob` 其中 `kob` 为该后端项目最上层包名称。

<img src="C:\Users\zpwang\Desktop\0_master\Long-termism\resources\img\SpringBoot后端项目创建1.png" style="zoom: 67%;" />

- 选择相关依赖,勾选`Web`中的`Spring Web` 和`Template Engines`中的`Thymeleaf` 后者用于实现前后端不分离

<img src="C:\Users\zpwang\Desktop\0_master\Long-termism\resources\img\SpringBoot后端项目创建2.png" style="zoom: 67%;" />

- 创建完成后的项目目录

<img src="C:\Users\zpwang\Desktop\0_master\Long-termism\resources\img\SpringBoot后端项目创建3.png" style="zoom: 67%;" />

- SpringBoot的入口函数

<img src="C:\Users\zpwang\Desktop\0_master\Long-termism\resources\img\SpringBoot后端项目创建4.png" style="zoom: 67%;" />

- 运行入口函数得到如下控制台信息, 

<img src="C:\Users\zpwang\Desktop\0_master\Long-termism\resources\img\SpringBoot后端项目创建5.png" style="zoom: 50%;" />

- 打开浏览器输入`127.0.0.1:8080`显示界面如下:

  <img src="C:\Users\zpwang\Desktop\0_master\Long-termism\resources\img\SpringBoot后端项目创建13.png" style="zoom:60%;" />

- 展示前后端不分离模式写法: `SpringBoot`后端主要是用来实现函数的,每一个`url`链接对应一个函数,`url`中的参数就是函数参数, 在`SpringBoot`中函数可以写在任何地方,只有加个`controller`就可以实现映射, 一般会新建一个软件包`controller,`在该软件包中实现所有的后端函数

<img src="C:\Users\zpwang\Desktop\0_master\Long-termism\resources\img\SpringBoot后端项目创建6.png" style="zoom: 67%;" />

- 项目分为四大块:PK, 对局记录, 排行榜, 用户中心. 在`controller`中创建一个包`pk`,在`pk`包中创建一个类`IndexController`用于控制`pk`页面的主页面

<img src="C:\Users\zpwang\Desktop\0_master\Long-termism\resources\img\SpringBoot后端项目创建7.png" style="zoom: 50%;" />

<img src="C:\Users\zpwang\Desktop\0_master\Long-termism\resources\img\SpringBoot后端项目创建8.png" style="zoom: 50%;" />

- 如何将`url`链接绑定到对应的函数呢?需要加上一个注解`@Controller`, 这个类中能够响应的所有`url`链接都是在`pk`目录下, 所以在`pk`包中的`IndexController`类中再增加一层父目录注解`@RequestMapping("/pk/")`

  ```java
  package com.kob.backend.controller.pk;
  import org.springframework.stereotype.Controller;
  import org.springframework.web.bind.annotation.RequestMapping;
  @Controller
  @RequestMapping("/pk/")
  public class IndexController {
  }
  ```

- 如果我们要通过`pk`包中的`IndexController`类返回一个页面, 需要先创建页面, 这里如果是前后端不分离的模式, 就在`resources`的`templates`中创建一个`pk`目录, 在其中创建一个`HTML`文件, 并在`body`中写入一个`<h1>Hello World</h1>`

<img src="C:\Users\zpwang\Desktop\0_master\Long-termism\resources\img\SpringBoot后端项目创建11.png" style="zoom:50%;" />

<img src="C:\Users\zpwang\Desktop\0_master\Long-termism\resources\img\SpringBoot后端项目创建12.png" style="zoom:60%;" />

- 之后可以在`IndexController`类中编写如下代码

  ```java
  package com.kob.backend.controller.pk;
  
  import org.springframework.stereotype.Controller;
  import org.springframework.web.bind.annotation.RequestMapping;
  
  @Controller
  @RequestMapping("/pk/")
  public class IndexController {
      @RequestMapping("index/")
      public String index(){
          return "pk/index.html";
      }
  }
  ```

- 重新运行, 浏览器中输入`127.0.0.1:8080/pk/index/`即可看到`Hello World`

- 如果采用前后端分离模式, 后端中返回的不再是`HTML`页面, 而是一些数据

- 后端可以在如下目录中的`application.properties`文件中加入如下信息修改端口号: `server.port=3000` , 否则会与`vue`所用的端口发生冲突

  <img src="C:\Users\zpwang\Desktop\0_master\Long-termism\resources\img\SpringBoot后端项目创建14.png" style="zoom: 67%;" />

- 前端通过下述代码通过`url`调用后端函数时, 会遇到跨域问题, 因为`SpringBoot`在域名`127.0.0.1:3000`而前端在`127.0.0.1:8080`

  ```vue
  <template>
    <div>
        <div>
            Bot昵称:{{ bot_name }}
      </div>
        <div>
            Bot战力:{{ bot_rating}}
      </div>
      </div>
  	<router-view></router-view>
  </template>
  
  <script>
  import $ from 'jqeury';
  import { ref } from 'vue';
  
  export default {
    name: "App",
      setup: () => {
          let bot_name = ref("");
          let bot_rating = ref("");
          $.ajax({
              url: "http://127.0.0.1:3000/pk/geetbotinfo/",
          });
          return {
              bot_name,
              bot_rating
          }
      }
  }
  </script>
  <style>
  </style>
  ```

  <img src="C:\Users\zpwang\Desktop\0_master\Long-termism\resources\img\SpringBoot后端项目创建15.png" style="zoom:60%;" />

- 在`backend`中创建`config`包并创建`CorsConfig`类, 并在其中填入下述代码即可

  ```java
  package com.kob.backend.config;
  
  import org.springframework.context.annotation.Configuration;
  
  import jakarta.servlet.*;
  import jakarta.servlet.http.HttpServletRequest;
  import jakarta.servlet.http.HttpServletResponse;
  import java.io.IOException;
  
  @Configuration
  public class CorsConfig implements Filter {
      @Override
      public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain) throws IOException, ServletException {
          HttpServletResponse response = (HttpServletResponse) res;
          HttpServletRequest request = (HttpServletRequest) req;
  
          String origin = request.getHeader("Origin");
          if(origin!=null) {
              response.setHeader("Access-Control-Allow-Origin", origin);
          }
  
          String headers = request.getHeader("Access-Control-Request-Headers");
          if(headers!=null) {
              response.setHeader("Access-Control-Allow-Headers", headers);
              response.setHeader("Access-Control-Expose-Headers", headers);
          }
  
          response.setHeader("Access-Control-Allow-Methods", "*");
          response.setHeader("Access-Control-Max-Age", "3600");
          response.setHeader("Access-Control-Allow-Credentials", "true");
  
          chain.doFilter(request, response);
      }
  
      @Override
      public void init(FilterConfig filterConfig) {
  
      }
  
      @Override
      public void destroy() {
      }
  }
  ```

### 前端界面

- web项目
  - 需要安装的插件: vue-router, vuex
  - 需要安装的依赖: jquery, bootstrap
  - 任务-serve-运行-输出 打开对应网址
- acapp项目
  - 需要安装的插件: vuex
  - 需要安装的依赖: 无
  - 任务-serve-运行-输出 打开对应网址
- 去掉router/index.js中首行的中`createWebHashHistory`中的`Hash` 以及下方的另一个`Hash`,目的是为了去掉链接中的"#"符号
- 安装Vue3的过程
  - [Vue3官网](https://vuejs.org/)
  - [安装Nodejs](https://nodejs.org/en/)
  - 打开PowerShell 或者 Gitbash执行`npm i -g @vue/cli`
  - 启动vue自带的图形化项目管理界面 `vue ui` 
  - 常见问题1：Windows上运行vue，提示无法加载文件，表示用户权限不足。
    解决方案：用管理员身份打开终端，输入set-ExecutionPolicy RemoteSigned，然后输入y
  - 常见问题2：在新的系统中`git clone`项目之后，在`vue ui`中导入项目之前需要进入项目所在文件夹执行`npm install`命令

## Vue前端设计

1. 分析页面:

   - 导航栏:每个页面都有,故添加至组件方便复用 [bootstrap 导航栏模板](https://v5.bootcss.com/)
   - 内容区

2. components中增加NavBar(使用bootstrap中得到的模板)

3. App.vue中导入NavBar组件

   ```vue
   <template> 
     <NavBar />  
     <router-view/>
   </template>
   
   <script>      // js中 导入相关组件
   import NavBar from '@/components/NavBar.vue'
   import "bootstrap/dist/css/bootstrap.min.css"
   import "bootstrap/dist/js/bootstrap"
   
   export default {
     components:{
       NavBar
     }
   }
   </script>
   ```

4. 修改NavBar以符合设计需求

   ```vue
   <template>
       <nav class="navbar navbar-expand-lg bg-body-tertiary">
           <div class="container-fluid">
             <a class="navbar-brand" href="#">King of Bots  </a>
   
             <div class="collapse navbar-collapse" id="navbarText">
               <ul class="navbar-nav me-auto mb-2 mb-lg-0">
                 <li class="nav-item">
                   <a class="nav-link " aria-current="page" href="#"> 对战</a>
                 </li>
                 <li class="nav-item">
                   <a class="nav-link" href="#">对局列表</a>
                 </li>
                 <li class="nav-item">
                   <a class="nav-link" href="#">排行榜</a>
                 </li>
               </ul>
   
               <ul class="navbar-nav">
                   <li class="nav-item dropdown">
                       <a class="nav-link dropdown-toggle" href="#" role="button" data-bs-toggle="dropdown" aria-expanded="false">
                         Kedjor
                       </a>
                       <ul class="dropdown-menu">
                         <li><a class="dropdown-item" href="#">我的Bot</a></li>
                         <li><a class="dropdown-item" href="#">else</a></li>
                         <li><hr class="dropdown-divider"></li>
                         <li><a class="dropdown-item" href="#">Quit</a></li>
                       </ul>
                     </li>
                 </ul>
             </div>
           </div>
         </nav>
   </template>
   
   <script>
   
   </script>
   
   <style scoped>
   
   </style>
   ```

5. 目前点击导航栏网址是不发生变化的,现在希望能够进行网址跳转,点击导航栏中相应的`nav-link`可以跳转至相应的页面,首先我们需要先将每个页面写好对应组件,页面组件放到`components`中也可以,一般放到`views`中,虽然定义在`views`中,但是每个`.vue`文件仍然是作为一个组件

   - pk
   - record
   - ranklist
   - user/bot
   - 404 Note found

6. views中增加各个页面的视图结构,由于每个页面中可能包含多个组件,故每个页面创建一个文件夹,如下所示:

   <img src="../resources/img/SpringBoot 前端目录.png" alt="目录" style="zoom: 67%;" />

- 每个页面的基础空白模板如下
   ```vue
   <template>
   </template>
   <script>
   </script>
   <style scoped>
   </style>
   ```


7. 现在希望将`Views`中写好的页面与网址进行绑定

   在`App.vue`中的`template`中 有一个`router-view`, 这个会自动根据网址的变化进行变化, `router-view`的变化方式定义在`router`文件夹中的`index.js`文件中

   按照如下方式定义:先将组件`import`进来,再在`routes`中将`path`与对应的组件进行绑定,之后网址发生相应变化时,就会自动跳转至对应的组件页面

   并且再routes额外增加一个如果网址为根目录"/",需要重定向至`pk`页面, 这里定义的`name`可以后续应用于`router-link`功能

   ```js
   import { createRouter, createWebHistory } from 'vue-router'
   import PkIndexView from '@/views/pk/PkIndexView'
   import RanklistIndexView from '@/views/ranklist/RanklistIndexView'
   import RecordIndexView from '@/views/record/RecordIndexView'
   import UserBotIndexView from '@/views/user/bot/UserBotIndexView'
   import NotFound from '@/views/error/NotFound'
   
   const routes = [
     {
       path: "/",
       name: "Home",
       redirect:"/pk/",
     },
     {
       path: "/pk/",
       name: "pk_index",
       component: PkIndexView,
     },
     {
       path: "/record/",
       name: "record_index",
       component: RecordIndexView,
     },
     {
       path: "/ranklist/",
       name: "ranklist_index",
       component: RanklistIndexView,
     },
     {
       path: "/user/bot/",
       name: "user_bot_index",
       component: UserBotIndexView,
     },
     {
       path: "/404/",
       name: "404_index",
       component: NotFound,
     },
     
   ]
   
   const router = createRouter({
     history: createWebHistory(),
     routes
   })
   
   export default router
   ```

8. 现在只能通过输入网址来控制页面的切换,我们希望能够通过点击`NavBar`中的内容来控制页面的跳转,这需要在`NavBar`中进行相应的修改,只要更改每个`<li>`中的`href`即可

   ```bash
   <li class="nav-item">
   	<a class="nav-link" href="/record/">对局列表</a>
   </li>
   ```

9. 目前点击`NavBar`中的内容时,页面会刷新一下,前后端分离的写法可以实现点击`NavBar`不刷新,`router`组件提供了相应功能,将`<a></a>`换成`<router-link></router-link>`,其中`:to="{name: 'home'}"`对应的`name`即为`router`文件夹下的`index.js`文件中所定义的

   ```vue
   <a class="navbar-brand" href="/pk/">King of Bots  </a>
   <router-link class="navbar-brand" :to="{name: 'home'}">King of Bots</router-link> 
   ```

10. 我们现在希望将每个页面中的内容用`Card`框起来,使得页面更加美观,由于每个界面都会用到这样一个container,故将其单独写为一个组件,在`components`新建一个文件`ContentField.vue`,并写入以下内容,其中最内层的`<slot>`提供了一个内容填充区域,当其他页面`import`该组件后会自动将内容填充进其中

    - 这里可以有个快速写法`div.container>div.card>div.card-body>`再按下回车即可

    ```vue
    <template>
        <div class="container">
            <div class="card">
                <div class="card-body">
                    <slot></slot>
                </div>
            </div>
        </div>
    </template>
    
    <script></script>
    
    <style scoped></style>
    ```

- `PkIndexView.vue`中可以如下使用`ContentField`组件
  
    ```vue
    <template>
        <ContentField>
            对战
        </ContentField>
    </template>
    
    <script>
    import ContentField from '@/components/ContentField'
    
    export default {
        components: {
            ContentField
        }
    }
    </script>
    
    <style scoped>
    
    </style>
    ```
    

11. 最后还有一个导航栏的聚焦高亮功能,高亮实现方式只是在`NavBar.vue`中的`<router-link>`的`class="nav-link"`后面加上一个`active`,这里需要根据当前所处页面来调整是否增加`active`

    故需要先判断出当前出于哪个页面,这可以通过`vue-router`获得

    ```vue
    <script>
    import { useRoute } from 'vue-router'
    import { computed } from 'vue'
    
    export default {
        setup() {
            const route = useRoute();
            let route_name = computed(() => route.name)
            return {
                route_name
            }
        }
    }
    </script>
    ```

    上述代码可以得到`route_name`,再按如下方式使用`? :`表达式判断定义`class`

    ```vue
    <li class="nav-item">
    	<router-link :class="route_name == 'pk_index' ? 'nav-link active' : 'nav-link'" :to="{name: 'pk_index'}">对战</router-link>
    </li>
    ```

### 对战地图

1. 在`assets`目录中建两个文件夹`iamges`和`scripts`，将背景图片放入前者，并在后者中新建`AcGameObject.js`实现游戏对象基类，其中包含下面一些元素

   - `start`执行一次动作
   - `update`每帧更新一次
   - `on_destroy`删除之前执行
   - `destroy`删除时动作

   ```js
   const AC_GAME_OBJECTS = [];
   
   export class AcGameObject {
       constructor() {
           AC_GAME_OBJECTS.push(this);
           this.has_call_start = false;
           this.timedelta = 0; // 用于计算物体移动速度
       }
       start() { // 只执行一次
       }
       update() { // 每一帧执行一次
       }
       on_destroy() { // 删除之前执行
       }
       destroy() {
           this.on_destroy();
   		// 回收每个对象
           for (let i in AC_GAME_OBJECTS) {
               const obj = AC_GAME_OBJECTS[i];
               if(obj === this) {
                   AC_GAME_OBJECTS.splice(i); //splice用于回收
                   break;
               }
           }   
       }
   }
   let last_timestamp; //上一次执行的时刻
   // 这段看不太懂...抄就完了
   const step = timestamp => {
       for (let obj of AC_GAME_OBJECTS) {
           if(!obj.has_call_start) {
               obj.has_call_start = true;
               obj.start();
           } else {
               obj.timedelta = timestamp - last_timestamp;
               obj.update();
           }
       }
       last_timestamp = timestamp;
       requestAnimationFrame(step)
   }
   requestAnimationFrame(step)
   ```

2. 对战地图中需要实现两个类，分别是地图和墙，每一帧都要重新渲染这两个对象，首先实现地图类，在`assets/scripts`中创建`GameMap.js`实现地图类，这个`js`脚本文件继承于`AcGameObject`基类，

   ```js
   import { AcGameObject } from "./AcGameObject";
   import { Wall } from "./Wall";
   export class GameMap extends AcGameObject {
       constructor(ctx, parent) { // 参数为画布和画布父元素，后者用于动态修改画布长宽
           super();
           this.ctx = ctx;
           this.parent = parent;
           this.L = 0;
           this.rows = 13;
           this.cols = 13;
           this.inner_walls_count = 20;
           this.walls = [];
       }
       check_connectivity(g, sx, sy, tx, ty) {
           if(sx == tx && sy == ty)return true;
           g[sx][sy] = true;
   
           let dx = [-1, 0, 1, 0], dy = [0, 1, 0, -1];
           for(let i = 0; i < 4; i++) {
               let x = sx + dx[i], y = sy + dy[i];
               if (!g[x][y] && this.check_connectivity(g, x, y, tx, ty)){
                   return true;
               }
           }
           return false;
       } 
       create_walls () {
           const g = [];
           for(let r = 0; r < this.rows; r++){
               g[r] = [];
               for(let c = 0; c < this.cols; c++){
                   g[r][c] = false;
               }
           }
           for(let r = 0; r < this.rows; r++){
               g[r][0] = g[r][this.cols - 1] = true;
           }
           for(let c = 0; c < this.cols; c++){
               g[0][c] = g[this.rows - 1][c] = true;
           }
           // 创建随机障碍物 
           for(let i = 0; i < this.inner_walls_count / 2; i++){
               for(let j = 0; j < 1000; j++){
                   let r = parseInt(Math.random() * this.rows);
                   let c = parseInt(Math.random() * this.cols);
                   if(g[r][c] || g[c][r])continue;
                   if(r == this.rows - 2 && c == 1 || r == 1 && c == this.cols - 2)continue;
                   g[r][c] = g[c][r] = true;
                   break;
               }
           }
           // 深度复制 g
           const copy_g = JSON.parse(JSON.stringify(g));
           if(!this.check_connectivity(copy_g, this.rows - 2, 1, 1, this.cols - 2)) return false;
           for(let r = 0; r < this.rows; r++){
               for(let c = 0; c < this.cols; c++){
                   if(g[r][c])this.walls.push(new Wall(r, c, this));
               }
           }      
           return true;
       }
       start() {
           for(let i = 0; i < 1000; i++){
               if(this.create_walls()){
                   break;
               }
           }
       }
       update_size() {
           this.L = parseInt(Math.min(this.parent.clientWidth / this.cols, this.parent.clientHeight / this.rows));
           this.ctx.canvas.width = this.L * this.cols;
           this.ctx.canvas.height = this.L * this.rows;
       }
       update() {
           this.update_size();
           this.render();
       }
       // 作图
       render() {
           const color_even = "#AAD751", color_odd = "#A2D149";
           for(let r = 0; r < this.rows; r++){
               for(let c = 0; c < this.cols; c++){
                   if((r + c) % 2 == 1){
                       this.ctx.fillStyle = color_even;
                   }
                   else {
                       this.ctx.fillStyle = color_odd;
                   }
                   this.ctx.fillRect(this.L * c, this.L * r, this.L, this.L);
               }
           }
       }
   }
   ```

3. `assets/scripts`中创建`Wall.js`实现障碍物类，用于作为`GameMap`中的障碍物填充

   ```js
   import { AcGameObject } from "./AcGameObject";
   
   export class Wall extends AcGameObject {
       constructor(r, c, gamemap) {
           super();
           this.c = c;
           this.r = r;
           this.gamemap = gamemap;
           this.color = "#B37226";
       }
       update () {
           this.render();
       }
       // 渲染单点元素
       render () {
           const L = this.gamemap.L;
           const ctx = this.gamemap.ctx;
           ctx.fillStyle = this.color;
           ctx.fillRect(this.c * L, this.r * L, L, L);
       }
   }
   ```

4. 在`components`中创建`PlayGround.vue`组件，作为`pk`页面中的底子，在`PkIndexView.vue`中添加如下代码使用`PlayGround`组件：

   ```vue
   <template>
       <PlayGround />
   
   </template>
   
   <script>
   import PlayGround from '@/components/PlayGround.vue'
   
   export default {
       components: {
           PlayGround
       }
   }
   </script>
   
   <style scoped>
   
   </style>
   ```

5. `PlayGround`组件的代码如下所示，定义了位置信息调用`GameMap`组件

   ```vue
   <template>
       <div class="playground">
           <GameMap>
           </GameMap>
       </div>
   </template>
   
   <script>
   import GameMap from './GameMap.vue';
   export default {
       components: {
           GameMap,
       }
   }
   </script>
   
   <style scoped>
   div.playground {
       width: 60vw;
       height: 80vh;
       margin: 40px auto;
   }
   </style>
   ```

6. 在`components`中创建`GameMap.vue`组件，引用`GameMap.js`并创建实例

   ```vue
   <template>
       <div ref="parent" class="gamemap">
           <canvas ref="canvas"></canvas>
       </div>
   </template>
   
   <script>
   import { GameMap } from '@/assets/scripts/GameMap.js'
   import { ref, onMounted } from 'vue'
   export default {
   
       setup() {
           let parent = ref(null);
           let canvas = ref(null);
           
           onMounted(() => {
               new GameMap(canvas.value.getContext('2d'), parent.value)
           });
   
           return {
               parent,
               canvas
           }
       }
   }
   
   </script>
   
   <style scoped>
   div.gamemap {
       width: 100%;
       height: 100%;
       display: flex;  
       justify-content: center;
       align-content: center;
   }
   </style>
   ```

7. 最后将`web/public/`目录下的`favicon.ico`替换为网络图标，至此`pk`网页雏形搭建完毕

## MySQL安装

- 安装完成将`/MySQL/MySQL Server 8.0/bin`添加到环境变量中
- 运行`mysql --version`验证安装是否完成
- 以管理员权限启动`PowerShell`通过`net stop mysql80`和`net start mysql80`来关闭和启动`mysql`
- `mysql -uroot -p`输入密码后登录,or`mysql -uroot -p123456`直接命令中跟密码（不安全）
  - `show databases;`
  - `create database kob;`
  - `use kob;`  // 进入数据库
  - `show tables;`
  - `drop database kob;`
  - `create table user(id int, username varchar(100), password varchar(100));`
  - `show tables;`  // 所有table
  - `drop table user;`
  - `insert into user values(1, 'wzp', 'pwzp');`
  - `select * from user;`
  - `select * from user where id=1;`
  - `delete * from user where id=2;`
  - `exit`or`quit`退出

### 如何让Spring后端操作MySQL

- IDEA 中右上角点击数据库符号，按步骤载入即可

- Spring中要调用数据库首先需要载入一些**依赖**

  - 到网站[Maven仓库地址](https://mvnrepository.com/)中搜索复制即可，`maven`和`gradle`都有，添加后在IDEA中刷新加载

    ```bash
    Spring Boot Starter JDBC
    Project Lombok
    MySQL Connector/J
    mybatis-plus-boot-starter
    mybatis-plus-generator
    # Mybatis依赖传递的直接依赖中有个mybatis-spring 默认版本2.1.2 若使用springboot3.0以上版本会导致不适配
    mybatis-spring 3.0.3 #需要额外添加高版本依赖，否则启动项目时报错
    
    spring-boot-starter-security
    jjwt-api
    jjwt-impl
    jjwt-jackson
    ```

  - 依赖问题参考[MyBatisPlus使用时报错Invalid value type for attribute ‘factoryBeanObjectType‘](https://blog.csdn.net/lonely__snow/article/details/134897066) [原因分析](https://github.com/mybatis/spring/pull/865)

- 在`resources`中的`application.properties`中添加以下信息：

  ```properties
  server.port=3001
  
  spring.datasource.username=root
  spring.datasource.password=Xwzp581012
  spring.datasource.url=jdbc:mysql://localhost:3306/kob?serverTimezone=Asia/Shanghai&useUnicode=true&characterEncoding=utf-8
  spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
  ```

- Spring调用数据库实现业务逻辑的分层结构

  - `pojo`层：向下连接数据库层，将数据库中的`table`转为`Java`中`Class`
  - `mapper`层：将`pojo`层中`Class`中的操作映射为`sql`语句
  - `service`层：作为实现业务逻辑的基本单元，对`mapper`层操作进行组合
  - `controller`层：负责请求转发，接受前端页面`url`传入的参数，分发任务给`Service`层

- 故整体项目结构大致如下：

  - `backend`
    - `config`
    - `pojo`
    - `mapper`
    - `service`
    - `controller`

- 实现`pojo`层：创建`Class`命名为数据库`table`名称`User`(大小写好像无所谓？) 在其中写入以下内容：其中通过三个`@`可以快速实现机械重复的基础类函数的编写

  ```java
  package com.kob.backend.pojo;
  
  import lombok.AllArgsConstructor;
  import lombok.Data;
  import lombok.NoArgsConstructor;
  
  @Data
  @NoArgsConstructor
  @AllArgsConstructor
  public class User {
      private Integer id;
      private String username;
      private String password;
  }
  ```

- 实现`mapper`层(为何为`interface`?)：

  ```java
  package com.kob.backend.mapper;
  
  import com.baomidou.mybatisplus.core.mapper.BaseMapper;
  import com.kob.backend.pojo.User;
  import org.apache.ibatis.annotations.Mapper;
  
  @Mapper
  public interface UserMapper extends BaseMapper<User> {
      // 这里只需要定义接口
      // 接口由Mybatisplus实现好了，直接调用
  }
  ```

- 实现`service`层

- 实现`controller`层
  - 目录结构分为四个包，分别对应前端的四个页面`pk`、`user`、`record`、`ranklist`

  - 以`user`为例，在`user`包中创建`UserController`类
    - 在类定义的外面加上注解`@RestController`
    
    - 在内容加注解`RequestMapping` or `GetMapping` or `PostMapping`，并定义函数`public List<User> getAll()`，这里的`User`即为`pojo`中定义的类，该函数希望将所有用户作为列表返回
    
    - 如何返回？调用`Mapper`层
    
      ```java
      package com.kob.backend.controller.user;
      
      import com.baomidou.mybatisplus.core.conditions.query.QueryWrapper;
      import com.kob.backend.pojo.User;
      import com.kob.backend.mapper.UserMapper;
      import org.springframework.beans.factory.annotation.Autowired;
      import org.springframework.web.bind.annotation.GetMapping;
      import org.springframework.web.bind.annotation.PathVariable;
      import org.springframework.web.bind.annotation.RestController;
      
      import java.util.List;
      
      
      @RestController
      public class UserController {
      
          @Autowired
          UserMapper userMapper;
      
          @GetMapping("/user/all/")
          public List<User> getAll() {
              return userMapper.selectList(null);
          }
      
          @GetMapping("/user/{userId}")
          public User getuser(@PathVariable int userId){
              QueryWrapper<User> queryWrapper = new QueryWrapper<>();
              queryWrapper.eq("id", userId);
              return userMapper.selectOne(queryWrapper);
          }
      
          @GetMapping("/user/add/{userId}/{username}/{password}/")
          public String addUser(@PathVariable int userId,
                                @PathVariable String username,
                                @PathVariable String password) {
              User user = new User(userId, username, password);
              userMapper.insert(user);
              return "Add User Successfully";
          }
      
          @GetMapping("/user/delete/{userId}/")
          public String deleteUser(@PathVariable int userId){
              userMapper.deleteById(userId);
              return "Delete User Successfully";
          }
      }
      ```

- 实现`Security`目前任何人都可以访问网页操作数据库，我们希望只有登录后才允许操作，添加依赖`spring-boot-starter-security`，该依赖添加后再启动主程序通过`url`调用时就会弹出登录界面，该界面由`Security`实现，默认用户名为`user`，默认密码在启动`Springboot`时输出的命令行中，在`Service`子目录下添加`impl`，再new一个class为`UserDetailServiceImpl`，并令其实现接口`UserDetailService`用于接入数据库信息

  ```java
  package com.kob.backend.service.impl;
  
  import com.baomidou.mybatisplus.core.conditions.query.QueryWrapper;
  import com.kob.backend.pojo.User;
  import com.kob.backend.mapper.UserMapper;
  import com.kob.backend.service.impl.utils.UserDetailsImpl;
  import org.springframework.beans.factory.annotation.Autowired;
  import org.springframework.security.core.userdetails.UserDetails;
  import org.springframework.security.core.userdetails.UserDetailsService;
  import org.springframework.security.core.userdetails.UsernameNotFoundException;
  import org.springframework.stereotype.Service;
  
  @Service
  public class UserDetailServiceImpl implements UserDetailsService {
  
      @Autowired
      UserMapper userMapper;
  
      @Override
      public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
          QueryWrapper<User> queryWrapper = new QueryWrapper<>();
          queryWrapper.eq("username", username);
          User user = userMapper.selectOne(queryWrapper);
          if(user == null) {
              throw new RuntimeException("用户不存在");
          }
          return new UserDetailsImpl(user);
      }
  }
  ```

- 在`impl`中还需要新建一个包`utils`，在其目录下新建类`UserDetailsImpl`，并在其中加入以下内容，这里需要修改`Account`的一些权限默认值：

  ```java
  package com.kob.backend.service.impl.utils;
  
  import lombok.AllArgsConstructor;
  import lombok.Data;
  import lombok.NoArgsConstructor;
  import org.springframework.security.core.GrantedAuthority;
  import com.kob.backend.pojo.User;
  import org.springframework.security.core.userdetails.UserDetails;
  
  import java.util.Collection;
  
  @Data
  @NoArgsConstructor
  @AllArgsConstructor
  public class UserDetailsImpl implements UserDetails {
  
      private User user;
  
      @Override
      public Collection<? extends GrantedAuthority> getAuthorities() {
          return null;
      }
  
      @Override
      public String getPassword() {
          return user.getPassword();
      }
  
      @Override
      public String getUsername() {
          return user.getUsername();
      }
  
      @Override
      public boolean isAccountNonExpired() {
          return true;
      }
  
      @Override
      public boolean isAccountNonLocked() {
          return true;
      }
  
      @Override
      public boolean isCredentialsNonExpired() {
          return true;
      }
  
      @Override
      public boolean isEnabled() {
          return true;
      }
  }
  ```

- 但目前登录输入密码会失败，原因是缺少加密模块，在`config`包中新建类`SecurityConfig`，并在其中加入以下内容：

  ```java
  package com.kob.backend.config;
  
  import org.springframework.context.annotation.Bean;
  import org.springframework.context.annotation.Configuration;
  import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
  import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
  import org.springframework.security.crypto.password.PasswordEncoder;
  
  @Configuration
  @EnableWebSecurity
  public class SecurityConfig {
  
      @Bean
      public PasswordEncoder passwordEncoder() {
          return new BCryptPasswordEncoder();
      }
  }

- 在`Test`中的`BackendApplicationTests`中写入

  ```java
  package com.kob.backend;
  
  import org.junit.jupiter.api.Test;
  import org.springframework.boot.test.context.SpringBootTest;
  import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
  import org.springframework.security.crypto.password.PasswordEncoder;
  
  @SpringBootTest
  class BackendApplicationTests {
      @Test
      void contextLoads() {
          PasswordEncoder passwordEncoder = new BCryptPasswordEncoder();
          System.out.println(passwordEncoder.encode("pwzp"));
          //System.out.println(passwordEncoder.matches("pwzp", "$2a$10$XOLMwJOUWG1dxAWqUXsT4u/rYdVbSHjsIPLJI4AVrfu1MiYRoiXpK"));
      }
  }
  ```

- 将输出得到的`password`对应的密文替代数据库表中的密码字段，之后便可正常输入`pwzp`进行登录，幕后发生的事情是在`config`中添加了`SecurityConfig`字段之后在`spring-boot-starter-security`依赖中的登录界面里输入密码后，密码会自动经过编码再通过我们实现的`UserDetailImpl`和`UserDetailServiceImpl`两个类与数据库中信息进行匹配并返回登录结果，授予权限。

- 我们希望添加用户时，用户写入数据库的密码就是密文，使得即使数据库被破解，用户密码也不会遭到泄露，修改`UserController`类中的`addUser`函数为

  ```java
  @GetMapping("/user/add/{userId}/{username}/{password}/")
  public String addUser(@PathVariable int userId,
                        @PathVariable String username,
                        @PathVariable String password) {
  
      PasswordEncoder passwordEncoder = new BCryptPasswordEncoder();
      String encodedPassword = passwordEncoder.encode(password);
      User user = new User(userId, username, encodedPassword);
      userMapper.insert(user);
      return "Add User Successfully";
  }
  ```

### 将Session控制登录方式改为Token控制

