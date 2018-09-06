# awk用法

# **语法**

```
awk [options] ‘program’ var=value file…
awk [options] -f program file var=value file…
awk [options] 'BEGIN{ action;… } pattern{ action;… } END{action;… }' file ...
```



## **option**

-F  指明 输入时用到的字段分隔符
-v var=value:  自定义变量



## **program**

pattern{action statements;..}

​      pattern 和action:

​              pattern 部分决定动作语句何时触发及触发事件（BEGIN,END）

​              action statements对 对 数据进行处理，放在{} 内指明（ print, printf ）

### **分割符、域和记录**

​      awk 执行时， 由 分隔符分隔的 字段（域）标记$1,$2..\$n称 称为 域标识。$0 为所有域



 **print 格式**：print item1, item2, ...

#### 例子

```
awk '{print "hello,awk"}'
awk –F: '{print}' /etc/passwd
awk –F: ‘{print “wang”}’ /etc/passwd
awk –F: ‘{print $1}’ /etc/passwd
awk –F: ‘{print $0}’ /etc/passwd
awk –F: ‘{print $1”\t”$3}’ /etc/passwd
tail –3 /etc/fstab |awk ‘{print $2,$4}’
```

## **awk 变量**

-  FS ：输入字段分隔符， 默认为空白字符

  ```
  awk -v FS=':' '{print $1,$3,$7}’ /etc/passwd
  awk –F: '{print $1,$3,$7}’ /etc/passwd
  ```

- OFS ：输出字段分隔符， 默认为空白字符

  ```
  awk -v FS=‘:’ -v OFS=‘:’ '{print $1,$3,$7}’ /etc/passwd
  ```

-  RS ：输入记录分隔符， 指定输入时的换行符，原换行符仍有效

  ```
  awk -v RS=' ' ‘{print }’ /etc/passwd
  ```

- ORS ：输出记录分隔符， 输出时用指定符号代替换行符

  ```
  awk -v RS=' ' -v ORS='###'‘{print }’ /etc/passwd
  ```

-  NF ：字段 数量

  ```
  awk -F： ： ‘{print NF}’ /etc/fstab, 引用内置变量不用$
  awk -F: '{print $(NF-1)}' /etc/passwd
  ```

-  NR ：行号

  ```
  awk '{print NR}' /etc/fstab ; awk END'{print NR}' /etc/fstab
  ```

- FNR ：各文件分别计数,

  ```
  awk '{print FNR}' /etc/fstab /etc/inittab
  ```

-  FILENAME ：当前文件名

  ```
  awk '{print FILENAME}’ /etc/fstab
  ```

- ARGC ：命令行参数的个数

  ```
  awk '{print ARGC}’ /etc/fstab /etc/inittab
  awk ‘BEGIN {print ARGC}’ /etc/fstab /etc/inittab
  ```

- ARGV ：数组，保存的是命令行所给定的各参数

  ```
  awk ‘BEGIN {print ARGV[0]}’ /etc/fstab  /etc/inittab
  awk ‘BEGIN {print ARGV[1]}’ /etc/fstab  /etc/inittab
  ```

### **自定义变量**

1. -v var=value

   变量名区分字符大小写

2. 在program 中直接定义

```
awk -v test='hello gawk' '{print test}' /etc/fstab
awk -v test='hello gawk' 'BEGIN{print test}'
awk 'BEGIN{test="hello,gawk";print test}'
```

## **printf 命令**

printf  “FORMAT ”, item1, item2, ...

> - 必须指定FORMAT
> - 不会 自动换行，需要显式给出换行控制符，\n
> - FORMAT 中需要分别为后面每个item 指定格式符

###  **格式符**

格式符:与item 一一对应

- %c:  显示字符的ASCII码 码
- %d, %i:  显示十进制整数
- %e, %E: 显示科学 计数 法数值
- %f ：显示为浮点数
- %g, %G ：以科学计数法或浮点形式显示数值
- %s ：显示字符串
- %u ：无符号整数
- %%:  显示% 自身

### **修饰符**

- \#[.\#] :第一个数字控制显示的宽度；第二个# 表示小数点后精度，%3.1f
- -:  左对齐（默认） 右对齐） %-15s
- \+：显示数值的号 正负符号 %+d

#### **例子**

```
 awk -F: ‘{printf "%s",$1}’ /etc/passwd
 awk -F: ‘{printf "%s\n",$1}’ /etc/passwd
  awk -F: ‘{printf "Username: %s\n",$1}’ /etc/passwd
  awk -F: ‘{printf “Username: %s,UID:%d\n",$1,$3}’ /etc/passwd
  awk -F: ‘{printf "Username: %15s,UID:%d\n",$1,$3}’  /etc/passwd
  awk -F: ‘{printf "Username: %-15s,UID:%d\n",$1,$3}’  /etc/passwd
```

### **操作符**

x+y, x-y, x*y, x/y, x^y, x%y

-x:  转换为负数

+x:  转换为数值



=, +=, -=, *=, /=, %=, ^= , ++, --

\>, >=, <, <=, !=, ==

### **模式匹配符**

~ ：左边是否和 右边匹配包含
!~ ：是否不匹配
awk –F: '$0 ~ /root/{print $1}‘ /etc/passwd
awk '$0 !~ /root/‘ /etc/passwd

  逻辑 操作符：与 与&& ，或||  ，非!

```
awk –F: '$3>=0 && $3<=1000 {print $1}' /etc/passwd
awk -F: '$3==0 || $3>=1000 {print $1}' /etc/passwd
awk -F: ‘!($3==0) {print $1}' /etc/passwd
awk -F: ‘!($3>=500) {print $3}’ /etc/passwd
```

条件表达式（三目表达式）

`selector?if-true-expression:if-false-expression`

```
awk -F: '{$3>=1000?usertype="Common User":usertype="Sysadmin or SysUser";printf "%15s:%-s\n",$1,usertype}' /etc/passwd
```

## **awk PATTERN**

 根据pattern 条件，过滤匹配的行，再做处理

- 如果未指定： 空模式，匹配每一行

-  /regular expression/ ：仅 处理能够模式匹配 到 的行，需要用/ / 括起来

  ```
  awk '/^UUID/{print $1}' /etc/fstab
  awk '!/^UUID/{print $1}' /etc/fstab
  ```

- relational expression:  关系表达式，结果 有“真” 有“假”，结果 为“真”才会 被处理

  真：结果为非0 值，非空字符串
  假：结果为 空字符串或0值 值

  ```
  awk ‘!0’ /etc/passwd ; awk ‘!1’ /etc/passwd
  awk –F: '$3>=1000{print $1,$3}' /etc/passwd
  awk -F: '$3<1000{print $1,$3}' /etc/passwd
  awk -F: '$NF=="/bin/bash"{print $1,$NF}' /etc/passwd
  awk -F: '$NF ~ /bash$/{print $1,$NF}' /etc/passwd
  ```

BEGIN/END 模式

BEGIN{}: 仅在开始处理文件中的文本之前执行一次

END{} ：仅在文本处理完成之后执行 一次

```
awk -F : 'BEGIN {print "USER USERID"} {print $1":"$3} END{print "end file"}' /etc/passwd
awk -F: 'BEGIN{print " USER UID \n ---------------"}{print $1,$3}'END{print "=============="} /etc/passwd
```

## awk action

-  Expressions: 算术，比较表达式等
- Control statements ：if, while等 等
- Compound statements ：组合语句
- input statements
- output statements ：print等





 **{ statements;… }**

-  if(condition) {statements;…}
- if(condition) {statements;…} else {statements;…}
-  while(conditon) {statments;…}
- do {statements;…} while(condition)
-  for(expr1;expr2;expr3) {statements;…}
-  break
- continue
- delete array[index]
-  delete array
- exit

### **awk 控制语句if-else**

```
if(condition) statement [else statement]
if(condition1){statement1}else if(condition2){statement2}else{statement3}
```

使用场景：对awk 取得的整行或某个字段做条件判断

例子:

```
awk -F: '{if($3>=1000)print $1,$3}' /etc/passwd
awk -F: '{if($NF=="/bin/bash") print $1}' /etc/passwd
awk '{if(NF>5) print $0}' /etc/fstab
awk -F: '{if($3>=1000) {printf "Common user: %s\n",$1} else{printf "root or Sysuser: %s\n",$1}}' /etc/passwd
awk -F: '{if($3>=1000) printf "Common user: %s\n",$1;else printf "root or Sysuser: %s\n",$1}' /etc/passwd
df -h|awk -F% '/^\/dev/{print $1}'|awk '$NF>=80{print $1,$5}‘
awk 'BEGIN{ test=100;if(test>90){print "very good"}else if(test>60){ print "good"}else{print "no pass"}}'
```

###  **while 循环**

```
while(condition){statement;…}
```

条件“真”，进入循环；条件“假”， 退出循环

使用场景:对一行内的多个字段逐一类似处理时使用,对数组中的各元素逐一处理时使用

```
awk '/^[[:space:]]*linux16/{i=1;while(i<=NF){print $i,length($i); i++}}' /etc/grub2.cfg
awk '/^[[:space:]]*linux16/{i=1;while(i<=NF) {if(length($i)>=10){print $i,length($i)}; i++}}' /etc/grub2.cfg
```

### **do-while 循环**

```
do {statement;…}while(condition)
```

无论真假，至少执行一次循环体

```
awk 'BEGIN{ total=0;i=0;do{total+=i;i++;}while(i<=100);print total}‘
```

### **for 循环**

```
for(expr1;expr2;expr3) {statement;…}
```

for(variable assignment;condition;iteration process){for-body}







