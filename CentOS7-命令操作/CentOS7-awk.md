#### 模式

- /正则表达式/：使用通配符的扩展集。

- 关系表达式：使用运算符进行操作，可以是字符串或数字的比较测试。

- 模式匹配表达式：用运算符`~`（匹配）和`~!`（不匹配）。

- BEGIN语句块、pattern语句块、END语句块。

  BEGIN：awk 开始处理输入文件中的文本之前执行初始化操作，是初始化 各种变量（包括内置变量，全局变量）的极佳位置。

  END：awk 在处理了输入文件中的所有行之后执行这个块。通常， END 块用于执行最终计算或打印应该出现在输出流结尾的摘要信息。

  ##### 例：

  ​	首先可以看到所有的命令都在 ‘ ’ 里，BEGIN and END 以及记录行正常的操作部分，都有单独的代码块，用 {} 区分。另外我们可以看到，awk支持自定义变量，还可以进行各种运算。实际上awk也是一个轻量级的编程语言。这里代码“BEGIN {size=0}”可以省略，awk默认size=0.

  ```shell
  # 统计某个文件夹下的文件占用的字节数
  >> ll |awk 'BEGIN {size=0} {size=size+$5} END{print "[end]size is ",size/2014/1024, "M"}'
  [end]size is 3.3 M
  ```



##### 脚本基本结构

```shell
awk 'BEGIN{ print "start" } pattern{ commands } END{ print "end" }' file

# 常用
awk '{命令}' file1, file2, ...
其他命令的输出 ｜ awk '{命令}'
```

一个awk脚本通常由：BEGIN语句块、能够使用模式匹配的通用语句块、END语句块3部分组成，这三个部分是可选的。任意一个部分都可以不出现在脚本中，脚本通常是被**单引号**中。



##### 常用命令

使用`print $NF`可以打印出一行中的最后一个字段，使用`$(NF-1)`则是打印倒数第二个字段，其他以此类推：

```shell
echo -e "line1 f2 f3n line2 f4 f5" | awk '{print $NF}'
```

打印每一行的第二和第三个字段：

```shell
awk '{ print $2,$3 }' filename
```

统计文件中的行数：

```shell
awk 'END{ print NR }' filename
```

// 中是模式匹配：

```shell
awk '/rodos/' filename
```



##### awk内置变量

| 变量名  | 含义                                                  |
| ------- | ----------------------------------------------------- |
| $0      | 当前记录行                                            |
| $1 ~ $n | 当对当前记录行进行分隔时，表示分隔后第几个字段        |
| FS      | 字段分隔符，默认是空格                                |
| RS      | 记录行分隔符，默认为换行符                            |
| NF      | 对记录行进行分隔后，字段的个数                        |
| NR      | 已经读出对记录数，从1开始，多个文件时继续累加计数     |
| FNR     | 已经读出对记录数，从1开始，多个文件时每个文件单独计数 |
| ORS     | 输出时的记录行分隔符， 默认是换行符                   |
| OFS     | 输出时的字段分隔符， 默认是空格                       |

###### 例： $0~$n 和 -F

```shell
>> cat awk_test.txt
a,b,c,d
e,f,g,h

>> awk '{print $0}' awk_test.txt 
a,b,c,d
e,f,g,h
>> awk -F ',' '{print $1,$2,$3,$4}' awk_test.txt
a b c d
e f g h
```

###### 例：FS 和 RS

```shell
>> awk -F ',' '{print $1 FS $2,$3,$4}' awk_test.txt
a,b c d
e,f g h

>> awk -F ',' '{print $1 RS $2,$3,$4}' awk_test.txt
a
b c d
e
f g h

# 若想用多个分隔符分隔记录行，只需要将多个分隔符放在 [分隔符1 隔符2 ..] 中；如果想用一个或者多个相同分隔符进行分隔，可以写成 [分隔符1 隔符2 ..]+

>> echo 'I am Poe,my qq is 0000001' | awk -F '[" ",]' '{print $3 " " $7}'
Poe 0000001

>> echo 'I am Poe,,my qq is 0000001' | awk -F '[" ",]+' '{print $3 " " $7}'
Poe 0000001
```

###### 例：NF、NR和FNR

```shell
# NF和NR，结果中有行号，即为NR， 输出结果中的 4 即为字段数。
>> cat awk_test.txt
a,b,c,d
e,f,g,h

>> awk -F ',' '{print NR") "$0"t size is "NF}' awk_test.txt
1) a,b,c,d	size is 4
2) e,f,g,h	size is 4

# FNR及其和NR的区别
>> cat file.txt
aaaaaa
bbbbbb

>> awk -F ',' '{print NR") "$0}' awk_test.txt file.txt
1) a,b,c,d
2) e,f,g,h
3) aaaaaa
4) bbbbbb

>> awk -F ',' '{print FNR") "$0}' awk_test.txt file.txt
1) a,b,c,d
2) e,f,g,h
1) aaaaaa
2) bbbbbb

>> awk -F ',' '{if (NR==FNR) print $0}' awk_test.txt file.txt
a,b,c,d
e,f,g,h
```

###### 例：OFS 和 ORS

```shell
>> echo '1 2 3' | awk 'BEGIN {OFS="|"} {print $1,$2,$3}' # 实例5
1|2|3

>> cat awk_test.txt | awk -F ',' '{print $1$2}'
ab
ef

>> cat awk_test.txt | awk -F ',' 'BEGIN {ORS="---"} {print $1$2}' # 实例6
ab---ef---
```



##### 运算符

| 运算符  | 描述                             |
| ------- | -------------------------------- |
| +  -    | 加，减                           |
| *  /  & | 乘，除与求余                     |
| +  -  ! | 一元加，减和逻辑非               |
| ^  ***  | 求幂                             |
| ++  --  | 增加或减少，作为前缀或后缀       |
| ~  ~!   | 匹配正则表达式，不匹配正则表达式 |
| $       | 字段引用                         |
| 空格    | 字符串连接符                     |
| in      | 数组中是否存在某键值             |
| ?:      | C条件表达式                      |

###### 例：三目运算符 ?:

```shell
>> awk 'BEGIN{a="b";print a=="b"?"ok":"err"}'
ok
>> awk 'BEGIN{a="b";print a=="c"?"ok":"err"}'
err
```

 

##### awk正则表达式

| 元字符 | 功能                            | 示例    | 解释                                                         |
| ------ | ------------------------------- | ------- | ------------------------------------------------------------ |
| ^      | 行首定位符                      | /^root/ | 匹配所有以root开头的行                                       |
| $      | 行尾定位符                      | /root$/ | 匹配所有以root结尾的行                                       |
| .      | 匹配任意单字符                  | /r..t/  | 匹配字母r，然后两个任意字符，再以t结尾的行，比如root, r33t 等 |
| *      | 匹配0个或多个前导字符(包括回车) | /a*ool/ | 匹配0个或多个ahi后紧跟ool的行，如ool, aaaaool 等             |
| +      | 匹配1个或多个前导字符           | /a+b/   | 匹配1个或多个a，紧跟b的行，如ab，aab等                       |
| ?      | 匹配0个或1个前导字符            | /a?b/   | 匹配 b 或者 ab 的行                                          |
| []     | 匹配指定字符组内任意1个字符     | /^[abc] | 匹配以字母a 或b 或c 开头的行                                 |

###### 示例

```shell
# awk的正则表达式写在2个 / 之间

# 输出当前目录下所有的文件夹的名称
ls -l | awk '/^d/{print $9}'

# 以分号作为分隔符，匹配第五个字段包含‘root’的行
>> awk -F ':' '$5~/root/{print $0}' /etc/passwd
```



#### 实例

##### log文件

```shell
# 大小为0.18亿行

# 输出前两行
>> cat awk_log.log | head -n 2
2019-10-22 18:11:42,726 feature_name_unify.py[line:99] ERROR: d_type, origin_name, sample_type, exam_name: exam,awk,linux,awk命令简介
2019-10-22 18:11:42,727 feature_name_unify.py[line:99] ERROR: d_type, origin_name, sample_type, exam_name: exam,grep,linux,awk命令简介1

# 统计前20行按逗号分隔后倒数第三个位置会出现哪几种字符，及其频数
>> cat awk_log.log | head -n 20 | awk -F ',' '{a[$6]++} END {for (x in a) print x "t" a[x]}'
awk	7
grep	6
seed	3
ls	3
pwd	1
# 解析：这个例子先将这个log文件的前20行输出，以便awk处理；在使用awk处理时，先使用逗号进行分隔；后面对记录行的每一个操作都写在单引号 ‘’ 里，每一个{}是一个代码块；对于代码块 {a[$6]++} ，a表示一个数组，这个数组可以理解为python里的dict，$6是其索引；初始时，a[$6] = 0，后面随着处理的记录行增加，每来一个相同的索引，就加1；END {for (x in a) print x “t” a[x]}在处理完所有记录行后执行，此时用一个循环，打印出数组a的索引及其值。就如同打印出python字典里的key和value。
```



##### 公司人员信息

```shell
>> cat company.csv | head -n 3
部门,姓名,性别
技术研发部,张三,男
财务部,韩梅梅,女

# 统计一个公司每个部门的人数及具体姓名
>> cat company.csv | head -n 16 ｜ awk -F ',' '{if (NR==1) next}{a[$1]++;b[$1]=b[$1]","$2} END {for (x in a) print x "t" a[x] "t" b[x}'
技术研发部	5	,张三,李四,王五,李雷,tom
财务部	4	,张一,李二,王三,Bob
商务部	3	,张二,李三,王四
战略部门	2	,张五,王六
总裁办	1	,三一
# 解析：这里新出现了一个语法 ‘next’， 它的作用等同于 continue。这个命令出现，就不会执行后面的命令了。这里它的作用是跳过第一行，因为第一行是csv文件的header。这个例子看起来毫无意义，但是在实际处理中，特别是一些log文件，数据量几千万到亿级，用python处理代码写起来也很简单，但是论速度，肯定远远不及awk。
```

