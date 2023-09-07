## Java 日志配置

**logback，log4j2是现代的高性能日志实现框架**

##### 基本知识：

```
日志门面 commons-logging，slf4j
日志实现 log4j，jdk-logging，logback，log4j2
=============================================================================

可以参考配置：
https://www.jianshu.com/p/7b5860be190f

=============================================================================

slf4j与log4j2集成
slf4j-api
log4j-api
log4j-core
log4j-slf4j-impl （用于log4j2与slf4j集成）
=============================================================================

slf4j与logback集成
slf4j-api
logback-core
logback-classic（已含有对slf4j的集成包）

处理可能造成依赖循环产生GC

第一种：jcl-over-slf4j 与 slf4j-jcl 冲突  如果这两者共存的话，必然造成相互委托，造成内存溢出
	jcl-over-slf4j： commons-logging切换到slf4j
	slf4j-jcl : slf4j切换到commons-logging
第二种：log4j-over-slf4j 与 slf4j-log4j12 冲突
	log4j-over-slf4j ： log4j1切换到slf4j
	slf4j-log4j12 : slf4j切换到log4j1
第三种：jul-to-slf4j 与 slf4j-jdk14 冲突
	jul-to-slf4j ： jdk-logging切换到slf4j
	slf4j-jdk14 : slf4j切换到jdk-logging

```

##### 参考稳定关于日志之前的依赖关系：

https://my.oschina.net/pingpangkuangmo/blog/410224

```
log4j1:
  log4j：log4j1的全部内容
log4j2:
  log4j-api:log4j2定义的API
  log4j-core:log4j2上述API的实现
logback:
  logback-core:logback的核心包
  logback-classic：logback实现了slf4j的API
commons-logging:
  commons-logging:commons-logging的原生全部内容
  log4j-jcl:commons-logging到log4j2的桥梁
  jcl-over-slf4j：commons-logging到slf4j的桥梁

日志桥接：
slf4j转向某个实际的日志框架
slf4j-jdk14：slf4j到jdk-logging的桥梁
slf4j-log4j12：slf4j到log4j1的桥梁
log4j-slf4j-impl：slf4j到log4j2的桥梁
logback-classic：slf4j到logback的桥梁
slf4j-jcl：slf4j到commons-logging的桥梁

某个实际的日志框架转向slf4j
jul-to-slf4j：jdk-logging到slf4j的桥梁
log4j-over-slf4j：log4j1到slf4j的桥梁
jcl-over-slf4j：commons-logging到slf4j的桥梁
=============================================================================
```



gradle配置参考：

```groovy
buildscript {
    // 定义全局变量
    ext {
        slf4j_version = '1.7.25'
        log4j2_version = '2.11.1'
        logback_version = '1.2.3'
      // logback_version = '1.2.3'
    }
}

// 全局排除依赖
configurations {
    // 支持通过group、module排除，可以同时使用
    all*.exclude group: 'commons-logging', module: 'commons-logging' // common-logging
    all*.exclude group: 'log4j', module: 'log4j' // log4j
    all*.exclude group: 'ch.qos.logback', module: 'logback-core' // logback
    all*.exclude group: 'ch.qos.logback', module: 'logback-classic' // slf4j -> logback
    all*.exclude group: 'org.slf4j', module: 'slf4j-jdk14' // slf4j -> jdk-logging
    all*.exclude group: 'org.slf4j', module: 'slf4j-jcl' // slf4j -> common-logging
    all*.exclude group: 'org.slf4j', module: 'slf4j-log4j12' // slf4j -> log4j
}

// 引入依赖
dependencies {
    // log
    compile "org.slf4j:slf4j-api:$slf4j_version"
    compile "org.slf4j:jul-to-slf4j:$slf4j_version"
    compile "org.slf4j:jcl-over-slf4j:$slf4j_version"
    compile "org.slf4j:log4j-over-slf4j:$slf4j_version"
  	// log4j2 配置
    compile "org.apache.logging.log4j:log4j-core:$log4j2_version"
    compile "org.apache.logging.log4j:log4j-slf4j-impl:$log4j2_version" //slf4j到log4j2的桥梁
  	// logback 配置
	  //compile "org.apache.logging.log4j:log4j-api:$log4j2_version"
  	//compile "org.apache.logging.log4j:log4j-to-slf4j:$log4j2_version"
    //compile "ch.qos.logback:logback-classic:$logback_version"  // slf4j到logback的桥梁
}

```

