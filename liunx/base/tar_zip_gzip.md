## tar

#### 压缩、解压

```shell
tar -zxvf elasticsearch-6.2.3.tar.gz -C /usr/local

--delete : 从tar包中删除某个文件
-r, --append : 将文件追加到tar归档文件中
-t, --list : 列出tar归档文件中包含的文件或目录
-u, --update : 将已更新的文件追加到tar归档文件中
-x, --extract, --get : 释放tar归档文件中文件及目录
-C, --directory=DIR : 执行归档动作前变更工作目录到 目标DIR
-f, --file=ARCHIVE : 指定 (将要创建或已存在的) 归档文件名
-j, --bip2 : 对归档文件使用 bzip2 压缩
-J, --xz : 对归档文件使用 xz 压缩
-p, --preserve-permissions : 保留原文件的访问权限
-v, --verbose : 显示命令整个执行过程
-z, gzip : 对归档文件使用 gzip 压缩

tar -zcvf /home/www/images.tar.gz /home/www/images

本地复制到远程：scp -r 目录名 root@主机地址:目标路径。
远程复制到本地：scp -r root@主机地址:目录名 目标路径。

-P或--absolute-names：文件名使用绝对名称，不移除文件名称前的 “/” 号
	注意：网上有些文档是 将 -P 参数加在 f 参数后面，那么这样是会报错的

tar -zcvPf /home/tianbeiping1/code.tar.gz /home/tianbeiping1/code
```

## gzip(选项)(参数)

```
压缩： gzip -r *
解压： gzip -drv *
显示列出压缩文件：gzip -l *


-a或——ascii：使用ASCII文字模式；
-d或--decompress或----uncompress：解开压缩文件；
-f或——force：强行压缩文件。不理会文件名称或硬连接是否存在以及该文件是否为符号连接；
-h或——help：在线帮助；
-l或——list：列出压缩文件的相关信息；
-L或——license：显示版本与版权信息；
-n或--no-name：压缩文件时，不保存原来的文件名称及时间戳记；
-N或——name：压缩文件时，保存原来的文件名称及时间戳记；
-q或——quiet：不显示警告信息；
-r或——recursive：递归处理，将指定目录下的所有文件及子目录一并处理；
-S或&lt;压缩字尾字符串&gt;或----suffix&lt;压缩字尾字符串&gt;：更改压缩字尾字符串；
-t或——test：测试压缩文件是否正确无误；
-v或——verbose：显示指令执行过程；
-V或——version：显示版本信息；
-&lt;压缩效率&gt;：压缩效率是一个介于1~9的数值，预设值为“6”，指定愈大的数值，压缩效率就会愈高；
--best：此参数的效果和指定“-9”参数相同；
--fast：此参数的效果和指定“-1”参数相同。
-num 用指定的数字num调整压缩的速度，-1或--fast表示最快压缩方法（低压缩比），-9或--best表示最慢压缩方法（高压缩比）。系统缺省值为6。
```

## zip

### zip [选项] <压缩文件名> <文件/目录列表>

> zip -r 压缩包名 待压缩的文件和目录列表

```
选项	作用
-r	递归地将一个目录及其所有子目录和文件压缩到ZIP文件中
-q	在压缩文件时启用静默模式，即不显示压缩过程的详细信息
-d	从现有的ZIP文件中删除指定的文件或目录
-u	用于更新现有的ZIP文件，将新的文件或修改后的文件添加到ZIP存档中
-f	用于刷新（更新）现有ZIP文件中的指定文件。
-m	用于移动（归档）文件到一个ZIP压缩文件中，并在移动后将源文件删除。
-e	用于对ZIP压缩文件进行加密。
-z	为压缩文件添加注释
```

#### unzip [选项] 压缩文件.zip

```
-d 目录：指定解压目录。
-q：静默模式，不显示解压进度信息。
-o：覆盖已存在的文件。
-j：只解压文件，不保留原始目录结构。
```

