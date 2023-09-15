# JDK包中证书太少

## 错误类型405

## 查看根证书

进入目录：${JAVA_HOME}/jre/lib/security

> cacerts 文字路径 ${JAVA_HOME}/jre/lib/security/cacerts

执行命令：keytool -list -v -keystore cacerts > ./output.txt

> keytool  在${JAVA_HOME}/jre/bin

查找根证书名字，如若：digicertglobalrootg2 

默认密码：changeit

/export/servers/jdk1.8.0_60/jre/bin/keytool -list -v -keystore cacerts > ./output.txt

## 处理方式

直接本地找一个更多包含正式的 cacerts文件，替换原本jdk版本的文件，并重启服务

