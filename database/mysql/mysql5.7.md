



## 修改密码和授权

```sql
update user set authentication_string=password('123456') where user='root';
grant all privileges on *.* to 'root'@'%' identified by '123456' with grant option;
flush privileges;
```

