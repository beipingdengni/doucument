

# docker 操作

查找所有容器有连接mysql

```shell
docker ps |grep 'yanxi'|awk '{print $1}' > ~/docker_id.txt;\
while read docker_id;do echo $docker_id; \
result=$(docker exec $docker_id bash -c 'if [ -d /export/Instances ] ;then ls /export/Instances/; netstat -nltpa |grep "3306"|grep -v grep;fi');\
echo -e "$result\n";\
done <~/docker_id.txt;\
rm -rf ~/docker_id.txt

```

# redis 操作

脚本执行

```shell
#!/bin/sh
host=127.0.01
port=6387
pwd=pwd
redis_exec_path=/export/Instances/redis/src/
$redis_exec_path/redis-cli -c -h $host -p $port -a $pwd keys "mykey:*" > keys.txt
more keys.txt | grep -v ^$ | while read mykey
do
  result=`$redis_exec_path/redis-cli -h $host -p $port -c -a $pwd ttl $mykey` 
  if [ $result -eq -1 ]
    then
      echo $mykey
  fi
done
```

直接在交互界面上操作

```shell
scan_key='mykey*';\
redis_path=/export/Instances/redis/src/redis-cli;\
redis_host=127.0.0.1;
pwd=pwd;\
$redis_path -p 6379 -h $redis_host -a $pwd \
keys $scan_key > ~/keys.txt && \
while read mykey;\
do result=$($redis_path -p 6379 -h $redis_host -a $pwd ttl $mykey);\
if [ $result -eq -1 ];then \
echo $mykey $result;\
fi;\
done < ~/keys.txt

```

