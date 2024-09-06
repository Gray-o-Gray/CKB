# Shell语言编程

!!!

在shell中，true表示值为0，false为非零值！

## **shell变量**

定义变量时，变量名不加美元符号。

```bash
Your_name="runoob.com"
```

**注意点：**

变量名和等号之间不能有空格

命名只能使用英文字母、数字和下划线。首个字符不能以数字开头

不能使用标点符号

不能使用bash里的关键字

for file in `ls /etc`或for file in $(ls /etc) 循环赋值

### 使用变量

使用一个定义过的变量： `$your_name` 和 `${your_name}`

花括号是为了帮助解释器识别变量的边界

```bash
echo "I am good at ${skill}Script"  # I am good at ****Script
echo "I am good at $skillScript"    # I am good at 
								    # 原因为$skill被错误识别为了$skillScript
									# $skillScript未定义，因此是一个空值 
```

建议都加{}

已定义的变量可以被重新赋值

```bash
readonly var  # readonly将变量定义为只读变量，只读变量的值不能被改变
unset var     # unset删除变量，删除后使用echo输出将没有任何输出。unset不能删除只读变量
```

### 字符串

字符串可以用单引号，也可以用双引号，也可以不用引号

单引号的限制：单引号里的任何字符都会原样输出，单引号字符串中的变量是无效的

单引号字串中不能出现单独一个的单引号（对单引号使用转义符后也不行），但可以成对出现，作为字符串拼接使用。

双引号：双引号里可以有变量

双引号里可以出现转义字符

字符串拼接尽量使用双引号，因为单引号中的变量是无效的。

```bash
Greeting="hello, "$your_name" !"
Greeting_1="hello, $your_name !"
# 两者输出相同
```

获取字符串长度`${#string}`，若string为数组时，`${#string}`等价于`${#string[0]}`

提取字符串`${string:index:len}`，`index`为索引（从0计数），`len`为提取的长度

查找子字符串：

```bash
String="runoob is a great site"
Echo `expr index "$string" io` # 输出4, 注意是`（反引号），不是’（单引号）
```

### 数组

只能有一维数组

```bash
# 定义 
array_name=(val0 val1 val2 val3)
Array_name[0]
```

获取数组的长度

获取数组元素的个数`${#array_name[@]}`

获取数组元素的个数`${#array_name[*]}`

获取数组单个元素的长度`${#array_name[n]}`

## 传递参数

在执行shell脚本时，可以向脚本传递参数，脚本内获取参数的格式为：$n。

n代表一个数字，1为执行脚本的第一个参数，2为执行脚本的第二个参数，以此类推。

用来处理参数的特殊字符

| 参数处理| 说明|
| --- | --- |
| $#| 传递到脚本的参数个数|
| $*| 以一个单字符串显示所有向脚本传递的参数。如"$*"用「"」括起来的情况、以"$1 $2 … $n"的形式输出所有参数。|
| $$| 脚本运行的当前进程ID号|
| $!| 后台运行的最后一个进程的ID号|
| $@| 与$*相同，但是使用时加引号，并在引号中返回每个参数。如"$@"用「"」括起来的情况、以"$1" "$2" …"$n" 的形式输出所有参数。|
| $-| 显示Shell使用的当前选项，与set命令功能相同。|
| $?| 显示最后命令的退出状态。0表示没有错误，其他任何值表明有错误。|

* *$* 与 $@ 区别：

相同点：都是引用所有参数。

不同点：只有在双引号中体现出来。假设在脚本运行时写了三个参数 1、2、3，，则 " * " 等价于 "1 2 3"（传递了一个参数），而 "@" 等价于 "1" "2" "3"（传递了三个参数）。

## 数组

读取数组元素值：`${array_name[index]}`

关联数组：

声明：declare -A array_name

-A 选项就是用于声明一个关联数组

关联数组的键是唯一的

```bash
# 实例1
declare -A site=(["google"]="www.google.com", ["runoob"]="www.runoob.com", ["taobao"]="www.taobao.com")

# 实例2：
declare -A site
site["google"]="www.google.com"
site["runoob"]="www.runoob.com"
site["taobao"]=www.taobao.com
```

使用 @ 或 * 可以获取数组中的所有元素

`${my_array[*]}` / `${my_array[@]}`

在数组前加一个感叹号 ! 可以获取数组的所有键

`${!site[*]}` / `${!site[@]}`

## 基本运算符

### 算术运算符

| 运算符| 说明| 举例|
| --- | --- | --- |
| +| 加法| `expr $a + $b` 结果为 30。|
| -| 减法|  `expr $a - $b` 结果为 -10。|
| *| 乘法| `expr $a \* $b` 结果为200。|
| /| 除法| `expr $b / $a` 结果为 2。|
| %| 取余| `expr $b % $a` 结果为 0。|
| =| 赋值| a=$b 把变量 b 的值赋给 a。|
| ==| 相等。用于比较两个数字，相同则返回 true。| [ $a == $b ] 返回 false。|
| !=| 不相等。用于比较两个数字，不相同则返回 true。| [ $a != $b ] 返回 true。|

注意：条件表达式要放在方括号之间，并且要有空格，例如:

```bash
[$a==$b] # 是错误的
[ $a == $b ] # 必须写成 
```

### 关系运算符

| 运算符| 说明| 举例|
| --- | --- | --- |
| -eq| 检测两个数是否相等，相等返回 true。 | [ $a -eq $b ] 返回 false。|
| -ne| 检测两个数是否不相等，不相等返回 true。| [ $a -ne $b ] 返回 true。 |
| -gt | 检测左边的数是否大于右边的，如果是，则返回 true。 | [ $a -gt $b ] 返回 false。 |
| -lt| 检测左边的数是否小于右边的，如果是，则返回 true。 | [ $a -lt $b ] 返回 true。 |
| -ge | 检测左边的数是否大于等于右边的，如果是，则返回 true。 | [ $a -ge $b ] 返回 false。 |
| -le | 检测左边的数是否小于等于右边的，如果是，则返回 true。 | [ $a -le $b ] 返回 true。 |

### 布尔运算符

| 运算符 | 说明 | 举例 |
| --- | --- | --- |
| ! | 非运算，表达式为 true 则返回 false，否则返回 true。 | [ ! false ] 返回 true。 |
| -o | 或运算，有一个表达式为 true 则返回 true。 | [ $a -lt 20 -o $b -gt 100 ] 返回 true。 |
| -a | 与运算，两个表达式都为 true 才返回 true。 | [ $a -lt 20 -a $b -gt 100 ] 返回 false。 |

### 逻辑运算符

| 运算符 | 说明 | 举例 |
| --- | --- | --- |
| && | 逻辑的 AND | [[ $a -lt 100 && $b -gt 100 ]] 返回 false |
| \|\| | 逻辑的 OR | [[ $a -lt 100 \|\| $b -gt 100 ]] 返回 true |

### 字符串运算符

| 运算符| 说明| 举例|
| --- | --- | --- |
| =| 检测两个字符串是否相等，相等返回 true。| [ $a = $b ] 返回 false。|
| !=| 检测两个字符串是否不相等，不相等返回 true。| [ $a != $b ] 返回 true。|
| -z| 检测字符串长度是否为0，为0返回 true。| [ -z $a ] 返回 false。|
| -n| 检测字符串长度是否不为 0，不为 0 返回 true。| [ -n "$a" ] 返回 true。|
| $| 检测字符串是否不为空，不为空返回 true。| [ $a ] 返回 true。|

### 文件测试运算符

| 操作符 | 说明 | 举例 |
| --- | --- | --- |
| -b file | 检测文件是否是块设备文件，如果是，则返回 true。 | [ -b $file ] 返回 false。 |
| -c file | 检测文件是否是字符设备文件，如果是，则返回 true。 | [ -c $file ] 返回 false。 |
| -d file | 检测文件是否是目录，如果是，则返回 true。 | [ -d $file ] 返回 false。 |
| -f file | 检测文件是否是普通文件（既不是目录，也不是设备文件），如果是，则返回 true。 | [ -f $file ] 返回 true。 |
| -g file | 检测文件是否设置了 SGID位，如果是，则返回 true。 | [ -g $file ] 返回 false。 |
| -k file | 检测文件是否设置了粘着位(Sticky Bit)，如果是，则返回 true。 | [ -k $file ] 返回 false。 |
| -p file | 检测文件是否是有名管道，如果是，则返回 true。 | [ -p $file ] 返回 false。 |
| -u file | 检测文件是否设置了 SUID位，如果是，则返回 true。 | [ -u $file ] 返回 false。 |
| -r file | 检测文件是否可读，如果是，则返回 true。 | [ -r $file ] 返回 true。 |
| -w file | 检测文件是否可写，如果是，则返回 true。 | [ -w $file ] 返回 true。 |
| -x file | 检测文件是否可执行，如果是，则返回 true。 | [ -x $file ] 返回 true。 |
| -s file | 检测文件是否为空（文件大小是否大于0），不为空返回 true。 | [ -s $file ] 返回 true。 |
| -e file | 检测文件（包括目录）是否存在，如果是，则返回 true。 | [ -e $file ] 返回 true。 |
| -S file | 判断某文件是否 socket。 |  |
| -L file | 检测文件是否存在并且是一个符号链接。 |  |

## #echo命令

- 文本输出：echo命令后面跟上输出的文本
- echo输出自动换行，echo -n 表示不换行输出
- 显示转义字符：echo "\"It is a test\"" 双引号的转义无限制
- 显示变量
    
    read命令从标准输入中读取一行，并把输入行的每个字段的值指定给shell变量
    
    ```bash
    read name
    echo "$name It is a test"
    ```
    
- 显示换行 -e 开启转义：
    
    ```bash
    echo -e "OK! \n"
    ```
    
- 显示不换行：
    
    ```bash
    echo -e "OK! \c"
    ```
    
- 显示结果定向至文件：
    
    ```bash
    echo "It is a test" > myfile
    ```
    
- 原样输出字符串，不进行转义或取变量（用单引号包住）：
    
    ```bash
    echo '$name\"'
    ```
    
- 显示命令执行结果（使用反引号`）：
    
    ```bash
    echo `date`
    ```
    

## #printf

1. printf 和echo一样都是输出命令
2. printf命令模仿C程序库里的printf()程序
3. printf由POSIX标准所定义，因此使用printf的脚本比使用echo的脚本有更好的移植性
4. printf使用引用文本或空格分隔的参数，可以在printf中使用格式化字符串，还可以制定字符串的宽度、左右对齐方式等。
5. 默认的printf不会像echo自动添加换行符，需要手动添加\n

示例：

```bash
printf "%-10s %-8s %-4s\n" 姓名 性别 体重kg
printf "%-10s %-8s %-4.2f\n" 郭靖 男 66.1234
```

（其中，%s %c %d %f 都是格式替代符）

%s 输出一个字符串

%d 输出一个整型

%c 输出一个字符

%f 输出实数，以小数形式输出

%-10s 指一个宽度为10字符（-表示左对齐，没有表示右对齐），任何字符都会被显示在10个字符宽的字符内。如果不足则自动以空格填充，超过也会将内容全部显示出来

%-4.2f 指格式化为小数，其中.2指保留2位小数。

如果参数多于格式指定的参数量，多出来的参数仍然会按该格式输出

如果没有足够的参数，%s将用NULL（空值）代替，%d用0代替

### printf 的转义序列

| 序列 | 说明 |
| --- | --- |
| \a | 警告字符，通常为ASCII的BEL字符 |
| \b | 后退 |
| \c | 抑制（不显示）输出结果中任何结尾的换行字符（只在%b格式指示符控制下的参数字符串中有效），而且，任何留在参数里的字符、任何接下来的参数以及任何留在格式字符串中的字符，都被忽略 |
| \f | 换页（formfeed） |
| \n | 换行 |
| \r | 回车（Carriage return） |
| \t | 水平制表符 |
| \v | 垂直制表符 |
| \\\ | 一个字面上的反斜杠字符 |
| \ddd | 表示1到3位数八进制值的字符。仅在格式字符串中有效 |
| \0ddd | 表示1到3位的八进制值字符 |

## #test命令

test命令用于检查某个条件是否成立，它可以进行**数值**、**字符**和**文件**三个方面的测试。

返回值为真或假。

### 数值相关参数

| 参数 | 说明 |
| --- | --- |
| -eq | 等于则为真 |
| -ne | 不等于则为真 |
| -gt | 大于则为真 |
| -ge | 大于等于则为真 |
| -lt | 小于则为真 |
| -le | 小于等于则为真 |

代码中的 [] 执行基本的算数运算：

```bash
result=$[a+b]
```

### 字符串相关参数

| 参数 | 说明 |
| --- | --- |
| = | 等于则为真 |
| != | 不相等则为真 |
| -z 字符串 | 字符串的长度为零则为真 |
| -n 字符串 | 字符串的长度不为零则为真 |

### 文件相关参数

| 参数 | 说明 |
| --- | --- |
| -e 文件名 | 如果文件存在则为真 |
| -r 文件名 | 如果文件存在且可读则为真 |
| -w 文件名 | 如果文件存在且可写则为真 |
| -x 文件名 | 如果文件存在且可执行则为真 |
| -s 文件名 | 如果文件存在且至少有一个字符则为真 |
| -d 文件名 | 如果文件存在且为目录则为真 |
| -f 文件名 | 如果文件存在且为普通文件则为真 |
| -c 文件名 | 如果文件存在且为字符型特殊文件则为真 |
| -b 文件名 | 如果文件存在且为块特殊文件则为真 |

Shell 还提供了与( -a )、或( -o )、非( ! )三个逻辑操作符用于将测试条件连接起来，其优先级为： ! 最高， -a 次之， -o 最低。

## # 流程控制

### if 条件语句

```bash
if condition
then
	command
	command
fi

# 一行的形式：
if condition; then command; fi
```

```bash
# if-else
if condition
then
	command
	command
else
	command
fi

# if-else if
if condition
then
	command
elif condition
then 
command
fi
```

判断语句command的形式：[…] 和 ((…))

[...] 判断语句中大于使用 -gt，小于使用 -lt。

((...)) 作为判断语句，大于和小于可以直接使用 > 和 <。

### for循环

```bash
for var in item1 item2 … itemN
do
	command
	command
done
```

### while循环

用途：

1. 不断执行一系列命令
2. 从输入文件中读取数据

基本形式：

```bash
while condition
do
	command
done
```

while循环可用于读取键盘信息。下面的例子中，输入信息被设置为变量FILM，按<Ctrl-D>结束循环。

示例

```bash
echo '按下 <CTRL-D> 退出'
echo -n '输入你最喜欢的网站名: '
while read FILM
do
    echo "是的！$FILM 是一个好网站"
done
```

### 无限循环语法格式

```bash
while :
do
	command
done

# 或

while true
do
	command
done

# 或

for (( ; ; ))
```

### until 循环

until 循环执行一系列命令直至条件为 true 时停止。

until 循环与 while 循环在处理方式上刚好相反。

一般 while 循环优于 until 循环，但在某些时候—也只是极少数情况下，until 循环更加有用。

语法格式

```bash
until condition
do
    command
done
```

condition 一般为条件表达式，如果返回值为 false，则继续执行循环体内的语句，否则跳出循环。

### case ... esac

case ... esac 为多选择语句，与其他语言中的 switch ... case 语句类似，是一种多分支选择结构，每个 case 分支用右圆括号开始，用两个分号 ;; 表示 break，即执行结束，跳出整个 case ... esac 语句，esac（就是 case 反过来）作为结束标记。

可以用 case 语句匹配一个值与一个模式，如果匹配成功，执行相匹配的命令。

```bash
case 值 in
模式1)
    command
    command
    ;;
模式2)
    command
    command
    ;;
esac
```

case 工作方式如上所示，取值后面必须为单词 in，每一模式必须以右括号结束。取值可以为变量或常数，匹配发现取值符合某一模式后，其间所有命令开始执行直至 ;;。

取值将检测匹配的每一个模式。一旦模式匹配，则执行完匹配模式相应命令后不再继续其他模式。如果无一匹配模式，使用星号 * 捕获该值，再执行后面的命令。

```bash
echo '输入 1 到 4 之间的数字:'
echo '你输入的数字为:'
read aNum
case $aNum in
    1)  echo '你选择了 1'
    ;;
    2)  echo '你选择了 2'
    ;;
    3)  echo '你选择了 3'
    ;;
    4)  echo '你选择了 4'
    ;;
    *)  echo '你没有输入 1 到 4 之间的数字'
    ;;
esac
```

### 循环中断

**break 命令**

break 命令允许跳出所有循环（终止执行后面的所有循环）。

```bash
#!/bin/bash
while :
do
    echo -n "输入 1 到 5 之间的数字:"
    read aNum
    case $aNum in
        1|2|3|4|5) echo "你输入的数字为 $aNum!"
        ;;
        *) echo "你输入的数字不是 1 到 5 之间的! 游戏结束"
            break
        ;;
    esac
done
```

**continue**

continue 命令与 break 命令类似，只有一点差别，它不会跳出所有循环，仅仅跳出当前循环。

对上面的例子进行修改：

```bash
#!/bin/bash
while :
do
    echo -n "输入 1 到 5 之间的数字: "
    read aNum
    case $aNum in
        1|2|3|4|5) echo "你输入的数字为 $aNum!"
        ;;
        *) echo "你输入的数字不是 1 到 5 之间的!"
            continue
            echo "游戏结束"
        ;;
    esac
done
```

## # 函数

函数的定义格式：

```bash
[ function ] funname [()]
{
    action;
    [return int;]
}
```

说明：

1、可以带function fun() 定义，也可以直接fun() 定义,不带任何参数。

2、参数返回，可以显示加：return 返回，如果不加，将以最后一条命令运行结果，作为返回值。 return后跟数值n(0-255)

函数返回值在调用该函数后通过 $? 来获得。

注意：所有函数在使用前必须定义。这意味着必须将函数放在脚本开始部分，直至shell解释器首次发现它时，才可以使用。调用函数仅使用其函数名即可。

### 函数参数

在Shell中，调用函数时可以向其传递参数。在函数体内部，通过 $n 的形式来获取参数的值，例如，$1表示第一个参数，$2表示第二个参数...

（和调用shell文件时给参数的方式相同）

示例

```bash
#!/bin/bash
# author:菜鸟教程
# url:www.runoob.com

funWithParam(){
    echo "第一个参数为 $1 !"
    echo "第二个参数为 $2 !"
    echo "第十个参数为 $10 !"
    echo "第十个参数为 ${10} !"
    echo "第十一个参数为 ${11} !"
    echo "参数总数有 $# 个!"
    echo "作为一个字符串输出所有参数 $* !"
}
funWithParam 1 2 3 4 5 6 7 8 9 34 73
```

注意，$10 不能获取第十个参数，获取第十个参数需要${10}。当n>=10时，需要使用${n}来获取参数。

### 用来处理参数的特殊字符

| 参数处理 | 说明 |
| --- | --- |
| $# | 传递到脚本的参数个数 |
| $* | 以一个单字符串显示所有向脚本传递的参数。如"$*"用「"」括起来的情况、以"$1 $2 … $n"的形式输出所有参数。 |
| $$ | 脚本运行的当前进程ID号 |
| $! | 后台运行的最后一个进程的ID号 |
| $@ | 与$*相同，但是使用时加引号，并在引号中返回每个参数。如"$@" 用「"」括起来的情况、以"$1" "$2" …"$n" 的形式输出所有参数。 |
| $- | 显示Shell使用的当前选项，与set命令功能相同。 |
| $? | 显示最后命令的退出状态。0表示没有错误，其他任何值表明有错误。 |

## # shell 输入/输出重定向

大多数 UNIX 系统命令从你的终端接受输入并将所产生的输出发送回到您的终端。一个命令通常从一个叫标准输入的地方读取输入，默认情况下，这恰好是你的终端。同样，一个命令通常将其输出写入到标准输出，默认情况下，这也是你的终端。

**重定向命令列表**如下：

| 命令 | 说明 |
| --- | --- |
| command > file | 将输出重定向到 file。 |
| command < file | 将输入重定向到 file。 |
| command >> file | 将输出以追加的方式重定向到 file。 |
| n > file | 将文件描述符为 n 的文件重定向到 file。 |
| n >> file | 将文件描述符为 n 的文件以追加的方式重定向到 file。 |
| n >& m | 将输出文件 m 和 n 合并。 |
| n <& m | 将输入文件 m 和 n 合并。 |
| << tag | 将开始标记 tag 和结束标记 tag 之间的内容作为输入。 |

需要注意的是文件描述符 0 通常是标准输入（STDIN），1 是标准输出（STDOUT），2 是标准错误输出（STDERR）。

### 输入重定向说明

和输出重定向一样，Unix 命令也可以从文件获取输入，语法为：command1 < file1

这样，本来需要从键盘获取输入的命令会转移到文件读取内容。

实例：

统计 users 文件的行数,执行以下命令：

```bash
$ wc -l users
2 users # 输出
```

也可以将输入重定向到 users 文件：

```bash
$  wc -l < users
2
```

说明： 第一个例子，wc命令会获取到文件的信息，所以在输出中包含文件名。第二个例子通过输入重定向，wc命令只获得了文件中的内容，不包含文件的信息。

command1 < infile > outfile

同时替换输入和输出，执行command1，从文件infile读取内容，然后将输出写入到outfile中。

### 重定向深入讲解

一般情况下，每个 Unix/Linux 命令运行时都会打开三个文件：

1. 标准输入文件(stdin)：stdin的文件描述符为0，Unix程序默认从stdin读取数据。
2. 标准输出文件(stdout)：stdout 的文件描述符为1，Unix程序默认向stdout输出数据。
3. 标准错误文件(stderr)：stderr的文件描述符为2，Unix程序会向stderr流中写入错误信息。

默认情况下，command > file 将 stdout 重定向到 file，command < file 将stdin 重定向到 file。

2 表示标准错误文件(stderr)。

如果希望 stderr 重定向到 file，可以这样写：

```bash
$ command 2>file
```

如果希望 stderr 追加到 file 文件末尾，可以这样写：

```bash
$ command 2>>file
```

如果希望将 stdout 和 stderr 合并后重定向到 file，可以这样写：

```bash
$ command > file 2>&1
# 或者
$ command >> file 2>&1
```

如果希望对 stdin 和 stdout 都重定向，可以这样写：

```bash
$ command < file1 >file2
```

command 命令将 stdin 重定向到 file1，将 stdout 重定向到 file2。

## Here Document

Here Document 是 Shell 中的一种特殊的重定向方式，用来将输入重定向到一个交互式 Shell 脚本或程序。

它的基本的形式如下：

```bash
command << delimiter
    document
delimiter
```

它的作用是将两个 delimiter 之间的内容(document) 作为输入传递给 command。

注意：

结尾的delimiter 一定要顶格写，前面不能有任何字符，后面也不能有任何字符，包括空格和 tab 缩进。

开始的delimiter前后的空格会被忽略掉。

实例

在命令行中通过 wc -l 命令计算 Here Document 的行数：

```bash
$ wc -l << EOF
    欢迎来到
    菜鸟教程
    www.runoob.com
EOF
3          # 输出结果为 3 行
```

我们也可以将 Here Document 用在脚本中，例如：

```bash
#!/bin/bash
# author:菜鸟教程
# url:www.runoob.com

cat << EOF
欢迎来到
菜鸟教程
www.runoob.com
EOF
```

执行以上脚本，输出结果：

欢迎来到

菜鸟教程

www.runoob.com

### /dev/null 文件

如果希望执行某个命令，但又不希望在屏幕上显示输出结果，那么可以将输出重定向到 /dev/null：

```bash
$ command > /dev/null
```

/dev/null 是一个特殊的文件，写入到它的内容都会被丢弃；如果尝试从该文件读取内容，那么什么也读不到。但是 /dev/null 文件非常有用，将命令的输出重定向到它，会起到"禁止输出"的效果。

如果希望屏蔽 stdout 和 stderr，可以这样写：

```bash
$ command > /dev/null 2>&1
```

注意：0 是标准输入（STDIN），1 是标准输出（STDOUT），2 是标准错误输出（STDERR）。

这里的 2 和 > 之间不可以有空格，2> 是一体的时候才表示错误输出。

## # shell文件包含

Shell 文件包含的语法格式如下：

```bash
. filename   # 注意点号(.)和文件名中间有一空格
# 或
source filename
```

实例

创建两个 shell 脚本文件。

test1.sh 代码如下：

```bash
#!/bin/bash
# author:菜鸟教程
# url:www.runoob.com
url="http://www.runoob.com"
```

test2.sh 代码如下：

```bash
#!/bin/bash
# author:菜鸟教程
# url:www.runoob.com
#使用 . 号来引用test1.sh 文件
. ./test1.sh
# 或者使用以下包含文件代码
# source ./test1.sh
echo "菜鸟教程官网地址：$url"
```

接下来，我们为 test2.sh 添加可执行权限并执行：

```bash
$ chmod +x test2.sh
$ ./test2.sh
菜鸟教程官网地址：http://www.runoob.com # 输出
```

注：被包含的文件 test1.sh 不需要可执行权限。
