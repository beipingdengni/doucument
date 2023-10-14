分组、人员

```shell
# 创建分组
groupadd elk

# 创建人员、增加密码
useradd admin
passwd admin

# 人员分配组  将admin用户添加到elk组
usermod -G elk admin

# 分配权限
# chown将指定文件的拥有者改为指定的用户或组 -R处理指定目录以及其子目录下的所有文件
chown -R admin:elk /usr/local
chown -R admin:elk /usr/upload

# 切换用户
su admin

# 切换root权限
sudo -s

```