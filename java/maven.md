# maven管理

## 发布配置

```xml
<distributionManagement>
    <repository>
        <id>central</id>
        <name>JD maven2 repository-releases</name>
        <url>部署的服务地址URL -- 地址 </url>
    </repository>
</distributionManagement>
```

## 强制更新包

```shell
mvn clean install -U
```

## 清理包依赖

```shell
#项目全部清除 该命令可以清除本地仓库中所有的缓存，包括依赖库和插件等
mvn dependency:purge-local-repository

#命令行删除本地依赖； 如果只需要清除指定的依赖库缓存，将com.example:example-artifact替换为需要清除的依赖库的坐标
mvn dependency:purge-local-repository -DreResolve=false -DmanualInclude=groupId:artifactId
#或
mvn dependency:purge-local-repository -DgroupId=com.example -DartifactId=myapp -Dversion=1.0.0
```

## 项目配置拉取最新包

```xml
<repositories>
    <repository>
        <id>central</id>
        <name>libs-releases</name>
        <url>maven仓库中心URL-发布地址</url>
        <snapshots>
            <enabled>false</enabled>
        </snapshots>
    </repository>
    <repository>
        <id>snapshots</id>
        <name>libs-snapshots</name>
        <url>maven仓库中心URL-快照地址</url>
        <snapshots>
            <enabled>true</enabled>
            <updatePolicy>always</updatePolicy>
        </snapshots>
    </repository>
</repositories>
```

# 基础操作





## PMD、checkstyle



### 忽略检查

```shell
mvn clean package \
-Dpmd.skip=true\
-Dcheckstyle.skip=true\
-Dmaven.test.skip=true\
-U -f pom.xml 
-s ~/.m2/settings.xml
```

