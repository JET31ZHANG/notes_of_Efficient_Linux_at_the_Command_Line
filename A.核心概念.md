
[下载地址](https://resources.oreilly.com/examples/0636920601098)

# 组合命令

Starting with just 6 Linux Commands, you'll rapidly build multi-stage pipelines that solve diverse business problems.

如何排列组合多个命令，完成操作。

## 输入、输出与管道

- stdin
- stdout
- |

## 6个命令

### 命令一：wc

```bash
# 输出文件的行数、单词数和字符数
$ wc animals.txt
# 仅输出行数
$ wc -l animals.txt
# 仅输出单词数
$ wc -w animals.txt
# 仅输出字符数
$ wc -c animals.txt
# 当前目录中共有多少个可见文件
$ ls -1 | wc -l
# wc 输出中共有 4 个单词：3个整数和1个文件名
$ wc animals.txt | wc -w
# 计算出第二个 wc 的输出（"4"）中的行数、单词数、字符数
$ wc animals.txt | wc -w | wc       # 2 包含了1个末尾的换行符
```

```bash
# 当标准输出为屏幕时，ls 会输出排列成多列，方便阅读
$ ls /bin
# 当标准输出被重定向时，ls 只有一列输出。
$ ls /bin | cat 

# ls 的一些强制选项：
# 1. -1 强制 ls 将结果输出到一行上
# 2. -C 强制 ls 输出多列结果
```

### 命令二：head

```bash
# 输出 animals.txt 的前 3 行
$ head -n3 animals.txt
# 输出 animals.txt 的前 10 行
$ head animals.txt
# 统计 animals.txt 前 3 行的字数
$ head -n3 animals.txt | wc -w
# 列出目录 /bin 中的前 5 个文件名
$ ls /bin | head -n5
```

### 命令三：cut

```bash
# 输出文件 animals.txt 中的所有书名（书名出现在第2列）
$ cut -f2 animals.txt
# 输出文件 animals.txt 中的前 3 行书名
$ cut -f2 anmals.txt | head -n3
# 在之前基础上，剪切多个字段 
$ cut -f1,3 animals.txt | head -n3
# 指定数字范围
$ cut -f2-4 animals.txt | head -n3
# 输出 animals.txt 文件每一行的前三个字符（可通过逗号指定（1,2,3）或指定范围（1-3））
$ cut -c1-3 animals.txt
# 提取 animals.txt 作者的姓氏：剪切出第4个字段，通过管道使用选项 -d 将分割符从制表符改为逗号
$ cut -f4 animals.txt | cut -d, -f1
```

### 命令四：grep （输出包含指定字符串的行）

```bash
# 输出 animals.txt 中包含字符串 "Nutshell" 的每一行文本
$ grep Nutshell animals.txt
# 通过 -v 选项输出不包含指定字符串的文本行
$ grep -v Nutshell animals.txt
# 输出名称以.txt 结尾的文件中包含字符串"Perl"的文本行
$ grep Perl *.txt
# 目录 usr 中有多少个子目录：（ls -l 每一行开头的标记"d"表示目录）
# 使用 cut 剪切出第一列；使用 grep d 只保留包含 "d" 的文本行；通过 wc -l 统计行数
$ ls -l /usr | cut -c1 | grep d | wc -l
```

### 命令五：sort

```bash
# 按照升序排列输出结果
$ sort animals.txt
# 按照降序排列输出结果
$ sort -r animals.txt
# sort 可按字母顺序（默认）或数字顺序（选项 -n）排列输出结果
# 剪切 animals.txt 第3个字段，并按升序排列
$ cut -f3 animals.txt | sort -n
# 剪切 animals.txt 第3个字段，并按降序排序
$ cut -f3 animals.txt | sort -nr
# animals.txt 最新的一本书的出版年份
$ cut -f3 animals.txt | sort -nr | head -n1

# 输出的是最大值
$ ... | sort -nr | head -n1
# 输出的是最小值
$ ... | sort -n | head -n1

# /etc/passwd 按字母顺序排列所有用户，查看前5行
$ head -n5 /etc/passwd
# 截取用户名
$ head -n5 /etc/passwd | cut -d: -f1
# 排序
$ head -n5 /etc/passwd | cut -d: -f1 | sort
# 检查系统中是否有 lp 的账号（grep -w 只输出整个单词匹配的用户名，部分匹配的单词不算在内）
$ head -n5 /etc/passwd | grep -w lp 
```

### 命令六：uniq （检查文件中相邻且重复的文本行）

```bash
# 去除 letters 的重复数据
$ uniq letters
# 统计各个字母的出现次数
$ uniq -c letters

# grades 里哪个级别的学生最多：使用 cut 分割成绩，然后排序
$ cut -f1 grades | sort
# 使用 uniq 统计相邻行数
$ cut -f1 grades | sort | uniq -c
# 按结果的数字倒序排序，人数最多的级别显示在第一行
$ cut -f1 grades | sort | uniq -c | sort -nr
# 只输出第一行结果
$ cut -f1 grades | sort | uniq -c | sort -nr | head -n1
# 只要成绩，不要人数
$ cut -f1 grades | sort | uniq -c | sort -nr | head -n1 | cut -c9
```

## 检测重复文件

```bash
# 目录 detecting_duplicate_files 内的文件是否有重复文件？
# 计算文件校验和，使用 cut 截取每一行的前 32 个字符，通过 sort 将重复的行防盗相邻的位置上
$ md5sum *.jpg | cut -c1-32 | sort
# 通过 uniq 统计重复数据
$ md5sum *.jpg | cut -c1-32 | sort | uniq -c 
# 按数字顺序，从高到低排列结果
$ md5sum *.jpg | cut -c1-32 | sort | uniq -c | sort -nr
# 去掉没有重复的文件
$ md5sum *.jpg | cut -c1-32 | sort | uniq -c | sort -nr | grep -v "      1 "
# 使用 grep 查找具有指定校验和的文件
$ md5sum *.jpg | grep 146b163929b6533f02e91bdf21cb9563
# 使用 cut 清理输出
$ md5sum *.jpg | grep 146b163929b6533f02e91bdf21cb9563 | cut -c35-
```

# Shell 简介

When you type a command an press Enter, some parts of the command are the responsibility of the shell, and some are the responsibility of the program it invokes. Knowing which is which will make you a wiser Linux user.

## Shell 的含义

## 文件名的模式匹配

```bash
# 在 100 个文件中搜索单词 "Linux" （假设这些文件名分别为 chapter1 ~ chapter100）
$ grep Linux chapter1 chapter2 chapter3 chapter4 chapter5 ...
# shell 将模式 "chapter*" 扩展成一个列表，其中包含 100 个匹配的文件名。接着，shell 会运行 grep。
$ grep Linux chapter*
# 只在 chapter1 ~ chapter9 中搜索单词 "Linux"
$ grep Linux chapter?
# 表示搜索 chapter10 ~ chapter99
$ grep Linux chapter??
# 搜索 chapter1 ~ chapter5
$ grep Linux chapter[12345]
# 搜索 chapter1 ~ chapter5 （用 "-" 表示范围）
$ grep Linux chapter[1-5]
# 搜索偶数章（第2、4、6、8、... 章）
$ grep Linux chapter*[02468]
# 匹配所有以大写字母开头，包含一个下划线、且以符号@结尾的文件名
$ ls [A-Z]*_*@
```

```bash
# 利用模式匹配查找目录 /etc 中所有名称以 ".conf" 结尾的文件
$ ls -1 /etc/*.conf
```

shell 的模式匹配仅适用于文件和目录路径。不能用它指定用户名，主机名及其他类型的参数。

## 变量求值

```bash
# 通过命令 printenv，在标准输出中显示变量 HOME 和 USER 的值
$ printenv HOME
$ printenv USER
# 对变量求值时，shell 将变量名替换成变量的值。只需在变量名前加一个 $，就可以对变量进行求值。
$ echo My name is $USER and my files are in $HOME
```

### 变量来自何方
USER 和 HOME 是 shell 预定义的变量。它们的值是在用户登陆时自动设置的（还有很多变量是在用户登陆后赋值）。按照习惯，一般这类预定义的变量名都会大写。

```bash
# 将目录赋给一个变量
$ work=$HOME/Projects
$ cd $work
$ pwd 

# 某个命令需要一个目录作为参数，可指定 $work
$ cp myfile $work
$ ls $work

# 注意：在定义变量时，等好两边不能有空格；否则，shell 会认为命令行的第一个单词是需要运行的程序，而等号和值是该程序的参数，便会返回错误的消息。
```

### 变量及其背后的神秘逻辑

```bash
# shell 在运行 echo 前，求出 $HOME 的值
$ echo $HOME
# 在 echo 看来，输入的是如下命令
$ echo /home/zm
```

### 模式与变量

```bash
# 假设某个目录包含两个子目录：mammals 和 reptiles，而子目录 mammals 包含两个文件：lizard.txt 和 snake.txt
$ ls 
mammals     reptiles
$ ls mammals
lizard.txt  snake.txt 

# 移动 lizard.txt 和 snake.txt 到子目录 reptiles 
# 方法1：
$ mv mammals/*.txt reptiles
# 以上语句等价于：
$ echo mammals/*.txt 	
mammals/lizard.txt mammals/snake.txt
$ mv mammals/lizard.txt mammals/snake.txt reptiles

# 方法2：使用变量，shell 会将他们直接替换成值，并不会因为它们是文件路径而做特殊处理：
FILES="lizard.txt snake.txt"
mv mammals/$FILES reptiles
# 以上语句等价于：
$ echo mammals/$FILES						
mammals/lizard.txt snake.txt
$ mv mammals/lizard.txt snake.txt reptiles

```

## 利用别名简化命令

```bash
$ alias g=grep      # 不带参数的命令
$ alias ll="ls -l"  # 带有参数的命令，注意引号不可省略
```

**别名若与某个已有的命令同名，则会取代 shell 中的命令，即 "覆盖" 命令。**

```bash
# 查看 shell 中所有的别名及其值
$ alias

# 查看某个别名的值（alias 别名名称）
$ alias g

# 删除 shell 中某个别名
$ unalias g
```

## 重定向输入与输出

shell 将标准输出重定向到一个文件。

```bash
# 输出文件 animals.txt 匹配的文本行，
$ grep Perl animals.txt

# 利用 shell 的输出重定向，将以上输出发送到一个文件中。
# 如果 outfile 不存在，就新建一个。如果存在，则重定向会覆盖文件内容。
$ grep Perl animals.txt > outfile   
$ cat outfile

# 将输出添加到文件末尾，需要使用 >>
$ grep Perl animals.txt > outfile               # 创建或覆盖 outfile
$ echo There was just one match >> outfile      # 添加到 outfile 末尾
$ cat outfile
```

输入重定向：将标准输入从键盘重定向到某个文件。重定向标准输入需要使用 <，再指定文件名。

```bash
$ wc animals.txt        # 从指定的文件中读取输入：wc 通过参数接收文件 animals.txt，所以 wc 知道该文件存在。
$ wc < animals.txt      # 读取重定向后的标准输入：wc 调用不带任何参数。wc 读取标准输入即键盘，但 shell 将标准输入重定向到文件 animals.txt。所以 wc 并不知道文件 animals.txt 的存在。

# 同一个命令同时重定向输入和输出
$ wc < animals.txt > count
$ cat count 

# grep 读取重定向的标准输入，通过管道将结果传递给 wc，
# 由 wc 写入重定向的标准输出，产生 count 文件
$ grep Perl < animals.txt | wc > count 
$ cat count 
```

标准错误（stderr）与重定向
标准错误用于接收错误消息。标准错误和标准输出的结果是一样的，但内部机制完全不同。**可以通过符号 "2>" 后接文件名的形式，重定向标准错误。**

```bash
$ cp noneistent.txt file.txt 2> errors
$ cat errors
cp: cannot stat 'noneistent.txt': No such file or directory

# 将标准错误添加到文件末尾，则需要指定 "2>>" 以及文件名
$ cp noneistent.txt file.txt 2>> errors
$ cp another.txt file.txt 2>> errors
$ cat errors
cp: cannot stat 'noneistent.txt': No such file or directory
cp: cannot stat 'another.txt': No such file or directory

# 将标准输出和标准错误同时重定向到同一个文件，需要定义 "&>" 以及文件名
$ echo This file exists > goodfile.txt              # 创建一个文件
$ cat goodfile.txt nonexistent.txt &> all.output
$ cat all.output
This file exists
cat: nonexistent.txt: No such file or directory
```

## 利用引用和转义阻止 shell 计算

文件名有空格，强制shell将空格当成文件名的一部分的方法：

- 使用单引号(')：单引号内的字符就是其字面意义，即使在 shell 中由特殊含义的字符也会被当成普通字符处理。

```bash
$ cat "Efficient Linux Tips.txt"
```

- 使用双引号(")：双引号内的字符就是其字面意义，但 $ 和其他几个特殊字符除外。

```bash
$ cat 'Efficient Linux Tips.txt'
```

- 使用反斜杠(\)：告知 shell 下一个字符应当采用其字面意义。

```bash
cat Efficient\ Linux\ Tips.txt
```

即使在双引号内，反斜杠也可以转义字符。

```bash
$ echo "The value of \$HOME is $HOME"
The value of $HOME is /home/frankcheung
```

**在单引号内反斜杠不起作用。可以在双引号内转义双引号字符**

```bash
$ echo 'The value of \$HOME is $HOME'
The value of \$HOME is $HOME

$ echo "This message \"sort of\" interesting"
This message "sort of" interesting
```

**行末的反斜杠可以阻止不可见的换行符**，shell 可以跨多行书写

```bash
$ echo "This is a very long message that needs to extend \
onto multiple lines"
This is a very long message that needs to extend onto multiple lines
```

```bash
$ cut -f1 grades | sort | uniq -c | sort -nr | head -n1 | cut -c9 
# 提高管道命令的可读性
$ cut -f1 grades \
    | sort \
    | uniq -c \
    | sort -nr \
    | head -n1 \
    | cut -c9
```

别名前面加上一个反斜杠表示转义该别名，即 shell 会寻找同名的命令，无视别名的覆盖作用

```bash
$ alias less="less -c"      # 定义一个别名
$ less myfile               # 运行别名，实际上调用的是 less -c
$ \less myfile              # 运行标准的 less 命令，而非别名
```

## 查找程序

搜索路径：一个预设目录列表，它保存在 shell 变量 "PATH" 中：

```bash
$ echo $PATH

# 搜索路径中的目录由冒号(:)分隔
# 为了看得更清楚，用管道将上述输出传递给 tr 命令，将这些冒号转换成换行符
# tr 命令：将一个字符转换成另一个
$ echo $PATH | tr : "\n"
```

查找某个程序:

- 使用 ls

```bash
$ ls ls
```

- 使用 which

```bash
$ which ls
```

- 使用 type

```bash
$ type ls
```


## 环境与初始化文件

.bashrc 的一个示例

```bash
# Set the search path
PATH=$HOME/bin:/usr/local/bin:/usr/bin:/bin
# Set the shell promp
PS1='$'
# Set your preferred text editor
EDITOR=emacs
# Start in my work directory
cd $HOME/Work/Projects
# Define an alias
alias g=grep
# Offer a hearty greeting 
echo "Welcome to Linux, firend!"
```

强制某个正在运行的 shell 重新读取并执行 $HOME/.bashrc (即初始化文件)

```bash
$ source $HOME/.bashrc
$ . $HOME/.bashrc
```

# 重复运行历史命令

学习盲打，达到每分钟输入 100 个单词的水平。

## 查看命令的历史记录

history 命令

```bash
$ history

# 输出最近的 3 条命令
$ history 3

# history 按照从早到晚的顺序分页显示历史记录
$ history | less

# history 按照从晚到早的顺序分页显示历史记录
$ history | sort -nr | less

# 只输出包含单词 "cd" 的命令
$ history | grep -w cd

# 清楚当前 shell 中的历史记录
$ history -c
```

## 重复调用历史记录的命令

### 通过方向键浏览历史记录

查找刚运行过的 2-3 个命令中的一个，则直接浏览更为便利。

有关命令历史记录的常见问题：

- shell 的历史记录会保存多少条命令？

最多可以保存500条命令，可以在 shell 的变量 HISTSIZE 更改：

```bash
$ echo $HISTSIZE

# 改成 10000，10000条历史记录大概占用内存 200KB
# 将 HISTSIZE 设置成 -1，存储的命令就没有上限了
$ HISTSIZE=10000
```

- 哪些命令会被添加到历史记录？

shell 会将用户输入的所有内容原样（未经过计算）添加到历史记录。

- 重复的命令会被添加到历史记录吗？

取决于变量 HISTCONTROL 的值。
默认情况，这个值是未设定的，所以每个命令都会被添加到历史记录。
如果这个值设定为 ignoredups（推荐这样做），则连续重复的命令就不会添加到历史记录中。

```bash
$ HISTCONTROL=ignoredups
```

- 每个 shell 都有单独的历史记录吗？还是说所有 shell 共享一个历史记录？

每个交互式 shell 都有一个单独的历史记录。

- 新启动的交互式 shell 有历史记录吗？为什么？

只要交互式 shell 存在，它就会将自己的历史记录写入文件 $HOME/.bash_history，或写入 shell 变量 HISTFILE 中保存的路径：

```bash
$ echo $HISTFILE
```

交互式 shell 每次启动都会加载该文件（.bash_history），所以 shell 一启动就有历史记录。
变量 HISTFILESIZE 控制写入该文件的历史记录数量。
（通过修改 HISTSIZE 来控制内存中的历史记录数量，同时应该修改 HISTFILESIZE）

```bash
$ echo $HISTFILESIZE
$ HISTFILESIZE=10000
```

### 历史记录展开

历史记录展开是 shell 的一个功能，可以通过特殊的表达式访问命令的历史记录。

```bash
$ echo $HISTSIZE
$ !!                # 表示前一个命令

# 想引用最近的，以某个特定字符串开头的命令，可以在该字符串前面加一个叹号
$ !grep     # 重复运行最近的一个 grep 命令

# 引用最近的，包含给定字符串的一个命令，且该字符串可出现在任意位置而不只是开头
$ !?grep?

# 指定绝对位置（命令 history 输出中显示在命令左侧的ID）的方式，从 shell 历史记录中获取某个特定的命令。
$ history | grep hosts
1203 cat /etc/hosts
$ !1203
cat /etc/hosts

# 指定负数值，表示根据相对位置，获取历史记录中的某个命令
$ !-3       # 倒数第三个命令

# 输出历史记录中的命令，但不执行
$ !-3:p                 # 输出倒数第三个命令，但不执行，添加到历史记录末尾
head -n2 /etc/hosts
$ !!                    # 运行命令
head -n2 /etc/hosts     # 输出并运行命令

# 使用 echo 在标准输出中显示 "!!" 的值，然后统计以下单词 "wc" 的出现次数
$ ls -l /etc | head -n3         # 运行任意命令
$ echo "!!" | wc -w             # 统计前一个命令的单词数
```

### 避免删除错误的文件

- 方案1：

```bash
# 在每次执行删除前，shell 都会显示确认提示，防止删错文件
$ alias rm='rm -i'
```

- 方案2：

```bash
# 1. 验证：在运行 rm 之前，先运行 ls，看看哪些文件匹配指定的模式
$ ls *.txt
a.txt b.txt c.txt

# 2. 在删除文件前，先用 head 命令看一看文件内容，确认选对了文件，再运行 rm !$
$ head a.txt
$ head b.txt
$ head c.txt

# 3. 删除：在确认 ls 的输出正确后，再运行 `rm !$` (!$ 表示前一个命令中输入的最后一个单词)
$ rm !$
rm *.txt

# shell 提供了一个历史记录展开 "!*"，它匹配前一个命令输出的所有参数，不只是最后一个参数
# 但（*）存在被误认为文件名模式匹配的风险。
$ ls *.txt *.o *.log
a.txt b.txt c.txt main.o output.log parser.o
$ rm !*
rm *.txt *.o *.log
```

### 命令历史记录的增量搜索

1. 在 shell 提示符下，按下 Ctrl+R （逆向增量搜索）
2. 输入前一个命令的任意一部分：开头、中间、末尾
3. 随着输入一个字符，shell 会自动显示最近的历史记录中与输入的字符匹配的命令
4. 在看到想要的命令后，按回车键即可运行

关于增量搜索的几个技巧：
- 连续两次 Ctrl+R，可以调出最近搜索并执行过的字符串；
- 如果想停止增量搜索，并继续当前命令，则可以按 Esc 键、Ctrl-J 或任何命令行编辑健，比如向左或向右方向键；
- 如果想退出增量搜索，并清空命令行，则可以按 Ctrl-G 或 Ctrl-C。

## 命令行编辑

### 在命令内移动光标

编辑命令行时移动光标的按键

|按键|动作|
|:-:|:-:|
|向左方向键|向左移动一个字符|
|向右方向键|向右移动一个字符|
|Ctrl+向左方向键|向左移动一个单词|
|Ctrl+向右方向键|向右移动一个单词|
|Home 键|移动到命令行的开头|
|End 键|移动到命令行的末尾|
|退格键|删除光标前的一个字符|
|删除键|删除光标处的一个字符|

### 历史记录展开的脱字符表示法

```bash
# 错误运行了如下命令：错把 "jpg" 写成了 "jg"
$ md5sum *.jg | cut -c1-32 | sort | uniq -c | sort -nr 

# 输入错误的文本，再输入正确的文本，和两个脱字符(^)
$ ^jg^jpg       # 将前一个命令中的 jg 替换成 jpg
md5sum *.jpg | cut -c1-32 | sort | uniq -c | sort -nr 

# 如果前一个命令出现多次 "jg"，那么只有第一个会被改为 "jpg"
```

利用历史记录展开实现更强大的替换，将源字符串改为目标字符串，替换格式为 `s/source/target`

```bash
# 通过历史记录展开调出命令，然后加上冒号(:)，再加上 sed 风格的替换表达式
# 调出前一个命令，将 "jg" 替换成 "jpg"（只替换第一个出现的字符串）
$ !!:s/jg/jpg

# 调用一个以 md5sum 开头的命令，将 "jg" 替换成 "jpg"
$ !md5sum:s/jg/jpg
```

### Emacs 或 Vim 风格的命令行编辑

Emacs 中的 Meta 键通常指 Esc键（按下然后放开）或Alt键（按住）。
shell 默认的是 Emacs 风格的编辑方式。如果喜欢 Vim 风格的编辑方式，需要运行以下命令（或将该命令添加到文件 $HOME/.bashrc）

```bash
# 选择 Vim 风格的编辑方式
$ set -o vi
# 选择 Emacs 风格的编辑方式
$ set -o emacs
```

Emacs 或 Vim 风格编辑按键
|动作|Emacs|Vim|
|:-:|:-:|:-:|
|向前移动一个字符|Ctrl+f|h|
|向后移动一个字符|Ctrl+b|l|
|向前移动一个单词|Meta-f|w|
|向后移动一个单词|Meta-b|b|
|移动到行首|Ctrl-a|0|
|移动到行末|Ctrl-e|$|
|转置（交换）两个字符|Ctrl-t|xp|
|转置（交换）两个单词|Meta-t|无|
|将下一个单词的首字母转换成大写|Meta-c|w~|
|将下一个单词整个转换成大写|Meta-u|无|
|将下一个单词整个转换成小写|Meta-l|无|
|转换当前字符的大小写|n/a|~|
|原样插入下一个字符，包括控制字符|Ctrl-v|Ctrl-v|
|删除前一个字符|Ctrl-d|x|
|删除后一个字符|退格键或Ctrl-h|X|
|剪切前一个单词|Meta-d|dw|
|剪切后一个单词|Meta- 退格键或 Ctrl-w|db|
|从光标处剪切到行首|Ctrl-u|d^|
|从光标处剪切到行末|Ctrl-k|D|
|删除整行|Ctrl-e Ctrl-u|dd|
|粘贴最后一次删除的文本|Ctrl-y|p|
|粘贴钱铁板中的下一个文本（只能在粘贴命令后使用）|Meta-y|无|
|撤销前一个编辑操作|Ctrl-_|u|
|撤销前面所有的编辑|Meta-r|U|
|从插入模式切换到命令模式|无|Esc|
|从命令模式切换到插入模式|无|i|
|中止正在进行的编辑模式|Ctrl-g|无|
|清空屏幕|Ctrl-l|Ctrl-l|

"无" 表示没有直接的按键，但可以通过一系列按键实现。

# 浏览文件系统

## 快速访问特定目录

### 快速回到根目录

```bash
$ pwd
/etc
$ cd            # 运行 cd 命令，不带参数...
/home/smith     # 回到了根目录

# 想进入根目录下的子目录
# 方法1：使用 shell 的变量 HOME
$ cd $HOME/Work
# 方法2：使用波浪号(~)
$ cd ~/Work
# 验证一下
$ echo $HOME ~

# 在波浪号(~)后面输入其他用户名，用于指定其他用户的根目录
$ echo ~jones
/home/jones
```

### Tab 键自动补齐

```bash
$ cd /usr
$ ls
bin games include lib local sbin share src
# 访问子目录 share
$ cd sha<Tab>       # shell 自动补齐目录名 $ cd share
# 键入的字符越多，歧义越少，匹配越好
```

大多数命令都可以使用 Tab 键自动补齐。
- 输入 cd 命令，Tab 键会自动补齐目录名；
- 输入 cat、grep、sort 命令，Tab 键自动补齐文件名；
- 输入 ssh 命令，Tab 键自动补齐用户名。

### 利用别名或变量跳到经常访问的目录

```bash
# 创建 cd 操作的别名。只需要运行别名，就可以跳转到该目录。
# in a shell configuration file:
$ alias work="cd $HOME/Work/Projects/Web/src/include"
$ work
$ pwd
/home/smith/Work/Projects/Web/src/include

# 创建一个变量来存储该目录路径
$ work=$HOME/Work/Projects/Web/src/include
$ cd $Work
$ pwd
/home/smith/Work/Projects/Web/src/include
$ ls $Work/css      # 通过其他方式使用变量
main.css mobile.css
```

使用别名编辑需要频繁编辑的文件

方法1：频繁访问某个目录编辑特定的文件，可以定义一个别名，通过绝对路径编辑该文件，无需变更目录。

```bash
# 通过 rcedit 编辑文件 $HOME/.bashrc，无论在文件系统的何处，都无须运行 cd 命令
# Place in a shell configuration file and source it:
$ alias rcedit='$EDITOR $HOME/.bashrc'
```

弊端：
- 很难记住这么多的别名或变量；
- 有可能意外地创建一个与已有命令同名的别名，从而引起冲突。

方法2：创建一个 shell 函数，名为 qcd，它可以接受一个字符串（如 work、recipes）作为参数。

```bash
# 跳转至其他目录的函数
# Define the qcd function
qcd () {
    # Accept 1 argument that's a string key, and perform a different
    # "cd" operation for each key.
    case "$1" in
        work)
            cd $HOME/Work/Projects/Web/src/include
            ;;
        recipes)
            cd $HOME/Family/Cooking/Recipes
            ;;
        video)
            cd /data/Arts/Video/Collection
            ;;
        beatles)
            cd $HOME/Music/mp3/Artists/B/Beatles
            ;;
        *)
            # The supplied argument was not one of the supported keys
            echo "qcd: unknown key '$1'"
            return 1
            ;;
    easc
    # Helpfully print the current directory name to indicate where you are 
    pwd
}
# Set up tab completion （自定义的 Tab 键自动补齐）
complete -W "work recipes video beatles" qcd 
```

将这个函数保存到 shell 配置文件（如 $HOME/.bashrc）中，然后执行该脚本。键入 qcd，及支持的字符串就可以快速访问相关目录了。

```bash
$ qcd <Tab><Tab>
beatles recipes video work
$ qcd v<Tab><Enter>     # 补齐以 "v" 开头的单词 "video"
/data/Arts/Video/Collection
```

### 利用 CDPATH 快速浏览大型文件系统的速度

"cd 搜索路径"：通过shell变量 CDPATH 配置cd 搜索路径，指定格式与 PATH 相同：目录列表，以分号分隔。

假设 CDPATH 包含以下四个目录：
$HOME:$HOME/Projects:$HOME/Family/Memories:/usr/local
键入 `$ cd Photos`，cd 就会依次检查下述目录是否存在，直到找到子目录，或搜索完所有目录，却未发现相应的子目录。
1. 当前目录是否包含 Photos
2. $HOME/Photos
3. $HOME/Projects/Photos
4. $HOME/Family/Memories/Photos
5. /usr/local/Photos
cd 尝试了四次，并成功跳转至 $HOME/Family/Memories/Photos，如果 $CDPATH 中有两个名为 Photos 的子目录，则选择比较靠前的目录。

可以将最重要或经常使用的父目录添加到CDPATH。

```bash
$ CDPATH=/usr       # 设置 CDPATH
$ cd /tmp           # 没有输出，不会查找 CDPATH
$ cd bin            # cd 命令会查找 CDPATH
/usr/bin            # ......并输出新的当前目录
```

### 合理地组织根目录加快浏览文件系统的速度

**按照顺序将以下目录添加到 CDPATH**
1. $HOME

```bash
$ pwd
/etc                        # 位于根目录之外
$ cd Work
/home/smith/Work
$ cd Family/School          # 跳转至 $HOME 的下一层 
/home/smith/Family/School
```

2. $HOME 中的主要子目录

```bash
$ pwd
/etc                        # 位于根目录之外
$ cd School
/home/smith/Family/Scholl   # 跳转至 $HOME 的下两层 
```

3. 父目录的相对路，即两个点（..）

```bash
$ pwd
/usr/bin                    # 当前目录
$ ls ..
bin include lib src         # 同级目录
$ cd lib
/usr/lib                    # 跳转至同级目录

```

```bash
# CDPATH 应包含以下目录：根目录、根目录的四个子目录、父目录的相对路径
# Place in a shell configuration file and source it:
export CDPATH=$HOME:$HOME/Work:$HOME/Family:$HOME/Linux:$HOME/Music:..
```

当 CDPATH 目录下所有子目录的名称都唯一时，该技巧的效果最佳。如果子目录重名，则可能达不到预期的效果。（比如，$HOME/Music 和 $HOME/Linux/Music，命令 cd 首先会检查 $HOME 中是否包含 Music 子目录，由于 $HOME/Linux 的位置靠后，因此搜索可能无法找到 $HOME/Linux/Music）

```bash
# 检查 $HOME 目录结构的前两个层级中是否包含重名的子目录
# 列出 $HOME 中所有的子目录和子子目录，通过 cut 剪切出子子目录，排序子子目录的列表，通过 uniq 统计出现次数
# 如果显示统计数量超过 1，则表示有重名的子目录
$ cd
$ ls -d */ && (ls -d */*/ | cut -d/ -f2-) | sort | uniq -c | sort -nr | less
```

## 快速返回特定目录

### 通过 "cd -" 在两个目录之间来回切换

```bash
$ pwd
/home/smith/Finances/Bank/Checking/Statements
$ cd /etc
# 回到之前的 Statements 目录
$ cd -
/home/smith/Finances/Bank/Checking/Statements

# 在两个目录之间来回跳转，可以反复运行 cd -，但 shell 只能保存上一个目录
$ pwd
/usr/local/bin
$ cd /etc               # shell 会记录下 /usr/local/bin
$ cd -                  # shell 会记录下 /etc
/usr/local/bin
$ cd -                  # shell 会记录下 /usr/local/bin
/etc

# 某次忘了输入参数，直接跳转到了自己的主目录下：
$ cd                    # shell 会记录下 /etc，shell 记录的前一个目录不再是 /usr/local/bin 
$ cd -                  # shell 记录的前一个目录是主目录
/etc
$ cd -                  # shell 记录的前一个目录是 /etc
/home/smith
```

### 通过 pushd 和 popd 在多个目录之间来回切换

使用名为 "目录栈" 的 shell 功能，利用其内置的 pushd、popd、irs 命令在多个目录来回切换。
目录栈：在当前 shell 中访问过并决定保存下来的目录列表。通过入栈(push)和出栈(pop)两个操作控制栈内保存的目录。

- 目录的入栈操作 pushd
    - 将指定的目录添加到栈的顶部；
    - 执行 cd 命令，进入该目录；
    - 输出栈内保存的目录，可以指定按照从顶到底的顺序输出。

```bash
# 构建一个目录栈，其中包含四个目录，将它们逐个放入栈内
$ pwd
/home/smith/Work/Projects/Web/src
$ pushd /var/www/html
/var/www/html ~/Work/Projects/Web/src           # 每个 pushd 命令执行完成后，shell 都会输出整个栈
$ pushd /etc/apache2
/etc/apache2 /var/www/html ~/Work/Projects/Web/src
$ pushd /etc/ssl/certs
/etc/ssl/certs /etc/apache2 /var/www/html ~/Work/Projects/Web/src
$ pwd
/etc/ssl/certs          # 当前目录是最顶部的目录
```

- 看目录栈 dirs （该命令不会修改栈）

```bash
$ dirs
/etc/ssl/certs /etc/apache2 /var/www/html ~/Work/Projects/Web/src

# 纵向输出栈，使用选项 -p
$ dirs -p
/etc/ssl/certs 
/etc/apache2 
/var/www/html 
~/Work/Projects/Web/src

# 给每一行加上编号（从0开始）
$ dirs -p | nl -v0
    0   /etc/ssl/certs 
    1   /etc/apache2 
    2   /var/www/html 
    3   ~/Work/Projects/Web/src

# 更简单的办法
$ dirs -v
 0  /etc/ssl/certs 
 1  /etc/apache2 
 2  /var/www/html 
 3  ~/Work/Projects/Web/src

# 为这种纵向格式定义一个别名
# Place in a shell configuration file and source it:
$ alias dirs='dirs -v'
```

- 从栈中弹出目录 popd
    - 移除栈顶部的一个目录；
    - 执行 cd 命令，进入栈顶部的目录；
    - 输出栈内保存的目录，可以指定按照从顶到底的顺序输出。

```bash
# 假设有如下四个目录
$ dirs
/etc/ssl/certs /etc/apache2 /var/www/html ~/Work/Projects/Web/src
# 重复运行 popd 命令就可以从顶到底遍历这些目录
$ popd
/etc/apache2 /var/www/html ~/Work/Projects/Web/src
$ popd
/var/www/html ~/Work/Projects/Web/src
$ popd
~/Work/Projects/Web/src
$ popd
bash: popd: directory stack empty
$ pwd
~/Work/Projects/Web/src

# 为 pushd 和 popd 创建两个字符的别名
# Place in a shell configuration file and source it:
$ alias gd=pushd
$ alias pd=popd
```

- 在栈中的目录之间来回切换

直接运行 pushd 不带任何参数，可以在栈最顶部的两个目录之间来回切换。

```bash
$ dirs
/etc/apache2 ~/Work/Projects/Web/src /var/www/html
$ pushd
~/Work/Projects/Web/src /etc/apache2 /var/www/html
$ pushd
/etc/apache2 ~/Work/Projects/Web/src /var/www/html
$ pushd
~/Work/Projects/Web/src /etc/apache2 /var/www/html
```

pushd 与 cd - 类似，但 pushd 没有只存储一个目录的限制。

- 通过 pushd 挽救因错误运行 cd 命令而丢失的目录

```bash
# 通过 pushd 在多个目录之间来回跳转，不小心运行了 cd 命令，并丢失了一个目录
$ dirs
~/Work/Projects/Web/src /var/www/html /etc/apache2
$ cd /etc/ssl/certs
$ dirs
/etc/ssl/certs /var/www/html /etc/apache2

# 将丢失的目录重新加入占中，无需输入冗长的路径
$ pushd -       # 返回 shell 的前一个目录 ~/Work/Projects/Web/src，并将它添加到目录栈
~/Work/Projects/Web/src /var/www/html /etc/apache2
$ pushd         # 交换栈顶的两个目录，并返回 /etc/ssl/certs。重新将 ~/Work/Projects/Web/src 添加到栈的第二个位置，即错误运行 cd 命令之前的位置。
/etc/ssl/certs ~/Work/Projects/Web/src  /var/www/html /etc/apache2

# 为 pushd - 和 pushd 创建一个别名
# Place in a shell configuration file and source it:
$ alias slurp='pushd - && pushd'
```

- 深入目录栈

pushd 和 popd 可以接受一个整数（正或负）参数，用于深入栈中的目录。
`$ pushd +N` ：
- 正参数（+N）表示将栈顶部的N个目录换到底部，并执行 cd 命令，进入新的顶部目录。
- 负参数（-N）表示按照相反的方向（从底部到顶部）移动，然后再执行 cd 命令。

```bash
$ dirs
/etc/ssl/certs ~/Work/Projects/Web/src  /var/www/html /etc/apache2
$ pushd +1
~/Work/Projects/Web/src  /var/www/html /etc/apache2 /etc/ssl/certs
$ pushd +2
/etc/apache2 /etc/ssl/certs ~/Work/Projects/Web/src  /var/www/html
# 如果栈很长
$ dirs -v
 0  /etc/apache2 
 1  /etc/ssl/certs 
 2  ~/Work/Projects/Web/src  
 3  /var/www/html
# 将 /var/www/html 换到栈的顶部（并将其作为当前目录）
$ pushd +3

# 直接跳至栈的底部
$ dirs
/etc/apache2 /etc/ssl/certs ~/Work/Projects/Web/src  /var/www/html
$ pushd -0
/var/www/html /etc/apache2 /etc/ssl/certs ~/Work/Projects/Web/src

# 移除从栈顶开始第 N 个位置的目录，负参数（-N）表示从栈底部向上数。
# 注意：这个参数从 0 开始
$ dirs 
/var/www/html /etc/apache2 /etc/ssl/certs ~/Work/Projects/Web/src
$ popd +1
/var/www/html /etc/ssl/certs ~/Work/Projects/Web/src
$ popd +2
/var/www/html /etc/ssl/certs
```