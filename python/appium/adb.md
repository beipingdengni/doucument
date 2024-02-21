# adb

## 常用指令

```shell
# 连接设备
adb connect localhost:5000
adb connect 192.168.3.142:5555 # 192.168.3.142是自己手机连接WIFI后的ip
adb kill-server # 关闭服务
adb start-server # 重启服务

# 查看设备
adb device
# 安装
adb install app-release.apk 
# 设备：可以是IP 或者 设置名称
adb install -s 指定设备 -r app.apk

adb shell pm list packages -3 #(===>查看外部安装包)
#adb shell dumpsys package com.jingdong.app.mall

# 查看当前正在运行的Activity
adb logcat | grep Displayed
```



## 安装如下

> brew install android-sdk
>
> ```shell
> echo 'export ANDROID_HOME=/usr/local/share/android-sdk' >> ~/.zshrc
> echo 'export PATH=$PATH:$ANDROID_HOME/tools' >> ~/.zshrc
> echo 'export PATH=$PATH:$ANDROID_HOME/platform-tools' >> ~/.zshrc
> source ~/.zshrc
> ```
>
> 手动下载安装：https://developer.android.com/tools/releases/platform-tools?hl=zh-cn
>
> 配置： ~/.zshrc

# 基础使用

```shell
# 连接设备
adb connect localhost:5000
adb connect 192.168.3.142:5555 # 192.168.3.142是自己手机连接WIFI后的ip
adb kill-server # 关闭服务
adb start-server # 重启服务

# 查看设备
adb device
# 安装
adb install app-release.apk 
# 设备：可以是IP 或者 设置名称
adb install -s 指定设备 -r app.apk

# 查看设备的 cpu 和 内存占用情况
adb shell top
# 查看占用内存前 N 的app应用（N 代表数字）
adb shell top -m N
# 查看进程列表
adb shell ps
# 查看所有的包名
adb shell pm list packages
adb shell pm list packages -3 #(===>查看外部安装包)
# 查看某个包的具体信息
adb shell dumpsys package XXX
	#adb shell dumpsys package com.jingdong.app.mall
# 查看当前resume的是哪个activity   
adb shell dumpsys activity | grep mFocusedActivity
# 查看当前正在运行的Activity
adb logcat | grep ActivityManager
# 查看当前正在运行的Activity
adb logcat | grep Displayed

# 将本机中的文件写入到设备
	# 在 Terminal 中输入下命令，其实这一步和git中的 push 有些相似之处
	adb push <本机文件路径/文件名> <设备路径>
# 将设备中的文件读取到本机中
	# 在 Terminal 中输入以下命令，其实这一步也和git中的 pull 比较相似
	adb pull <设备中文件路径/文件名> <本机路径>

# adb shell am monitor（只有在启动或退出的时候才会打印）
# 查询本机所有软件包  adb shell pm list packages
# 6.查询本机所有软件包  adb shell pm list packages
# 7.输出和安装包相关联的文件 adb shell pm list packages -f
# 8.输出本机禁用的包 adb shell pm list packages -d
# 9.输出本机启用的包 adb shell pm list packages -e
# 10.打印输出系统包名 adb shell pm list packages -s
# 11.打印输出第三方安装包 adb shell pm list packages -3    (===> 查看外部安装包)
# 12.输出包和安装信息（安装来源） adb shll pm list packages -i
# 13.输出包和为安装包信息（安装包来源） adb shell pm list packages -u
# 14.通过adb打开应用 adb shell am start -n com.android.settings/com.android.settings.Settings
# 15.卸载系统应用 adb shell rm system/priv-app/WBZ/WBZ.apk
# 16.无root权限进行卸载系统应用：1.通过adb shell进入shell模式；2.输入pm uninstall -k --user 0 应用包名进行系统软件卸载。
# 17. 查看设备cpu类型：adb shell getprop ro.product.cpu.abi
# 18.ping命令 adb shell ping -c 4 www.baidu.com，ping4次后停止ping

# 模拟器
emulator -avd test1
# -s 可以在多个模拟器中指定一个进行操作
# -r 覆盖安装
adb -s emulator-5554 install -r ./JioChat_Poc_V1.0.7.1026_1912.apk
adb -s emulator-5554 uninstall com.jiochat.jiochatapp
```

## 搜索京东

```shell
adb shell monkey -v -v -v -p com.jingdong.app.mall 10
```