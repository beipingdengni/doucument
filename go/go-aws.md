



## go aws

### 安装

```bash
go get -U github.com/aws/aws-sdk-go
```

### 代码

```go
package main

import (
	"fmt"
	"os"
	"time"

	"github.com/aws/aws-sdk-go/aws"
	"github.com/aws/aws-sdk-go/aws/credentials"
	"github.com/aws/aws-sdk-go/aws/session"
	"github.com/aws/aws-sdk-go/service/s3"
)

const bucket string = "csevent"
const host string = "https://demo.com/csevents3oss"

func main() {
  // 数据的文件
	filename := os.Args[1]
	// 初始化配置
	var client *s3.S3 = initCfg()

  // 执行上传
	putOss(filename, client)

  // 检查文件是否存在
	//checkOss(filename, client)
}

// 初始化配置
func initCfg() *s3.S3 {
	ak := "ak 值"
	sk := "sk 值"
	creds := credentials.NewStaticCredentials(ak, sk, "")
	_, err := creds.Get() // 验证是否正确
	// 检查错误
	if err != nil {
		panic(err)
	}
  // 指定服务器
	config := &aws.Config{
		Region:      aws.String("cn-north-1"),
		Endpoint:    aws.String("s3.aws.com"),
		DisableSSL:  aws.Bool(true),
		Credentials: creds,
	}
	sess, err := session.NewSession(config) // 创建会话
	if err != nil {
		panic(err)
	}
	return s3.New(sess)
}
// 上传文件
func putOss(filename string, client *s3.S3) {
	// 获取文件
	file, err := os.Open(filename)
	if err != nil {
		exitErrorf("Unable to open file %q, %v", filename, err)
	}
	defer file.Close() // 执行完成方法关闭
	state, _ := file.Stat() // 查询文件状态
  nowStr := time.Now().Format("2006-01-02 15:04:05") // 获取当前时间,类似java： yyyy-MM-dd HH:mm:ss
	key := fmt.Sprintf("/%s/%s-%s", bucket, nowStr, state.Name()) //组装key
	fmt.Println("上传文件路径：" + key)
	fmt.Printf("访问地址：%s/%s\r\n", host, key)
	// /csevent/20231213182127-文章.mp4
	putObjectInput := &s3.PutObjectInput{
		Bucket: aws.String(bucket),
		Key:    aws.String(key),
		Body:   file,
		// ACL: （默认可以不带任何验证，直接访问： https://crm.jdazcn.com/csevents3oss/csevent/20231213182127-京东保险客服中心质检规则V2.0.mp4）
		//　指定ACL权限。一般上传图片类的都是期望可以在前端通过url获取资源的，为public-read。具体可以参考 https://docs.aws.amazon.com/zh_cn/AmazonS3/latest/userguide/acl-overview.html#CannedACL
		// ACL:    aws.String("public-read"),
	}
	response, err := client.PutObject(putObjectInput) // 执行上传
	if err != nil {
		exitErrorf("上传异常", err.Error())
	}
	fmt.Printf("上传成功%v \n", response) // 返回结果
}

// 检查oss是否存在key的文件
func checkOss(key string, client *s3.S3) {
	headObject := &s3.HeadObjectInput{
		Bucket: aws.String(bucket),
		Key:    aws.String(key),
	}
	response, err := client.HeadObject(headObject)
	if err != nil {
		exitErrorf("文件检查异常", err.Error())
	}
	fmt.Printf("文件存在 %v \r\n ", response)
}

// 异常检查
func exitErrorf(msg string, args ...interface{}) {
	fmt.Fprintf(os.Stderr, msg+"\n", args...)
	os.Exit(1)
}
```

