
其他 shell 支持的 bash 功能

| bash 功能 | dash | fish | ksh | tcsh | zsh |
| :-: | :-: | :-: | :-: | :-: | :-:|
| alias 内置命令 | Y | Y，但运行“alias别名名称" 不会输出别名 | Y | 不用写等号：alias g grep | Y |
| 通过&在后台运行命令 | Y | Y | Y | Y | Y |
| bash -c | dash -c | fish -c | ksh -c | tcsh -c | zsh -c |
| bash 命令 | dash | fish | ksh | tcsh | zsh |
| bash 的位置：/bin/bash | /bin/dash | /bin/fish | /bin/ksh | /bin/tcsh | /bin/zsh |
| 变量 BASH_SUBSHELL |  |  |  |  |  |
| 大括号表达式 {} | 使用 seq | 必须写成{a,b,c} 不能写成 {a..c} | Y | 使用 seq | Y |
| cd - （来回切换目录） | Y | Y | Y | Y | Y |
| cd 内置命令 | Y | Y | Y | Y | Y |
| 变量 CDPATH | Y | set CDPATH值 | Y | set cdpath=（目录1 目录2 ...） | Y |
| 命令替换：$() | Y | 使用() | Y | 使用反引号 | Y |
| 命令替换：反引号 | Y | 使用() | Y | Y | Y |
| 命令行编辑，使用方向键 |  | Y | YA | Y | Y |
| 命令行编辑，使用Emacs按键 |  | Y | YA | Y | Y |
| 命令行编辑，使用 Vim 按键，set -o vi |  |  | Y | 运行 bindkey -v | Y |
| complete 内置命令 |  | 语法不同B | 语法不同B | 语法不同B | compdefB |
| 条件列表：&& 和 \|\| | Y | Y | Y | Y | Y |
| $HOME 中的配置文件 | .profile | .config/fish/config.fish | profile.kshrc | .cshrc | .zshenv, .zprofile, .zshrc, .zlogin, .zlogout |
| 控制结构：for循环、if语句等 | Y | 语法不同 | Y | 语法不同 | Y |
| dirs 内置命令 |  | Y |  | Y | Y |
| echo 内置命令 | Y | Y | Y | Y | Y |
| 使用 \ 转义别名 | Y |  | Y | Y | Y |
| 使用 \ 进行转义 | Y | Y | Y | Y | Y |
| exec 内置命令 | Y | Y | Y | Y | Y |
| 使用 $? 查询退出代码 | Y | $status | Y | Y | Y |
| export 内置命令 | Y | set -x 名称值 | Y | setenv 名称值 | Y |
| 函数 | YC | 语法不同 | Y |  | Y |
| 变量 HISTCONTROL |  |  |  |  | 查看帮助手册中以 HIST_ 开头的文件名 |
| 变量 HISTFILE |  | set fish_history 路径 | Y | set histfile= 路径 | Y |
| 变量 HISTFILESIZE |  |  |  | set savehist= 路径 | +SAVEHIST |
| history 内置命令 |  | Y，但是命令没有编号 | history 是 hist -l 的别名 | Y | Y |
| history -c |  | history clear | 删除 ~/.sh_history 并重启 ksh | Y | history -p |
| 历史记录展开：!和^ |  |  |  | Y | Y |
| 历史记录增量搜索 Ctrl-R |  | 键入命令的开头，然后按向上箭头搜索，向右箭头选择 | YAD | YC | YF |
| history 编号 |  | histroy - 编号 | history -N 编号 | Y | history - 编号 |
| 通过方向键查看历史记录 |  | Y | YA | Y | Y |
| 通过Emacs的按键查看历史记录 |  | Y | YA | Y | Y |
| 通过 vim 的按键查看历史记录 set -o vi |  |  | Y | Run bindkey -v | Y |
| 变量 HISTSIZE |  |  | Y |  | Y |
| 作业控制：fg,bg，Ctrl-Z,jobs | Y | Y | Y | YG | Y |
| 模式匹配：*, ?, [] | Y | Y | Y | Y | Y |
| 管道 | Y | Y | Y | Y | Y |
| Popd 内置命令 |  | Y |  | Y | Y |
| 进程替换：<() |  |  | Y |  | Y |
| 变量 PS1 | Y | set PS1 值 | Y | set prompt=值 | Y |
| Pushd 内置命令 |  | Y |  | Y，但不支持负数参数 | Y |
| 双引号 | Y | Y | Y | Y | Y |
| 单引号 | Y | Y | Y | Y | Y |
| 标准错误重定向（2>） | Y | Y | Y |  | Y |
| 标准输入重定向（<），标准输出重定向（>, >>） | Y | Y | Y | Y | Y |
| 同时重定向标准输出和标准错误（&>） | 在末尾添加 2>&1 H  |  | 在末尾添加 2>&1 H | >& | Y |
| 通过 source 或点(.)执行某个文件 | 只支持点（.）I | Y | YI | 只支持 source | Y |
| 启动子shell：() | Y |  | Y | Y | Y |
| tab 键自动补齐文件名 |  | Y | YA | Y | Y |
| Type 内置命令 | Y | Y | type 是 whence -v 的别名 | 不支持，但 which 是一个内置命令 | Y |
| unalias 内置命令 | Y | function --erase | Y | Y | Y |
| 变量定义：变量名 = 值 | Y | set 变量名值 | Y | set 变量名=值 | Y |
| 变量计算：$变量名 | Y | Y | Y | Y | Y |


- A. 默认情况下此功能禁用。可以通过运行 set -o emacs 启用。就版本的 ksh 可能有不同的行为。
- B. 可以使用 complete 命令或类似的命令来自定义命令补齐规则，但具体方法因 shell 不同而有很大不同。
- C. 函数：此 shell 不支持以关键字 function 开头的新风格的定义。
- D. 历史记录增量搜索在 ksh 中的工作方式不同。按 Ctrl-R，键入一个字符串，然后按回车键就可以调出包含该字符串的最新命令。再按一次 Ctrl-R 和回车键就可以继续搜索下一个匹配的命令，以此类推。按回车键执行。
- E. 要想在 tcsh 中使用 Ctrl-R 执行历史记录的增量搜索，需要运行命令bindkey ^R i-search-back （并将其添加到 shell 的配置文件中）。另外，搜索的行为也与bash 略有不同。
- F. 在 vi 模式下，需要输入 / 以及搜索字符串，然后按回车键。按 n 可以跳转到下一个搜索结果。
- G. 作业控制：tcsh 无法像其他 shell 那样智能地跟踪默认的作业编号，所以必须提供作业编号，如 %1,作为 fg 和 bg 的参数。
- H. 同时重定向标准输出和标准错误，此 shell 中的语法是：命令 > 文件 2>&1。最后一部分 “2>&1” 表示 “将标准错误（即文件描述符2）”重定向到标准输出（即文件描述符1） 
- I. 在这个 shell 中执行文件需要明确制定源文件的路径。例如，./myfile 表示当前目录中的文件，否则 shell 就找不到该文件。或者，也可以将文件放入 shell 搜索路径中的目录中。
