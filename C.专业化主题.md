# 提高键盘输入的效率

## 窗口管理

### 即时启动 shell 和浏览器

为常用操作定义快捷键
- 打开一个shell窗口
    - gnome-terminal
    - konsole: Ctr-Windows-T
    - xterm
- 打开一个网页浏览器窗口
    - firefox
    - google-chrome: Ctrl-Windows-C
    - opera

工作目录
- 在桌面环境中通过快捷键启动 shell 时，它会在登陆 shell 中创建一个子进程，该子进程的当前目录是该用户的主目录；
- 在命令行中运行 gnome-terminal 或 xterm 或 使用终端程序的菜单打开一个新窗口，终端程序会打开一个新 shell 。它是终端 shell 的子进程。其当前目录与父目录相同，不一定是该用户的主目录。

### 一次性窗口

一次性 shell 终端窗口，使用后关闭。
一次性 浏览器窗口，用完就关闭，如有需要再次查看该页面，只需翻看浏览器的历史记录。

### 浏览器键盘快捷键

Firefox Chrome Opera 最常用的键盘快捷键

|动作|键盘快捷键|
|:-:|:-:|
|打开一个新窗口|Ctrl-N|
|打开一个新的私人/隐身窗口|Ctrl-Shift-P（Firefox），Ctrl-Shift-N（Chrome 和 Opera）|
|打开一个新的选项卡|Ctrl-T|
|关闭选项卡|Ctrl-W|
|循环浏览选项卡|Ctrl-Tab（向前）、Ctrl-Shift-Tab（向后）|
|跳转到地址栏|Ctrl-L（或 Alt-D 或 F6）|
|在当前页面中查找（搜索）文本|Ctrl-F|
|显示浏览器的历史记录|Ctrl-H|


### 切换窗口和桌面

切换窗口：Alt-Tab，Alt-Shift-Tab（反向切换）
循环浏览桌面上属于同一应用程序的所有窗口：Alt-`，Alt-Shift-`（向后循环）


## 通过命令行访问网页

### 通过命令行启动浏览器窗口

```bash
$ firefox &
$ google-chrome &
$ opera &
# 如果浏览器已在运行，则可以省略符号 &

# 防止在命令行输出诊断信息，并弄乱 shell 窗口，在首次启动浏览器时将所有输出重定向到 /dev/null 
$ firefox &> /dev/null &

# 通过命令行打开浏览器访问某个URL：
$ firefox https://oreilly.com
$ google-chrome https://oreilly.com
$ opera https://oreilly.com

# 强制打开一个新窗口
$ firefox --new-window https://oreilly.com
$ google-chrome --new-window https://oreilly.com
$ opera --new-window https://oreilly.com

# 打开一个私有窗口或隐身模式的浏览器
$ firefox --private-window https://oreilly.com
$ google-chrome --incognito https://oreilly.com
$ opera --private https://oreilly.com

# 为经常访问的站点定义别名：
# Place in a shell configuration file and source it:
alias oreilly="firefox --new-window https://oreilly.com"

# 通过一个文件保存经常访问的URL：使用 grep、cut或其他命令提取 URL，在命令行中通过命令替换将 URL 传递给浏览器。
$ cat urls.txt
duckduckgo.com      My search engine
nytimes.com         My newspaper
spotify.com         My music
$ grep music urls.txt | cut -f1
spotify.com
$ google-chrome https://$(grep music urls.txt | cut -f1)        # 访问 spotify

# 通过编号查询包裹
$ cat packages.txt
170EW7360669374701          UPS     Shoes
568733462924                FedEx   Kitchen blender
9505510823011761842873      USPS    Care package from Mom
```

```bash
# 脚本 track-it，可以打开快速查询页面：找到对应的 URL，在末尾加上相应的查询编号。
#!/bin/bash
PROGRAM=$(basename $0)
DATAFILE=packages.txt
# Choose a browser command: firefox, opera, google-chrome
BROWSER="opera"
errors=0

cat "$DATAFILE" | while read line; do
    track=$(echo "$line" | awk '{print $1}')
    service=$(echo '$line' | awk '{print $2}')
    case "$service" in
        UPS)
            $BROWSER "https://www.ups.com/track?tracknum=$track" &
            ;;
        FedEx)
            $BROWSER "https://www.fedex.com/fedextrack/?trknbr=$track" &
            ;;
        USPS)
            $BROWSER "https://tools.usps.com/go/TrackConfirmAction?tLabels=$track" &
            ;;
        *)
            >&2 echo "$BROWSER: Unknown service '$service'"
            errors=1
            ;;
    esac
done
exit $errors
```

### 利用 curl 和 wget 获取 HTML

curl 会在标准输出上显示结果。
wget 会将输出保存到文件。

```bash
$ curl https://efficientlinux.com/welcome.html
Welcome to Efficient Linux.com

$ wget https://efficientlinux.com/welcome.html
--2025-11-29 09:50:54--  https://efficientlinux.com/welcome.html
Connecting to 127.0.0.1:7890... connected.
Proxy request sent, awaiting response... 200 OK
Length: 32 [text/html]
Saving to: ‘welcome.html’

welcome.html        100%[===================>]      32  --.-KB/s    in 0s      

2025-11-29 09:50:57 (5.69 MB/s) - ‘welcome.html’ saved [32/32]

$ cat welcome.html
Welcome to Efficient Linux.com!

: '
    某些网站不支持 curl 和 wget 获取网页内容
    使这两个命令伪装成另一个浏览器，告知这两个程序更改其用户代理
'
$ wget -U Mozilla url
$ curl -A Mozilla url

: '
    假如网站 efficientlinux.com 有一个目录 images，它包含1.jpg ~ 20.jpg，要下载这些文件。
    其网址如下：
    https://efficientlinux.com/images/1.jpg
    https://efficientlinux.com/images/2.jpg
    https://efficientlinux.com/images/3.jpg
    ...
'
# 低效的方法：逐个访问 URL，每次访问一个，然后下载每个图像
# 高效的方法：使用 wget
# 1. 利用 seq 和 awk 生成URL
$ seq 1 20 | awk '{print "https：//efficientlinux.com/images/" $1 ".jpg"}'
# 2. 在 awk 程序中添加一个字符串 "wget"，将生成的命令通过管道传给 bash
$ seq 1 20 \ 
    | awk '{print "wget https：//efficientlinux.com/images/" $1 ".jpg"}' \
    | bash
# 2. 或者使用 xargs 创建并执行 wget 命令(如果包含特殊字符，选择 xargs 更好)
$ seq 1 20 | xargs -I@ wget https://efficientlinux.com/images/@.jpg

```

### 利用 HTML-XML-utils 处理 HTML

安装 html-xml-utils 

- 利用 curl （或 wget）获取 HTML 源代码；
- 利用 hxnormalize 确保 HTML 格式正确；
- 通过 CSS 选择器获取想要的值；
- 利用 hxselect 分隔各个值，并通过管道将输出结果传递给其他命令，进行进一步的处理。


从网页获取区号数据，生成该示例中使用的 areacodesl.txt 文件：

[HTML区号表（供下载和处理）](https://efficientlinux.com/areacodes.html)
|Area code|State|Location|
|:-:|:-:|:-:|
|201|NJ|Hackensack, Jersey City|
|202|DC|Washington|
|203|CT|New Haven, Stamford|
|204|MB|entire province|
|205|AL|Birmingham, Tuscaloosa|
|206|WA|Seattle|
|207|ME|entire state|
|208|ID|entire state|
|209|CA|Modesto, Stockton|
|210|TX|San Antonio|
|212|NY|New York City, Manhattan|

```bash
# 用 curl 获取 HTML 源代码，通过选项 -s 禁止在屏幕上显示消息。
# 通过管道将输出结果传递给 hxnormalize -x 整理代码，再传递给 less，分页显示输出结果。
$ curl -s https://efficientlinux.com/areacodes.html \
    | hxnormalize -x \
    | less
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01//EN"
"http://www.w3.org/TR/html4/strict.dtd">

<html>
  <head>
    <meta content="text/html; charset=UTF-8" http-equiv="content-type"
       />
    <title>Area code test</title>

    <style><![CDATA[
      #ac {
          border: 1px solid black;
      }
      #ac tr:nth-child(even) {
          background-color: #eeeeee;
      }
    ]]></style></head>

  <body>
    <h1>Area code test</h1>

    <table id="ac">
      <tbody>
        <tr>
          <th>Area code</th>

          <th>State</th>

          <th>Location</th>
        </tr>

        <tr>
          <td class="ac">201</td>

          <td class="state">NJ</td>

          <td class="cities">Hackensack, Jersey City</td>
        </tr>

        <tr>
          <td class="ac">202</td>

          <td class="state">DC</td>

          <td class="cities">Washington</td>
        </tr>

        <tr>
          <td class="ac">203</td>

          <td class="state">CT</td>

          <td class="cities">New Haven, Stamford</td>
        </tr>

        <tr>
          <td class="ac">204</td>

          <td class="state">MB</td>

          <td class="cities">entire province</td>
        </tr>

        <tr>
          <td class="ac">205</td>

          <td class="state">AL</td>

          <td class="cities">Birmingham, Tuscaloosa</td>
        </tr>

        <tr>
          <td class="ac">206</td>

          <td class="state">WA</td>

          <td class="cities">Seattle</td>
        </tr>

        <tr>
          <td class="ac">207</td>

          <td class="state">ME</td>

          <td class="cities">entire state</td>
        </tr>

        <tr>
          <td class="ac">208</td>

          <td class="state">ID</td>

          <td class="cities">entire state</td>
        </tr>

        <tr>
          <td class="ac">209</td>

          <td class="state">CA</td>

          <td class="cities">Modesto, Stockton</td>
        </tr>

        <tr>
          <td class="ac">210</td>

          <td class="state">TX</td>

          <td class="cities">San Antonio</td>
        </tr>

        <tr>
          <td class="ac">212</td>

          <td class="state">NY</td>

          <td class="cities">Modesto, Stockton</td>
        </tr>

        <tr>
          <td class="ac">210</td>

          <td class="state">TX</td>

          <td class="cities">San Antonio</td>
        </tr>

        <tr>
          <td class="ac">212</td>

          <td class="state">NY</td>

          <td class="cities">New York City, Manhattan</td>
        </tr>
      </tbody>
    </table>
  </body>
</html>

: '
    页面HTML表格的 CSS ID 为 #ac，其三列（区号、州和位置）的 CSS 类分别是 ac/ state/ cities
    <table id="ac">
      <tbody>
        <tr>
          <th>Area code</th>

          <th>State</th>

          <th>Location</th>
        </tr>

        <tr>
          <td class="ac">201</td>

          <td class="state">NJ</td>

          <td class="cities">Hackensack, Jersey City</td>
        </tr>
        ...
      </tbody>
    </table>
'
# 运行 hxselect，从每个表格单元格中提取区域代码，并通过选项 -c 过滤掉输出中的标签 td。
# 输出的结果只有一行且很长，各个字段间由指定的字符分隔（使用 -s 选项），选择字符 @，方便查看。
$ curl -s https://efficientlinux.com/areacodes.html \
    | hxnormalize -x \
    | hxselect -c -s@ '#ac .ac, #ac .state, #ac .cities'
201@Hackensack, Jersey City@202@Washington@203@New Haven, Stamford@204@entire province@205@Birmingham, Tuscaloosa@206@Seattle@207@entire state@208@entire state@209@Modesto, Stockton@210@San Antonio@212@New York City, Manhattan@% 

: '
    1. 将输出管道传给 sed，编写正则表达式将输出转换为三列（以制表符分隔）：
    - 区号：由数字组成 [0-9]*
    - @ 符号
    - 美国州的缩写，两个大写字母，[A-Z][A-Z]
    - @ 符号
    - 城市：任何不包含 @ 符号的文本 [^@]*
    - @ 符号
    2. 将以上组合起来：[0-9]*@[A-Z][A-Z]@[^@]*@
    3. 在区域代码、州、城市的前后加 \，形成三个子表达式：\([0-9]*\)@\([A-Z][A-Z]\)@\([^@]*\)@
    4. 用制表符分隔上述三个子表达式并加上换行符，然后作为替换字符串传给 sed，可以生成 areacodes.txt 文件的格式：\1\t\2\t\3\n
    5. 拼接上述正则表达式和替换字符串，得到完整的 sed 脚本：s/\([0-9]*\)@\([A-Z][A-Z]\)@\([^@]*\)@/\1\t\2\t\3\n/g
'
$ curl -s https://efficientlinux.com/areacodes.html \
    | hxnormalize -x \
    | hxselect -c -s'@' '#ac .ac, #ac .state, #ac .cities' \
    | sed 's/\([0-9]*\)@\([A-Z][A-Z]\)@\([^@]*\)@/\1\t\2\t\3\n/g'
201	NJ	Hackensack, Jersey City
202	DC	Washington
203	CT	New Haven, Stamford
204	MB	entire province
205	AL	Birmingham, Tuscaloosa
206	WA	Seattle
207	ME	entire state
208	ID	entire state
209	CA	Modesto, Stockton
210	TX	San Antonio
212	NY	New York City, Manhattan
```

**注意：某些旧版的 hxselect 只能处理两个 CSS选择器**
1. 下载最新的 [hxselect 版本](https://www.w3.org/Tools/HTML-XML-utils/)
2. 执行以下命令编译并安装

```bash
$ ./configure && make 
$ sudo make install
```

**处理冗长的正则表达式**
如果 sed 脚本非常长，如 “s/\([0-9]*\)@\([A-Z][A-Z]\)@\([^@]*\)@/\1\t\2\t\3\n/g”。可将其拆分，将正则表达式的各个部分储存在几个 shell 变量内，然后将变量组合起来，如下：

```bash
# The three parts of the regular expression
# Use single quotes to prevent evaluation by the shell
areacode='\([0-9]*\)'
state='\([A-Z][A-Z]\)'
cities='\([^@]*\)'

# Combine the three parts, separated by @ symbols
# Use double quotes to permit variable evaluation by the shell
regexp="$areacode@$state@$cities@"

# The replacement string.
# Use single quotes to prevent evaluation by the shell
replacement='\1\t\2\t\3\n'

# The sed script now becomes much simpler to read:
# s/$regexp/$replacement/g
# Run the full command:
curl -s https://efficientlinux.com/areacodes.html \
    | hxnormalize -x \
    | hxselect -c -s'@' '#ac .ac,#ac .state,#ac .cities' \
    | sed "s\/$regexp/$replacement/g"
```


### 利用基于文本的浏览器获取网页内容

用命令安装 lynx 和 links

```bash
# lynx 和 links 都可以通过选项 -dump 下载页面
$ lynx -dump https://efficientlinux.com/areacodes.html > tempfile
$ cat tempfile

# 可利用 lynx 和 links 检查某个看似可疑的链接。但它们不能保证百分百安全。
```

## 通过命令行控制剪贴板

Linux 上的复制和粘贴操作属于一种更普通的机制，叫“X选择（复制目的地）”。
大多数基于X构建的Linux桌面环境都支持两种文本选择方式：
- 剪贴板：与其他操作系统的剪贴板完全相同；
- X 选择（主选择）：如，在终端窗口中使用鼠标选中某些文本，这段文本就会写入主选择。

通过常见的终端程序访问 X 选择
|操作|剪切板|主选择|
|:-:|:-:|:-:|
|复制（鼠标）|打开右键菜单并选择复制|单击并拖动；或双击选择当前单词；或三次单击选择当前行|
|粘贴（鼠标）|打开右键菜单并选择粘贴|按鼠标中键（通常是滚轮）|
|复制（键盘）|Ctrl-Shift-C|无|
|粘贴（键盘），gnome-terminal|Ctrl-Shift-V 或 Ctrl-Shift-Insert|Shift-Insert|
|粘贴（键盘），Konsole|Ctrl-Shift-V 或 Shift-Insert|Ctrl-Shift-Insert|

### 将选择连接到标准输入和标准输出

命令 xclip 可将 X 选择连接到标准输入和标准输出。
**不要将带有 xclip 的命令复制并粘贴到 shell 窗口中，须要手动输入命令。因为复制操作可能会覆盖命令中 xclip 访问的同一个 X 选择，从而导致命令产生意外结果。**

```bash
# 将文本复制到某个应用程序中
: '
    一般程序：
    1. 运行 Linux 命令，并将其输出重定向到文件
    2. 查看文件
    3. 使用鼠标将文件内容复制到剪贴板
    4. 将内容粘贴到另一个应用程序中

    使用 xclip：
    1. 将 Linux 命令的输出通过管道传递给 xclip
    2. 将内容粘贴到另一个应用程序中

    或者将文本粘贴到文件中，然后使用 Linux 命令进行处理：
    1. 使用鼠标复制应用程序的一堆文本
    2. 将其粘贴到文本文件中
    3. 使用 Linux 命令处理文本文件
    而使用 xclip -o 可跳过中间的文本文件：
    1. 使用鼠标复制应用程序中的一堆文本
    2. 将 xclip -o 的输出传递给其他 Linux 命令进行处理
'

# 默认，xclip 会读取标准输入并写入主选择。
# 读取文件
$ xclip < myfile.txt
# 读取管道
$ echo "Efficient Linux at the Command Line" | xclip

$ xclip -o                                  # 复制到标准输出
Efficient Linux at the Command Line 
$ xclip -o > anotherfile.txt                # 复制到文件
$ xclip -o | wc -w                          # 统计单词数
6

# 任何将输出结果写入标准输出的复合命令都可以将结果通过管道传递给 xclip：
$ cut -f1 grades | sort | uniq -c | sort -nr | head -n1 | cut -c9 | xclip
# 清空主选择，需要运行 echo -n，将其值设置为空字符串
# -n 选项很重要，否则 echo 会在标准输出上写入一个换行符，且这个换行符会被写入主选择。
$ echo -n | xclip
# 将文本复制到剪贴板而不是主选择
$ echo https://oreilly.com/ | xclip -selection clipboard    # 复制
$ xclip -selection clipboard -o                             # 粘贴
https://oreilly.com

# 通过命令替换启动 Firefox 浏览器窗口，并访问上述 URL
$ firefox $(xclip -selection clipboard -o)

```

Linux 还提供另一个命令 xsel。它不仅可以读写 X 选择，还有其他功能。如，清空选择(xsel -c)、附加到选择(xsel -a)


# 节省时间的小技巧

## 速效方案

### 从 less 跳转到编辑器

通过 less 打开文本文件，在浏览过程中需要编辑文件，只需按下 v 键就可以编辑了。退出编辑后，回到原来的位置。可以将环境变量 EDITOR 或 VISUAL 设置为编辑命令。

```bash
# 将默认编辑器设置为 emacs。编辑器默认设置是 vim。
VISUAL=emacs
EDITOR=emacs
```

### 编辑包含特定字符串的文件

```bash
# 编辑当前目录中包含某个字符串（或匹配某个正则表达式）的所有文件
# 利用 grep -l 生成文件名列表，通过命令替换将其传递给编辑器（vim） 
$ vim $(grep -l string *)
# 选项 -r 递归，从当前目录（点）开始，编辑整个目录树（当前目录及所有子目录）中包含该字符串的所有文件
$ vim $(grep -lr string .)
# 更快搜索大型目录树，利用 find 和 xargs 替代 grep -r
$ vim $(find . .type f -print0 | xargs -0 grep -l string)
```

### 输入错误

```bash
# 将一些常见的输入错误定义成别名
$ alias firfox=firefox
$ alias les=less
$ alias meacs=emacs

# 不要定义与现有 Linux 命令具有相同名称的别名，否则会覆盖该命令。
# 通过 which 或 type 搜索一下别名，然后运行命令 man 以确保没有其他同名命令：
$ type firfox
$ man firfox
```

### 快速创建空文件

```bash
# 利用 touch 创建大量用于测试的空文件
$ mkdir tmp                     # 创建一个目录
$ cd tmp 
$ touch file{0000..9999}.txt    # 创建 10000 个文件
$ cd ..
$ rm -rf tmp                    # 删除目录和文件

# 将 echo 命令的输出重定向到文件，它会创建一个空文件
# 必须使用 -n 选项，否则最终得到的文件包含一个换行符，即非空文件
$ echo -n > newfile2
```

### 一次处理文件中的一行

```bash
# 通过 while 循环读取文件
$ cat myfile | while read line; do 
    ...此处是一些处理...
done

# 计算文件（/etc/file）每一行的长度：
$ cat /etc/file | while read line; do
    echo "$line" | wc -c
done
```

### 支持递归的命令

```bash
# 以递归的方式针对整个目录树执行任何 Linux 命令：
$ find . -exec your commmand here \;

$ ls -R                     # 以递归的方式列出目录及其内容
$ cp -r  or $ cp -a         # 以递归的方式复制目录及其内容
$ rm -r                     # 以递归的方式删除目录及其内容
$ grep -r                   # 通过正则表达式搜索整个目录树
$ chmod -R                  # 以递归的方式修改文件权限
$ chown -R                  # 以递归的方式修改文件的所有权
$ chgrp -R                  # 以递归的方式修改文件的组所有权
```

### 查阅帮助文档

## 长期的学习

### 阅读 bash 帮助文档

### 学习 cron crontab 以及 at

利用 crontab 设置定期运行的命令
1. 定义默认的编辑器
2. 运行 `crontab -e` 编辑个人的计划命令文件
3. 编辑指定文件 crontab.

crontab 文件中的没一行都是一个计划命令（cron作业），由 6 个字段组成。前 5 个字段分别指定了计划作业的时间：分、小时、天、月、星期。第 6 个字段是运行的 Linux 命令。例如：
* * * * * 命令      每分钟运行一次命令
30 7 * * * 命令     每天 07:30 运行一次命令
30 7 5 * * 命令     每个月第 5 天 07:30 运行一次命令
30 7 5 1 * 命令     1月5日 07:30 运行一次命令
30 7 * * 1 命令     每周一 07:30 运行一次命令

指定这 6 个字段后，保存文件，退出编辑器。

at 命令可在制定的日期和时间运行一次命令。

```bash
# 明天晚上 10 点发送电子邮件提醒刷牙：
$ at 22:00 tomorrow 
at> echo brush your teeth | mail $USER
at> ^D          # 按下 Ctrl-D 结束输入
# 列出待处理的 at 作业
$ atq
699 Sun Nov 14 22:00:00 20211 a smith
# 查看 at 作业中的命令
$ at -c 696 | tail
...
echo brush your teeth | mail $USER
# 在执行前删除作业
$ atrm 699

```

### 学习 rsync

rsync：复制第一个目录与第二个目录的差异。
rsync 可通过 ssh 连接复制到远程服务器。

```bash
# / 表示复制 dir1 中的文件
# 如果没有 / ，则连同目录 dir1 一起复制过去，即创建 dir2/dir1
$ rsync -a dir1/ dir2
```

rsync 常用选项：
- -v：在复制文件时输出文件名称
- -n：模拟复制，结合选项 -v 可确认哪些文件将被复制
- -X：告知 rsync 不要超出当前文件系统

[《Linux的Rsync示例》](https://linuxconfig.org/rsync-command-examples)


### 学习脚本语言

**shell脚本不擅长处理包含空白字符的文件名。**

```bash
# 试图删除某个文件的bash脚本：
#!/bin/bash
BOOKTITLE="Slow Inefficient Linux"
rm $BOOKTITLE           # 错！不要采用这种写法
: '
    脚本试图删除一个名为 Slow Inefficient Linux 的文件，操作错误。
    因为它试图删除三个文件，分别名为 Slow，Inefficient，Linux。
    shell 在调用 rm 前替换变量 $BOOKTITLE，并将其扩展为空格相隔的三个单词，即 rm Slow Inefficient Linux
    应写成：
    rm "$BOOKTITLE"
    这时，shell 会将其替换为：
    rm "Slow Inefficient Linux"
'
# 可以利用 shell 启动复杂的命令，创建简单的脚本，但在实际工作中，应采用一种编程语言（Perl/ Python/ Ruby/ PHP）
```

### 利用 make 处理非编程任务

make 根据规则自动更新文件，旨在加速软件开发。

```bash
: '
    有四个文件：chapter1.txt chapter2.txt chapter3.txt book.txt
    book.txt 包含上述三个章节的文件，每当某一章发生变化时，就需要重新组合各个章节，并更新 book.txt 
'
$ cat chapter1.txt chapter2.txt chapter3.txt > book.txt

: '
    已知条件：
    1. 一堆文件；
    2. 与文件相关的规则，即任何章节文件发生变化 book.txt 都需要更新；
    3. 执行更新操作的命令。

    make 的工作原理：
    读取一个配置文件，通常名为 Makefile，然后进行相应的操作，该配置文件中保存了相应的规则和命令。
'
# 例如，如下 Makefile 中的规则声明 book.txt 必须根据三个章节文件更新：
# book.txt: chapter1.txt chapter2.txt chapter3.txt

# 如果规则的目标的更新时间晚于任何依赖项，则 make 认为目标已过期。
# 如果在规则后制定需要执行的命令，则 make 会运行该命令更新目标：
# book.txt: chapter1.txt chapter2.txt chapter3.txt
#           cat chapter1.txt chapter2.txt chapter3.txt > book.txt
# 要应用规则，只需运行 make：
$ ls 
Makefile    chapter1.txt    chapter2.txt    chapter3.txt
$ make
cat chapter1.txt chapter2.txt chapter3.txt > book.txt                   # 由 make 执行
$ ls 
Makefile    book.txt    chapter1.txt    chapter2.txt    chapter3.txt
$ make
make: 'book.txt' is up to date.
$ vim chapter2.txt                                                      # 更新其中一章
$ make 
cat chapter1.txt chapter2.txt chapter3.txt > book.txt

: '
    将 Asciidoc 文件转换为 HTML 文件的 make 规则：
    %.html:     %.asciidoc
                asciidoctor -o $@ $<
    含义：要创建扩展名为 .html（%.html）的文件，首先需要查找相应的扩展名为 .asciidoc（%.asciidoc）的文件。
    如果 HTML 文件的时间早于 AsciiDoc 文件，则需要根据依赖文件（$<）运行命令 asciidoctor，来重新生成 HTML 文件，并将输出写入目标 HTML 文件（-o $@）。
'
$ ls ch11*
ch11.asciidoc
$ make ch11.html
asciidoctor -o ch11.html ch11.asciidoc
$ ls ch11*
ch11.asciidoc ch11.html
$ firefox ch11.html             # 浏览 HTML 文件
```

[make学习指南](https://makefiletutorial.com/)

### 利用版本控制管理日常文件

Git 常用命令：

```bash
# 将当前目录转存到 Git 仓库
$ git init 
# 将变更后的文件添加到一个看不见的“预发布区”
$ git add .
# 创建新版本，并通过注释描述你对文件所作的更改
$ git commit -m"Changed X to Y"
# 查看版本历史记录
$ git log
```
