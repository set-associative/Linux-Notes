## Linux Administration with sed and awk

https://www.bilibili.com/video/BV1Lp411o7qh?p=4&vd_source=7fa1062756a4a6344a4a995006b2193e

### Introduction to sed

* `sed -n ' 1, 5 p' /etc/passwd` 只打印一到五行的内容

* `sed -n ' /^michael/ p ' /etc/passwd` 打印以michael开头的行

* `sed ' /^"/ d' ~/.vimrc` 打印删除.vimrc中的所有注释后的文件内容

* `sed ' /^#/ d ; /^$/ d' /etc/ntp.conf` 将删除注释和空行后的内容打印出来

* `sed -i.bak ' /^#/ d; /^$/ d' /etc/netp.conf` 对文件内容进行更改，并备份

### Introduction to awk

* `awk ' {print $0 } ' /etc/passwd` 相当于 `cat /etc/passwd`, 此处的 `$0` 可省略

* `awk ' BEGIN {print "users"} {print }  END { print "Total Users", NR}' /etc/passwd` 以 `BEGIN`或 `END`开头的命令仅仅执行一次， 而`main block`会针对每行执行一次； `NR`变量即 number of records

* `awk -F":" ' {print $1} ' /etc/passwd`展示所有的用户名，`-F` 用于指定`field seperators` (FS) 

### Introduction to grep

* grep 即 Global Regular Expression Print

* `cat /proc/cpuinfo  | grep -c name ` 查找name在/proc/cpuinfo文件中出现的次数; 该命令也可写为 `grep -c name /proc/cpuinfo`

### Parse CSV File

parsecsv.sh

```bash
#!/bin/bash

# IFS represents input field seperators
OLDIFS="$IFS"; IFS=","
while read product price quantity
do
  echo -e "\e[1;33m$product \
  =====================\e[0m\n\
  Price : \t $price \n\
  Quantity : \t $quantity \n"
done < $1

IFS=OLDIFS
```

tools

```textile
drill,99,5
hammer,5,100
brush,5,100
lamp,25,30
screwdriver,5,23
table-saw,1099,3
```

```bash
$ bash parsecsv.sh tools
drill   =====================
  Price :      99 
  Quantity :      5 

hammer   =====================
  Price :      5 
  Quantity :      100 

brush   =====================
  Price :      5 
  Quantity :      100 

lamp   =====================
  Price :      25 
  Quantity :      30 

screwdriver   =====================
  Price :      5 
  Quantity :      23 

table-saw   =====================
  Price :      1099 
  Quantity :      3 
```

使用`-A` 可以指定匹配行之后的需要展示的行; `-B 和 -C`也是差不多的用法

```bash
$ bash parsecsv.sh tools | grep -A2 hammer 
hammer   =====================
  Price :      5 
  Quantity :      100 
```

### Mastering Regular Expression

所有内容都可以`man grep`查询 -- REGULAR EXPRESSIONS部分

* Anchors -- `^`, `$`

* Ranges -- `[]`, `[^]`

* Boundaries -- `\b`, `\B`

* Quantifiers -- `(){}`, `?`, `+`, `*`

* grep命令带上`-v` option可用于反向匹配

* grep命令带上`-E`option可使用`extended regular expression`

* grep 命令带上`-i`option可进行忽略大小写地匹配

### Print with sed

* `sed ' p ' /etc/passwd`  --  不带上`-n` option的话，鸡肋

* `sed -n ' p ' /etc/passwd` -- 标准用法

* `sed -n ' 1,3 p' /etc/passwd` 打印一到三行

* `sed -n ' /^root/ p ' /etc/passwd` 打印匹配pattern的行

### Substitute with sed

* `sed ' [range] s/<string>/<replacement>/g ' /etc/passwd`

* `sed ' /^gretchen s@/bin/bash/@/bin/sh@g' /etc/passwd`  此处仍然可以用forward slash作为substitute命令的分隔符，但是后面的path需要使用backslash转义

### Append \ Insert \ Delete with sed

* `sed ' /^server 3/ a server ntp.example.com' /etc/ntp.conf`

* `sed ' /^server 0/ i a server ntp.example.com' /etc/ntp.conf``

* `sed ' /^server\s[0-9\.ubuntu/ d ' /etc/net.conf`

* 请注意：Delete命令会将一整行都删掉，如果想要删除一行中的部分内容，请用substitute命令（将向删除的部分用空白替换）

### Multiple statements sed files with sed

sed命令可以带上多个expressions，从而使得整条命令过于冗长，为了避免这种情况可以使用`-f`来指定sed files，将冗长的sed expressions写道sed files中;

例如，以下两种写法完全一样

```bash
$ sed '{
> /^server 0/ i ntp.example.com
> /^server\s[0-9]\.ubuntu/ d
> } ' /etc/net.conf
```

```bash
$ cat ntp.sed
/^server 0/ i ntp.example.com
/^server\s[0-9]\.ubuntu/ d
$ sed -f ntp.sed /etc/ntp.conf
```

### In-place edits with sed

`sed -i.bak`, 此处`.bak`被用于命名备份文件

### Substitution Groups in sed

* 在sed中的substitute命令中，可以用`\(\)` 来定义`Groups`

* `sed 's@\([^,]*\)@\U\1@' <fifle>`

* `sed 's@\([^,]*\),\([^,]*\)@@\U\1\L\2' <file>`

### Executing Commands with sed

当某文件的某行的内容是一条命令的一部分时，可以使用`e`和`s` 命令来中执行这条命令

* `sed ' /^\// s/^/tar -rf catelog.tar /e' cat.list`

* `sed ' /^\// s/^/rm -f /e' cat.list`

* 注意替换的部分是行首(`^`), 并且替换字符串中包含了空格

### Using sed with vim

sed命令在terminal的使用格式为`sed ' [range] s/<string>/<replacement>/g ' /etc/passwd`;

但是在vim中使用sed命令只需要`[range] s/<string>/<replacement>/g` （即sed expressions）就可以了

注意： vim中要确保处于`normal mode`, 并按下`:`, 才能使用sed命令

### Reading and Writing to Files

在vim中， 可以将指定行写入到其他文件中

```vim
# filename可以自定义
:4,10 w filename 
```

也可以将其他文件读入到当前文件中

```vim
# filename可以自定义
: r filename
```

### Fundamentals of awk

与sed命令类似，awk命令也可用`-f` option指定一个awk file，来避免单个命令过长

```bash
$ cat users.awk
# FS is field seperators
BEGIN { FS=":" ; print "Username" }
{ print $1 }
# NR is number of recoreds
END { print "Total users = " NR } 
$ awk -f users.awk /etc/passwd
```

```bash
cat users.awk
# FS is field seperators
BEGIN { FS=":" ; print "Username" }
# count 是一个自定义的变量，而不是内置变量，将count换成i, 对结果毫无影响
/root/ { print $1 ; count++}
# NR is number of recoreds
END { print "Total users = " count } 
$ awk -f users.awk /etc/passwd
```
