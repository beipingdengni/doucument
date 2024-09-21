## ItChat

https://github.com/littlecodersh/ItChat



> pip3 install itchat-uos==1.5.0.dev0
>
> 安装以上才正常运行
>
> https://pypi.org/project/itchat-uos/1.5.0.dev0/
>
> https://github.com/why2lyj/ItChat-UOS



java实现版本：https://github.com/yaphone/itchat4j、

python扩展版本使用：https://github.com/youfou/wxpy（类似的基于Python的微信机器人）





# wxauto

https://github.com/cluic/wxauto

> Windows版本微信客户端（非网页版）自动化，可实现简单的发送、接收微信消息，简单微信机器人

```shell
python3 -m pip uninstall wxauto
python3 -m pip install wxauto -i https://pypi.tuna.tsinghua.edu.cn/simple 
查看wxauto是否安装成功
python3 -c "import wxauto; print(wxauto.VERSION)"
```

代码演示

```python
from wxauto import *

# 获取当前微信客户端
wx = WeChat()

# 获取会话列表
wx.GetSessionList()

# 向某人发送消息（以`文件传输助手`为例）
msg = '你好~'
who = '文件传输助手'
wx.SendMsg(msg, who)  # 向`文件传输助手`发送消息：你好~


# 向某人发送文件（以`文件传输助手`为例，发送三个不同类型文件）
files = [
    'D:/test/wxauto.py',
    'D:/test/pic.png',
    'D:/test/files.rar'
]
who = '文件传输助手'
wx.SendFiles(filepath=files, who=who)  # 向`文件传输助手`发送上述三个文件

# 下载当前聊天窗口的聊天记录及图片
msgs = wx.GetAllMessage(savepic=True)   # 获取聊天记录，及自动下载图片
```

