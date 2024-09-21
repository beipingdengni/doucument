

# TAG 相关命令

## 创建标签

创建两种类型的标签：轻量级标签和附注标签。

轻量级标签：git tag <tagname>

创建本地tag: git tag -a <tag_name> <commit_id>

推送tag到远程: git push --tags origin <remote_branch_name>

附注标签：git tag -a <tagname> -m "your message"

## 查看标签

查看本地所有标签：git tag

查看远程所有标签：git ls-remote --tags

查看指定tag的详细信息：git show <tag>

## 推送标签

推送单个标签：git push origin <tagname>

推送所有标签：git push origin --tags

## 删除标签

删除本地标签：git tag -d <tagname>

删除远程标签：git push origin :refs/tags/<tagname>

## 检出标签

检出标签到工作区：git checkout <tagname>

注意：这样会使得HEAD处于一个无分支的状态，可以通过创建一个新的分支来处理这种状态： git checkout -b <branchname> <tagname>

## 签出指定标签的代码

git clone -b <tagname> <repository>

例如： git clone -b v1.0 https://github.com/user/repo.git

