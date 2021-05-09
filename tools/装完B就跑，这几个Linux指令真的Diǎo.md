![在这里插入图片描述](https://img-blog.csdnimg.cn/2020040517314897.gif)
### 2.1 sl
先看一下呼啸而过的火车；
安装指令如下；
```bash
sduo apt-get install sl
```
执行结果如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201124092931937.gif)
### 2.2 htop
图形化Linux系统性能监测工具，屌不屌；

安装指令如下；
```bash
sduo apt-get install htop
```
执行结果如下；
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201124093014523.gif)
### 2.3 gcp
以前用cp复制文件总是看不懂进度，现在它来了，`gcp`给你解决烦恼；

安装指令如下；
```bash
sudo apt-get install gcp
```
执行结果如下；![在这里插入图片描述](https://img-blog.csdnimg.cn/202011240930533.gif)
### 2.4 hollywood
无形装逼，最为致命，这个指令可以满足你们的需求，且看；

安装指令如下；
```bash
sudo apt-get install hollywood
```
执行结果如下；![在这里插入图片描述](https://img-blog.csdnimg.cn/20201124093140541.gif)
### 2.5 cmatrix
看过黑客帝国没，今天看一下一条指令撸字符雨；

安装指令如下；
```bash
sudo apt-get install cmatrix
```
执行结果如下；![在这里插入图片描述](https://img-blog.csdnimg.cn/2020112409324727.gif)
### 2.6 asciiview
黑与白的世界依然可以五彩缤纷，图片字符串化。

安装指令如下；
```bash
sudo apt-get install aview imagemagick
asciiview kobe.jpg
```
执行结果如下；![在这里插入图片描述](https://img-blog.csdnimg.cn/20201124093324680.gif)
### 2.7 ninvaders
上班划个水，太空侵略者玩一下也不错；

安装指令如下；
```bash
sudo apt-get install ninvaders
```
执行结果如下；
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201124093404358.gif)
### 2.8 bastet
杰哥：游戏暂时介绍两个吧，玩物丧志，俄罗斯方块了解一下；

安装指令如下；
```bash
sudo apt-get install bastet
```
执行结果如下；
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201124093434759.gif)
### 2.9 pipe
手撸个pipe脚本，各种绘制，炫不炫？

脚本如下所示；
```bash
p=1
f=75 s=13 r=2000 t=0
w=$(tput cols) h=$(tput lines)
# ab -> idx = a*4 + b
# 0: up, 1: right, 2: down, 3: left
# 00 means going up   , then going up   -> ┃
# 12 means going right, then going down -> ┓
sets=(
    "┃┏ ┓┛━┓  ┗┃┛┗ ┏━"
    "│╭ ╮╯─╮  ╰│╯╰ ╭─"
    "│┌ ┐┘─┐  └│┘└ ┌─"
    "║╔ ╗╝═╗  ╚║╝╚ ╔═"
)
v="${sets[0]}"
RNDSTART=0
NOCOLOR=0

OPTIND=1
while getopts "p:t:f:s:r:RCh" arg; do
case $arg in
    p) ((p=(OPTARG>0)?OPTARG:p));;
    t) ((OPTARG>=0 && OPTARG<${#sets[@]})) && v="${sets[OPTARG]}";;
    f) ((f=(OPTARG>19 && OPTARG<101)?OPTARG:f));;
    s) ((s=(OPTARG>4 && OPTARG<16 )?OPTARG:s));;
    r) ((r=(OPTARG>=0)?OPTARG:r));;
    R) RNDSTART=1;;
    C) NOCOLOR=1;;
    h) echo -e "Usage: $(basename $0) [OPTION]..."
        echo -e "Animated pipes terminal screensaver.\n"
        echo -e " -p [1-]\tnumber of pipes (D=1)."
        echo -e " -t [0-$((${#sets[@]} - 1))]\ttype of pipes (D=0)."
        echo -e " -f [20-100]\tframerate (D=75)."
        echo -e " -s [5-15]\tprobability of a straight fitting (D=13)."
        echo -e " -r LIMIT\treset after x characters, 0 if no limit (D=2000)."
        echo -e " -R \t\trandom starting point."
        echo -e " -C \t\tno color."
        echo -e " -h\t\thelp (this screen).\n"
        exit 0;;
    esac
done

cleanup() {
    tput rmcup
    tput cnorm
    exit 0
}
trap cleanup SIGHUP SIGINT SIGTERM

for (( i=1; i<=p; i++ )); do
    c[i]=$((i%8)) n[i]=0 l[i]=0
    ((x[i]=RNDSTART==1?RANDOM*w/32768:w/2))
    ((y[i]=RNDSTART==1?RANDOM*h/32768:h/2))
done

tput smcup
tput reset
tput civis
while ! read -t0.0$((1000/f)) -n1; do
    for (( i=1; i<=p; i++ )); do
        # New position:
        ((${l[i]}%2)) && ((x[i]+=-${l[i]}+2,1)) || ((y[i]+=${l[i]}-1))

        # Loop on edges (change color on loop):
        ((${x[i]}>w||${x[i]}<0||${y[i]}>h||${y[i]}<0)) && ((c[i]=RANDOM%8))
        ((x[i]=(x[i]+w)%w))
        ((y[i]=(y[i]+h)%h))

        # New random direction:
        ((n[i]=RANDOM%s-1))
        ((n[i]=(${n[i]}>1||${n[i]}==0)?${l[i]}:${l[i]}+${n[i]}))
        ((n[i]=(${n[i]}<0)?3:${n[i]}%4))

        # Print:
        tput cup ${y[i]} ${x[i]}
        [[ $NOCOLOR == 0 ]] && echo -ne "\033[1;3${c[i]}m"
        echo -n "${v:l[i]*4+n[i]:1}"
        l[i]=${n[i]}
    done
    ((r>0 && t*p>=r)) && tput reset && tput civis && t=0 || ((t++))
done

cleanup
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201124093518958.gif)
### 2.10 oneko
养只猫，你们看看，可以追着鼠标跑。

安装指令如下；
```bash
sudo apt-get install oneko
```
执行结果如下；![在这里插入图片描述](https://img-blog.csdnimg.cn/20201124093551473.gif)
### 2.11 炸弹
怎么样？怎么样！！牛不牛逼？Diao不Diao？上面的都见过是吧？给你来个更牛的！
```bash
:(){ :|:& };:
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201124093754604.gif)
结果：`系统奔溃`了，刚刚跑了一个fork炸弹，耗尽了系统的资源，然后就死机了。`fork` 炸弹之后输入`ps`指令，发现已经耗尽了系统的进程，无法再进行fork得到子进程，得到的执行结果如下所示；
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201124093914473.png)
### 2.12 删库跑路
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201124093957334.gif)





