# shell_tips

1. #!/usr/bin/env bash
2. bash 和 sh 是不同的,在一些命令的处理上会有区别,所以不要无脑用 `sh ./run.sh` 去执行别人写的shell
3. set -ex 比较常用, -e 代表 error则退出, -x 代表输出执行的命令本身
4. wget不支持socks5代理协议,所以直接用curl吧
5. curl -fsSL 常用, -L表示跟随重定向, -f表示Fail silently (no output at all) on HTTP errors主要是为了在服务端错误时让curl命令直接返回错误,默认情况可能输出表示错误的html,此时curl命令本身可能不是错误的, -s 表示silent,不输出进度条+错误信息, -S表示Show error even when -s is used
6. curl -o minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 用于将输出写到文件
7. curl 默认将输出写到/dev/stdin,而wget默认写到文件
8. 下面几条等价:
9. curl -fsSL get.docker.com | sudo bash /dev/stdin --mirror Aliyun (不如直接显示指明从stdin读输入)
10. curl -fsSL get.docker.com | sudo bash -s -- --mirror Aliyun
11. curl -fsSL get.docker.com | sudo bash -s 123 (如果脚本本身没有命名参数,那么就不需要--,--用于区分sh本身的参数还是要执行脚本的参数)
12. bash -s 表示从stdin读输入,和 `bash -` 一个意思
13. 但是`bash -` 不能接参数,所以一般是 `curl -fsSL get.docker.com | bash -`,但这个其实等价于 `curl -fsSL get.docker.com | bash`
14. 一定要注意"" 和 '' 和 不加任何引号的区别!
15. "" 双引号会直接对内部的变量等等特殊符号解析渲染,比如 addr=123 && echo "$addr"将会输出123
16. '' 表示最原本的字符串,不会对内部的字符做任何解析渲染,比如 addr=123 && echo "$addr"将会输出$addr字面量,所以如果你想模拟将一段命令输出到stdout然后传给sh的stdin,注意一定要用单引号,比如`echo 'echo "$0 $1 $2"' | bash /dev/stdin arg1test arg2test`
17. 不加引号的话,所以的空白字符(无论是变量内部的,还是变量之间的)都会变成一个`空格`,这意味着换行也会变成空格! 比如echo $a 如果变量$a是"asdf<换行>asdf",那么实际的输出结果是"asdf asdf" 注意这里换行不能用\n,如果想模拟的话,可以这样:a="asdf回车asdf" 
18. a=1 b=2 && echo $a          $b 最终输出1 2,多空格变成一个空格,如果$a本身是"1换行2",那么输出1 2 2,变量内部字符串的空白字符也都被替换成单空格
19. 在echo 一些命令的结果时一定要加单引号,避免把输出变成一行,使之按原样输出...
```shell
liwm29@wymli-NB1:~/bc_sns/script$ a="11
> asdf"
liwm29@wymli-NB1:~/bc_sns/script$ echo $a
11 asdf
liwm29@wymli-NB1:~/bc_sns/script$ echo '$a'
$a
liwm29@wymli-NB1:~/bc_sns/script$ echo "$a"
11
asdf
```
20. cd `dirname $0` 将pwd换成脚本所在的路径,用于使用相对路径,一般脚本开头这样写:
```shell
#!/usr/bin/env bash
set -ex
cd `dirname $0`
```
21. 反引号`等同于$(),用于执行命令
22. source ./utils.sh 用于将其他脚本加载到当前环境
23. 一个单独的echo即可换行
24. 好网站: https://explainshell.com/
25. 在https://github.com/rancher/k3d/blob/main/install.sh 看到的脚本,mark一下
```shell
#!/usr/bin/env bash

APP_NAME="k3d"
REPO_URL="https://github.com/rancher/k3d"

: ${USE_SUDO:="true"}
: ${K3D_INSTALL_DIR:="/usr/local/bin"}

# initArch discovers the architecture for this system.
initArch() {
  ARCH=$(uname -m)
  case $ARCH in
    armv5*) ARCH="armv5";;
    armv6*) ARCH="armv6";;
    armv7*) ARCH="arm";;
    aarch64) ARCH="arm64";;
    x86) ARCH="386";;
    x86_64) ARCH="amd64";;
    i686) ARCH="386";;
    i386) ARCH="386";;
  esac
}

# initOS discovers the operating system for this system.
initOS() {
  OS=$(uname|tr '[:upper:]' '[:lower:]')

  case "$OS" in
    # Minimalist GNU for Windows
    mingw*) OS='windows';;
  esac
}

# runs the given command as root (detects if we are root already)
runAsRoot() {
  local CMD="$*"

  if [ $EUID -ne 0 -a $USE_SUDO = "true" ]; then
    CMD="sudo $CMD"
  fi

  $CMD
}

# verifySupported checks that the os/arch combination is supported for
# binary builds.
verifySupported() {
  local supported="darwin-386\ndarwin-amd64\ndarwin-arm64\nlinux-386\nlinux-amd64\nlinux-arm\nlinux-arm64\nwindows-386\nwindows-amd64"
  if ! echo "${supported}" | grep -q "${OS}-${ARCH}"; then
    echo "No prebuilt binary for ${OS}-${ARCH}."
    echo "To build from source, go to $REPO_URL"
    exit 1
  fi

  if ! type "curl" > /dev/null && ! type "wget" > /dev/null; then
    echo "Either curl or wget is required"
    exit 1
  fi
}

# checkK3dInstalledVersion checks which version of k3d is installed and
# if it needs to be changed.
checkK3dInstalledVersion() {
  if [[ -f "${K3D_INSTALL_DIR}/${APP_NAME}" ]]; then
    local version=$(k3d version | grep 'k3d version' | cut -d " " -f3)
    if [[ "$version" == "$TAG" ]]; then
      echo "k3d ${version} is already ${DESIRED_VERSION:-latest}"
      return 0
    else
      echo "k3d ${TAG} is available. Changing from version ${version}."
      return 1
    fi
  else
    return 1
  fi
}

# checkTagProvided checks whether TAG has provided as an environment variable so we can skip checkLatestVersion.
checkTagProvided() {
  [[ ! -z "$TAG" ]]
}

```
26. `: ${USE_SUDO:="true"}` 这里${}内部好理解,就是给变量$USE_SUDO一个默认值,如果$USE_SUDO为空,就将其设置为true, : ${} 这里的冒号:代表不执行这条指令,如果不加冒号,一个单独的${}独列一行,shell会默认将其认为是一个命令去执行,但是这里我们的期望其实是单纯的赋值,所以前面加colon,看文档里面,This utility shall only expand command arguments. 仅仅用于`参数拓展`
27. : 常见的用法有 a. : comment b. 放在if then else语句里面类似python的pass c. : >file 清空文件 d. : ${a:=123} 如果变量空,就给变量赋默认值
28. `ARCH=$(uname -m)` 可以看到,-m是machine的意思,所以架构用-m ,事实上,-m machine -p processor -h hardware 在一般的pc上都是一样的,可能在android,stawberry等特殊的平台上会不同
29. 架构的类型在代码上表示的很清楚了,常见的就是amd64(x86_64,x64)x86(386),arm64,arm,前面两个是intel的64位和32位机器,后面两个是arm的64和32位机,注意每种类型别名挺多的!
30. 架构arch可以认为是cpu指令集的区别,`OS=$(uname|tr '[:upper:]' '[:lower:]')`,软件除了涉及指令集,还是os内核相关的.
31. `tr`命令: Translate, squeeze, and/or delete characters from standard input, writing to standard output. `tr [:lower:] [:upper:] ` 表示把所有小写字母转换成大写,等价于`tr a-z A-Z`
32. `local CMD="$*"` local命令用于函数内部定义局部变量,否则默认的变量声明赋值是全局的. `$*`代表脚本所有的输入参数,也就是$1 $2 ...,不包括$0, 使用时:`runAsRoot chmod +x "$K3D_TMP_FILE"` 相当于`sudo chmod +x "$K3D_TMP_FILE"`
33. `if ! echo "${supported}" | grep -q "${OS}-${ARCH}"; then` ,echo用于把${supported}送到stdout,进而进到grep的stdin, grep -q 表示quiet,不输出匹配结果.  `If the reserved word ! precedes a pipeline, the exit status of that pipeline is the logical negation of the exit status as described above. ` 这里注意的是exit status中,0是成功,非0是失败,而grep中匹配到了是0,没有匹配到是1,所以匹配到了那么就会进入else,没匹配到就会进入then
34. $? 指示上一条命令的退出状态,比如 exit 0
35. if判断中,字符串判断和数字判断是不同命令,-eq,-gt,-lt,-ge,-le 用于数字, =,!=,>,<用于字符串, -z,-n也是判断字符串是否空
36. 我理解shell中一切都是字符串,用-eq这些只是用数字来解析字符串,所以`if [ "$?" -eq "0" ]` 即使加上双引号也是没问题的,注意`[]`和内部的判断语句之间一定有空格,就像赋值语句一定不能有空格一样`a=1而不能a = 1`
37. 所以`if [ "$?" -eq "0" ]` 最好加上双引号,而不是`if [ $? -eq 0 ]` 虽然这里没有问题,但如果是`if [ $a = asdf ];then echo q;fi;`就报错,必须`if [ "$a" = asdf ];then echo q;fi;`,所以不如两边都直接加引号
38. `[ "$(whoami)" != 'root' ] && ( echo you are using a non-privileged account; exit 1 )` 在bash中可以使用上面的用法,相当于if then的语法糖
39. `if ! type "curl" > /dev/null && ! type "wget" > /dev/null; then` 这里 `type`就是用来判断命令是否存在,type本意是判断命令的类型是内置命令还是外部命令.与之相似的还有which,hash都可以用来判断命令是否存在;  > /dev/null 是为了不显示输出,如果命令本身有quiet选项也可以指定-q
40. `[[ ]]`是bash的,在`[[和]]`之间所有的字符都不会发生文件名扩展或者单词分割，但是会发生参数扩展和命令替换
41. `[]`等价于test,比如test -n "$a" 相当于 `[ -n "$a"]`
42. 使用test或`[]`时,必须加上双引号,比如`[ -n "$a" ]`,但是`[[]]`就可以`[[ -n $a ]]`

```shell
liwm29@wymli-NB1:~/bc_sns/deploy$ a=
liwm29@wymli-NB1:~/bc_sns/deploy$ test -n "$a"
liwm29@wymli-NB1:~/bc_sns/deploy$ echo $?
1
liwm29@wymli-NB1:~/bc_sns/deploy$ test -n $a
liwm29@wymli-NB1:~/bc_sns/deploy$ echo $?
0
liwm29@wymli-NB1:~/bc_sns/deploy$ [[ -n "$a" ]]
liwm29@wymli-NB1:~/bc_sns/deploy$ echo $?
1
liwm29@wymli-NB1:~/bc_sns/deploy$ [[ -n $a ]]
liwm29@wymli-NB1:~/bc_sns/deploy$ echo $?
1
```
43. 注意shell中,0是true/success,1是false/fail
44. `参数扩展`就是通过符号$获得参数中存储的值,可以认为${} 内部发生的所有操作都是参数拓展,比如引用${a},赋值默认值${a:=1},替换${a//pattern/string},字符串长度${#a},大小写转换${parameter,,}${parameter,}${parameter^^}${parameter^} 一个字符^,就是首字母大写或小写,两个字符^^,,就是全部大小写; tr命令也可以达到相同的效果
45. a="a.exe"    ${a//.exe/} 则删去所有.exe  ${a//.exe/.123}将.exe替换成.123 注意不是正则
46. () 用于声明数组, `a=(1 2 3)`,   数组长度:`echo ${#a[*]}` 遍历数组: `for ((i = 0; i < ${#a[*]}; i++));do echo ${a[$i]};done` `for i in ${a[*]};do echo $i;done`; 注意不能是for i in $a,意味$a仅仅表示a数组的第一个元素,必须`${a[*]}或${a[@]}`
47. 如果for语句中是循环一个字符串,那么是按newline换行分隔,比如`for i in $(ls);do echo $i;done;` 这里ls命令的输出就是一行一行的字符串
48. `checkTagProvided() {[[ ! -z "$TAG" ]]}` 函数如果没有返回值,就默认返回最后一条语句的返回值,也就是这里对$TAG字符串判空的判断
49. `pushd`,`popd`  用于切换目录, 相比于`cd`的优点是能回到原目录,且因为是模拟堆栈,可以回溯多次. 而`cd -`就只能回最近访问的目录.
50. `pushd $dir` 进入$dir, `popd` 返回上一次访问的目录
51. docker in docker ,一般分为在docker中启动sibling docker,还是在docker中启动child docker. 如果是兄弟docker,只需要把host上的docoker daemon用于ipc的uds(unix domain socket)mount到docker中即可,然后借助docker cli(也需要mount)或者官方提供的docker sdk来运行容器. 如果是启动child docker,就需要在docker中运行一个新的docker daemon,此时新运行的docker对host是不可见的,这种隔离性是一些ci程序所期望的.
