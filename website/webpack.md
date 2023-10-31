## 安装webpack、webpack-cli、webpack-dev-server

```shell
npm i webpack-cli -g

npm i --save-dev clean-webpack-plugin common-config-webpack-plugin html-webpack-plugin webpack webpack-cli webpack-dev-server
或
yarn add  --dev  clean-webpack-plugin common-config-webpack-plugin html-webpack-plugin webpack webpack-cli webpack-dev-server

yarn add -D webpack-dev-server   或 pnpm add -D webpack-dev-server

#本地安装依赖   @3.6.0 指定版本安装
npm install --save-dev webpack@3.6.0

#版本查看
webpack –v
```

## webpack基础配置

### 组合安装

```sh
const webpack = require('webpack');

const configuration = require('./webpack.config.js');
let compiler = webpack(configuration);
new webpack.ProgressPlugin().apply(compiler);
compiler.run(function (err, stats) {
  // ...
});
```

###  webpack.config.js 文件

```js

const path = require('path');
const webpack = require('webpack');

const CleanWebpackPlugin = require('clean-webpack-plugin').CleanWebpackPlugin;
const CommonConfigWebpackPlugin = require('common-config-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
    mode: 'development', // development production
    // entry: './src/main.js',
  	// entry: path.join(__dirname,'src/index.js'),
    entry: {
        main: './src/main.js',
        app: './src/app.js',
    },
    output: {
        // path: path.resolve(__dirname, 'dist', '[fullhash]'),
      	// path.join(__dirname,'dist')
        path: path.resolve(__dirname, 'dist'), 
        filename: '[name].[contenthash].bundle.js',
    },
    module: {
        rules: [
            { test: /\.css$/, use: 'css-loader' },
            { test: /\.ts$/, use: 'ts-loader' },
            {
                test: /\.(png|jpg|gif|jpeg)$/,
                use: [{
                    loader: 'url-loader',
                    options: {
                        limit: 8000,
                        name: 'img/[name].[hash:8].[ext]'
                    }
                }]
            }
        ]
    },
    plugins: [
      new CleanWebpackPlugin(),
      new CommonConfigWebpackPlugin(),
        new webpack.ProgressPlugin(),
        new HtmlWebpackPlugin({ template: './index.html' })
    ],
};
```

## npm 安装依赖(package.js)

```json
{
  "scripts": {
    "build": "webpack"
  },
  "devDependencies": {
    "css-loader": "^6.8.1",
    "html-webpack-plugin": "^5.5.3",
    "ts-loader": "^9.5.0",
    "url-loader": "^4.1.1",
    "webpack-dev-server": "^4.15.1"
  }
}
```

### 无配置使用webpack

https://github.com/merkle-open/webpack-config-plugins/tree/master

在线快速生成webpack模版：https://webpack-config-plugins.js.org/

#### webpack 无配置js运行

```
npm i --save-dev webpack webpack-cli common-config-webpack-plugin

// 直接使用此，指令就能允许webpack 打包
npx webpack --plugin common-config-webpack-plugin
```

#### webpack-dev-server 无配置js允许

```
npm i --save-dev webpack common-config-webpack-plugin html-webpack-plugin

webpack-dev-server --plugin common-config-webpack-plugin --plugin html-webpack-plugin
```

#### common-config-webpack-plugin依赖展示

```
common-config-webpack-plugin
  ├── js-config-webpack-plugin
  ├── ts-config-webpack-plugin
  ├── scss-config-webpack-plugin
  └── asset-config-webpack-plugin
      ├── font-config-webpack-plugin
      └── image-config-webpack-plugin

// 全部配置
const JsConfigWebpackPlugin = require('js-config-webpack-plugin');
const TsConfigWebpackPlugin = require('ts-config-webpack-plugin');
const ScssConfigWebpackPlugin = require('scss-config-webpack-plugin');
const FontConfigWebpackPlugin = require('font-config-webpack-plugin');
const ImageConfigWebpackPlugin = require('image-config-webpack-plugin');

module.exports = {
  plugins: [
    new JsConfigWebpackPlugin(),
    new TsConfigWebpackPlugin(),
    new ScssConfigWebpackPlugin(),
    new FontConfigWebpackPlugin(),
    new ImageConfigWebpackPlugin(),
  ],
};
```



## 依赖插件

### new HtmlWebpackPlugin

```js
{
  plugins: [
        new HtmlWebpackPlugin({ template: './index.html' }),
		    new HtmlWebpackPlugin({ template: './app.html' })
    ]
}
// 可以配置多个模版，每个模版使用不同
title
生成的html文档的标题。配置该项，它并不会替换指定模板文件中的title元素的内容，除非html模板文件中使用了模板引擎语法来获取该配置项值，如下ejs模板语法形式：
原文有误，修改为：<title><%= htmlWebpackPlugin.options.title %></title>

filename
输出文件的文件名称，默认为index.html，不配置就是该文件名；此外，还可以为输出文件指定目录位置（例如'html/index.html'）
**关于filename补充两点：**
1、filename配置的html文件目录是相对于webpackConfig.output.path路径而言的，不是相对于当前项目目录结构的。
2、指定生成的html文件内容中的link和script路径是相对于生成目录下的，写路径的时候请写生成目录下的相对路径。

template
本地模板文件的位置，支持加载器(如handlebars、ejs、undersore、html等)，如比如 handlebars!src/index.hbs； 
**关于template补充几点：**     // path.join(__dirname, 'default_index.html'), //模版文件
1、template配置项在html文件使用file-loader时，其所指定的位置找不到，导致生成的html文件内容不是期望的内容。
2、为template指定的模板文件没有指定任何loader的话，默认使用ejs-loader。如 template: './index.html'，若没有为.html指定任何loader就使用ejs-loader

templateContent
string | function，可以指定模板的内容，不能与template共存。配置值为function时，可以直接返回html字符串，也可以异步调用返回html字符串。

inject
向template或者templateContent中注入所有静态资源，不同的配置值注入的位置不经相同
1、true或者body：所有JavaScript资源插入到body元素的底部
2、head: 所有JavaScript资源插入到head元素中
3、false： 所有静态资源css和JavaScript都不会注入到模板文件中

favicon
添加特定favicon路径到输出的html文档中，这个同title配置项，需要在模板中动态获取其路径值

hash
true|false，是否为所有注入的静态资源添加webpack每次编译产生的唯一hash值，添加hash形式如下所示：
html <script type="text/javascript" src="common.js?a3e1396b501cdd9041be"></script>

chunks //[早期版本]
// {"publicPath":"","js":["main.c152138b98f22cb0fa5e.bundle.js","app.aaa3ebf0a9a2fbc398ea.bundle.js"],"css":[]}
 <h1><%= JSON.stringify(htmlWebpackPlugin.files)  %></h1> 
允许插入到模板中的一些chunk，不配置此项默认会将entry中所有的thunk注入到模板中。在配置多个页面时，每个页面注入的thunk应该是不相同的，需要通过该配置为不同页面注入不同的thunk；

excludeChunks
这个与chunks配置项正好相反，用来配置不允许注入的thunk。

chunksSortMode
none | auto| function，默认auto； 允许指定的thunk在插入到html文档前进行排序。
值可以指定具体排序规则；auto基于thunk的id进行排序； none就是不排序

xhtml
true|fasle, 默认false；是否渲染link为自闭合的标签，true则为自闭合标签

cache
true|fasle, 默认true； 如果为true表示在对应的thunk文件修改后就会emit文件

showErrors
true|false，默认true；是否将错误信息输出到html页面中。这个很有用，在生成html文件的过程中有错误信息，输出到页面就能看到错误相关信息便于调试。

minify
{....}|false；传递 html-minifier 选项给 minify 输出，false就是不使用html压缩，minify具体配置参数请点击html-minifier
```

#### 模版

```html
<!DOCTYPE html>
<html style="font-size:20px">
<head>
    <meta charset="utf-8">
    <title><%= htmlWebpackPlugin.options.title %></title>
    <% for (var css in htmlWebpackPlugin.files.css) { %>
    <link href="<%=htmlWebpackPlugin.files.css[css] %>" rel="stylesheet">
    <% } %>
</head>
<body>
<div id="app"></div>
<% for (var chunk in htmlWebpackPlugin.files.js) { %>
<script type="text/javascript" src="<%=htmlWebpackPlugin.files.js[chunk] %>"></script>
<% } %>
</body>
</html>

// 自定义输出
files 下，默认属性
  publicPath: string;
	js: string[];
	css: string[];
	manifest?: string;
	favicon?: string; 

"htmlWebpackPlugin": {
  "files": {
    "css": [ "inex.css" ],
    "js": [ "common.js", "index.js"],
    "chunks": {
      "common": {
        "entry": "common.js",
        "css": [ "index.css" ]
      },
      "index": {
        "entry": "index.js",
        "css": ["index.css"]
      }
    }
  }
}
<% for (var chunk in htmlWebpackPlugin.files.chunks) { %>
<script type="text/javascript" src="<%=htmlWebpackPlugin.files.chunks[chunk].entry %>"></script>
<% } %>

```

### new CleanWebpackPlugin

> 启动清理文件

### new CommonConfigWebpackPlugin

> 



## webpack基本使用

运行 webpack 并且监听文件变化

```shell
npx webpack init ./my-app --force --template=default

# 编译项目
npx webpack build --config ./webpack.config.js --stats verbose

npx webpack watch --mode development   

npx webpack info --output markdown

环境参数
npx webpack --node-env production   # process.env.NODE_ENV = 'production'
npx webpack --env prod   #{ prod: true }
npx webpack --env platform=app --env production  #{ platform: "app", production: true }
npx webpack --env app.platform="staging" --env app.name="test"  #{ app: { platform: "staging", name: "test" }

npx webpack --progress #查看 webpack 的编译进度
npx webpack --analyze #可以使用 webpack-bundle-analyzer 插件来分析你 webpack 输出的 bundle
```

