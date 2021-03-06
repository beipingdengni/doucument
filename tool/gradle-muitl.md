Gradle 使用

父模块设置

```groovy
// 所以模块使用
allproject{
  
}
// 所以子模块使用
subprojects {
  // 使用插件
  apply plugin: 'java'
  apply plugin: 'maven'
  apply plugin: "idea"
 // 设置编译使用编码
  sourceCompatibility = 1.8
  targetCompatibility = 1.8
  // 使用UTF-8
  tasks.withType(JavaCompile) {
    options.encoding = 'UTF-8'
  }
  configurations.all {
    // [设置本地缓存的更新策略 seconds 、 minutes 、hours ]
    resolutionStrategy.cacheChangingModulesFor 10, 'seconds'
  }
  // 组织名称 与 版本号
  group 'com.tian'
  version '1.1.0-SNAPSHOT'
  // 设置的环境参数
  ext {
    compileJava.options.encoding = 'UTF-8'
    compileTestJava.options.encoding = 'UTF-8'
    springCloudVersion = 'Greenwich.SR2'
    // true jar上传使用RELEASE ，false 使用 SNAPSHOT
    mavenUpload = false
    maven_version = System.getProperty("maven_version") ?: (mavenUpload ? "${project.version}".replaceAll('SNAPSHOT', 'RELEASE') : "${project.version}")
    maven_username = System.getProperty("maven_username") ? System.getProperty("maven_username") : "admin"
    maven_password = System.getProperty("maven_password") ? System.getProperty("maven_version") : "123456"
  }
  
  repositories {
    mavenLocal()
    maven { url 'https://maven.aliyun.com/repository/central' }
    //http://maven.aliyun.com/nexus/content/groups/public/
    mavenCentral()
  } 
}

```

子模块使用

```groovy
// 上传jar 配置
uploadArchives {
  repositories {
    mavenDeployer {
      pom.version = maven_version
      if (mavenUpload) {
        pom.version = "${project.version}".replaceAll('SNAPSHOT', 'RELEASE')
        repository(url: 'http://localhost:8081/repository/maven-releases/') {
          authentication(userName: maven_username, password: maven_password)
        }
      } else {
        snapshotRepository(url: 'http://localhost:8081/repository/maven-snapshots/') {
          authentication(userName: maven_username, password: maven_password)
        }
      }
    }
  }
}
/**
* uploadArchives 下repository(url:''){
*	 authentication(userName: maven_username, password: maven_password)
*  snapshots(enabled:!mavenUpload,updatePolicy:'always',checksumPolicy:'warn')
* }
*/
```

##### 配置文件外引用

```
// 1 所有模块配置
allprojects {
    apply from: "${rootProject.projectDir}/common.gradle"
}

// 2、修改 web 模块，web/build.gradle 增加配置：
apply from: "${rootProject.projectDir}/common.gradle"
// ===============================================================================
// ScalaLanguageSourceSet [scala 语言]
sourceSets {
   main {
      java {
         srcDir 'java-sources'
      }
      resources {
         srcDir 'resources'
      }
   }
}
// ===============================================================================
//获取 common.gradle 依赖插件配置
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'net.researchgate:gradle-release:2.4.0' // release plugin
    }
}
```

##### 扩展功能

```
gradle release // 执行所有模块
gradle :release // 只执行父模块
gradle web:release // 只执行 web 子模块
gradle web:release api:release //只执行 web、api子模块
// ===============================================================================
//插件使用
apply plugin: 'application'

group = "com.gradle.api"
version = "${rootProject.version}" //也可以不写，默认继承父模块 version
description = "this is gradle api demo"
archivesBaseName = 'gradle_api'

mainClassName = "com.gradle.api.Main" //指定 Jar 启动 Main
// ===============================================================================
clean 本地包
    task deleteDescriptors(type: Exec) { //执行shell命令
        executable "sh"
        args "-c", "rm -rf ~/.gradle/caches/modules-2/metadata-	2.16/descriptors/com.company.appname" 
        //此处的“com.company.appname“就是之前定义的“repositoryGroup“。
    }

    task clean(type: Delete, dependsOn: deleteDescriptors) { //clean工程时顺带执行上述任务
        delete rootProject.buildDir
    }
// ===============================================================================
gradle-release 插件     文档：https://www.ctolib.com/mip/gradle-release.html

plugins {
    id 'net.researchgate.release' version '2.6.0'
}

// config release task
release {
    tagCommitMessage = "[Gradle Release Plugin] - creating tag: "
    scmAdapters = [
            net.researchgate.release.GitAdapter,
    ]
}
// ===============================================================================
// pom 设置基本数据
 pom.project {
   name 'Example Application'
   packaging 'jar'
   // optionally artifactId can be defined here 
   description 'A application used as an example on how to set up 
   pushing its components to the Central Repository . '
   url 'http://www.example.com/example-application'

   scm {
     connection 'scm:git:git@github.com:username/project.git'
     developerConnection 'scm:git:git@github.com:username/project.git'
     url 'https://github.com/username/project'
   }

   licenses {
     license {
       name 'The Apache License, Version 2.0'
       url 'http://www.apache.org/licenses/LICENSE-2.0.txt'
     }
   }

   developers {
     developer {
       id 'manfred'
       name 'Manfred Moser'
       email 'manfred@sonatype.com'
     }
   }
 }
```

