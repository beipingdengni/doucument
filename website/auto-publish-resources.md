## webpack 打包流程

1、我们打包的时候，会先合并 Webpack config 文件和命令行参数，合并为options。
2、将 options 传入Compiler构造方法，生成compiler 实例，并实例化了 Compiler上的 Hooks。
3、compiler 对象执行run方法，并自动触发beforeRun、run、beforeCompile、compile 等关键 Hooks。
4、调用 Compilation构造方法创建 compilation对象，compilation负责管理所有模块和对应的依赖，创建完成后触发 make Hook。
5、执行compilation.addEntry() 方法，addEntry 用于分析所有入口文件，逐级递归解析，调用 NormalModuleFactory 方法，为每个依赖生成一个 Module 实例，并在执行过程中触发 beforeResolve、resolver、afterResolve、module等关键 Hooks。
6、将第 5 步中生成的 Module 实例作为入参，执行 Compilation.addModule() 和 Compilation.buildModule() 方法递归创建模块对象和依赖模块对象。
7、调用seal方法生成代码，整理输出主文件和 chunk，并最终输出。



## 自定义webpack组件

官方文档： https://webpack.docschina.org/contribute/writing-a-plugin/

https://blog.csdn.net/qq_43568455/article/details/128648671

知乎参考：https://zhuanlan.zhihu.com/p/661670534



## webpack打包自动上传到指定目录

https://www.cnblogs.com/lijun12138/p/17502947.html



## webpack-minio-plugin

https://www.npmjs.com/package/webpack-minio-plugin?activeTab=code



## webpack-ossplus-plugin

https://github.com/lianhr12/webpack-ossplus-plugin



## webpack-alioss-plugin

https://github.com/borenXue/webpack-alioss-plugin/tree/master



## ssh-webpack-plugin

https://github.com/luchanan/ssh-webpack-plugin/tree/master



## ssh2-sftp-client





```js
const JSZip = require('jszip');
const { RawSource } = require('webpack-sources');
/* 
  将本次打包的资源都打包成为一个压缩包
  需求:获取所有打包后的资源
*/
const pluginName = 'CompressAssetsPlugin';

class CompressAssetsPlugin {
  constructor({ output }) {
    this.output = output;
  }

  apply(compiler) {
    // AsyncSeriesHook 将 assets 输出到 output 目录之前调用该钩子
    compiler.hooks.emit.tapAsync(pluginName, (compilation, callback) => {
      // 创建zip对象
      const zip = new JSZip();
      // 获取本次打包生成所有的assets资源
      const assets = compilation.getAssets();
      // 循环每一个资源
      assets.forEach(({ name, source }) => {
        // 调用source()方法获得对应的源代码 这是一个源代码的字符串
        const sourceCode = source.source();
        // 往 zip 对象中添加资源名称和源代码内容
        zip.file(name, sourceCode);
      });
      // 调用 zip.generateAsync 生成 zip 压缩包
      zip.generateAsync({ type: 'nodebuffer' }).then((result) => {
        // 通过 new RawSource 创建压缩包
        // 并且同时通过 compilation.emitAsset 方法将生成的 Zip 压缩包输出到 this.output
        compilation.emitAsset(this.output, new RawSource(result));
        // 调用 callback 表示本次事件函数结束
        callback();
      });
    });
  }
}

module.exports = CompressAssetsPlugin;
```

