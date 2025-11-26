# 扩充工具箱

## 生成文本

### date 命令：按照各种格式输出日期和时间。

```bash
$ date              # 默认格式
$ date +%Y%-m%-d    # 格式：年 - 月 - 日
$ date +%H:%M:%S    # 格式：时 : 分 : 秒
$ date +"I cannot believe it's already %A!"     # 星期
I cannot believe it's already Tuesday!
```

### seq 命令：输出一个数字序列。

```bash
$ seq 1 5           # 输出 1~5 之间的所有数字，包括 1 和 5
$ seq 1 2 10        # 第一个参数和第三个参数表示范围，中间的数字表示增幅
$ seq 3 -1 0        # 增幅为负，seq 按照降序输出数字序列
$ seq 1.1 0.1 2     # 增幅为小数，则 seq 会输出浮点小数，增幅为 0.1
$ seq -s/ 1 5       # 用斜杠分隔各个值
$ seq -w 8 10       # 根据需要添加前导零，来统一所有值的宽度（相同的字符数）
```

### 大括号扩展(shell特性)：可以输出一系列数字或字符。

```bash
$ echo {1..10}          # 从 1 到 10，正序展开
$ echo {10..1}          # 从 10 到 1，倒序展开
$ echo {01..10}         # 加上前导零（统一宽度）

# 常用方式：通过 shell 表达式{x..y..z}，生成从x到y的值，增幅为z
$ echo {1..1000..100}       # 从 1 到 1000，增幅为 100
$ echo {1000..1..100}       # 从 1000 到 1，倒序展开
$ echo {01..1000..100}      # 加上前导零

# 大括号扩展的输出只有一行，字符间以空格分隔。但可以将它的输出通过管道传递给其他命令。
$ echo {A..Z}               # 生成字母序列
$ echo {A..Z} | tr -d ' '   # 删除空格
$ echo {A..Z} | tr ' ' '\n' # 将空格换成换行符

# 创建一个别名，用于输出英文字母表中的第 n 个字母：
$ alias nth="echo {A..Z} | tr -d ' ' | cut -c"
$ nth 10
J
```

### find：输出文件路径。

find 命令可以列出目录中的文件，而且还能递归到子目录中，输出完整的路径。输出结果不以字母顺序排列。

```bash
$ find /etc/ -print     # 列出 /etc 中所有的文件，递归到子目录
# -type 指定只输出文件或目录
$ find . -type f -print     # 只输出文件
$ find . -type d -print     # 只输出目录
# -name 指定只输出与模式匹配的文件名。模式需要加引号或进行转义，避免 shell 在执行命令前处理表达式。
$ find /etc -type f -name "*.conf" -print		# 所有以 .conf_ 结尾的文件
/etc/logrotate.conf
/etc/systemd/logind.conf
/etc/systemd/timesyncd.conf
.
.
.
# -iname 指定在匹配名称时不区分大小写
$ find . -iname "*.txt" -print
```

针对输出中的每个文件路径执行一个Linux命令
1. 构造一个 find 命令，并省略 -print；
2. 在 -exec 后面指定执行的命令。使用表达式 {} 指定文件路径应出现在命令中的位置；
3. 以带引号或转义的分号结尾，例如 ";" 或 \;。

```bash
# 在文件路径名的两侧分别输出一个 @ 符号
$ find /etc -exec echo @ {} @ ";"
@ /etc @
@ /etc/issue.net @
@ /etc/nanorc @
.
.
.

# 输出 /etc 及其子目录中所有 .conf 文件的详细信息（ls -l）
$ find /etc -type f -name "*.conf" -exec ls -l {} ";"

# 删除目录 $HOME/tmp 及其子目录中名称以波浪号（~）结尾的文件
# 先运行 echo rm, 确认哪些文件要被删除，然后再删除文件
$ find $HOME/tmp -type f -name "*~" -exec echo rm {} ";"
$ find $HOME/tmp -type f -name "*~" -exec rm {} ";"         # 执行删除
```

### yes：重复输出同一行。

yes 命令可以为交互式程序提供输入，以确保这些程序可以在无人看管的情况下运行。

```bash
$ yes                                   # 默认会重复输出 "y"，直到被终止
$ yes woof!                             # 重复输出其他字符串
$ yes "Efficient Linux" | head -n3      # 连续3次输出一个字符串
```

fsck 程序：检查 Linux 文件系统的错误，它会提示用户是否继续，并等待用户输入 y 或 n。如果将 yes 命令的输出管道连接到 fsck，它就可以代表用户响应每个提示，即使用户离开，fsck 也可以持续运行，直到完成。

## 分离文本

第1章介绍过的命令
- grep：输出与某个字符串相匹配的文本；
- cut：输出文件的某些列；
- head：输出文件开头的几行。

### 深入了解 grep

```bash
# grep 可以输出某个文件中与指定字符串相匹配的文本行：
$ cat frost
Whose woods these are I think I know.
His house is in the village though;
He will not see me stopping here;
To watch his woods fill up with show.
This is not the end of the poem.

$ grep his frost                        # 输出包含 "his" 的文本行
To watch his woods fill up with snow.
This is not the end of the poem.        # "This" 匹配 "his"

# -w 表示必须匹配整个单词
$ grep -w his frost
To watch his woods fill up with snow.

# -i 表示可以忽略字母大小写
$ grep -i his frost
His house is in the village though;     # 匹配 "His"
To watch his woods fill up with snow.   # 匹配 "his"
This is not the end of the poem.        # "This" 匹配 "his"

# -l 表示只输出包含匹配文本行的文件名，不输出文本行本身
$ grep -l his *                         # 哪些文件包含字符串 "his"
frost
```

命令 grep、awk和sed共通的正则表达式写法

|                匹配条件                |                     写法                     |                                        示例                                        |
| :------------------------------------: | :-------------------------------------------: | :--------------------------------------------------------------------------------: |
|            一行开头的字符串            |                       ^                       |                               `^a`：以a开头的一行                               |
|            一行结尾的字符串            |                      \$                      |                           `!$`：以感叹号(!)结尾的一行                           |
|         单个字符（换行符除外）         |                       .                       |                             `...`：连续三个任意字符                             |
| 文字插入符号、美元符号或其他特殊符号 c |                      \c                      |                             `\$`：一个美元符号（$）                             |
|        表达式 E 出现零次或多次        |                      E*                      |                         `_*`：下划线（_）出现零次或多次                         |
|          集合中的任意一个字符          |                 [ 字符集合 ]                 |                           `[aeiouAEIOU]`：任意元音字符                           |
|         集合以外的任意一个字符         |                 [ ^字符集合 ]                 |                       `[^aeiouAEIOU]`：元音之外的任意字符                       |
|   指定范围（c1与c2之间）内的任意字符   |                    [c1-c2]                    |                                `[0-9]`：任意数字                                |
|  指定范围（c1与c2之间）之外的任意字符  |                   [^c1-c2]                   |                             `[^0-9]`：任意非数字字符                             |
|       表达式 E1 或 E2 中的某一个       | `grep` 和 `sed: E1\ \|E2` `awk: E1\ \|E2` |                 `one\ \|two`：one或two；`one\ \|two`：one 或 two                 |
|          将表达式 E 放到一组          |    `grep` 和 `sed:\(E\)` `awk: (E)`    | `\(one\ \|two\)*`：one或two出现零次或多次；`(one\|two)*`：one或two出现零次或多次 |

在 grep 命令中使用正则表达式

```bash
# 以大写字母开头的文本行
$ grep '^[A-Z]' myfile
# 所有非空文本行（匹配空行，然后再利用 -v 忽略这些文本行）
$ grep -v '^$' myfile
# 所有匹配 "cookie" 或 "cake" 的文本行
$ grep 'cookie\|cake' myfile
# 至少包含 5 个字符的文本行
$ grep '.....' myfile
# 所有小于号出现在大于号之前的文本行，例如 HTML 的代码
$ grep '<.*>' page.html


# 
# 假设在文件 frost 中搜索包含如下字符串的文本行： "w" 加一个点 （w.），这样会输出错误的结果。
# 因为点在正则表达式中表示：任意字符。
$ grep w. frost
Whose woods these are I think I know.
He will not see me stopping here.
To watch his woods fill up with snow.

# 为了解决以上问题，使用转义字符
$ grep 'w\.' frost
Whose woods these are I think I know.
To watch his woods fill up with snow.

# 有多个字符需要转义的情况下：
# 方法1：-F 选项强制 grep 忽略正则表达式，并逐字搜索输入中的每个字符。
$ grep -F w. frost
Whose woods these are I think I know.
To watch his woods fill up with snow.
# 方法2：
$ fgrep w. frost
Whose woods these are I think I know.
To watch his woods fill up with snow.

# -f 找出与一组字符串匹配的文本
: '
    要找出文件 /etc/passwd 中所有的 shell
    但 /etc/passwd 的每一行都包含有关用户的信息，且各个字段之间以冒号分隔。
    每行的最后一个字段是用户登录时启动的程序。
    $ cat /etc/passwd
    root:x:0:0:root:/root/:/bin/bash                    # 第七个字段是 shell
    daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin     # 第七个字段不是 shell
    文件 /etc/shells 列出了 Linux 系统中所有合法的 shell：
    $ cat /etc/shells
'
: ' 通过 cut 命令提取第七个字段，通过 sort -u 消除重复，
    通过 grep -f 对照一下 /etc/shells，就可以列出 /etc/passwd 中所有合法的 shell。
    添加 -F 选项，确保即使 /etc/shells 中的文本包含特殊字符进行比较
'
$ cut -d: -f7 /etc/passwd | sort -u | grep -f /etc/shells -F
```

### tail 命令：输出文件的最后几行（默认为 10 行）

假设有一个名为 alphabet 的文件，总共 26 行，每行包含一个字母：

```bash
$ cat alphabet 
A is for aardvark
B is for bunny
C is for chipmunk
.
.
.
X is for xenorhabdus
Y is for yak
Z is for zebu

$ tail -n3 alphabet             # 输出最后三行
$ tail -n+25 alphabet           # 从文件的第25行开始输出
$ head -n4 alphabet | tail -n1  # 输出第 4 行
$ head -n8 alphavet | tail -n3  # 输出第6-8行。输出 M~N行：用 head 提取前N行，用 tail 分理出最后的 N-M+1 行。
```

### awk{print} 命令

```bash
$ less /etc/hosts
127.0.0.1	localhost
255.255.255.255	broadcasthost
::1             localhost

: '
    # 将主机名分离出来，难点在于每个主机名前都有任意数量的空格。
    # cut 需要各列整齐排列，并指定列号（-c），或有一个统一的字符分隔（-f）。
    # 输出每行的第二个单词：

    # awk 可以指定任何一列，指定方法是 $加列号，$7 表示第7列。
      列号为10以上的数字，需要数字加圆括号，如$(25)。
      指定最后一个字段，可以使用 $NF (NF 表示 "字段数量")
      指定整行，使用 $0。
'
$ awk '{print $2}' /etc/hosts
localhost
myhost

# awk 不会在各个值之间输出空白，如果想要空白，可以用逗号（,）分隔各个值：
$ echo Efficient fun Linux | awk '{print $1 $3}'        # 没有空格
EfficientLinux
$ echo Efficient fun Linux | awk '{print $1, $3}'       # 有空格
Efficient Linux

$ df / /data                            # df 输出 Linux 系统上磁盘的已使用容量和剩余容量
$ df / /data | awk '{print $4}'         # 分理出每一行的某一列，比如第4列
$ df / /data | awk 'FNR>1 {print $4}'   # 让 awk 命令只输出编号大于 1 的行

# 输入中包含的分隔符不是空格而是其他字符。-F 选项，用正则表达式指定字段分隔符
$ echo efficient:::::linux | awk -F':*' '{print $2}'    # 任意数量的冒号
```

## 组合文本

```bash
$ cat poem1
It is an ancient Mariner,
And he stoppeth one of three.

$ cat poem2
'By thy long grey beard and glittering eye,

$ cat poem3
Now wherefore stopp'st thou me?

$ cat poem1 poem2 poem3
It is an ancient Mariner,
And he stoppeth one of three.
'By thy long grey beard and glittering eye,
Now wherefore stopp'st thou me?

```

### tac 命令：反向组合文本文件。

遇到已按时间顺序排列，但无法通过命令 sort -r 反向排序的数据，就可以使用 tac 来处理。

```bash
$ cat poem1 poem2 poem3
It is an ancient Mariner,
And he stoppeth one of three.
'By thy long grey beard and glittering eye,
Now wherefore stopp'st thou me?

$ cat poem1 poem2 poem3 | tac               # 反向输出文件
Now wherefore stopp'st thou me?
'By thy long grey beard and glittering eye,
It is an ancient Mariner,
And he stoppeth one of three.

$ tac poem1 poem2 poem3
And he stoppeth one of three.               # 先反转第一个文件
It is an ancient Mariner,
'By thy long grey beard and glittering eye, # 第二个文件
Now wherefore stopp'st thou me?             # 第三个文件
```

反转网络服务器的日志文件，按照从新到旧的顺序显示：
日志文件输出的每一行都按时间顺序排列，且带有时间戳，但它们不是字母或数字顺序，所以不能使用 sort -r 来反向排序。但命令 tac 可以反向输出这些记录，而且不需要考虑时间戳。

### paste 命令：以列对列的方式组合文本文件。

paste：将每个文件以列对列的方式结合起来，并以制表符分隔。
cut：以制表符分隔的各列提取出来。

```bash
$ cat title-words1
EFFICIENT
AT
COMMAND

$ cat title-words2
linux
the 
line

$ paste title-words1 title-words2
EFFICIENT   Linux
AT      the
COMMAND line

$ paste title-words1 title-words2 | cut -f2     # cut 和 paste 是一对儿
linux
the 
line

$ paste -d, title-words1 title-words2           # 将分隔符换成其他字符，比如逗号(,)，指定选项 -d (delimiter)
EFFIECIENT,linux
AT,the
COMMAND,line

$ paste -d, -s title-words1 title-words2        # 转置输出：将各行结合在一起（而不是各列），则可以指定选项 -s
EFFICIENT,AT,COMMAND
linux,the,line

$ paste -d "\n" title-words1 title-words2       # 将分隔符改为换行符(\n)，paste 可交叉显示来自两个或多个文件的数据
EFFICIENT
linux
AT
the
COMMAND
line
```

### diff 命令：对比两个文件的文本，并输出二者的差异。

```bash
$ cat file1
Linux is all about efficiency.
I hope you will enjoy this book.

$ cat file2 
MacOS is all about efficiency.
I hope you will enjoy this book.
Have a nice day.

$ diff file1 file2
1c1                                     # 1c1：第一个文件的第1行 与 第二个文件的第 1 行有差异，
< Linux is all about efficiency.        # 1c1 后面紧跟着 file1 的相关行。< 表示本行来自第一个文件
---                                     # 三条短线分隔符
> MacOS is all about efficiency.        # file2 的相关行。> 表示本行来自第二个文件
2a3                                     # 2a3 表示添加，意思是文件 file2 中包含第三行内容，而 file1 的第二行之后没有这些内容
> Have a nice day.                      # 2a3 后面是 file2 多出来的一行内容： Having a nice day.

# 利用 grep 和 cut 单独显示文件中含有差异的行：
$ diff file1 file2 | grep '^[<>]'
< Linux is all about efficiency.
> MacOS is all about efficiency.
> Have a nice day.
$ diff file1 file2 | grep '^[<>]' | cut -c3-
Linux is all about efficiency.
MacOS is all about efficiency.
Have a nice day.
```

## 文本转换

### tr 命令：将指定字符转换成其他字符。

```bash
$ echo $PATH | tr : "\n"        # 将冒号转换成换行符
$ echo efficient | tr a-z A-Z   # 将 efficient 改成全大写
EFFICIENT
$ echo Efficient | tr A-Z a-z   # 将 Efficient 改成全小写
efficient
$ echo Efficient  Linux | tr " " "\n"   # 将空格换成换行符
$ echo efficient linux | tr -d ' \t'    # 用选项 -d 删除空格和制表符
```

### rev 命令：反转一行中的字符。

```bash
$ echo Efficient Linux! | rev
!xuniL tneiciffE

$ cat cekebrities
Jamie Lee Curtis
Zooey Deschanel
Zemdays Maree Stoermer Coleman
Rihanna

# 提取每一行的最后一个单词（Curtis、Deschanel、Coleman、Rihanna）
# 如果文件的每一行都有数量相等的字段，可以用 cut -f 提取。
# 该文件每行的字段都不同，可以用 rev 反转每一行的文本，提取第一个字段，再反转一次。

$ rev celebrities
sitruC eeL eimaJ
lenahcseD yeooZ
nameloC remreotS eeraM syadmeZ
annahiR
$ rev celebrities | cut -d' ' -f1
sitruC
lenahcseD
nameloC
annahiR
$ rev celebrities | cut -d' ' -f1 | rev
Curtis
Deschanel
Coleman
Rihanna
```

### awk 与 sed 命令：通用的转换操作。

为了成功使用 awk 和 sed 需要：
- 理解这两个命令可以完成哪些类型的操作，并根据需要应用它们；
- 阅读帮助文档，通过 [Stack Exchange](https://oreil.ly/0948M) 和其他在线资源寻找完整的解决方案。

```bash
$ sed 10q myfile            # 输出前 10 行，然后退出
$ awk 'FNR<=10' myfile      # 循环输出编号 <= 10 的文本行

$ echo image.jpg | sed 's/\.jpg/.png/'              # 将 .jpg 替换成 .png 
$ echo "linux efficient" | awk '{print $2, $1}'     # 交换两个单词
```

- awk 命令的基本知识

awk: 将文件（标准输入）的文本转换成任何其他文本。

```bash
$ awk 程序 输入文件
```

可以将一个或多个 awk 程序保存在文件中，并通过 -f 选项调用它们，这些程序按顺序运行
```bash
$ awk -f 程序文件1 -f 程序文件2 -f 程序文件3 输入文件
```

awk 程序包含一个或多个动作，当输入与指定的模式匹配时，就执行这些动作。
程序中的每个指令的形式：模式 { 动作 }

常见的模式：
- 关键字 BEGIN：在 awk 处理输入之前执行该动作，且只执行一次。
- 关键字 END：在 awk 处理完所有输入之后执行动作，且只执行一次。
- 前后都有斜杠的正则表达式
- 其他 awk 特有的表达式

检查输入行的第3个字段($3)是否以大写字母开头，则匹配模式为：$3~/^[A-Z]/
跳过输入的前 5 行的匹配模式为：FNR>5

- 如果某个动作没有指定模式，则对输入的每一行都运行一次

```bash
# 与 rev celebrities | cut -d' ' -f1 | rev 输出相同的结果
$ awk '{print $NF}' celebrities

# 没有指定动作，只有模式。原样输出匹配的输入
$ echo efficient linux | awk '/efficient/'
efficient linux

: '
    将 animals.txt 转换成一份整洁的参考书单
    原格式：python programming Python 2010   Lutz, Mark
    转换成：Lutrz, Mark (2010). "Programming Python"

    需要重新规整三个字段，还要添加一些字符，比如括号和双引号，
'
$ awk -F'\t' '{print $4, "(" $3 ").", "\"" $2 "\""}' animals.txt
Lutz, Mark (2010). "Programming Python"
Barrett, Daniel (2005). "SSH, The Secure Shell"
Schwartz, Randal (2012). "Intermediate Perl"
Bell, Charles (2014). "MySQL High Availability"
Siever, Ellen (2009). "Linux in a Nutshell"
Boney, James (2005). "Cisco IOS in a Nutshell"
Roman, Steven (1999). "Writing Word Macros"
# 只处理包含 "horse" 的书籍
$ awk -F'\t' '/^horse/{print $4, "(" $3 ").", "\"" $2 "\""}' animals.txt
Siever, Ellen (2009). "Linux in a Nutshell"
# 只处理2010年及以后的书
$ awk -F'\t' '$3~/^201/{print $4, "(" $3 ").", "\"" $2 "\""}' animals.txt
Lutz, Mark (2010). "Programming Python"
Schwartz, Randal (2012). "Intermediate Perl"
Bell, Charles (2014). "MySQL High Availability"

: '
    通过一个 BEGIN 指令输出一个标题，
    再加上一些短线（-）来缩进书单的每一行，
    并通过 END 指令引导读者了解更多信息。
'
$ awk -F'\t' \
    'BEGIN {print "Recent books:"} \
    $3~/^201/{print "-", $4, "(" $3 ").", "\"" $2 "\""} \
    END {print "For more books, search the web"}' \
    animals.txt
Recent books:
- Lutz, Mark (2010). "Programming Python"
- Schwartz, Randal (2012). "Intermediate Perl"
- Bell, Charles (2014). "MySQL High Availability"
For more books, search the web

# 从 1 累加到 100
$ seq 1 100 | awk '{s+=$1} END {print s}'
5050
```

[awk教程1](https://www.tutorialspoint.com/awk/index.htm)
[awk教程2](https://riptutorial.com/awk)

- 改进重复文件的检测

```bash
# 构建一个通过校验和检测重复的 JPEG 文件的管道，但这个管道无法输出文件名
$ md5sum *.jpg | cut -c1-32 | sort | uniq -c | sort -nr | grep -v "     1"

$ md5sum *.jpg 
: ' 
    用 awk 的数组和循环输出文件名：

    1. awk 检查 md5sum 输出的每一行，将校验和分离出来($1)，将其作为 counts 数组的键。
    每当 awk 遇到同一个校验和，就将该元素的值加1（运算符++），至此awk脚本只统计了每个校验和的数量。
    $ md5sum *.jpg | awk '{counts[$1]++}'
    2. 使用 for 循环，通过键遍历数组，并按顺序处理每个元素：for (变量 in 数组) 处理数组[ 变量 ]
    例如，for (key in counts) pirnt array[key]
    将循环放入 END 指令内，在统计完所有数量后，运行这个循环
    $ md5sum *.jpg \
    | awk '{counts[$1]++} \
            END {for (key in counts) print counts[key]}'
    3. 输出校验和：（每个数组键就是一个校验和）统计完成后，输出数组键
'
$ md5sum *.jpg \
|   awk '{counts[$1]++} \
        END {for (key in counts) print counts[key] " " key}'

: '
    收集并输出文件名：添加一个数组 names，键同样是校验和。
    在 awk 处理每一行输出时，将文件名($2)附加到names数组相应的元素中，并以空格作为分隔符。
    在 END 的循环内，在输出校验和（键）之后，输出一个冒号以及该校验和的文件名：
'
$ md5sum *.jpg \
    | awk '{counts[$1]++; names[$1]=names[$1] " " $2} \
        END {for (key in counts) print counts[key] " " key ":" names[key]}'

: '
    如果行开头显示的数字为1，则表示该校验和只出现了一次，即没有重复。
    通过管道将输出传递给 grep -v，就可以去除这些行，然后通过 sort -nr 按数字从大到小排列。
'

$ md5sum *.jpg \
    | awk '{counts[$1]++; names[$1]=names[$1] " " $2} \
        END {for (key in counts) print counts[key] " " key ":" names[key]}' \
    | grep -v '^1 ' \
    | sort -nr
```

- sed 命令的基本知识

```bash
$ sed 脚本 输入文件
$ sed -e 脚本1 -e 脚本2 -e 脚本3 输入文件               # 指定多个脚本，并依照次序处理输入
$ sed -f 脚本文件1 -f 脚本文件2 -f 脚本文件3 输入文件     # 将 sed 脚本保存到文件中，然后通过 -f 选项调用，它们会按照次序执行
# sed 最常见类型的脚本是替换字符串，
# 语法：s/regexp/replacement/ 。其中 regexp 是匹配每个输入行的正则表达式，replacement 是替换匹配文本的字符串。
$ echo Efficient Windows | sed "s/Windows/Linux/"   # 将一个单词替换成另一个

# 功能等同于 $ rev celebrities | cut -d' ' -f1 | rev
# 匹配最后一个空格前的所有字符（.*），然后将它们替换成空即可
$ sed 's/.* //' celebrities         

$ echo Efficient Stuff | sed "s/stuff/linux"        # 区分大小写，不匹配
Efficient Stuff

# 选项 i 表示匹配不分大小写
$ echo Efficient Stuff | sed "s/stuff/linux/i"      # 不区分大小写，匹配
Efficient Linux

$ echo efficient stuff | sed "s/f/F/"               # 只替换第一个 "f"
eFficient stuff

# 选项 g 表示全局
$ echo efficient stuff | sed "s/f/F/g"              # 替换所有出现的 "f"
eFFicient stuFF

# 根据编号删除行
$ seq 10 14 | sed 4d                                # 删除第 4 行
10
11
12
14

# 删除与正则表达式相匹配的行
$ seq 101 200 | sed '/[13579]$/d'                   # 删除末尾是奇数的行
```

在命令行运行 sed 脚本的时候，可以在周围添加引号，防止 shell 对 sed 的特殊字符求值。根据需要选择单引号或双引号。
当正则表达式本身包含正斜杠（需要转义）时，就可以改用其他字符。这三个 sed 脚本的效果是一样的：s/one/two/ s_one_two s@one@two@

- 利用 sed 匹配子表达式

```bash
$ ls 
image.jpg.1 image.jpg.2 image.jpg.3
: '
    要生成新文件名，如 image1.jpg、image2.jpg、image3.jpg 
    sed 通过其子表达式的功能，将文件名拆分为多个部分，并重新排列。

'
# 先创建一个匹配文件名的正则表达式： image\.jpg\.[1-3]
# 将文件名最后一个数字移到前面，定义一个子表达式： image\.jpg\.([1-3]\)
# 新文件名的形式为：image\1.jpg
$ ls | sed "s/image\.jpg\.\([1-3]\)/image\1.jpg"
image1.jpg
image2.jpg
image3.jpg

# 假设文件名为小写字母组成的单词：
$ ls
apple.jpg.1 banana.jpg.2 carrot.jpg.3

# 创建三个子表达式，分别代表主文件名、扩展名和最后的数字：
# \([a-z][a-z]*\)             \1：一个或多个字母的主文件名
# \([a-z][a-z][a-z]\)         \2：三个字母的文件扩展名
# \([0-9]\)                   \3：一个数字

# 转换后的文件名应表示为 \1\3.\2 将其提供给 sed
$ ls | sed "s/\([a-z][a-z]*\)\.\([a-z][a-z][a-z]\)\.\([0-9]\)/\1\3.\2/"
```

## 进一步扩展工具箱

```bash
# 查找 Linux 系统已安装的命令，或 apropos
$ man -k width
$ apropos
```

流行的包管理器：apt、dnf、emerge、pacman、rpm、yum、zypper

Ubuntu 或 Debian Linux 的命令：

```bash
$ sudo apt update               # 下载最新的元数据
$ apt-file search string        # 搜索字符串
```

# 父、子与环境

shell 不断重复：
- 显示提示符
- 从标准输入读取命令
- 计算并运行命令

Linux 很好地隐藏了一个事实：shell 知识一个普通的程序。

在第2章的基础上，深入探讨 shell ：
- shell 程序位于何处？
- 不同 shell 实例之间有何关联？
- 为什么不同的 shell 实例可以拥有相同的变量、值、别名和其他上下文？
- 如何通过编辑配置文件改变 shell 的默认行为？

## shell 是可执行文件

```bash
$ cd /bin && ls -l bash cat ls      # bash cat ls 保存在 /bin 里
$ cat /etc/shells                   # 列出所有有效的 shell
$ echo $SHELL                       # 正在运行的 shell
```

```bash
# halshell: 一个拒绝运行命令的 shell。
#!/bin/bash
# Print a prompt
echo -n '$ '
# Read the user's input in a loop. Exit when the user presses Ctrl-D.
while read line; do
# Ignore the input $line and print a message 
echo "I'm sorry, I'm afraid I can't do that"
# Print the next prompt
echo -n '$ '
done
```

```bash
# shell 新实例
$ bash
$

# 为了区分新实例，通过设置 shell 变量 PS1，将其提示符修改为 %%，再运行几个命令：
$ PS1="%% "                     # 提示符发生了变化
%% ls
animals.txt

%% echo "This is a new shell."
This is a new shell.

%% exit
$                               # 新的 bash 实例已终止
```

## 父子进程

每个进程都有自己的环境。环境包括：
- 当前目录
- 搜索路径
- shell 提示符
- shell 变量中保存的其他重要信息
在创建子进程时，其环境大部分来自父进程的环境。
**每当运行一个命令时，就会创建一个子进程。在子进程运行的任何修改，只会影响子进程shell，而且这个修改在退出子进程shell时就会失效。同样，父进程中的任何变更也不会影响到正在运行的子进程。但是，父进程的变更会影响到之后创建的子进程。因为每个子进程都会在启动时复制父环境。**

在子进程运行命令很重要：运行的任何程序都可以通过 cd 命令浏览整个文件系统，但在它退出时，当前shell（父shell）不会改变其当前目录。

例如，在根目录中创建一个名为 cdtest 的shell脚本，其中包含一个 cd 命令。

cdtest 脚本：
```bash
#!/bin/bash
cd /etc
echo "Here is my current directory:"
pwd
```

```bash
# 赋予该脚本可执行的权限：
$ chmod +x cdtest

# 输出当前目录名称，运行该脚本：
$ pwd
/home/smith
$ ./cdtest                          # cdtest 在子进程中运行，它有自己的环境
Here is my current directory:
/etc

# 现在检查你的当前目录
$ pwd
/home/smith
```

cd 是 shell 的一个内置命令：（如果 cd 是外部程序，就不能改变目录了。）程序都在子进程运行，无法影响到父进程。
管道会启动多个子进程，管道的每个命令对应一个子进程，

## 环境变量

一些环境变量及其用途：
- HOME：用户主目录的路径，在登录系统时，登录shell自动设定该值。
    vim 和 emacs 等会读取变量 HOME，找到并读取其配置文件。（$HOME/.vim 或 $HOME/.emacs）
- PWD：shell的当前路径。
    每当通过 cd 命令切换到其他目录时，shell 自动设置这个值。
    命令pwd 可以读取这个变量，并输出 shell 当前的目录名。
- EDITOR：首选的文本编辑器（或路径），其值一般由shell的配置文件设定。
    其程序可以读取这个变量，并根据需要启动正确的编辑器。

```bash
# 查看shell环境变量：（printenv 输出不包括局部变量）
$ printenv | sort -i | less  

# 显示局部变量：在变量名前加上$，通过 echo 输出
$ title="Efficient Linux"
$ echo $title
Efficient Linux
$ printenv title            # 不会产生任何输出
```

### 创建环境变量

```bash
# 使用 export 命令，将某个局部变量变成环境变量。
$ MY_VARAIBLE=10                    # 局部变量
$ export MY_VARIABLE                # 通过 export 将它变成环境变量
$ export ANOTHER_VARIABLE=20        # 通过一条命令设置并导出变量

# export 指明变量及其值应当从当前 shell 复制到以后启动的子进程中。
# 局部变量不会复制到子进程。
$ export E="I am an environment variable"   # 设定一个环境变量
$ L="I am just a local variable"            # 设定一个局部变量
$ echo $E
I am an environment variable 
$ echo $L
I am just a local variable 
$ bash                                      # 运行一个子进程 shell
$ echo $E                                   # 环境变量被复制
I am an environment variable
$ echo $L                                   # 局部变量没有被复制
                                            # 输出为空字符串
$ exit                                      # 退出子进程 shell
```

```bash
# 子进程的变量是父进程的副本，不会影响到父进程shell
$ export E="I am the original value"        # 设定一个环境变量
$ bash                                      # 运行一个子进程 shell
$ echo $E
I am the original value                     # 父进程的值被复制
$ E="I was modified in a child"             # 修改子进程的值
$ echo $E
I was modified in a child
$ exit                                      # 退出子进程 shell
$ echo $E
I an the orginal value                      # 父进程的值没有变化
```

### 警惕 "全局" 变量

环境变量不是全局的，每个 shell 实例都有自己的副本。修改一个shell中的环境变量，不会影响到其他正在运行的shell，只会影响到该shell之后创建的子进程。

环境变量HOME 和 PATH等变量能够在所有shell实例中保持相同的值
- 方法1：子进程复制父进程的变量
HOME 或 PATH 等变量的值是在登录shell时自动配置和导出的。所有之后的shell都是登录shell的子进程。
- 方法2：不同的实例读取相同的配置文件
局部变量不会复制到子进程，它们的值是在Linux的配置文件中设置的，如$HOME/.bashrc。shell的每个实例在启动时都会读取并执行相应的配置文件。这样，每个子进程shell都会拥有这些局部变量。

## 子进程 shell 与子 shell

子进程shell（child shell）：复制父进程的一部分（如，环境变量），但不会复制父进程的局部（未导出的）变量或别名。

```bash
$ alias             # 列出所有别名
alias gd='pushd'
alias l='ls -CF'
alias pd='popd'

$ bash --norc       # 运行一个子进程 shell，并忽略 bashrc 文件
$ alias             # 列出所有别名：都是未知的
$ echo $HOME        # 环境变量都是已知的
/home/smith
$ exit              # 退出子进程 shell
```

子shell（subshell）：完整地复制父shell，包括父shell的所有变量、别名、函数等。**如果要在子shell启动一个命令，需要在命令前后加括号。**

```bash
$ (ls -l)           # 在子 shell 中运行 ls -l
-rw-r--r-- 1 smith smith 325 Oct 13 22:19 animals.txt 
$ (alias)           # 在子 shell 中查看别名
alias gd=pushd
alias l=ls -CF
alias pd=popd
.
.
.
$ (l)               # 运行父 shell 的别名
animals.txt
```

检查shell的实例是否在子shell中：

```bash
# 如果在子shell中，BASH_SUBSHELL 为 1，否则为 0。
$ echo $BASH_SUBSHELL   # 检查当前 shell
0                       # 不是子 shell

$ bash                  # 运行子进程 shell
$ echo $BASH_SUBSHELL   # 检查子进程 shell
0                       # 不是子 shell
$ exit                  # 退出子进程 shell

$ (echo $BASH_SUBSHELL) # 运行子 shell
1                       # 这是一个子 shell
```

## 配置环境

bash 标准配置文件类型
- 启动文件：登录时自动执行的配置文件，只对登录shell有效。该文件可能会设置和导出环境变量。但是，在该文件中定义别名没有任何作用（别名不会复制到子进程中）。
- 初始化文件：供登录shell之外的每个shell实例执行的配置文件。（手动运行交互式shell或非交互式shell脚本）初始化文件命令可能会设置变量或定义别名。
- 清理文件：在登录shell退出前执行的配置文件。该文件可能会执行 clear 命令，确保登出时清空屏幕。

bash 的标准配置文件
|文件类型|由谁运行|系统内的位置|个人文件的位置|
|:-:|:-:|:-:|:-:|
|启动文件|登录shell，启动时运行|/etc/profile|\$HOME/.bash_profile、\$HOME/.bash_login 和 \$HOME/.profile|
|初始化文件|交互式shell（非登录shell），启动时运行|/etc/bash.bashrc|$HOME/.bashrc|
||shell脚本，启动时运行|将变量 BASH_ENV 设置为初始化文件的绝对路径（例如，BASH_ENV=/usr/local/etc/bashrc）|将变量 BASH_ENV 设置为初始化文件的绝对路径（例如，BASH_ENV=/usr/local/etc/bashrc）|
|清理文件|登录shell，退出时运行|/etc/bash.bash_logout|$HOME/.bash_logout|

**用户主目录中可供用户使用的启动文件：.bash_profile、.bash_login、.profile。多数用户选择其一即可。如果还要运行其他shell（如，Bourne shell、Korn shell），需要注意这些shell也会读取 .profile。如果将 bash 专用的命令放入其中，可能会运行失败。这时应将 bash 专用的命令放入 .bash_profile 或 .bash_login 中。**

将个人启动文件和个人初始化文件分开的原因：
- 希望明确划分个人启动文件和个人初始化文件的职责：
    （尽管个人启动文件只是执行个人初始化文件 $HOME/.bashrc，所以所有交互式shell都拥有大概相同的配置。）个人启动文件负责设置并导出环境变量，方便复制到子进程中。但$HOME/.bashrc 负责定义所有的别名（不会复制到子进程）。
- 登录到图形化窗口（GNOME、KDE、Unity、Plasma）是看不到登录shell的：
    此时，只与登录shell的子进程交互，因此可以将大部分或全部配置放到 $HOME/.bashrc 中。但若主要从SSH客户端登录，则与登录shell直接交互，所以shell的配置很重要。

建议将个人启动文件通过 source 命令执行个人初始化文件：

```bash
# Place in $HOME/.bash_profile or other personal startup file 
if [ -f "$HOME/.bashrc" ]
then
  source "$HOME/.bashrc"
fi
```

### 重新读取配置文件

在修改启动文件或初始化文件后，可以强制 shell 重新读取这些文件

```bash
$ source ~/.bash_profile        # 使用内置的 "source" 命令
$ .~/.bash_profile              # 使用一个点（.）
```

### 统一获取配置文件

把配置文件上传到 github 的免费账号或其他版本控制软件中，这样可以在任何一台Linux统一下载、安装和更新配置文件。

# 运行命令的 11 种方法

## 列表技巧

### 技巧一：条件列表：每个命令都依赖于前一个命令的成功与否。

```bash
# 在目录 dir 创建一个文件 new.txt
$ cd dir            # 进入目录
$ touch new.txt     # 创建文件

$ cd dir && touch new.txt

# 在修改文件前，先创建一个备份，然后修改源文件，最后删除备份
# 前提：前一个命令运行成功
$ cp myfile.txt myfile.safe         # 创建备份
$ nano myfile.txt                   # 修改源文件
$ rm myfile.safe                    # 删除备份

$ cp myfile.txt myfile.safe && nano myfile.txt && rm myfile.safe 

: '
    修改某文件后要运行的命令序列：
    1. git add 准备提交文件;
    2. git commit;
    3. git push 提交变更

    && 表示 只有第一个命令运行成功，才执行第二个命令。
    || 表示 只有第一个命令运行失败，才执行第二个命令。
'
$ git add . && git commit -m"fixed a bug" && git push

# 先尝试进入 dir，如果失败，则创建 dir
$ cd dir || mkdir dir 

# If a directory can't be entered, exit with an error code of 1
$ cd dir || exit 1

: '
    先尝试进入目录 dir，如果失败就创建该目录，并进入其中。
    如果所有命令都以失败告终，则输出一条错误信息。
'
$ cd dir || mkdir dir && cd dir || echo "I failed"
```

表示成功或失败的退出代码（每个Linux命令在结束时都会产生的一个结果），惯例是0表示成功，非0表示失败。如要查看 shell 最后一个命令的退出代码，可输出特殊的shell变量 "问号" （?）:

```bash
$ ls myfile.txt
myfile.txt
$ echo $?               # 输出变量 ? 的值
0                       # 成功
$ cp nonexistent.txt somewhere.txt
cp: cannot stat 'nonexistent.txt': No such file or directory
$ echo $?
1                       # 复制失败
```

### 技巧二：无条件列表：命令一个接一个地运行。

用分号分离各个命令，它们就会依次运行。某个命令的成败不影响列表后面的命令。
无条件列表产生的结果大多与单独键入各个命令并在每个命令后按回车是相同的。它们最大的不同是：退出代码。在无条件列表中，除了最后一个命令外，其余命令的退出代码都会被丢弃。只有列表中最后一个命令的退出代码会赋给 shell 变量问号（?）.


```bash
# 无条件睡眠两个小时，然后备份我的重要文件
$ sleep 7200; cp -a ~/important-file /mnt/backup_drive

# 一个最原始的提醒系统：睡眠 5 分钟，然后给我发一封电子邮件
$ sleep 300; echo "remember to walk the dog" | mail -s reminder $USER

$ mv file1 file2; mv file2 file3; mv file3 file4
$ echo $?
0               # 仅代表命令 "mv file3 file4" 的退出代码
```

## 替换技巧：将命令的某个文本替换成其他文本。

### 技巧三：命令替换：将命令替换成它的输出。

```bash

: '
    假设有几千个歌曲的文件，每个文件都保存了歌曲名称、艺术家姓名、专辑名称和歌词：
    Title: Carry On Wayward Son
    Artist: Kansas
    Album: Leftoverture

    Carry on my wayward son     
    There will be peace when you are done

    希望按照艺术家将这些文件组织到子目录中。
'
# 手动执行：使用 grep 搜索艺术家（如 kansas）的所有歌曲
$ grep -l "Artist: Kansas" *.txt
carry_on_wayward_son.txt
dust_in_the_wind.txt
belexes.txt
# 然后将每个文件移动到目录 Kansas 中
$ mkdir kansas
$ mv carry_on_wayward_son.txt kansas
$ mv dust_in_the_wind.txt kansas
$ mv belexes.txt kansas

# 让 shell 将所有包含字符串 Artist: Kansas 的文件都移动到目录 Kansas 下
# grep -l 获取的文件名列表传递给 mv
# $(任意命令) 表示执行括号里的命令，然后将命令替换成输出。 但是如果文件名包含空格或其他特殊字符就不行了。
$ mv $(grep -l "Artist: Kansas" *.txt) kansas
# 上面这条命令相当于：
$ mv carry_on_wayward_son.txt dust_in_the_wind.txt belexes.txt kansas

: '
    假设下载了一份PDF格式的银行对账单，其中包含多年的数据。
    下载的文件名中包含对账单的年月日，如 eStmt_2021-08-26.pdf 
    若要查看当前目录中最新的对账单，可以手动查看：
    列出目录，找到最近的文件（列表中的最后一个文件），用okular打开PDF文件
'
# 编写一个命令，通过命令替换，将以上输出传递给okular
# ls 会列出所有的对账单，tail 只输出最后一个
$ okular $(ls eStmt*pdf | tail -n1)

# 过去，命令替换使用的是反引号（`）。以下两个命令是等效的：
$ echo Today is $(date +%A).
Today is Saturday.
$ echo Today is `date +%A`.
Today is Saturday.

# 大多数 shell 都支持反引号（`）。但是，语法$()更方便嵌套：
$ echo $(date +%A) | tr a-z A-Z                     # 一个命令
SATURDAY
$ echo Today is $(echo $( date +%A) | tr a-z A-Z)!  # 嵌套命令
Today is SATURDAY!

# 在脚本中，命令替换的常见用法是将某个命令的输出保存到变量中：
# 变量名 =$(此处是命令)

# 获取包含 Kansas 歌曲的文件名，然后将它们保存到一个变量中，就可以使用命令替换
$ kansasFiles=$(grep -l "Artist: Kansas" *.txt)
# 输出有可能包含多行，为了保证换行符能正确传递，确保在使用变量时，加上引号：
$ echo "$KansasFiles"

```

### 技巧四：进程替换：将命令替换成某个文件（类似于文件）。

```bash
: '
    假设在一个目录中，保存了很多 JPEG 格式的图像文件，名称分别为 1.jpg ~ 1000.jpg。
    但有些文件神秘消失了，你想知道是哪些。
'
# 生成一个这样的目录：
$ mkdir /tmp/jpegs && cd /tmp/jpegs
$ touch {1..1000}.jpg
$ rm 4.jpg 981.jpg

# 一种苯办法：列出目录中所有的文件，按数字排序，再通过肉眼找出丢失的文件。
$ ls -1 | sort -n | less 
1.jpg
2.jpg
4.jpg       # 3.jpg 消失了
5.jpg

: '
    更强大的方法：使用 diff 命令，从 1.jpg ~ 1000.jpg 逐一对比现有文件名与完整的名称列表。
'
# 方法1：使用临时文件。
# 1. 将现有文件名按顺序存储在一个临时文件 original-list 中
$ ls *.jpg | sort -n > /tmp/original-list
# 2. 将完整的名称列表从 1.jpg ~ 1000.jpg 输出到另一个临时文件 ful-list 中
# 使用 seq 生成 1-1000 的整数，然后再通过 sed 在每一行末尾加上".jpg"
$ seq 1 1000 | sed 's/$/.jpg/' > /tmp/full-list
# 3. 通过 diff 命令比较两个临时文件，发现 4.jpg 和 981.jpg 缺失。
$ diff /tmp/original-list /tmp/full-list
3a4
> 4.jpg
978a981
> 981.jpg
# 4. 最后删除两个临时文件
$ rm /tmp/original-list /tmp/full-list

： '
    省略生成临时文件的步骤，直接比较两个名称列表：
    难点：diff 无法比较来自标准输入的两个列表，它需要文件作为参数。
    解决办法：进程替换（将两个列表都当成文件传递给 diff）：
    <(任意命令)
'

$ cat <(ls-1 | sort -n)
1.jpg
2.jpg
...

$ cp <(ls-1 | sort -n) /tmp/listing
$ cat /tmp/listing
1.jpg
2.jpg
...

# 利用 diff 比较这两个文件：
# 1. 生成两个临时文件
$ ls *.jpg | sort -n
$ seq 1 1000 | sed 's/$/.jpg/'
# 2. 加上进程替换，diff 把它们当成文件
$ diff <(ls *.jpg | sort -n) <(seq 1 1000 | sed 's/$/.jpg')
3a4
> 4.jpg

# 3. 整理输出：利用 grep 去掉行开头的大于号（>），通过 cut 提取丢失的文件名
$ dif <(ls *.jpg | sort -n) <(seq 1 1000 | sed 's/$/.jpg') \
    | grep '>' | cut -c3-
4.jpg
981.jpg
```

进程替换是一个非 POSIX 功能

```bash
# 启用 shell 中的 POSIX 功能
$ set +o posix
```

标准输入、标准输出、标准错误的文件描述符分别为0、1、2。因此，重定向标准错误的写法为 >2

## 命令作为字符串的技巧

### 技巧五：将命令作为字符串传递给 bash

新启动的 bash 进程是一个子进程，它有自己的环境（当前目录、变量及其值）。任何子进程 shell 的变化都不会影响到当前运行的 shell。

```bash
# 运行 bash 启动一个交互式 shell，通过 -c 选项将命令当成字符串传递给 bash，
# bash 会将这个字符串作为命令运行，然后退出。
$ bash -c "ls -l"
-r-r--r-- 1 smith smith 325 Jul     3 17:44 animals.txt

# bash -c 访问目录 /tmp，并删除一个文件，然后退出。
$ pwd
/home/smith
$ touch /tmp/badfile                # 创建一个临时文件
$ bash -c "cd /tmp && rm badfile"
$ pwd 
/home/smith                         # 当前目录没有变化
```

bahs -c 适合以超级用户的身份运行某些命令。（将sudo 与输入/输出重定向组合到一起）

```bash
# 在系统目录 /var/log 中创建一个日志文件，普通用户没有日志文件的写入权限。
# sudo 命令获得超级用户权限并创建日志文件，失败原因：sudo 没有运行。
: '
    sudo 的作用范围只限于 echo，无法触及输出重定向。
    执行该命令时，shell 先处理重定向，但这一步失败了：
    1. 按下回车键；
    2. shell 开始处理这个命令，包括重定向 （>）;
    3. shell 尝试在受保护的目录 /var/log 中创建文件 custom.log；
    4. 但没有权限写入 /var/log，因此 shell 放弃并输出 "Permission denied"

    所以，需要告知 shell，“以超级用户的身份运行整个命令，包括输出重定向”。
    bash -c 适合解决这种问题。
'
$ sudo echo "New log file" > /var/log/cutom.log                 
bash: /var/log/custom.log: Permission denied

# 将想运行的命令构造成字符串：'echo "New log file" > /var/log/custom.log'，将它作为参数，传递给 sudo bash -c
$ sudo bash -c 'echo "New log file" > /var/log/custom.log'
[sudo] password for smith: xxxxxxxx
$ cat /var/log/custom.log
New log file 
```

### 技巧六：通过管道将命令传递给 bash

shell 会读取标准输入中键入的每个命令，即 bash 可以放入管道中。
**永远不要将不明的文本传递给 bash。必须清除在执行什么命令。**
适合依次执行多个类似的命令

```bash
$ echo "ls -l"
ls -l
# bahs 会把字符串 "ls -l" 当成命令运行
$ echo "ls -l" | bash

: '
    假设某个目录中有许多文件，按首字母将它们划分到各个子目录中，
    名为 apple 的文件被移到子目录a中，名为 cantaloupe 的文件会被移到子目录 c 中，以此类推。
'

# 1. 列出所有文件并排序：假定所有文件名至少包含两个字符（即匹配模式 ??*）,防止命令与子目录 a~z 冲突。
$ ls -1 ??*
apple
banana
cantaloupe
carrot
...

# 2. 用大括号扩展创建 26个子目录
$ mkdir {a..z}

# 3. 生成 mv 命令的字符串。
# 开头是 sed 的第一个正则表达式(\1)，将文件名的第一个字符提取出来： ^\(.\)
# 第二个正则表达式(\2)将文件名的其余字符提取出来： \(.*\)$
# 将两个正则表达式连起来： ^\(.\)\(.*\)$
# 构造 mv 命令：mv \1\2 \1
$ ls -1 ??* | sed 's/^\(.\)\(.*\)$/mv \1\2 \1/'
mv apple a 
mv banana b
mv cantaloupe c
mv carrot c
...

$ ls -1 ??* | sed 's/^\(.\)\(.*\)$/mv \1\2 \1/' | less  # 检查一下输出，通过 less 按页输出
$ ls -1 ??* | sed 's/^\(.\)\(.*\)$/mv \1\2 \1/' | bash  # 确认命令正确，将输出结果传递给 bash 执行 

: '
    上述命令在重复以下模式：
    1. 通过字符串操作生成一系列命令
    2. 通过less检查结果是否正确
    3. 通过管道，将结果传给 bash
'
```

### 技巧七：利用 ssh 远程执行字符串

本技巧只适用于熟悉 ssh 的用户。

```bash
# 访问远程主机的常见方式：
$ ssh myhost.example.com
# 在 ssh 命令后追加一个字符串
$ ssh myhost.example.com ls 

$ ssh myhost.example.com ls > outfile           # 在本地主机创建输出文件
$ ssh myhost.example.com "ls > outfile"         # 在远程主机创建输出文件

# 通过管道将 ls 命令传递给 ssh，然后在远程主机上运行。
# 这种写法与将它们传递给 bash，然后在本地运行一样。
$ echo "ls > outfile" | ssh myhost.example.com 

: '
    通过管道将命令传递给 ssh 时，远程主机可能会输出诊断信息或其他消息。
'
# 如果看到有关伪终端 tty 的消息（如，"Pseudo-terminal will not be allocated because stdin is not a terminal"）
# 使用 -T 选项运行 ssh，防止远程 ssh 服务器分配终端。
$ echo "ls > outfile" | ssh -T myhost.example.com

# 一般远程主机会显示欢迎消息 "Welcome to Linux!" 
# 明确告知 ssh 在远程主机上运行 bash
$ echo "ls > outfile" | ssh myhost.example.com bash 
```

### 技巧八：通过 xargs 运行一组命令

xargs 接受两个输入：
- 标准输入：由空格分隔的字符列表（例如，ls 或 find 生成的文件路径）实际上任何字符串都可以，即输入字符串。
- 命令行：不完整的命令，缺少一些参数，我称为 "命令模板"

xargs 能够结合输入字符串和命令模板，生成并运行完整的命令。

```bash
# 假设某个目录包含三个文件：
$ ls -1
apple
banana
cantaloupe

# 通过管道将这个列表传递给 xargs，将其作为输入字符串，并制定命令模板 wc -l
$ ls -1 | xargs wc -l   
3 apple 
4 banana
1 cantaloupe
8 total

# 利用 cat 输出这三个文件
$ ls -1 | xargs cat

: '
    以上命令存在两个缺点：
    1. 如果输入字符串包含特殊字符（比如空格），xargs 就会输出错误的结果。
    2. 这里不需要 xargs，

    使用 xargs 的原因：当输入字符远比简单的目录列表更复杂时。
'
$ wc -l *
3 apple
4 banana
1 cantaloupe
8 total

: '
    假如要用递归的方式，统计某个目录及其所有子目录名称以 .py 结尾的 python 源文件的行数。
'
# 1. 利用 find 生成所有文件的路径列表：
$ find . -type f -name \*.py -print

# xargs 可以针对每个文件路径，应用命令模板 wc -l，并生成递归结果。
# 为了安全，将选项 -print 换成 -print0，将 xargs 换成 xargs -0
$ find . -type f -name \*.py -print0 | xargs -0 wc -l
6 ./fruits/raspberry.py
3 ./vegetables/leafy/lettuce.py
...

# xargs 的三个重要选项： -0 -n -I
# -n 选项：控制 xargs 可以将多少个参数附加到每个生成的命令上。默认行为是只要在 shell 的限制范围内，就可以添加任意数量的参数。
$ ls | xargs echo               # 添加任意数量的输入字符串
apple banana cantaloupe carrot  # echo apple banana cantaloupe carrot 

$ ls | xargs -n1 echo           # 每个 echo 命令一个参数
apple                           # echo apple
banana                          # echo banana
cantaloupe                      # echo cantaloupe
carrot                          # echo carrot

$ ls | xargs -n2 echo           # 每个 echo 命令两个参数
apple banana                    # echo apple banana
cantaloupe carrot               # echo cantaloupe carrot 

$ ls | xargs -n3 echo           # 每个 echo 命令三个参数 
apple banana cantaloupe         # echo apple banana cantaloupe
carrot                          # echo carrot

# -I 选项：控制输入字符出现在生成的命令中的位置。默认，它们会附加到命令模板的末尾。
# -I 后面可以是任意字符串，该字符串可以作为命令模板中的占位府，准确指示输入字符串应插入的位置。
$ ls | xargs -I XYZ is my favorite food     # 使用 XYZ 作为占位符
apple is my favorite food
banana is my favorite food
cantaloupe is my favorite food
carrot is my favorite food
```

find 与 xargs 的安全性
**find 和 xargs 结合使用时，为防止输入字符串中出现意外的特殊字符，不能单独使用 xargs，必须添加选项 xargs -0。且不要使用 find -print，务必使用 find -print0**
```bash
$ find 选项 ... -print0 | xargs -0 选项...
```
通常，xargs 的输入字符串应由空格分隔。当输入字符串本身包含其他空格（如，文件名包含空格），就会出问题。默认，xargs 会将这些空格当成输入分隔符，从而导致字符串不完整，结果错误。(如，xargs 的输入为 “prickly pear.py”，则xargs会将其视为两个输入字符串)为避免这样的问题，应使用 "**xargs -0**"，将输入分隔符改为空字符（null，ASCII码为0的字符）

利用空字符（而非换行符）分隔输入字符串：用 "**-print0**" 替代 "**-print**"
ls 命令没有利用空字符分隔其输出的选项。可使用 tr 将换行符转变为空字符：

```bash
$ ls | tr '\n' '\0' | xargs -0 ...
```

或使用下述别名，列出当前目录中的文件名和子目录名，并以空字符串分隔，传递给 xargs：`alias ls0="find . -maxdepth 1 -print0"`


参数列表过长：如果某个命令过长，导致 shell 报错，则可以考虑使用 xargs。

```bash
: '
    假设当前目录包含一百万个文件，文件名为 file1.txt ~ file1000000.txt。
'
# 尝试通过模式匹配删除这些文件，会出错：
$ rm *.txt
bash: /bin/rm: Argument list too long

# 将文件列表通过管道传给 xargs 可删除这些文件。xargs 可将这个文件列表拆分为多个 rm 命令。
$ ls | grep '\.txt$' | xargs rm
$ find . -maxdepth 1 -name \*.txt -type f -print0 \
    | xargs -0 rm
```

## 进程控制技巧

### 技巧九：后台命令

#### 启动后台命令

```bash
$ wc -c my_extremely_huge_file.txt &    # 统计一个个巨大的文件中的字符数
```


#### 挂起命令并将其发送到后台

#### 作业与作业控制

#### 常见的作业控制操作

作业控制命令

|命令|含义|
|:-:|:-:|
|bg|将当前挂起的作业移动到后台|
|bg %n|将挂起的编号为 n 的作业移动到后台（如，bg %1）|
|fg|将当前后台的作业移动到前台|
|fg %n|将编号为 n 的后台作业移动到前台（如，bg %2）|
|kill %n|终止后台编号为 n 的作业（如，kill %3）|
|jobs|浏览 shell 的作业|

```bash
# 在后台运行一个作业直到完成
$ sleep 20 &
$ jobs 
$
# 运行一个后台作业，然后拿到前台
$ sleep 20 $ 
$ fg
$
# 运行一个前台作业，挂起，然后拿到前台
$ sleep 20

$ jobs

$ fg 

# 运行一个前台作业，然后发送到后台
$ sleep 20

$ bg

$ jobs 

$ 

# 
$ sleep 100 &           # 在后台运行 3 个命令

$ sleep 200 &

$ sleep 300 & 

$ jobs                  # 列出该 shell 的作业

$ fg %2                 # 把第 2 个作业拿到前台

^Z                      # 挂起第 2 个作业

$ jobs                  # 可以看到作业 2 被挂起（"stopped"）

$ kill %3               # 终止作业 3

$ jobs                  # 作业 3 不见了

$ bg %2                 # 被挂起的作业 2 拿到后台继续执行

$ jobs                  # 可以看到作业 2 又开始运行了

$

```

#### 后台的输出与输入

```bash
$ sort /usr/share/dict/words | head -n2 &

$ sort /usr/share/dict/words | head -n2 & > /tmp/results &

$ cat /tmp/results

$ cat &

```

后台命令：

### 技巧十：显式子 shell

```bash
$ (cd /usr/local && ls)

$ pwd 

$ tar xvf package.tar.gz

$ cat package.tar.gz | (mkdir -p /tmp/other && cd /tmp/other && tar xzvf -)

$ cat czf -dir1 | (cd /tmp/dir2 && tar xvf -)

$ tar czf -dir | ssh myhost '(cd /tmp/dir2 && tar xvf -)'
```

哪些技巧会创建子 shell？



### 技巧十一：进程替换

exec 作为 shell 的一个内置命令，将正在运行的shell（一个进程）替换成你的指令（另一个进程）。该命令退出后不会显示任何 shell 提示符。

```bash
$ bash                  # 运行一个子进程 shell
$ PS1="Doomed> "        # 修改新 shell 的提示符
Doomed> echo hello      # 随便运行一个命令
hello

# 运行 exec 命令，观察这个新 shell 被销毁
Doomed> exec ls         # 子进程 shell 别替换、运行，然后退出
animals.txt
$                       # A prompt from the original (parent) shell
```

运行 exec 可能会引发致命错误：如果 shell 在终端窗口中运行，则该窗口会关闭；如果是在登陆 shell 中运行，则你会登出系统。
运行 exec 的原因：
- 为了节省资源，不需要再启动一个进程。
- 可以为当前 shell 分配新的标准输入、标准输出和标准错误。

将一些信息保存到文件 /tmp/outfile 中。
如果使用常见的输出重定向，则需要将每个指令的输出依次重定向到 /tmp/outfile 

```bash
#！/bin/bash
echo "My name is $USER"                                 >  /tmp/outfile
echo "My current directory is $PWD"                     >> /tmp/outfile
echo "Guess how many lines are in the file /etc/hosts?" >> /tmp/outfile
wc -l /etc/hosts                                        >> /tmp/outfile
echo "Goodbye for now"                                  >> /tmp/outfile
```
exec 可以让整个脚本的输出重定向到 /tmp/outfile

```bash
#!/bin/bash
# Redirect stdout for this script
exec > /tmp/outfile2
# All subsequent commands print to /tmp/outfile2
echo "My name is $USER"
echo "My current directory is $PWD"
echo "Guess how many lines are in the file /etc/hosts?"
wc -l /etc/hosts
echo "Goodbye for now"
```

```bash
# 运行脚本，检查一下文件 /tmp/outfile 的结果：
$ cat /tmp/outfile 
My name is smith
My current directory is /home/smith
Guess how many lines are in the file /etc/hosts?
122 /etc/hosts
Goodbye for now
```

命令行的常用技巧
|问题|解决方法|
|:-:|:-:|
|将某个程序的标准输出发送到另一个程序的标准输入|管道|
|将输出（标准输出）输入到一个命令|命令替换|
|将输出（标准输出）提供给一个无法读取标准输入、只能读取磁盘文件的命令|进程替换|
|将一个字符串当作命令执行|bash -c 或通过管道将字符串传递给 bash|
|在标准输出上显示多个命令，并执行这些命令|通过管道传递给 bash|
|依次执行多个相似的命令|xargs，或将命令构造成字符串并传递给 bash|
|根据其他命令的成功与否管理当前命令|条件列表|
|一次运行多个命令|后台|
|根据其他命令的成功与否，一次运行多个命令|后台条件列表|
|在远程主机上运行一个命令|运行 "ssh host命令"|
|在管道中间更改名录|显式子 shell|
|稍后运行一个命令|无条件列表：sleep 要运行的命令|
|将输入或输出重定向到受保护的文件|运行 sudo bash -c "命令，文件"|

# 构建单行命令

## 准备就绪

### 不要死板

```bash
$ echo $(ls *.jpg)
$ bash -c 'ls *.jpg'
$ cat <(ls *.jpg)
$ find . -maxdepth 1 -type f -name \*.jpg -print 
$ ls > tmp && grep '\.jpg$' tmp && rm -f tmp
$ paste <(echo ls) <(echo \*.jpg) | bash
$ bash -c 'exec $(paste <(echo ls) <(echo \*.jpg))'
$ echo 'monkey *.jpg' | sed 's/monkey/ls/' | bash 
$ python -c 'import os; os.system("ls *.jpg")'
```

### 考虑从何处入手

```bash
$ echo {A..Z}

$ echo {A..Z} | awk '{print $(17)}'

$ echo {A..Z} | sed 's/ //g' | cut -c17

$ echo {1..12}

$ echo 2021-{01..12}-01 | xargs -n1 date +%B -d

$ ls

$ ls | awk '{print "echo -n", $0, "| wc -c"}'

$ ls | awk '{print "echo -n", $0, "| wc -c"}' | bash | sort -nr | head -n1

```

### 熟练掌握测试工具

## 插入一个文件名到序列中

```bash
$ ls 
ch01.asciidoc ch03.asciidoc ch05.asciidoc ch07.asciidoc ch09.asciidoc 
ch02.asciidoc ch04.asciidoc ch06.asciidoc ch08.asciidoc ch10.asciidoc 

: '
    在第2章和第3章之间再插入一章，即将 3-10章重命名为 4-11章
'
# 1. 通过大括号输出 ch03.asciidoc ~ ch10.asciidoc:
$ seq -w 10 -1 3 | sed 's/\(.*\)/ch\1.asciidoc/'
# 2. 通过大括号输出 ch04.asciidoc ~ ch11.asciidoc:
$ seq -w 11 -1 4 | sed 's/\(.*\)/ch\1.asciidoc/'
# 3. 构造 mv 命令，需要并排输出原来的文件名和新文件名
$ paste <(seq -w 10 -1 3 | sed 's/\(.*\)/ch\1.asciidoc/') \
        <(seq -w 11 -1 4 | sed 's/\(.*\)/ch\1.asciidoc/')
    | sed 's/^/mv /'
# 4. 通过管道传给 bash
$ paste <(seq -w 10 -1 3 | sed 's/\(.*\)/ch\1.asciidoc/') \
        <(seq -w 11 -1 4 | sed 's/\(.*\)/ch\1.asciidoc/')
    | sed 's/^/mv /'
    | bash 
$ ls ch*.asciidoc

: '
    1. 生成写到标准输出上的列表，并将其作为参数传递给命令；
    2. 通过 paste 和进程替换并排输出列表；
    3. 通过 sed 将命令添加到每行的开头。也就是将每行开头的字符(^)替换为程序名称和一个空格；
    4. 将结果传递给 bash。
'
```

## 检查匹配的文件

有一个目录包含数百个图像文件和文本文件，要确认每个图像文件都有对应的文本文件，反之，每个文本文件都有对应的图像文件。

```bash
$ ls
bald_eagle.jpg blue_jay.jpg cardinal.txt robin.jpg wren.jpg
bald_eagle.txt cardinal.jpg oriole.txt   robin.txt wren.txt

# 方案1
# 先创建两个列表（JPEG文件列表和文本文件列表），通过 cut 去除它们的文件扩展名 .txt 和 .jpg
$ ls *.jpg | cut -d. -f1
$ ls *.txt | cut -d. -f1

# 接着利用进程替换和diff比较这两个列表
$ diff <(ls *.jpg | cut -d. -f1) <(ls *.txt | cut -d. -f1)
2d1
< blue_jay
3a3
> oriole

# 用 grep 去掉开头不包含小于号或大于号的行。使用 awk 为每个文件名（$2）添加正确的扩展名，至于添加.jpg 还是 .txt取决于每行开头是小于号还是大于号
$ diff <(ls *.jpg | cut -d. -f1) <(ls *.txt | cut -d. -f1) \
    | grep '^[<>]'\
    | awk '/^</{print $2 ".jpg"} /^>/{print $2 ".txt"}'
blue_jay.jpg
oriole.txt
# 如果当前目录包含一个名为 yellow.canary.jpg 的文件，该文件名包含两个点，则会出现错误。原因是：两个cut命令会删除第一个点及其后面的字符，不考虑后一个点。
blue_jay.jpg
oriole.txt
yellow.jpg

# 将 cut 换成 sed，删除最后一个点到字符串末尾的字符：
$ diff <(ls *.jpg | sed 's/\.[^.]*$//') \
       <(ls *.txt | sed 's/\.[^.]*$//') \
    | grep '^[<>]'\
    | awk '/^</{print $2 ".jpg"} /^>/{print $2 ".txt"}'
```

```bash
# 方案2：将两个列表合并，去除匹配的文件对。
# 1. 使用 sed 删除文件的扩展名，用 uniq -c 统计每个字符串的出现次数：
# 数字2代表一对匹配的文件名，1代表一个部匹配的文件名
$ ls *.{jpg, txt} \
    | sed 's/\.[^.]*$//' \
    | uniq -c
    2 bald_eagle
    1 blue_jay
    2 cardinal
    1 oriole
    2 robin
    2 wren
    1 yellow.canary

# 2. 使用 awk 提取以空格和1开头的行，只输出第二个字段
$ ls *.{jpg, txt} \
    | sed 's/\.[^.]*$//' \
    | uniq -c \
    | awk '/^ *1 /{print $2}'
blue_jay
oriole
yellow.canary

# 3. 添加文件扩展名：通过命令替换将输出发送到 ls，shell 会执行模式匹配，由 ls 列出不匹配的文件名。
$ ls -1 $(*.{jpg, txt} \
    | sed 's/\.[^.]*$//' \
    | uniq -c \
    | awk '/^ *1 /{print $2 "*"}')
```

## 生成主目录的 CDPATH

编写一个单行命令，自动生成 CDPATH （这个单行命令可以插入到 bash 的配置文件中）

```bash
# 1. 查看 $HOME 的子目录列表：使用一个子shell，防止 cd 命令改变 shell 的当前目录。
$ (cd && ls -d */)
Family/     Finances/       Linux/      Music/      Work/

# 2. 利用 sed 将 "$HOME/" 添加到每个目录的前面：
$ (cd && ls -d */) | sed 's/^/$HOME\//g'
$HOME/Family/
$HOME/Finances/
$HOME/Linux/
$HOME/Music/
$HOME/Work/

# 替换字符串 "$HOME/" 包含了一个斜杠，sed 替换的分隔符也是斜杠，故在斜杠前使用了转义：$HOME\/
# 使用 @ 替换斜杠，就不需要转义了。
$ (cd && ls -d */) | sed 's@^@$HOME/@g'
$HOME/Family/
$HOME/Finances/
$HOME/Linux/
$HOME/Music/
$HOME/Work/

# 加入一个 sed 表达式，去掉最后的斜杠
$ (cd && ls -d */) | sed -e 's@^@$HOME/@g' -e 's@/$@@'

# 使用 echo 和命令替换输出一行结果
# 不需要通过在 cd 和 ls 周围加括号的方式显式创建子shell，因为命令替换会创建子shell。
$ echo $(cd && ls -d */) | sed -e 's@^@$HOME/@g' -e 's@/$@@'

# 添加一个目录 $HOME 和最后的相对目录 ..
$ echo '$HOME' \
       $(cd && ls -d */) | sed -e 's@^@$HOME/@g' -e 's@/$@@' \
       ..

# 将空格改为冒号，将所有输出通过管道传递给 tr
$ echo '$HOME' \
       $(cd && ls -d */) | sed -e 's@^@$HOME/@g' -e 's@/$@@' \
       .. \
    | tr ' ' ':'

# 添加 CDPATH 环境变量。
# 可将这个变量定义复制到 bash 的配置文件，方便随时生成 CDPATH。
$ echo 'CDPATH=$HOME' \
       $(cd && ls -d */) | sed -e 's@^@$HOME/@g' -e 's@/$@@' \
       .. \
    | tr ' ' ':'  
```

## 生成测试文件

生成一千个文件，其中包含可用于软件测试的随机文本。
解决方案：从一个大型文本文件中随机挑选单词，然后创建内容和长度都随机的一千个小文件。系统字典 /usr/share/dict/words 是一个完美的源文件，其中包含 104,334 个单词，一个单词一行。

```bash
$ wc -l /usr/share/dict/words
104334 /usr/share/dict/words 
```

构建这样一个单行命令，需要解决四个难题：
- 随机打乱字典文件
- 从字典文件中随机选择几行
- 创建一个输出文件来保存结果
- 运行一千次这种方案

```bash
# 使用 shuf 打乱字典顺序，每次运行产生十万多行输出，通过 head 提取前几行来确认打乱结果。
$ shuf /usr/share/dict/words | head -n3
pans
dreads
programs
$ shuf /usr/share/dict/words | head -n3
anatomically
Fafnir
inflatables

# 从打乱的字段中随机选取几行
# bash 的一个变量 $RANDOM 可提供 0-32767 之间的随机整数
# 生成行数随机的输出，通过管道将结果传给 wc -l 确认每次运行命令产生的输出行数
$ shuf -n $RANDOM /usr/share/dict/words | wc -l

# 产生一千个不同的文件名
$ pwgen
# 生成一个字符串，长度 10
$ pwgen -N1 10
# 使用命令替换，生成更像文本文件名的字符串
$ echo $(pwgen -N1 10).txt

# 通过 shuf -o 讲输出保存到文件中
$ mkdir -p /tmp/randomfiles && cd /tmp/randomfiles
$ shuf -n $RANDOM -o $(pwgen -N1 10).txt /usr/share/dict/words

# 检查结果
$ ls                            # 列出新文件
Ahxiedie2f.txt
$ wc -l Ahxiedie2f.txt          # 包含多少行
13544 Ahxiedie2f.txt
$head -n3 Ahxiedie2f.txt        # 查看前几行
saviors
guerillas
forecaster

# 运行一千次上述 shuf 命令，可以使用循环
for i in {1..1000}; do
    shuf -n $RANDOM -o $(pwgen -N1 10).txt /usr/share/dict/words
done 

```

```bash
# 可以将命令打包成字符串，然后通过管道传递给 bash。
# 1. 使用 echo 输出一次我们的命令，加上单引号，确保 $RANDOM 不后被替换，且 pwgen 不会运行
$ echo 'shuf -n $RANDOM -o $(pwgen -N1 10).txt /usr/share/dict/words'
# 2. 将其传给 bash
$ echo 'shuf -n $RANDOM -o $(pwgen -N1 10).txt /usr/share/dict/words' | bash
$ ls 
eiFohpies1.txt
# 3. 通过管道将 yes 的输出传递给 head，并让 head 输出一千次我们的命令，将结果传给 bash 
$ yes 'shuf -n $RANDOM -o $(pwgen -N1 10).txt /usr/share/dict/words' \
    | head -n 1000 \
    | bash
$ ls 
aagheSei1w.txt  Chiegaph7w.txt  Hab2noh8le.txt  Nee3fu7nas.txt  rohGheeB3c.txt
aashiL3Eey.txt  choh5oe4Qu.txt  haC3aix2uu.txt  nee8Ahquoh.txt  rohX2eesae.txt
Aathex8eoh.txt  chohl1nu6H.txt  HaeWi9ahSh.txt  neiDeem0Le.txt  roo0pi3OhL.txt

```

```bash
# 生成一千个随机图像文件：运行图形包 ImageMagick 的 convert 命令，生成由彩色方块组成的，大小为 100x100像素的随机图像：
$ yes 'convert -size 8x8 xc: +noise Random -scale 100x100 $(pwgen -N1 10).png' \
    | head -n 1000 \
    | bash
$ ls
$ display hohti3Aeze.png
```

## 生成空文件

生成一千个名为 file0001.txt ~ file1000.txt 的空文件

```bash
$ mkdir /tmp/empties            # 创建文件的目录
$ cd /tmp/empties
$ touch file(01...1000).txt     # 生成文件
```

通过系统字典随机获取字符串，生成意义的文件名

```bash
# 使用 grep 限制文件名仅为小写字母（避免空格、单引号、其他字符） 
$ grep '^[a-z]*$' /usr/share/dict/words

# 通过 shuf 打乱文件名，再利用 head 输出前一千个
$ grep '^[a-z]*$' /usr/share/dict/words | shuf | head -n1000

# 将结果通过管道传递给 xargs，并使用 touch 创建文件
$ grep '^[a-z]*$' /usr/share/dict/words | shuf | head -n1000 | xargs touch 
$ ls
abetted          diet             lancets           renegotiate
ably             diggers          landfills         renumber
abrasions        digitization     lanolin           replicated
abrogations      dinettes         lash              reprehensibly
...

```

# 处理文本文件

```bash
# 按字母顺序输出所有的用户名（即第一个字段）
$ cut -d: -f1 /etc/passwd | sort 
```

根据数字用户ID区分人类用户与系统用户，并向人类用户发送欢迎邮件。

```bash
# 1. 当数字用户ID（字段3）大于等于 1000 时，使用 awk 输出用户名（字段1）
$ awk -F: '$3>=1000 {print $1}' /etc/passwd

# 2. 通过管道用 xargs 生成欢迎消息
$ awk -F: '$3>=1000 {print $1}' /etc/passwd \
  | xargs -I@ echo "Hi there, @!"

# 3. 生成将欢迎消息传递给 mail 命令的命令（字符串），由 mail 命令将指定标题(-s)的电子邮件发送给指定的用户
$ awk -F: '$3>=1000 {print $1}' /etc/passwd \
  | xargs -I@ echo 'echo "Hi there, @!" | mail -s greeting @'

# 4. 将生成的命令通过管道传递给 bash，就可以发送电子邮件：
$ awk -F: '$3>=1000 {print $1}' /etc/passwd \
  | xargs -I@ echo 'echo "Hi there, @!" | mail -s greeting @' \
  | bash
```

设计能够与 Linux 命令完美配合的文本文件：

1. 搞清要解决的业务问题，涉及的数据；
2. 按照合适的格式，将数据存储在文本文件中；
3. 构建处理文件的Linux命令；
4. （可选）将这些命令保存到脚本、别名或函数中，方便日后使用。

## 示例：查找文件

假设主目录包含几万个文件和子目录，很难记住文件放在哪里。

```bash
# 用 find 找到 animals.txt，但是 find 很慢。
$ find $HOME -name animals.txt -print
```

第一步，搞清楚设计数据的业务问题：根据名称快速查找主目录的文件。
第二步，按照合适的格式，将数据存储在文本文件中。

运行一次 find 命令，构建所有文件和目录的列表，每行一个文件路径，并将结果保存在一个隐藏文件中。

```bash
$ find $HOME -print > $HOME/.ALLFILES
$ head -n3 $HOME/.ALLFILES
```

第三步，构建 Linux 命令来加速文件搜索，需要使用 grep。grep 遍历一个大型文件的速度比在一个大型目录树中运行 find 快很多。

```bash
$ grep animals.txt $HOME/.ALLFILES
```

第四步，保存命令，以便运行。
编写一个名为 ff 的单行脚本，主要任务是运行 grep，同时接受用户提供的选项和搜索字符串。

```bash
# 示例 9-1 脚本 ff
#！/bin/bash
# $@ means all arguments provided to the script 
grep "$@" $HOME/.ALLFILES
```

赋予脚本执行的权限，并将其放入搜索路径中的任意目录下，如个人的 bin 子目录下：

```bash
$ chmod +x ff
$ echo $PATH

$ mv ff ~/bin
```

运行 ff 脚本。

```bash
$ ff anima                  # 快速定位文件
$ ff -i animal | less       # 不区分大小的 grep
$ ff -i animal | wc -l      # 有多少个匹配
```

为了更新文件列表，应隔一段时间运行一次 find 命令。（还可以用 cron 创建计划作业）

## 检查域名的有效期限

1. 确定业务问题：假设你拥有多个互联网的域名，要看看它们什么时候过期。
2. 创建一个包含这些域名的文件。如 domain.txt，每行一个域名：

```txt
example.com
oreilly.com
efficientlinux.com
...
```

3. 创建一个命令，输出截止日期。首先是 whois （向域名注册商查询有关域名的信息）

```bash
$ whois example.com | less

# 表示截止日期的字符以 "Registry Expiry Date" 开头，可以使用 grep 和 awk 将其分离出来。
$ whois example.com | grep 'Registry Expiry Date:'
$ whois example.com | grep 'Registry Expiry Date:' | awk '{print $4}'

# 然后，通过命令 date --date 将日期字符串转换成更加方便阅读的格式：
$ date --date 2028-10-11T11:05:17Z
$ date --date 2028-10-11T11:05:17Z +'%Y-%m-%d'

# 使用命令替换将 whois 输出的日期字符串传递给 date 命令
$ echo $(whois example.com | grep 'Registry Expiry Date:' | awk '{print $4}')

$ date \
    -- date $(whois example.com \
            | grep 'Registry Expiry Date:' \
            | awk '{print $4}') \
    +'%Y-%m-%d'*

# 这样，查询域名注册商并输出截止日期的命令就创建好了
```

创建一个脚本名为 check-expiry 来运行上述命令并输出截止日期。

```bash
# 9-2 脚本 check-expiry
#!/bin/bash
expdate=$(date \
            --date $(whois "$1" \
                    | grep 'Registry Expiry Date:' \
                    | awk '{print $4}') \
            +'%Y-%m-%d')
echo "$expdate  $1"         # Two values separated by a tab
```

运行 check-expiry

```bash
$ ./check-expiry example.com
```

利用循环检查文件 domain.txt 中所有的域名，创建一个名为 check-expiry-all 的脚本

```bash
# 9-3 脚本 check-expiry-all
#!/bin/bash
cat domains.txt | while read domain; do
    ./check-expiry "$domain"
    sleep 5             # Be kind to the register's server 
done 
```

将所有输出重定向到一个文件 expiry.txt

```bash
$ ./check-expiry-all &> expiry.txt &
# 脚本运行完成，可以检查文件 expiry.txt 中包含的信息
$ cat expiry.txt

# 对日期进行排序，并找出下一个需要续订的域名
$ sort -n expiry.txt | head -n1

# 使用 awk 查找已过期或今天就要过期的域名，即截止日期（字段1）小于或等于今天
$ awk "\$1<=\"$(date +%Y-%m-%d)\"" expiry.txt

```

关于命令 awk，注意：

- 在日期字符串周围添加了双引号和美元符号，因此 shell 不会在运行 awk 之前替换字符串；
- 使用了字符串运算符 <= 来比较日期。这只是字符串比较，但它可以正确执行。因为日期格式 YYYY-MM-DD 按时间顺序排列后的结果与字母顺序是一致的。

## 建立区号数据库

文件名 areacodes.txt，其中包含美国的电话号码区号。
如下：
201	NJ	Hackensack, Jersey City (201/551 overlay)
202	DC	Washington
203	CT	New Haven, Stamford, southwestern (475 will overlay 203)
204	MB	entire province
205	AL	Birmingham, Tuscaloosa (659 will overlay 205)
.
.
.
989 MI Saginaw

```bash
# 使用 grep 按州查找区号，加上选项 -w 只匹配完整的单词
$ grep -w NJ areacodes.txt

# 按照区号查找城市
$ grep -w 202 areacodes.txt

# 按照文件中任意字符串查找城市
$ grep Washing areacodes.txt

# 利用 wc 统计区号的数量
$ wc -l areacodes.txt

# 查找区号最多的州 [获胜者是加利福尼亚州，缩写CA, 一共38个区号]
$ cut -f2 areacodes.txt | sort | uniq -c | sort -nr | head -n1

# 将文件转换成 CSV 格式，导入电子表格应用程序。输出第三个字段，这里加上双引号，防止文本中的逗号被当成 CSV 的分隔符
$ awk -F'\t' '{printf "%s,%s,\"%s\"\n", $1, $2, $3}' areacodes.txt \> areacodes.csv
$ head -n3 areacodes.csv

# 输出某一个州的所有区号
$ awk '$2~/^NJ$/{ac=ac FS $1} END {print "NJ:" ac}' areacodes.txt

# 或使用数组和 for 循环输出每一个州的区号
$ awk '{arr[$2]=arr[$2] " " $1} \
        END {for (i in arr) print i ":" arr[i]}' areacodes.txt \
    | sort 

# 将上述命令转换为别名、函数或脚本
# 9-4 脚本 areacode
#！/bin/bash
if [ -n "$1" ]; then 
    grep -iw "$1" areacodes.txt
fi

# 脚本 areacode 的功能：搜索文件 areacode.txt 中的任意单词，例如区号、州缩写或城市名称
```

## 构建密码管理器

密码文件名为 vault，包含三个字段，以制表符分隔：

- 用户名
- 密码
- 备注（任意文本）

1. 创建文件 vault，并添加数据。

```bash
$ touch vault 
$ chmod 600 vault 
$ emacs vault 
$ cat vault 
```

2. 将文件 vault 保存到：

```bash
$ mkdir ~/etc
$ mv vault ~/etc
```

3. **基本思路：通过模式匹配程序（grep、awk等）输出与指定字符串相匹配的数据。**

```bash
$ cd ~/etc
$ grep sally vault      # 匹配用户名
$ grep work vault       # 匹配备注
$ grep drop vault       # 匹配多行
```

A. 将这个功能保存到脚本中，并逐步改进。
创建脚本 pman

```bash
# pman
#!/bin/bash
# Just print matching lines 
grep "$1 $HOME/etc/vault"
```

将这个脚本添加到搜索路径中

```bash
$ chmod 700 pman
$ mv pman ~/bin
```

试运行这个脚本

```bash
$ pman goog
$ pman account 
$ pman facebook
```

B. pman 的第二个版本：添加了错误检查，定义了一些便于记忆的变量名

```bash
#!/bin/bash
# Capture the script name.
# $0 is the path to the script, and basename prints the final filename.
PROGRAM=$(basename $0)
# Location of the password vault
DATABASE=$HOME/etc/vault

# Ensure that at least one argument was provided to the scipt.
# The expression >&2 directs echo to print on stderr instead of stdout.
if [ $# -ne 1 ]; then 
    >&2 echo "$PROGRAM: loop up passwords by string"
    >&2 echo "Usage: $PROGRAM string"
    exit 1
fi
# Store the first argument in a fiendly, named variable
searchstring="$1"

# Search the vault and print an error message if nothing matches
grep "$searchstring" "$DATABASE"
if [ $? -ne 0 ]; then 
    >&2 echo "$PROGRAM: no matches for '$searchstring'"
    exit 1
fi
```

运行脚本：

```bash
$ pman
$ pman smith
$ pman xyzzy
```

这个技巧不利于扩展。如果文件 vault 包含几百行数据，并通过 grep 输出了 63 行匹配的数据，不便于找到所需的密码。

C. 改进脚本，在每一行的第三列添加一个唯一键（字符串），并优先搜索这个唯一键。

```bash
# 9-7 第三版 pman：优先搜索第三列中的键
#!/bin/bash
PROGRAM=$(basename $0)
DATABASE=$HOME/etc/vault

if [ $# -ne 1 ]; then
    >&2 echo "$PROGRAM: loop up passwords"
    >&2 echo "Usage: $PROGRAM string"
    exit 1
fi
searchstring="$1"

# Look for exact matches in the third column
match=$(awk '$3~/^'$searchstring'$/' "$DATABASE")

# If the search string doesn't match a key, find all matches 
if [ -z "$match" ]; then
    match=$(awk "/$searchstring/" "$DATABASE")
fi

# If still no match, print an error message and exit
if [ -z "$match" ]; then 
    >&2 echo "$PROGRAM: no matches for '$searchstring'"
    exit 1
fi

# Print the match 
echo "$match"
```

运行脚本：

```bash
$ pman dropbox
$ pman drop
```

D. 因纯文本文件存在安全风险，故使用标准的Linux加密程序 GnuPG 对其进行加密。

设置 Gunpg

```bash
$ gpg --quick-generate-key your_email_address default default never 

# 系统会提示输入密码两次。当运行完成后，可使用公钥加密密码文件，生成文件 vault.pgp
$ cd ~/etc
$ gpg -e -r your_email_address vault
$ ls vault*

# 测试一下，解密文件 vault.gpg 并将其内容显示在标准输出中
$ gpg -d -q vault.gpg 

```

修改脚本，将读取的文件从纯文本 vault 改为经过加密的 vault.gpg。解密 vault.gpg 并将标准输出通过管道传递给 awk，再进行模式匹配。

```bash
# 9-8 第四版 pman：使用加密的 vault
#!/bin/bash
PROGRAM=$(basename $0)
# Use the encrypted file 
DATABASE=$HOME/etc/vault.gpg

if [ $# -ne 1 ]; then
    >&2 echo "$PROGRAM: loop up passwords"
    >&2 echo "Usage: $PROGRAM string"
    exit 1
fi
searchstring="$1"

# Store the decrpted text in a variable 
decrpted=$(gpg -d -q "$DATABASE")

# Look for exact matches in the third column
match=$(echo "$decryped" | awk '$3~/^'$searchstring'$/')

# If the search string doesn't match a key, find all matches 
if [ -z "$match" ]; then
    match=$(echo "$decryped" | awk "/$searchstring/")
fi

# If still no match, print an error message and exit
if [ -z "$match" ]; then 
    >&2 echo "$PROGRAM: no matches for '$searchstring'"
    exit 1
fi

# Print the match 
echo "$match"
```

这个脚本显示的密码来自加密后的文件

```bash
$ pman dropbox
$ pman drop
```

密码文件支持注释（以#开头），为此需要更新脚本，将解密后的数据通过管道传递给 grep -v，过滤掉以 # 开头的行。

```bash
decrypted=$(gpg -d -q "$DATABASE" | grep -v '^#')
```

## 编辑加密文件

要修改加密文件，最直接、繁琐、不安全的方法是解密文件、编辑，然后再加密。

```bash
$ cd ~/etc
$ gpg vault.gpg
$ emacs vault 
$ gpg -e -r your_email_address vault 
$ rm vault 
```

emacs 和 vim 都提供了编辑 GnuPG 加密文件的模式

1. 将如下设置添加到 bash 的配置文件中，然后在 shell 中用 source 命令执行该设置

```bash
export GPG_TTY=$(tty)
```

emacs 拥有内置的 EasyPG 包。只需将如下设置添加到配置文件 $HOME/.emacs，并重启 emacs 即可。需要将如下命令中的字符串 "GnuPG ID here" 替换成与密钥关联的电子邮件地址。

```bash
(load-library "pinentry")
(setq epa-pinentry-mode 'loopback')
(setq epa-file-encrypt-to "GunPG ID here")
(pinentry-start)
```

2. 编辑加密文件时，emacs 会提示输入密码，并将解密后的文件保存到缓冲区，方便编辑。在编辑完毕，保存文件时，emacs 会自动加密缓冲区的内容。（若使用 vim，可选择插件 [vim-gnupg](https://github.com/jamessan/vim-gnupg)，并将如下设置添加到配置文件 $HOME/.vimrc）

```bash
let g:GPGreferArmor=1
let g:GPGDefaultRecipients=["GunPG ID here"]
```

创建一个别名来编辑密码文件：

```bash
alias pwedit="$EDITOR $HOME/etc/vault.gpg"
```
