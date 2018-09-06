# awk

[TOC]

GUN awk  简称gawk

> gawk ---pattern scanning and processing language

## 语法

gawk [option] 'program' FILE

​	program:PATTERN{ACTION STATEMENTS}

语句之间用分号分隔

### 选项

- -F 指明输入时用到的字段分隔符
- -v var=value 自定义变量

### program

pattern 和action： ：
• pattern 部分决定动作语句何时触发及触发事件
（ （BEGIN,END） ）
• action statements对 对 数据进行处理，放在{} 内指明
（ print, printf ）

分割符、域和记录
• awk 执行时， 由 分隔符分隔的 字段（域）标记$1,$2..$n称 称
为 域标识。$0 为所有域，注意：和shell 中变量$ 符含义不同
• 文件的每一行 称为记录
• 省略action行 ，则默认执行 print $0 的 的 操作

### 变量

内建变量:

FS 默认为空白字符 input field separator

​	OFS  output field separator 默认是空白字符

​	RS   输入时的换行符

​	ORS	 输出时的换行符  output record separator

​	NF number of field 字段数量

​		{print NF} 

​	NR number of record 行数

​	FNR 各文件分别计数  行数

​	FILENAME 当前文件名

​	ARGC 命令行参数个数

​	ARGV 数组 保存的是命令行给定的各个参数

自定义变量:

-v var=value

在program中直接定义

### printf命令

格式化输出 printf FORMAT ,item1,item2,....

> FORMAT 必须给出
>
> 不会自动换行 需要给出换行符 \n
>
> FORMAT 需要分别为后面的每个item指出一个格式化符号
>
> 格式符
>
> ​	%c 显示字符的ascii码
>
> ​	%d,%i显示十进制整数
>
> ​	%e 科学记数法
>
> ​	%f 浮点数
>
> ​	%s 字符串
>
> 修饰符
>
> ​	\#[.#] 第一个字符表示控制显示的宽度 第二个数字表示小数点后的精读
>
> ​	\- 左对齐
>
> ​	\+ 显示数字的符号

### 操作符

算术操作符

​	= - + ..

赋值操作符

​	= ,+= ,-= ++

比较操作符

\	>= >

模式匹配

​	~ 是否匹配

​	!~ 是否不匹配

逻辑操作符

​	&&

​	||

​	!

函数调用

条件表达式

## PATTERN

- empty 空模式 匹配每一行

- /regular expression/  仅匹配

- relational expression 关系表达式 结果有真有假 结果是真的才被处理,结果假被过滤      真:结果为非0值  非空字符串

- line ranges 行范围

  /part1/  /part2/

- BEGIN/END 模式

  BEGIN{} 在开始处理文件前执行一次

  END{}在文本处理完成之后执行一次

## 常用的action

1. expressions

2. control statements  :  if while

3. compound statements 组合语句
4. input statements
5. output statements

##控制语句
if(condition) {statements}
if(condition) {statements} else {statements}
while(condition) {statements}
do {statements} while {statements}
for{expr1,expr2..} {statements}
break
...


