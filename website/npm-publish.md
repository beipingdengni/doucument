

## 发布公共仓库

```shell
# 查看当前 npm 源配置
npm config ls
# 查看当前 npm 全部源配置
npm config ls -l
# 修改 npm 源地址为官方源
npm config set registry https://registry.npmjs.org/
# 将 npm 源地址修改为淘宝源（用于安装 npm 速度慢的时候使用）
npm config set registry  https://registry.npm.taobao.org/
```

> `npm config ls -l` 命令查看 `metrics-registry = "https://registry.npmjs.org/"` 是否为官方源，如果不是则使用上面命令设置为官方源

### 进行登录、发布包

```shell
npm login

npm logout

# 普通包
npm publish

# 私域包
# 包名：@username/packageName
# username 必须为当前登录的用户名，必须一致，否则会报错，然后加上参数 --access public
npm publish --access public
```

### package.json 参考创建

```json
{
  // 发布的包名，默认是上级文件夹名。不得与现在npm中的包名重复。包名不能有大写字母/空格/下滑线!
  "name": "#####",  //@aliyun/mysql //anliyun是私有仓库名称
  // 版本号，每次要更新
  "version": "1.0.0",
  // 包的描述
  "description": "仅供测试，别下载",
  // 文件入口，默认是 index.js，可修改
  "main": "index.js",
  "scripts": {
    // 测试命令，可以不填直接回车
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  // 作者名称
  "author": "###",
  // 包遵循的开源协议，默认是ISC
  "license": "ISC",
  // 因为组件包是公用的，所以 private 为 false
  "private": false,
  // 当前包需要依赖的第三方组件，如何安装使用依赖包，可以看看文章顶部的NPM命令介绍文章
  "dependencies": {},
  // "devDependencies": {}
  // 指定代码所在的仓库地址
  "repository": {
    "type": "git",
    "url": "https://github.com/beipigndenigni/self-custome-componet.git"
  },
  // bug在哪里提
  "bugs": {
     "url": "https://github.com/beipigndenigni/self-custome-componet/issues"
  },
  // 项目官网的地址
  "homepage": "https://github.com/beipigndenigni/self-custome-componet",
  // 指定打包后,包中存在的文件夹
  "files": [
    "dist",
    "src"
  ],
  // 指定了项目的目标浏览器的范围
  "browserslist": [
    "> 1%",
    "last 2 versions",
    "not ie <= 8"
  ],
  // 项目关键词，供搜索
  "keywords": [
    "测试"
  ]
}
```



## 私有化

> 参考文档：https://www.zhihu.com/question/564766133/answer/2745540660

项目中创建 `.npmrc` 文件

```shell
# 配置数据源
registry=https://registry.npm.taobao.org
# 私有化数据源
@aliyun:registry=https://registry.npm.taobao.org

# 下载私有仓库mysql依赖包
npm install @aliyun/mysql
```

### 在私有项目package.json中配置包信息

```sh
如果从0开发的项目可以直接npm init

如果项目有package.json可直接手动修改

注意：私有包名称必须是"@{组织名}/{npm-name}"
@jd/self-custom-component

例如：@jd/self-custom-component
```

### 发布、删除私有包

```shell
npm publish

npm unpublish 包名 --force
```

### 部署代理组件服务(verdaccio)

```sh
# 全局安装
npm install -g verdaccio
# 启动服务器
verdaccio

# 通过命令行启动的话，如果终端停止了，那我们的服务器也就停止了，因此一般我们通过pm2启动守护进程。
npm install -g pm2
pm2 start verdaccio
pm2 list

#docker来进行安装
docker run --name verdaccio -itd -v ~/docker/verdaccio:/verdaccio -p 4873:4873 verdaccio/verdaccio


nrm管理镜像源地址
# 设置npm使用的源为本地私服
npm set registry http://localhost:4873/
# 试用，安装
npm install lodash --registry http://localhost:4873

# 本地别名源管理
nrm add localnpm http://localhost:4873/
nrm add taobao http://localhost:4873/

```

### 项目增加配置切换数据源

> npm、yarn 切换回淘宝镜像，项目根目录下添加.npmrc、.yarnrc文件添加以下配置支持npm和yarn安装私有包。

```shell
# .npmrc 文件
# 指向内网私源 
@{组织名}:registry=http://ip:4873
 
# .yarnrc 文件
# 指向内网私源
"@{组织名}:registry" "http://ip:4873"
#如
"@jd:registry" "http://ip:4873"


# .yarnrc 文件 ，参考配置
# 其中registry “https://registry.npm.taobao.org“就是指定淘宝镜像源，是最重要的。其余是指定对应包的下载路径
registry "https://registry.npm.taobao.org"

sass_binary_site "https://npm.taobao.org/mirrors/node-sass/"
phantomjs_cdnurl "http://cnpmjs.org/downloads"
electron_mirror "https://npm.taobao.org/mirrors/electron/"
sqlite3_binary_host_mirror "https://foxgis.oss-cn-shanghai.aliyuncs.com/"
profiler_binary_host_mirror "https://npm.taobao.org/mirrors/node-inspector/"
chromedriver_cdnurl "https://cdn.npm.taobao.org/dist/chromedriver"
```

