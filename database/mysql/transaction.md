

## mysql 隔离级别



隔离级别

> Read Uncommitted（读取未提交内容）
> Read Committed（读取提交内容）
> Repeatable Read（可重读）
> Serializable（可串行化）

查看mysql 版本

>  select version();

查看隔离级别

> 查看系统隔离级别：select @@global.tx_isolation;
> 查看会话隔离级别(5.0以上版本)：select @@tx_isolation;
> 查看会话隔离级别(8.0以上版本)：select @@transaction_isolation;

修改隔离级别

> set session transaction isolation level repeatable read; 设置会话隔离级别为可重复读
> set session transaction isolation level read uncommitted; 设置会话隔离级别为读未提交
> set session transaction isolation level read committed; 设置会话隔离级别为读已提交

事物开启、提交、回滚

```sql
start transaction;  --开启事物
	-- select * from a for update
	-- insert / update / delete 
commit; -- 提交
rollback;  -- 回滚
```



