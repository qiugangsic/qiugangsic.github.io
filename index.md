# 命令行提示

- PS1是主提示环境变量，默认值是`[\u@\h \W\$]`
- PS1变量可以使用的参数
  - \d ：#代表日期，格式为weekday month date，例如："Mon Aug 1"
  - \H ：#完整的主机名称。
  - \h ：#仅取主机的第一个名字。
  - \t ：#显示时间为24小时格式，如：HH：MM：SS
  - \T ：#显示时间为12小时格式
  - \A ：#显示时间为24小时格式：HH：MM
  - \u ：#当前用户的账号名称
  - \v ：#BASH的版本信息
  - \w ：#完整的工作目录名称。家目录会以 ~代替
  - \W ：#利用basename取得工作目录名称，所以只会列出最后一个目录
  - \# ：#下达的第几个命令
  - \$ ：#提示字符，如果是root时，提示符为：# ，普通用户则为：$
  - \s : #shell名称
  - \a : an ASCII bell character (07)
  - \D{format}
            the format is passed to strftime(3) and the result is inserted into   - prompt string; an empty  for‐
          mat results in a locale-specific time representation.  The braces   - required
  - \e     an ASCII escape character (033)
  - \h     the hostname up to the first `.'
  - \H     the hostname
  - \j     the number of jobs currently managed by the shell
  - \l     the basename of the shell's terminal device name
  - \n     newline
  - \r     carriage return
  - \s     the name of the shell, the basename of $0 (the portion following   - final slash)
  - \t     the current time in 24-hour HH:MM:SS format
  - \T     the current time in 12-hour HH:MM:SS format
  - \@     the current time in 12-hour am/pm format
  - \A     the current time in 24-hour HH:MM format
  - \u     the username of the current user
  - \v     the version of bash (e.g., 2.00)
  - \V     the release of bash, version + patch level (e.g., 2.00.0)
  - \w     the  value  of  the PWD shell variable ($PWD), with $HOME abbrevia  - with a tilde (uses the value of the PROMPT_DIRTRIM variable)
  - \W     the basename of $PWD, with $HOME abbreviated with a tilde
  - \!     the history number of this command
  - \#     the command number of this command
  - \$     if the effective UID is 0, a #, otherwise a $
  - \nnn   the character corresponding to the octal number nnn
  - \\     a backslash
  - \[     begin a sequence of non-printing characters, which could be used to em  - a terminal control sequence
  -        into the prompt
  - \]     end a sequence of non-printing characters

- 我的设置：\n\! \u @ \h 当前目录：\W 当前时间：[$(date +%c)] SHELL版本：\s \v\n$
- color: \[\e[0;95;40m\]
可以使用 ANSI 颜色进行配置，使用\[\e[font;fg;bgm\]可以对后面的文字输出进行样式管理

font为字体编码，包括加粗、斜体、闪烁等各种特效，具体编码见表
编码 含义
1 加粗
2 半透明
3 斜体
4 下划线
5 闪烁
6 快速闪烁
7 反色
8 隐藏
9 删除线

fg为前景色编码，根据具体环境不同，有三种格式可以选择：

基本颜色: [颜色编码]
256 颜色: 38;5;[编码]
rgb 颜色: 38;2;[红色];[绿色];[蓝色]

编码 含义
30 黑色
31 红色
32 绿色
33 黄色
34 蓝色
35 品红
36 青色
37 白色
90 浅黑
91 浅红
92 浅绿
93 浅黄
94 浅蓝
95 淡紫
96 淡青
97 浅白

bg为背景色编码，根据具体环境不同，有三种格式可以选择：

基本颜色: [颜色编码]
256 颜色: 48;5;[编码]
rgb 颜色: 48;2;[红色];[绿色];[蓝色]

编码 含义
40 黑色
41 红色
42 绿色
43 黄色
44 蓝色
45 品红
46 青色
47 白色
100 浅黑
101 浅红
102 浅绿
103 浅黄
104 浅蓝
105 淡紫
106 淡青
107 浅白

\[\e[0m\]：重置背景颜色为透明
如：export PS1="\[\e[33;1m\]\u\[\e[31;1m\]@\[\e[33;1m\]\h \[\e[36;1m\]\w\[\e[34;1m\]\$ \[\e[0m\]"

## 带脚本的主题

如果需要更多的功能，则可以使用 bash 本身的功能来实现，如我们需要以下功能：

- 如果上一行成功执行，则开始的$为绿色，否则为红色
- 如果当前目录是一个 Git 项目，则显示当前的分支

首先，我们先不考虑PS1，在 Bash 中如何实现上面的功能呢？

前者实际上只是一个判断，判断$?是否为0即可

```C
if [ $? -eq 0 ]; then 
    echo -e '\[\e[1;32;40m\]Success\[\e[0m\]'; 
else
    echo -e '\[\e[1;31;40m\]Fail\[\e[0m\]';
fi
```

这里\[和\]在 echo 中未被转义，不过并不影响结果。
而后者

branch=$(git status 2>/dev/null | head -n 1 | cut -d " " -f 3 2>/dev/null); \
if [[ -n ${branch} ]]; then
    echo -e "@\[\e[1;91m\]${branch}\[\e[0m\]";
fi

综上所述，如果PS1的部分代码本身使用$()或反引号包裹，那么将会实时执行，从而实现对动态内容的展示。

最后的代码合起来为

export PS1='$(if [ $? -eq 0 ]; then echo -ne "\[\e[1;32m\]$\[\e[0m\]"; else echo -ne "\[\e[1;31m\]$\[\e[0m\]"; fi) \[\e[95m\]\t\[\e[0m\] \[\e[33m\]\u\[\e[0m\]@\[\e[96m\]\H\[\e[0m\]:\[\e[36m\]\w\[\e[0m\]$(branch=$(git status 2>/dev/null | head -n 1 | cut -d " " -f 3 2>/dev/null);if [[ -n ${branch} ]]; then echo -ne "@\[\e[1;91m\]${branch}\[\e[0m\]";fi) \[\e[1;37m\]>\[\e[0m\] '

## 更优雅的生成方式
上述内容在使用上没有任何问题，但是在维护上，问题很大。更好的办法是使用脚本生成PS1语句，这样会有更高的可维护性。

通过对各种颜色的显示进行封装，每部分单独处理，最后拼接到一起，这样可以使得项目维护性更高。

下面的代码可以在source后，在当前终端配置PS1，同时会输出一份可以复制的脚本，可以粘贴到.bashrc中，以便后续自动生成

```python
fg_color() {
    case "$1" in
        black)          echo 30;;
        red)            echo 31;;
        green)          echo 32;;
        yellow)         echo 33;;
        blue)           echo 34;;
        magenta)        echo 35;;
        cyan)           echo 36;;
        white)          echo 37;;
        lightBlack)     echo 90;;
        lightRed)       echo 91;;
        lightGreen)     echo 92;;
        lightYellow)    echo 93;;
        lightBlue)      echo 94;;
        lightMagenta)   echo 95;;
        lightCyan)      echo 96;;
        lightWhite)     echo 97;;
        orange)         echo 38\;5\;166;;
        *)              echo $1;;
    esac
}

bg_color() {
    case "$1" in
        black)          echo 40;;
        red)            echo 41;;
        green)          echo 42;;
        yellow)         echo 43;;
        blue)           echo 44;;
        magenta)        echo 45;;
        cyan)           echo 46;;
        white)          echo 47;;
        orange)         echo 48\;5\;166;;
        lightBlack)     echo 100;;
        lightRed)       echo 101;;
        lightGreen)     echo 102;;
        lightYellow)    echo 103;;
        lightBlue)      echo 104;;
        lightMagenta)   echo 105;;
        lightCyan)      echo 106;;
        lightWhite)     echo 107;;
        *)              echo $1;;
    esac;
}

text_effect() {
    case "$1" in
        reset)      echo 0;;
        bold)       echo 1;;
        weak)       echo 2;;
        italic)     echo 3;;
        underline)  echo 4;;
        blink)      echo 5;;
        quickBlink) echo 6;;
        reverse)    echo 7;;
        hide)       echo 8;;
        del)        echo 9;;
        *)          echo $1
    esac
}

color() {
    font=$(text_effect $1)
    fg=$(fg_color $2)
    bg=$(bg_color $3)

    code=""
    first=0
    for c in $font $fg $bg
    do 
        if [[ $c -ne "" ]]; then
            if [[ $first -ne 0 ]]; then
                code+=";"
            fi
            code+="$c"
            first=1
        fi
    done

    
    text=$4

    echo "\[\e[${code}m\]${text}\[\e[0m\]"
}


# export PS1='`a=$?; if [ $a -eq 0 ]; then echo -ne "\[\e[32;1m\]\$\[\e[0m\]"; else echo -ne "\[\e[31;1m\]\$\[\e[0m\]"; fi` \[\e[35;1m\]\t\[\e[0m\] \[\e[1;41m\]▶\[\e[0m\] \[\e[1;41m\]\u@\H:\w\[\e[0m\]  \[\e[1;41m\]▶\[\e[0m\]\$?'

prefix_ok="$(color bold green "" \$)"
prefix_fail="$(color bold red "" \$)"

prefix='$(if [ $? -eq 0 ]; then echo -ne '"\"${prefix_ok}\""'; else echo -ne '"\"${prefix_fail}\""'; fi)'
tm="$(color "" lightMagenta "" \\t)"
host="$(color "" lightCyan "" \\H)"
user="$(color "" yellow "" \\u)"
dir="$(color "" cyan "" \\w)"

git='$(branch=$(git status 2>/dev/null | head -n 1 | cut -d " " -f 3 2>/dev/null);if [[ -n ${branch} ]]; then echo -ne "@\[\e[1;91m\]${branch}\[\e[0m\]";fi)'

P="${prefix} ${tm} ${user}@${host}:${dir}${git} $(color bold white "" \>) "

echo "export PS1='${P}'"
echo "export PS1='${P}'" > theme.bash
export PS1=$P
```
