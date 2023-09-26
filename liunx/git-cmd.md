

## 基础操作

ssh生成key

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