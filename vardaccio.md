# 私有包服务verdaccion的搭建与踩坑

### 背景
1. 公司内部使用
2. 组件代码通过类似 npm 的命令行的方式进行复用
3. 下载速度更快

## 对于不需要发布，只用下载的用户，只需执行以下操作（下文可以不用看了，有上传需要的请楼下雅间坐）
内部私有仓库地址为 http://192.168.2.101:4873
1. npm set registry http://192.168.2.101:4873
2. yarn add [对应的包名]    
3. 下载外网包可以再切换为 npm set registry https://registry.npmjs.org/


=========================================================================






### verdaccion介绍
    verdaccio 是一个轻量级的私有npm代理注册。（sinopia fork）
    sinopia现在早已不更新，verdaccio是在它基础上维护升级

### verdaccion使用
1.安装 （建议两个都跑一遍）<br />
    npm：npm install -g verdaccio <br />
    yarn：yarn global add verdaccio
    
2.启动服务
    verdaccio >> verdaccio.log 2>&1 &     后台启动并写入日志
    verdaccio --listen 4000 --config ./config.yaml    指定配置启动（安装完verdaccio会自动出现config.yaml，记得相对路径要保持正确）

### 添加用户/注册登录

    npm adduser --registry  http://192.168.2.101:4873，
    按照提示输入自己 name、email、password

### 上传私有包
    npm publish --registry http://192.168.2.101:4873
    ps：这一步需要在自己想要上传的包的主目录下执行，同时对于有内部安装依赖的包 例如react包，我们需要将yarn/npm install以后的包再上传，而不是直接把githunb上的包拉下来去上传，这是因为直接拉下来的源文件并没有解析去执行postinstall，直接上传会报错，没有内部依赖的可以直接源文件上传的。执行时可以先在根目录下 npm init --scope==wsf,相当于npm初始化时添加了一个作用域

### 本地配置注册地址
    npm config list -l 查看默认配置
    
#### 将默认地址 https://registry.npmjs.org/ 改成私有地址
    npm set registry http://192.168.2.101:4873

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

