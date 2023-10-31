## while

```sh
#while循环方式实现读写
while read line
do
  echo $line #将读到的行写入文件
done <${readfile} #从文件中按行读取

# 读取
while read line;do echo $line; done <$readfile
```

第二种

```sh
cat datafile.txt | while read myline
do
 echo "LINE:"$myline
done

cat datafile.txt| while read myline;do echo "$myline\n";done
```



## cat

```sh
#for循环方式实现读写
for line in $(cat $readfile)
do
  echo $line
done
# 读取
for line in $(cat test.txt);do echo $line; done;
```

