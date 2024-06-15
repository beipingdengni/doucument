

## 启用module

### 临时开启 Go modules 功能

export GO111MODULE=on

### 永久开启 Go modules 功能

go env -w GO111MODULE=on

### 设置 Go 的国内代理，方便下载第三方包

go env -w GOPROXY=https://goproxy.cn,direct

