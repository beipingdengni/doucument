



```sh
npm install -g typescript
# tsc -v
# tsc app.ts
# tsc -w #监控自动编译

# tsc --init # 初始化项目
npm install --save-dev typescript ts-loader webpack
```

### tsconfig.json 文件生成

> tsc --init # 初始化项目
>
> 运行：tsc  或  tsc -w   或     npm run tsc

```json
{
  "compilerOptions": { // 编译时自动生成单独的类型声明文件 xxx.d.ts
    "outDir": "./dist", // 编译后输出目录
    /* target 用于指定变异之后的版本目标，主要版本有：ES3、ES5、ES2015、ES2016等. */
    "target": "ES5"
    "baseUrl": "./", // 基础路径
    // 需要处理的路径
    "paths": {
            "@/*": ["src/*"],
            "skeleton": ["skeleton"],
     },
    "declaration": true, // tsc --declaration
    "lib": ["dom", "es2021"], //lib选项指定加载dom和es2021这两个内置类型声明文件。编译选项noLib会禁止加载任何内置声明文件
    // 如果项目中使用了外部的某个第三方代码库，就需要这个库的类型声明文件。
    // 一般来说，如果这个库的源码包含了[vendor].d.ts文件，那么就自带了类型声明文件。其中的vendor表示这个库的名字，比如moment这个库就自带moment.d.ts。使用这个库可能需要单独加载它的类型声明文件。
    // 社区库包含各个工程的类型定义：https://github.com/DefinitelyTyped/DefinitelyTyped
    // ts 会自动加载node_modules/@types目录下的模块，但可以使用编译选项typeRoots改变这种行为。
    // 示例表示，ts 不再去node_modules/@types目录，而是去跟当前tsconfig.json同级的typings和vendor/types子目录，加载类型模块了。
    "typeRoots": ["./typings", "./vendor/types"],
    // 默认情况下，ts会自动加载typeRoots目录里的所有模块，编译选项types可以指定加载哪些模块
    "types" : ["jquery"] // npm install @types/jquery --save-dev
    
  },
  "files": [ // 加入编译
    "src/index.ts",
    "typings/moment.d.ts"
  ],
//idea 自动保存，执行打包处理
//  "compileOnSave": true,  
//
  "includes": ["./src/**/*", "./skeleton/**/*"],
// 排除哪些文件不打包
  "exclude": ["./node_modules", "./src/*.html", "./dist", "./cert", "./tests"],
}
```

## 安装服务运行

```json
// npm i -g live-server -D

// package.json
"scripts": {
  "mb": "tsc -w".
  "start": "tsc -w & live-server", // 这个不显示html页面
  "server": "live-server ./ --port=8081"  // 这个可以，然后 哎配合 tsc -w
}

```

lodash 是一个一致性、模块化、高性能的 JavaScript 实用工具库

> 类似Java的guava工具库

```sh
npm install --save lodash
npm install --save-dev @types/lodash
```

## 打包(Browserify)

> Browserify是一个node.js模块，主要用于改写现有的CommonJS模块，使得浏览器端也可以使用这些模块。使用下面的命令，在全局环境下安装Browserify。

npm install -g browserify

```json
//npm install browserify --save-dev

//package.json:
{
  "scripts": {
     "browserify": "browserify ./build/index.js > ./dist/laker.js"
  }
}
```

## 压缩(terser)

> 提供了压缩 js 代码的方式，可以移除无用代码、替换变量名等，减少编译后文件体积，提升加载速度。

```json
//npm install terser --save-dev

//package.json:
{
  "scripts": {
     "mini": "terser ./dist/laker.js -o ./dist/laker.min.js -c arguments,arrows=true -m toplevel,keep_classnames,keep_fnames"
  }
}
```

## 执行脚步(shx)

> 一个包装 ShellJS Unix 命令的包装器，为 npm 包脚本中简单的**类 Unix** `跨平台命令`提供了一个简单的解决方案。

```json
//npm install shx --save-dev

//package.json:
{
  "scripts": {
    "clean": "shx rm -rf build dist && shx echo Done"
  }
}
```

