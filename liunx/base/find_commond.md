# find命令中的参数以及作用

| 参数                                                         | 作用                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 根据文件名和inode查找                                        | -name "FILE_NAME"   文件名查找，支持使用glob：、？、[]、[^]<br/>-iname "FILE_NAME"  文件名查找，不区分大小写<br/>-inum n             按inode号查找<br/>-samefile name      查找与指定文件有相同inode号的文件，一般用于查找硬连接文件<br/>-links n            查找连接数为N的文件<br/>-regex "PATTERN"    以PATTERN匹配整个文件路径字符串，而不仅仅是文件名称<br/>使用:  find . -name "f" |
| 根据时间戳分为：<br/>访问时间(access time)<br/>修改时间(modify time)<br/>创建时间(create time)，<br/>分别简写为：<br/>atime/mtime/ctime | 根据时间戳查找：<br/>        以天为单位(time)：访问时间<br/>            -atime [+\|-]<br/>                +: 表示（#+1）天之外被访问过；<br/>                -: 表示#天之内被访问过；<br/>                无符号：表示短于（#+1）> x >=#天的时间段被访问过；<br/>            -mtime:修改时间<br/>            -ctime:创建时间<br/>         以分钟为单位(min)：<br/>            -amin [+\|-]:访问时间<br/>            -mmin:修改时间 <br/>           -cmin:创建时间<br/>-mtime -n +n：匹配修改内容的时间（-4指小于等于4天内的文件名；+4,大于等于5天前的文件名；4指前4~5那一天的文件） |
| 根据属主/属组查找                                            | -user username      查找属主为指定用户(UID)的文件<br/>-group groupname    查找属组为指定组(GID)的文件<br/>-uid UseerID        查找属主为指定的UID号的文件<br/>-gid GroupID        查找属组为指定的GID号的文件<br/>-nouser             查找没有属主的文件<br/>-nogroup            查找没有属组的文件 |
| 根据文件类型查找<br/>--type                                  | 匹配文件类型（后面的字母参数依次表示块设备、目录、字符设备、管道、链接文件、文本文件）<br/>f  普通文件<br/>d  目录文件<br/>l  符号链接文件 <br/>s  套接字文件<br/>b  块设备文件<br/>c  字符设备文件<br/>p  管道文件 |
| 根据文件大小查找<br/>find [path] -size [+\|-]N               | 其中N为文件大小，单位为c/k/M/G<br/>50k:   搜索49k~50k大小的文件   N-1~N<br/>+50k:  搜索大于50k的文件       N~······<br/>-50k:  搜索小于49k的文件       0~N-1<br/>find /app -size 2M |
| 指定搜索目录层级/深度<br/>-mindepth n<br/>-maxdepth n        | -maxdepth level 指定最大搜索目录深度level,指定的目录为第1级<br/>-mindepth level 指定最小搜索目录深度level。配合-maxdepth可搜索指定深度的文件。<br/>表示至多搜索到第 n-1 级子目录。0当前目录<br/>使用：find / -maxdepth 2 -name "*.conf"后面可跟用于进一步处理搜索结果的命令 -ok -print |
| 根据权限查找                                                 | -perm [/\|-] MODE<br/>    MODE：精确匹配三类对象(u,g,o)的权限<br/>    MODE：三类对象只要有一类对象中的三个权限位匹配一位即可，或关系(/444,三个只要有一 个“有”读权限(注意不是“是”读权限,))<br/>   -MODE：三类对象分别有对应权限，是否还有其他权限不在意，与关系(-444,三个都要“有”读权限，可以匹配到4,6,7；是否还有写或者执行权限不关心) <br/>  Notes：/或者-后面需要匹配的权限出现0(例如404)，,则表明不关心该对象权限(例如440,所属组有无权限都不关心)。如果单独的权限表示精确匹配，则表明该对象无任何权限 |
| 多条件查询                                                   | and    -a   与   #并且，也可以忽略<br/>or     -o   或   #<br/>not    !    非   #相反的条件<br/>find ./ -type f -o -type l  #查找普通文件和符号链接文件 |
| 处理动作                                                     | -print:默认的处理动作，显示至屏幕<br/>-ls   :类似于对查找到的文件执行“ls -l”命令<br/>-delete:删除查找到的文件 *慎用！*<br/>-fls file :查找到的所有文件的长格式信息保存至指定文件中，也可用重定向的方式<br/>-ok COMMAND {} \; :对查找到的每个文件执行由COMMAND指定的命令，对于每个文件执行命令之前，都会交互式要求用户确认<br/>-exec COMMAND {} \; :对查找到的每个文件执行由COMMAND指定的命令,没有-ok中的交互式确认。<br/>    其中{}用于引用查找到的文件名称自身，\;是配合-ok和-exec选项的<br/>find ./test1/ -type f -perm -001 |
| 获取某个时间段内的文件                                       | find . -type f -newermt '2022-08-17 00:00:00'  #找到比2022-08-17 0时0分0秒修改的文件<br/>find -newermt的真正形式是find -newer**XY** {variable}，旨在找到一些X属性比variable的Y属性更早的文件。 |

使用案例

```shell
# 查找压缩
find / -name *.jpg -type f -print | xargs tar -cvzf images.tar.gz

# 保留三天日志
find /export/Logs/* -name "*.log" -mtime +3 | xargs rm -rf

# 复制文件，文件到一个外部的硬盘驱动
ls *.jpg | xargs -n1 -i cp {} /external-hard-drive/directory

# find -newermt的真正形式是（find -newer**XY** {variable}），旨在找到一些X属性比variable的Y属性更早的文件
X指代find的目标文件属性；X可选a,c,m;Y可选a,c,m,t。acm意义分别为atime（访问时间），ctime(改变时间），mtime（修改时间）。
Y代表参照属性。t代表客观绝对时间，只作为参照属性存在，格式为yyyy-MM-dd hh:mm:ss
筛出mtime为今天的文件: find {path} -newermt `date +%F` -type f;
进行反选，在find中加入!: find . ! -newermt `date +%F` -type f;
find {path} ! -newermt `date +%F` -exec {order} {} \;  #最后一组{}表示exec之前筛到的文件; 
	# -newer f1 ! -newer f2:   匹配比文件f1新但比f2旧的文件 -a -o ! -not
```





`find` 命令在 Linux 中是一个非常强大的工具，用于在文件系统中搜索符合条件的文件。它可以根据文件名、大小、修改日期等多种条件来查找文件。下面是一些基本的使用方法和示例：

### 1. 基本语法

```bash
find [搜索路径] [选项] [动作]
```

- **搜索路径**：指定 `find` 命令开始搜索的目录路径。例如，使用 `.` 来表示当前目录，或者使用 `/` 来表示根目录。
- **选项**：指定搜索条件，比如按照文件名、文件类型、文件大小等来搜索文件。
- **动作**：对搜索结果进行操作，比如打印出文件名、删除文件等。如果不指定动作，默认动作是打印出找到的文件名。

### 2. 按名称搜索

```bash
find /path/to/search -name "filename"
```

- 例如，要在 `/home` 目录下搜索所有名为 `example.txt` 的文件，可以使用：
  ```bash
  find /home -name "example.txt"
  ```

### 3. 忽略大小写搜索

```bash
find /path/to/search -iname "filename"
```

- 例如，搜索 `/home` 目录下所有名字是 `example.txt`（不区分大小写）的文件：
  ```bash
  find /home -iname "example.txt"
  ```

### 4. 按类型搜索

```bash
find /path/to/search -type [f|d|l]
```

- 其中 `f` 表示普通文件，`d` 表示目录，`l` 表示符号链接。例如，搜索 `/home` 目录下所有的目录：
  ```bash
  find /home -type d
  ```

### 5. 按修改时间搜索

```bash
find /path/to/search -mtime +n -mtime -n
```

- `+n` 表示文件修改时间距现在 n 天之前，`-n` 表示文件修改时间距现在 n 天之内。例如，搜索 `/home` 目录下最近 7 天内修改过的文件：
  ```bash
  find /home -mtime -7
  ```

### 6. 按大小搜索

```bash
find /path/to/search -size [+|-]n[c|k|M|G]
```

- `n` 是大小，`c` 表示字节，`k` 表示千字节，`M` 表示兆字节，`G` 表示吉字节。`+n` 表示大于 n，`-n` 表示小于 n。例如，搜索 `/home` 目录下大于 50MB 的文件：
  ```bash
  find /home -size +50M
  ```

### 7. 结合使用 -exec 执行命令

```bash
find /path/to/search -type f -exec command {} \;
```

- 对找到的每个文件执行指定的命令。`{}` 是一个特殊字符串，对于每个匹配的文件，`{}` 会被替换成相应的文件名。例如，删除 `/home` 目录下所有 `.tmp` 文件：
  ```bash
  find /home -type f -name "*.tmp" -exec rm {} \;
  # 查找某时间段内
  find /path/to/search -type f -newer /tmp/start ! -newer /tmp/end -exec rm {} \;
  # 测试: 在使用-exec rm {} \;之前，你可以先用-exec ls {} \;来查看哪些文件会被选中，以避免误删除。
  ```

`find` 命令的功能非常强大，上面只是一些基本用法。通过阅读 `man find` 或者 `find --help` 可以获取更多高级用法和选项。
