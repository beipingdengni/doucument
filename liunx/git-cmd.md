

## 基础操作

```
ssh-keygen -t rsa -C "your_email@example.com"

输入生成路径，下生成文件名称：id_rsa_qq  默认都在~/.ssh/ 路径下

# 本地电脑产生多个key，用于推送
ssh-add ~/.ssh/id_rsa_qq

git remote set-url origin git地址
# git remote set-url --add origin it@github.com:beipingdengni/doucument.git
git remote add origin git地址
或者
git remote add 自定义远程名称 git地址

用法：git remote [-v | --verbose]
  或：git remote add [-t <分支>] [-m <master>] [-f] [--tags | --no-tags] [--mirror=<fetch|push>] <名称> <地址>
  或：git remote rename <旧名称> <新名称>
  或：git remote remove <名称>
  或：git remote set-head <名称> (-a | --auto | -d | --delete | <分支>)
  或：git remote [-v | --verbose] show [-n] <名称>
  或：git remote prune [-n | --dry-run] <名称>
  或：git remote [-v | --verbose] update [-p | --prune] [(<组> | <远程>)...]
  或：git remote set-branches [--add] <名称> <分支>...
  或：git remote get-url [--push] [--all] <名称>
  或：git remote set-url [--push] <名称> <新的地址> [<旧的地址>]
  或：git remote set-url --add <名称> <新的地址>
  或：git remote set-url --delete <名称> <地址>
```

## ssh生成key

```
ssh-keygen -t rsa -C '1271727449@qq.com' -f ~/.ssh/id_rsa_qq
```

>  添加环境ssh：ssh-add ~/.ssh/id_rsa_qq

```
ssh-add ~/.ssh/id_rsa_qq
# 跟踪master
git push --set-upstream origin master
```

### 同一代码，推送多个不同仓库，版本管理也不一样

```
# 添加
git remote add jingdai git地址
# 删除
git remoter rm jingdai
# 指定源推送
git push -u jingdai master
```

## 场景，同一代码推送多仓库

```
git remote add git地址   #默认推送到多个仓库 
```



![image-20230922175340924](/Users/tianbeiping1/Documents/study/doucument/liunx/imgs/git-cmd/image-20230922175340924.png)