# awk

过滤进程



### 自定义分隔符

```sh
awk -F: '{print $1}' /etc/passwd		#以冒号作为分隔符
awk -F"[:,_]" '{print $1}' /etc/passwd	#使用集合定义分隔符
```

### 调用系统变量、自定义变量

```sh
awk -v shell=$SHELL '{print shell}' /tmp/hosts	或者
awk '{print "'$SHELL'"}' /tmp/hosts		#双引号加单引号组合能正确获取系统变量

#自定义变量
awk -v x="bob" -v y=10 '{print x,y}' /tmp/hosts
```



1. **打印文件内容**：
   ```sh
   awk '{print}' filename
   ```

2. **打印文件的某一列**：
   ```sh
   awk '{print $1}' filename
   ```
   这里 `$1` 表示第一列，`$2` 表示第二列，以此类推。

3. **使用分隔符分割列**：
   ```sh
   awk -F, '{print $1}' filename
   ```
   `-F` 选项后面跟分隔符，上面的例子中使用逗号作为分隔符。

4. **模式匹配**：
   ```sh
   awk '/pattern/ {print $0}' filename
   ```
   只打印包含 `pattern` 的行。

5. **条件语句**：
   ```sh
   awk '$1 > 100 {print $0}' filename
   ```
   只打印第一列值大于 100 的行。

6. **内置变量**：
   - `NR`：表示当前处理的是第几行。
   - `NF`：表示当前行有多少字段。
   - `FS`：输入字段分隔符，默认是空格。
   - `OFS`：输出字段分隔符。
   - `RS`：输入记录分隔符，默认是换行符。
   - `ORS`：输出记录分隔符。

   例如，打印每行的字段数：
   ```sh
   awk '{print NF}' filename
   ```

7. **BEGIN 和 END 块**：
   `BEGIN` 在任何行被读取之前执行，`END` 在所有行被读取之后执行。
   ```sh
   awk 'BEGIN {print "Start"} {print $0} END {print "End"}' filename
   ```

8. **awk 脚本文件**：
   你可以将 `awk` 命令保存在一个文件中，然后执行该脚本文件。
   ```sh
   awk -f script.awk filename
   ```

`awk` 功能非常强大，以上只是一些基础用法。它还支持数组、函数和用户自定义变量等高级功能。
