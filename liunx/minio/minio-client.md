





## 选择客户端下载

> mac  arm芯片选择版本：https://dl.min.io/client/mc/release/darwin-arm64/

其他版本下载选择：https://dl.min.io/client/mc/release/

### 配置服务地址

```shell
# demo-test 别名 服务地址
mc config host add demo-test http://minio-test.local.com:9000 用户 密码
```

### 本地复制到远程

```shell
# 本地文件夹存储路径   别名/桶名称/存储层级定义
mc cp --recursive ./vue3/dist-v1/ demo-test/test/数据存储
#或使用
mc cp -r ./vue3/dist-v1/ demo-test/test/数据存储

# 案例
# 将文件v1.14.1，上传到my-crm-web目录下,  在远程my-crm-web目录下多一个v1.14.1
mc cp -r ./v1.14.1 demo-test/mybucket/vue/my-crm-web/
```

### 远程下载到本地

```sh
mc cp --recursive demo-test/documents/2014/  ./Backups/2014
```

### 查看、删除（相对比较重要使用）

```sh
ls          list buckets and objects
mb          make a bucket
rb          remove a bucket
cat         display object contents
mv          move objects
rm          remove objects
undo        undo PUT/DELETE operations
cp          copy objects
du          summarize disk usage recursively
find        search for objects
```

博客参考
https://zhuanlan.zhihu.com/p/558896919

