







sql优化

创建索引

Mysql5.6  提升 mysql8.0  【有一个并行】

超过5秒数据sql，拿出来制定时间到制定到数据执行

adb 数据计算有问题，rds同步数据过去存在问题

查询使用polardb只读链接

测试：分区表---不是分库分表

表建议在【200G---300G】

单库理论60T --- > 已经存在顾客使用：30T

polardb 保存高可用 -- 也有 主从 机器  【主从都有延迟 -- 毫秒级别】，12月上旬 全局session延迟，但能使用从库

开启会话查询一致性，session 事务拆分，查询sql使用【从库】，更新添加删除使用事物【主库】

创建索引过程中，未提交事物，导致锁表，CPU 变高

polardb 现在已经开启binlog

【500G】超过两台rds从库，polardb使用跟便宜

规划未来：8.0 使用全局索引【分区表】



测试数据库性能

主库sql转发从库

tcp抓包处理、应用服务器处理



drds 使用 java代理分布式【产品解决代理，向polardb更加融合，多个polardb集群】

DTS--->DRDS -- > 存在问题【迁移、同步】



polardb事物隔离级别：【一致性要求】
事物一致性

会话一致性

全局会话一致性