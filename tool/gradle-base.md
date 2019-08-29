## Gradle 基本



参考博客：http://www.zhaiqianfeng.com/2017/03/love-of-gradle-dependencies-1.html

```groovy
// 看包依赖情况，除了使用gradle dependencies外，还可以自定义任务
task listJars {
    doLast {
        configurations.compile.each { File file -> println file.name }
    }
}
//=============================================================================

configurations.all {
  
  // 有效期10分钟
    resolutionStrategy.cacheDynamicVersionsFor 10, 'minutes'
  // 有效期4小时
  //resolutionStrategy.cacheChangingModulesFor 4, 'hours'

	// 两种解决版本冲突的策略:Newest和Fail.默认策略是Newest，配置Fail模式
	resolutionStrategy.failOnVersionConflict()
	
　// 如果有冲突,强制依赖asm-all的3.31版本和commons-io的1.4版本
  resolutionStrategy.force 'asm:asm-all:3.3.1', 'commons-io:commons-io:1.4'
}
//=============================================================================

dependencies {

  // 使用本地jar
	runtime files('libs/a.jar', 'libs/b.jar')
  runtime fileTree(dir: 'libs', include: '*.jar')

  compile('org.hibernate:hibernate:3.1') {
    //如果有冲突，强制使用3.1版本
    force = true

    //排除传递中的依赖
    exclude module: 'cglib' //通过artifact的名字排除
    exclude group: 'org.jmock' //通过artifact的group名字排除
    exclude group: 'org.unwanted', module: 'iAmBuggy' //通过artifact的名字和grop名字排除

    //禁止该依赖传递
    transitive = false
  }
  
  runtime group: 'org.hibernate', name: 'hibernate', version: '3.0.5', transitive: true
}
//=============================================================================

//拷贝compile的所有依赖到指定目录allLibs
task copyAllDependencies(type: Copy) {
  from configurations.compile
  into 'allLibs'
}
//=============================================================================

// 编译打包所以依赖，到jar包 // 把依赖的包打包进可执行jar
jar {
 	from { configurations.compile.collect { it.isDirectory() ? it : zipTree(it) }} 
	manifest {
		attributes 'Main-Class':'App'
	}
}
//=============================================================================

// 如我们经常会把源码一起发布，可以这样来做
task sourcesJar(type: Jar, dependsOn: classes) {
    classifier = 'sources'
    from sourceSets.main.allSource
}
 
artifacts {
    archives sourcesJar
}
//=============================================================================

// Ivy和Maven都支持如下协议和认证
repositories {
    maven【ivy】 {
        url "sftp://repo.mycompany.com:22/maven2"
        credentials {
            username 'user'
            password 'password'
        }
    		//credentials(AwsCredentials) {
           // accessKey "someKey"
           // secretKey "someSecret"
           // // 可选
           // sessionToken "someSTSToken"
       // }
    }
}


```

