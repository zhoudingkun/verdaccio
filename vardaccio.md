# 私有包服务verdaccion的搭建与踩坑

### 背景
1. 公司内部使用
2. 组件代码通过类似 npm 的命令行的方式进行复用
3. 下载速度更快

## 快速操作指南（下文可以不用看了，有想详细了解verdaccio的请楼下看）
内部私有仓库地址为 http://static-repo.wanshifu.com:8007

1,先把npm地址指向私有仓库
 npm set registry http://static-repo.wanshifu.com:8007
2, npm install [对应的包名]    例如：npm install @wsf/test,在对应页面 import '@wsf/test';
当所下载的包在私有仓库找不到时，就会自动在外网里找
3,上传包到私有仓库
    在登录账号后，在根目录下，即上传
npm publish --registry http://static-repo.wanshifu.com:8007





=========================================================================
=================================楼下=====================================





### verdaccion介绍
    verdaccio 是一个轻量级的私有npm代理注册。（sinopia fork）
    sinopia现在早已不更新，verdaccio是在它基础上维护升级

### verdaccion使用
1.安装 （建议两个都跑一遍）<br />
    npm：npm install -g verdaccio <br />
    yarn：yarn global add verdaccio
    
2.启动服务
    第一种启动方式：verdaccio >> verdaccio.log 2>&1 &     后台启动并写入日志
        verdaccio --listen 4000 --config ./config.yaml    指定配置启动（安装完verdaccio会自动出现config.yaml，记得相对路径要保持正确）
    第二种启动方式：verdaccio

### 使用pm2启动verdaccio，保证该进程一直处于打开状态
1. 安装pm2      npm install -g pm2 --unsafe-perm
2. 使用pm2启动verdaccio   pm2 start verdaccio
3. 查看pm2 守护下的进程verdaccio的实时日志   pm2 show verdaccio 



### 添加用户/注册登录

    npm adduser --registry  http://static-repo.wanshifu.com:8007，
    按照提示输入自己 name、email、password

## 上传规范：
1. 执行时可以先在根目录下 npm init --scope==wsf,
    相当于npm初始化时添加了一个作用域，<br />
    package name使用@wsf + / + 组件名，例如 @wsf/test <br />
    description需要简要描述组件的功能，例如 页面轮播封装 <br />
    entry point:(入口文件)默认 <br />
    author:(作者)方便到时联系咨询 <br />
2. 编写md文件，便于其他人的使用

### 上传私有包
    npm publish --registry http://static-repo.wanshifu.com:8007
    ps：这一步需要在自己想要上传的包的主目录下执行，同时对于有内部安装依赖的包 
    例如react包，我们需要将yarn/npm install以后的包再上传，
    而不是直接把githunb上的包拉下来去上传,
    这是因为直接拉下来的源文件并没有解析去执行postinstall，直接上传会报错，没有内部依赖的可以直接源文件上传的。

### 本地配置注册地址
    npm config list -l 查看默认配置
    
#### 将默认地址 https://registry.npmjs.org/ 改成私有地址
    npm set registry http://static-repo.wanshifu.com:8007

    如果您使用HTTPS，请添加适当的CA信息
    “null”表示从操作系统获取CA列表）
    npm set ca null
    
### verdaccio配置文件config.yaml详解

    所有包的缓存目录
    storage: ./storage
    插件目录
    plugins: ./plugins

--------------------------------------------
    开启web 服务,能够通过web 访问
    web:
    WebUI is enabled as default, if you want disable it, just uncomment this line
    enable: false
    title: Verdaccio
--------------------------------------------
    验证信息
    auth:
    htpasswd:
        #  用户信息存储目录
        file: ./htpasswd
        # Maximum amount of users allowed to register, defaults to "+inf".
        # You can set this to -1 to disable registration.
        #max_users: 1000

    a list of other known repositories we can talk to
---------------------------------------------
    公有仓库配置
    uplinks:
    npmjs:
        url: https://registry.npmjs.org/
-------------------------------------------
    packages:
    '@*/*':
        # scoped packages
        access: $all
        publish: $authenticated

        #代理 表示没有的仓库会去这个npmjs 里面去找 ,
        #npmjs 又指向  https://registry.npmjs.org/ ,就是上面的 uplinks 配置
        proxy: npmjs
-------------------------------------------
    '**':
        # 三种身份,所有人,匿名用户,认证(登陆)用户
        # "$all", "$anonymous", "$authenticated"

        是否可访问所需要的权限
        access: $all

        发布package 的权限
        publish: $authenticated

        如果package 不存在,就向代理的上游服务发起请求
        proxy: npmjs

-------------------------------------------
    监听的端口 ,重点, 不配置这个,只能本机能访问
    listen: 0.0.0.0:4873


