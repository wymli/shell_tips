# shell_tips

1. #!bin/env bash
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
#!/bin/env bash
set -ex
cd `dirname $0`
```
21. 反引号`等同于$(),用于执行命令
22. source ./utils.sh 用于将其他脚本加载到当前环境
23. 一个单独的echo即可换行
24. 

