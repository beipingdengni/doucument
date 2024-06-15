

## 编译不同环境

```sh
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build main.go
CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build main.go

// main.go 为源文件； 默认：go build main.go 
```



##### 详解mysql中concat,concat_ws,concat_group函数
https://blog.csdn.net/weixin_44080445/article/details/118032842

##### GO 字符串
https://blog.csdn.net/Pola_/article/details/123833876

##### GO 参考使用
string转成int：
    int, err := strconv.Atoi(string)
string转成int64：
    int64, err := strconv.ParseInt(string, 10, 64)
int转成string：
    string := strconv.Itoa(int)
int64转成string：
    string := strconv.FormatInt(int64,10)

##### GO REDIS 博客
https://www.cnblogs.com/HJZ114152/p/17325053.html
https://blog.csdn.net/m1215339620/article/details/131568640
