

# 在线聊天指令

# 上行

> TRACE_ID 

心跳消息监听、监听client连接断连、监听client连接

> CMD_UP 上行到im的CMD指令

| cmd                        | 指令含义                                                     | 对应redis缓存key            |
| -------------------------- | ------------------------------------------------------------ | --------------------------- |
| READ                       | 消息回去回执【已读】<br />* 1. 修改消息状态为已读 <br />* 2. 删除信箱中的消息<br />* 3. 下行消息到关注消息已读状态的终端 | CHAT_MSG_BOX                |
| REVOKE                     | 撤回消息【撤回】                                             | CHAT_MSG_BOX                |
| GET_READ_MSG               | 用户在当前会话需要获取当前会话消息<br />MSG_DOWN 会下行消息  | CHAT_MSG_BOX                |
| REST_READ_NUM_AND_LAST_MSG | 已读消息最新的标记，存入缓存                                 | RESET_READ_NUM_AND_LAST_MSG |
| GET_SYSTEM_TIME            | 获取后端系统时间                                             |                             |
| IRIS_CMD_SEND              |                                                              |                             |
| CMD_COMPLETE               | CMD处理完成，回执                                            | CMD_BOX                     |
| NOTICE_COMPLETE            | NOTICE处理完成，回执                                         | NOTICE_BOX                  |

> MSG_UP上行到im的消息

> NOTICE_UP 通知上行 【暂未使用】

# 下行

> MSG_DOWN、NOTICE_DOWN、CMD_DOWN、ACK_DOWN、CMD_DOWN

| 指令        | 指令含义                                     | 缓存key                                                     |
| ----------- | -------------------------------------------- | ----------------------------------------------------------- |
| MSG_DOWN    | 消息下行                                     | CHAT_MSG_BOX                                                |
| NOTICE_DOWN | 通知下行<br />消息内容下行<br />系统踢人下行 | NOTICE_BOX<br />CHAT_MSG_BOX<br />SYSTEM_INSTRUCT_USER_KICK |
| CMD_DOWN    | 指令下行                                     | CMD_BOX                                                     |
| ACK_DOWN    | 服务收到消息回执                             |                                                             |

