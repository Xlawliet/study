# Shell

是一个==命令行解释器==，它接受应用程序/用户命令，然后调用操作系统内核。

Shell还是一个功能相当强大的编程语言，易编写，易调试，灵活性强；

## Shell常用解析器

sh

bash 系统默认的shell解析器



## Shell脚本入门

#### 脚本格式

以#!/bin/bash 开头（指定解析器）



## Shell中的变量

### 系统变量

#### 常用系统变量

$HOME 

$PWD

$SHELL

$USER

### 自定义变量

#### 定义变量 ：

可以由字母、数字、下划线组成，但是不能以数字开头，==环境变量名建议大写==

等号两侧不能带空格

在bash中，变量默认类型都是字符串类型，无法直接进行数值运算

**变量=值** （中间无法带等号）

**撤销变量**：unset

**声明静态变量**： readonly  变量=值    无法撤销

将变量**提升为全局环境**变量： export 变量



### 特殊变量

$n （功能描述：n为数字，$0代表该脚本名称，$1-$9代表第一到第九个参数，十以上的参数需要用大括号包含如 ${10}）

$#  功能描述：获取所有输入参数的个数，常用于循环

$*  这个变量代表命令行中所有的参数，$*所有的参数看成一个整体

$@ 这个变量也代表命令行中所有的参数，不过$@把每个参数区分对待

$? 最后一次执行的命令的返回状态  如果为0 说明上一个命令正确执行，非0则执行不正确



## 运算符

### 基本语法

1. $((运算式)) 或 $[运算式]
2. expr +,-,\*,/,% 加 减 乘 除 取余  expr运算符间要有空格

### 条件判断

####  [ condition ]  

（注意condition前后要有空格）      注意条件非空即为true，[atguigu] 返回true ,[] 返回 false

#### 常用判断条件

两个整数之间比较：

**= 字符串比较**

-lt 小于（less than）                 -le 小于等于 (less equal)

-eq 等于 (equal)                        -gt 大于(greater than)

-ge 大于等于( greater equal )    -ne 不等于(Not equal)

**按照文件权限进行判断**

-r  有读的权限(read)                 -w  有写的权限(write)

-x 有执行的权限(execute)

**按照文件类型进行判断**

-f 文件存在并且是一个常规的文件（file）

-e 文件存在（existence）                      -d 文件存在并是一个目录 （directory）

​                                           

## 条件判断语句

### 基本语法

#### if

```bash
if [ 程序判断式 ];then
	程序
elif[ 程序判断式 ];then
	程序

或者

if [ 条件判断式 ]
then
	程序
elif [ 程序判断式 ]
	程序
```

注意事项：

1. [ 条件判断式 ] ， 中括号和条件判断式之间必须又空格
2. **if后要有空格**



#### case

```bash
case $变量名 in
"值1")
	如果变量的值等值1，则执行程序1
	;;  # break
"值2")
	如果变量的值等值2，则执行程序2
	;;
......省略其他分支
 *) default语句
 	如果变量的值都不是以上的值，则执行此程序
 	;;
 esac
```

注意事项：

1. case行尾必须为单词in，每个模式匹配必须以右括号")"结束。
2. 双分号;; 表示命令序列结束 相当于java中的break
3. 最后的*)表示为默认模式，相当于java中的default



#### for循环

```bash
语法一：
for((初始值;循环控制条件;变量变换))
do
程序
done


语法二：
for 变量 in 值1 值2 值3...
do
	程序
done
将变量用""扩起后，使其成为整体
如果变量用$* 则会将输入的变量合并为一个变量
如果变量用$@ 则仍然是逐个赋值并逐个运行

```



#### while循环

```bash
基本语法
while[ 条件判断式 ]
do
	程序
done

s=0
i=1
while[ $i -le 100 ]
do
	s=$[$s+$i]
	i=$[$i+1]
done
```





## read读取控制台输入

read(选项)(参数)

选项：  

-p：指定读取值时的提示符；

-t：指定读取值时等待的时间（秒）。

参数：

​	变量：指定读取值的变量名

```bash
//提示7秒内，读取控制台输入的名称
read -t 7 -p "Enter your name in 7 seconds " NAME
echo $NAME

```



## 函数

### 系统函数

basename 基本语法

```bash
#basename[string/pathname][suffix] （功能描述：basename命令会删掉所有的前缀包括最后一个（‘/’）字符，然后将字符串显示出来）。
#选项： suffix为后缀，如果suffix被指定了，basename会将pathname或string中的suffix去掉。

例子：
basename /home/atguigu/banzhang.txt
得到 banzhang.txt

basename /home/atguigu/banzhang.txt .txt
得到 banzhang
```



dirname 基本语法

```bash
dirname 文件绝对路径   （功能描述：从给定的包含绝对路径的文件名中去除文件名（非目录部分），然后返回剩下的路径（目录的部分））

例如：
获取banzhang.txt文件的路径
dirname /home/atguigu/banzhang.txt
得到 /home/atguigu
```



### 自定义函数

```bash
基本语法
[ function ]funname[()]
{
	Action;
	[return int;]
}
funname

例子：
计算两个输入参数的和

function sum()
{
	s=0
	s=$[$1+$2]
	echo $s
}

read -p "input your paratemer1:"P1
read -p "input your paratemer2:"P2

sum $P1 $P2

```

1. 必须再调用函数地方之前，先声明函数，shell脚本是逐行运行的。不会像其他语言一样先编译。
2. 函数返回值，只能通过$？系统变量获得，可以显示加：return返回，如果不加，将以最后一条命令运行结果，作为返回值。return后跟数值n(0-255)



## Shell工具（重点）

### cut

cut的工作就是"剪"，在文件中负责剪切数据用的。cut命令从文件的每一行剪切字节、字符和字段并将这些字节、字符、字段输出

```bash
基本语法
cut[选项参数] filename
-f 列号，提取第几列
-d 分隔符，按照指定分隔符分割列 （默认分隔符为制表符）

#数据
touch cut.txt
vim cut.txt
shang bei
hai jing
wo  wo（此处两个空格）
lai  lai（此处两个空格）
le  le（此处两个空格）

获取第一列的操作：
cut -d " " -f 1 cut.txt
输出：
shang
hai
wo
lai
le

获取第二列、第三列的操作：
cut -d " " -f 2，3 cut.txt
bei
jing
 wo
 lai
 le
 
获取shang这个字符的操作：
cat cut.txt| grep shang    （读取出以shang开头的行）
输出
shang bei

cat cut.txt| grep shang| cut -d " " -f 1
输出
shang

选取系统PATH变量值，第2个“：”开始后的所有路径：
echo $PATH
/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/home/xzy/.local/bin:/home/xzy/bin

echo $PATH | cut -d : -f 3- （选出第三列以后的所有列）
/usr/local/sbin:/usr/sbin:/home/xzy/.local/bin:/home/xzy/bin

切割ifconfig后打印的ip地址：
ifconfig eth0|grep "inet" | cut -d : -f 2
```



### sed

是一种流编辑器，它一次处理一行内容。处理时，把当前处理的行存储在临时缓冲区中，称为“模式空间”，接着用sed命令处理缓冲区中的内容，处理完成后，把缓冲区的内容送往屏幕。接着处理下一行，这样不断重复，直到文件末尾。==文件内容并没有改变==，除非使用重定向存储输出。

```bash
sed[选项参数] ‘command’ filename
选项参数说明：
-e 直接在指令列模式上进行sed的动作编辑。

命令功能描述
a	新增，a的后面可以接字符串，在下一行出现
d	删除
s	查找并替换

数据准备：
touch sed.txt
vim sed.txt
shang bei
hai jing
wo  wo（此处两个空格）
lai  lai（此处两个空格）

le  le（此处两个空格）

将mei nv这个单词插入到sed.txt第二行下，打印：
sed "2a mei nv" sed.txt   (表示在第二行新增mei nv)
输出：
shang bei
mei nv
hai jing
wo  wo
lai  lai

le  le
但是源文件并不会改变

删除sed文件中包含有wo的行
sed "/wo/d" sed.txt
输出
shang bei
hai jing
lai  lai

le  le

将sed中wo替换为ni
sed "s/wo/ni/g" sed.txt  （global全部替换）
shang bei
hai jing
ni  ni
lai  lai

le  le

将sed文件中的第二行删除并将wo替换为ni
sed -e "2d" -e "s/wo/ni/g" sed.txt
输出
shang bei
ni  ni
lai  lai

le  le
```



### awk

一个强大的文本分析工具，把文件逐行的读入，以空格为默认分隔符将每行切片，切开的部分再进行分析处理。

```bash
awk[选项参数] ‘pattern1{action1} pattern2{action2}...’ filename
patter：表示awk在数据中查找的内容，就是匹配模式
action：在找到匹配内容时所执行的一系列命令
选项参数：
-F：指定输入文件拆分隔符
-v：赋值一个用户定义变量 

查询sed.txt中空行所在的行号
awk '/^${print NR}' sed.txt
```



### sort

将文件进行排序，并将排序结果标准输出

```bash
sort(选项)(参数)
选项：
-n：依照数值的大小进行排序
-r：以相反的顺序来排序
-t：设置排序时所用的分割字符
-k：指定需要排序的列

数据准备：
touch sort.sh
vim sort.sh
bb:40:5.4
bd:20:4.2
xz:50:2.3
cls:10:3.5
ss:30:1.6

按照第二列进行排序：
sort -t : -nrk 2 sort.sh
输出：
xz:50:2.3
bb:40:5.4
ss:30:1.6
bd:20:4.2
cls:10:3.5
```



例题：

```bash
由chengji.txt内容如下：
张三 40
李四 50
王五 60
使用linux命令计算第二列的和并输出
cat chengji.txt | awk -F " " '{sum+=$2} END{print sum}'
输出 150

shell脚本中如何检查一个文件是否存在，不存在该如何处理
#!/bin/bash

if [ -f file.txt ];
then
	echo "Exist！"
elif
	echo "Not Exist"
fi

使用shell脚本写出查找当前文件夹（/home）下所有的文本文件内容中含有字符"shen"的文件名称
grep -r "shen" /home | cut -d ":" -f 1
```











