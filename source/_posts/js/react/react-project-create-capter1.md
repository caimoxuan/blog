---
title: react入门-创建项目
date: 2019-12-12 00:02:01
thumbnail: 
categories:
    - js
tags:
    - react
---

## 从创建react项目开始

### 准备环境

首先我们需要了解创建项目前我们需要做哪些准备
1. 准备node环境
    我们首先需要安装node，因为使用node的npm命令能帮助我们快速管理js的相关依赖，可以理解成java中的maven；node的版本不能太低(js在兼容性方面确实做的不好，但是现在有了typescript不知道之后会不会有所改变)；
    安装node的教程有很多，如果安装完成在任意目录中执行npm报错，可能是没有配置环境变量(有的安装会自动配置)；

<!-- more -->
2. 通过npm安装create-react-app来帮助我们创建项目
    接下来使用npm来安装我们需要的东西（觉得速度慢可以自行设置淘宝镜像，或者是安装yarn代替npm）：
    ``` js
        // 使用-g 参数全局安装 这样任意目录下都能执行命令（可以自行配置`-g`安装的目录，方便查找）
        npm install -g create-react-app

        // 命令执行完成后我们能用这个创建我们的react项目（创建出来的项目是最基础的版本）执行以下任一一条即可
        npx create-react-app project_name / create-react-app project_name
    ```
    上面的步骤可能失败：可以尝试换成淘宝镜像重试
    如果安装的create-react-app的时间比较早，可能需要安装最新版本（重新install可以更新版本）

3. 查看创建的项目
    上面的命令会帮我们在命令的执行路径创建一个 `project_name` 的文件夹进入文件夹可以看见他的目录    
    ``` js
    ├── package.json            # 配置文件，管理启动命令、依赖配置、环境配置等
    ├── public                  # 这个是webpack的配置的静态目录
    │   ├── favicon.ico         # 网站的小图标
    │   ├── index.html          # 页面入口
    │   └── manifest.json       # 站点的一些配置，类似网页名称等
    ├── src
    │   ├── App.css             # 组件的css
    │   ├── App.js              # 组件逻辑从这里开始编辑
    │   ├── App.test.js
    │   ├── index.css           # 全局的css文件
    │   ├── index.js            # 全局的js文件，页面js入口，将react挂载到dom上一般没必要修改
    │   ├── logo.svg
    │   └── serviceWorker.js
    └── yarn.lock               # 可以使用yarn的命令代替npm
    ```
4. 先启动查看效果
    如果命令执行完成，没有在目录中看到`node_modules`这个目录，说明依赖还没有安装，需要先执行
    ``` js
    // 命令会安装所有在package.json中声明的所有依赖 （同样会比较慢， 可以用镜像或yarn）
    npm install  
    
    // 完成之后就可以执行
    npm start
    ```
    启动之后会使用默认浏览器访问 http://localhost:3000 （默认情况）

### 配置项目

完成以上步骤，react项目算是创建完成了，但是这是个基础的模板，对于简单开效果来说已经够了；这个时候可以随意修改 App.js中的代码，然后保存查看效果（npm start之后保存修改会热更新）；有些依赖是比较常见的，基本都会安装；
1. 如果项目不是单页项目，就会用到路由
   ``` js
    // 安装react路由来管理页面 --save 代表将module添加到package.json中，方便下次install自动安装
    npm install react-router --save
   ```
   关于路由的使用，简单几句话难以概括；
2. 如果不是什么定制项目的话，需要自己写一些组件来完成工作，那么可以参考使用ant design；
   ``` js
    // 安装ant design
    npm install antd --save
   ```
   antd帮助我们实现了开发中的大多数的组件，如果不满足需求可以再封装，对于ERP（后台管理系统）推荐使用 `ant design pro`；
3. 那么如果我们要请求后台接口，再jquery中封装了ajax，再react中同样可以使用，但是不推荐这种方式，因为react的理念时数据驱动页面，操作dom破坏了这种理念；所以我们可以安装axios来发送ajax请求；
   ``` js
   npm install axios --save
   ```
   这是一个全面的ajax封装，如果不满意，它提供了拦截器，前置拦截请求、后置拦截响应，异常处理等，可以自行配置；

有了以上的配置，要开发一个小系统就绰绰有余了；


### 优化模板

到这里开发来说是够了，但是如果个人有一些喜好啥的还是不够的，比如喜欢用less、sass代替css；喜欢用typescript等；我们就需要配置达到自己喜欢方式；每个人有不同的喜好，我们可以配置来制作一个自己喜欢的模板；
1. 使用typescript开发
``` js
// 开始创建项目就使用typescript
create-react-app project_name --typescript 

// 但是项目一开始创建没有选择typescript，也可以改成ts项目 稍微麻烦一点(不过一定要早点转，因为有的依赖在之后安装会安装ts版本的，很多依赖都有ts版本)
npm install --save typescript @types/node @types/react @types/react-dom @types/jest
```
2. 使用sass
react-app默认是支持sass的，只是少一个依赖不用多余配置
``` js
npm install --save node-sass
```

3. 其他

还有一些其他的配置，如less，这个比较麻烦一点；
配置图片的解析格式；
配置静态文件的引用路径（默认是绝对路径，这个很坑，打包完部署路径会有影响）；
配置环境变量并使用，这个在多环境（比如要调用的接口分不同环境url不同，可以使用环境变量打包的时候可以区分）；
...
还有其他扩展自己慢慢探索吧；