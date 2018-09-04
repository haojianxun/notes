# sed用法和例子

## 语法

```
sed [option]... 'script' inputfile...
```

### 语法详解

#### **[option]**

- -n ：不输出模式 空间内容到屏幕，即不自动 打印
- -e:  多点编辑
- -f ：/ PATH/SCRIPT_FILE :  从指定文件中读取编辑脚本
- -r:  支持使用扩展正则表达式
- -i: 原处编辑


#### **'script'**

是由'地址命令'组成,即由地址和命令2部分组成

##### **地址**

1. **不给地址**：对全文进行处理

2.   **单地址**：

   ​             \#:指定的行,\#代表数字的意思

   ​             /pattern/

3. **地址范围**：

   ​            \# ,\#  :在某个区间,比如'2,5'

   ​            \# ,+\# :比如'2,+6' 代表的是第二行和之后的6行

   ​            /pat1/,/pat2/:由2个模式匹配的行之间

   ​            \# ,/patter1/

4. 步行:~

   ​         1~2  奇数行
   ​         2~2  偶数行


##### **命令**              

- `d` :  删除模式空间匹配的行
- `p` :  显示模式空间中的内容
- `a [\]text`  ：在指定行 后面 追加文本 支持 使用\n 实现多行追加
- `i [\]text ` ：在行前面 插入文本
- `c [\]text` ：替换行为单行或多行文本
- `w /path/somefile` :  保存模式匹配的行至指定文件
- `r /path/somefile ` ：读取指定文件的文本至 模式空间中匹配 到的行后
- `=` :  为模式空间中的行打印行号
- `!` :模式空间中匹配行取反处理



 s/// ：查找替换, 支持使用其它分隔符，s@@@ ，s\#\#\# 

替换标记：
g:  行内全局替换
p:  显示替换成功的行
w  /PATH/TO/SOMEFILE ：将替换成功的 行 保存至文件中

## 例子

```
sed ‘2p’ /etc/passwd     //打印第二行及其以后的行
sed –n ‘2p’ /etc/passwd     //只打印第二行
sed –n ‘1,4p’ /etc/passwd    //打印1到4行之间的行
sed –n ‘/root/p’ /etc/passwd   //打印包含root的行
sed –n ‘2,/root/p’ /etc/passwd   //从2 行开始,打印包含root的行之间的行  
sed -n ‘/^$/=’ file  //显示空行行号
sed –n –e ‘/^$/p’ –e ‘/^$/=’ file   //打印空行和显示空行行号
sed ‘/root/a\superman’ /etc/passwd //行后追加superman
sed ‘/root/i\superman’ /etc/passwd  //行前
sed ‘/root/c\superman’ /etc/passwd  //代替行

sed ‘/^$/d’ file  //删除空白行
sed –n‘s/root/&superman/p’ /etc/passwd   //搜索root的行,并且把root替代为rootsuperman并打印出来   (在单词后)
sed –n‘s/root/superman&/p’ /etc/passwd   //在单词前,  把root替代为supermanroot
sed -i.bak '2,10d' f1  //把f1文件中的2到10行删除,并把剩下的内容保存为f1.bak
```



## sed高级用法

高级编辑命令：

- h:  把模式空间中的内容覆盖至保持空间中
- H ：把模式空间中的内容追加至保持空间中
- g:  从保持空间取出数据覆盖至模式空间
- G ：从保持空间取出内容追加至模式空间
- x:  把模式空间中的内容与保持空间中的内容进行互换
- n:  读取匹配到的行的下一行 覆盖 至模式空间
- N ：读取匹配 到的行的 下一行 追加 至 模式空间
- d:  删除模式空间中的行
- D ：删除 当前模式空间开端至\n 的内容 （ 不再传 至标准输出），放弃之后的命令，但是对剩余模式空间重新执行sed

### 高级用法例子

```
sed -n 'n;p' FILE  //打印文件偶数行
sed '1!G;h;$!d' FILE  //将文件倒叙打印,作用和tac一样
sed '$!N;$!D' FILE
sed '$!d' FILE
sed ‘G’ FILE
sed ‘g’ FILE
sed ‘/^$/d;G’ FILE
sed 'n;d' FILE
sed -n '1!G;h;$p' FILE
```









