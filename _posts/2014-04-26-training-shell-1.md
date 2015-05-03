---
layout: post
category: "python"
title: "shell技术分享[1]－ Awk技巧"
tags: ["shell技术分享"]
---

# 1､awk注意项
- 1) 为了避免碰到awk错误，可以总结出以下规律:
	- ① 确保整个awk_script用单引号括起来。
	- ② 确保awk_script内所有引号成对出现。
	- ③ 确保用花括号括起动作语句，用圆括号括起条件语句。
	- ④ 可能忘记使用花括号，也许你认为没有必要，但awk不这样认为，将按之解释语法。
	- ⑤ 如果使用字符串，一定要保证字符串被双引号括起来(在模式中除外)。
- 2) 在awk中，设置有意义的域名是一种好习惯，在进行模式匹配或关系操作时更容易理解。一般的变量名设置方式为name=$n。(这里name为调用的域变量名， n为实际域号。)
- 3) 通常在BEGIN部分给一些变量赋值是很有益的，这样可以在awk表达式进行改动时减少很多麻烦。
- 4) awk的基本功能是根据指定规则抽取输入数据的部分内容并输出，另一个重要的功能是对输入数据进行分析运算得到新的数据并输出，这是通过在awk_script中对字段变量($1、$2、$3...)从新赋值或使用更大的字段变量$n(n大于当前记录的NF)而实现的。
- 5) 使用字符串或正则表达式时，有时需要在输出中加入一新行或查询一元字符。这时就需要字符串屏蔽序列。awk中经常使用的屏蔽序列有: \b 退格键 \t tab键 \f 走纸换页 \ddd 八进制值 \n 新行 \r 回车键 \c 任意其他特殊字符。eg: \\为反斜线符号
- 6) awk的输出函数printf，基本上和C语言的语法类似。
	- ① 格式: printf ("输出模板字符串",参数列表)
	- ② 参数列表是以逗号分隔的列表，参数可以是变量、数字值或字符串。
	- ③ 输出模板字符串的字符串中必须包含格式控制符，有几个参数就要求有几个格式控制符。模板字符串中可以只有格式控制符而没有其它字符。
	- ④ 格式控制符: %[-][width][.prec]fmt % : 标识一个格式控制符的开始，不可省略。 - : 表示参数输出时左对齐，可省略。 width : 一个数字，表示参数输出时占用域的宽度，可省略。 .prec : prec是一个数值，表示最大字符串长度或小数点右边的位数，可省略。 fmt : 一个小写字母，表示输出参数的数据类型，不可省略。
	- ⑤ 常见的fmt : c ASCII字符 d 整数 e 浮点数，科学记数法 f 浮点数，如 123.44 g 由awk决定使用哪种浮点数转换e或f o 八进制数 s 字符串 x 十六进制数
	- ⑥ 举例: echo "65" | awk '{ printf ("%c\n",$0) }' // 将打印 A awk 'BEGIN{printf "%.4f\n",999}' //将打印 999.0000 awk 'BEGIN{printf "2 number:%8.4f%8.2f",999,888}' // 将打印 2 number:999.0000 888.000

#2､awk实例
<pre><code class="shell">

awk 用法：awk ' pattern {action} '

变量名 含义
ARGC 命令行变元个数
ARGV 命令行变元数组
FILENAME 当前输入文件名
FNR 当前文件中的记录号
FS 输入域分隔符，默认为一个空格
RS 输入记录分隔符
NF 当前记录里域个数
NR 到目前为止记录数
OFS 输出域分隔符
ORS 输出记录分隔符

1、awk '/101/' file 显示文件file中包含101的匹配行。
awk '/101/,/105/' file
awk '$1 == 5' file
awk '$1 == "CT"' file 注意必须带双引号
awk '$1 * $2 >100 ' file
awk '$2 >5 && $2<=15' file
2、awk '{print NR,NF,$1,$NF,}' file 显示文件file的当前记录号、域数和每一行的第一个和最后一个域。
awk '/101/ {print $1,$2 + 10}' file 显示文件file的匹配行的第一、二个域加10。
awk '/101/ {print $1$2}' file
awk '/101/ {print $1 $2}' file 显示文件file的匹配行的第一、二个域，但显示时域中间没有分隔符。
3、df | awk '$4>1000000 ' 通过管道符获得输入，如：显示第4个域满足条件的行。
4、awk -F "|" '{print $1}' file 按照新的分隔符“|”进行操作。
awk 'BEGIN { FS="[: \t|]" }
{print $1,$2,$3}' file 通过设置输入分隔符（FS="[: \t|]"）修改输入分隔符。

Sep="|"
awk -F $Sep '{print $1}' file 按照环境变量Sep的值做为分隔符。
awk -F '[ :\t|]' '{print $1}' file 按照正则表达式的值做为分隔符，这里代表空格、:、TAB、|同时做为分隔符。
awk -F '[][]' '{print $1}' file 按照正则表达式的值做为分隔符，这里代表[、]
5、awk -f awkfile file 通过文件awkfile的内容依次进行控制。
cat awkfile
/101/{print "\047 Hello! \047"} --遇到匹配行以后打印 ' Hello! '.\047代表单引号。
{print $1,$2} --因为没有模式控制，打印每一行的前两个域。
6、awk '$1 ~ /101/ {print $1}' file 显示文件中第一个域匹配101的行（记录）。
7、awk 'BEGIN { OFS="%"}
{print $1,$2}' file 通过设置输出分隔符（OFS="%"）修改输出格式。
8、awk 'BEGIN { max=100 ;print "max=" max} BEGIN 表示在处理任意行之前进行的操作。
{max=($1 >max ?$1:max); print $1,"Now max is "max}' file 取得文件第一个域的最大值。
（表达式1?表达式2:表达式3 相当于：
if (表达式1)
表达式2
else
表达式3
awk '{print ($1>4 ? "high "$1: "low "$1)}' file
9、awk '$1 * $2 >100 {print $1}' file 显示文件中第一个域匹配101的行（记录）。
10、awk '{$1 == 'Chi' {$3 = 'China'; print}' file 找到匹配行后先将第3个域替换后再显示该行（记录）。
awk '{$7 %= 3; print $7}' file 将第7域被3除，并将余数赋给第7域再打印。
11、awk '/tom/ {wage=$2+$3; printf wage}' file 找到匹配行后为变量wage赋值并打印该变量。
12、awk '/tom/ {count++;}
END {print "tom was found "count" times"}' file END表示在所有输入行处理完后进行处理。
13、awk 'gsub(/\$/,"");gsub(/,/,""); cost+=$4;
END {print "The total is $" cost>"filename"}' file gsub函数用空串替换$和,再将结果输出到filename中。
1 2 3 $1,200.00
1 2 3 $2,300.00
1 2 3 $4,000.00

awk '{gsub(/\$/,"");gsub(/,/,"");
if ($4>1000&&$4<2000) c1+=$4;
else if ($4>2000&&$4<3000) c2+=$4;
else if ($4>3000&&$4<4000) c3+=$4;
else c4+=$4; }
END {printf "c1=[%d];c2=[%d];c3=[%d];c4=[%d]\n",c1,c2,c3,c4}"' file
通过if和else if完成条件语句

awk '{gsub(/\$/,"");gsub(/,/,"");
if ($4>3000&&$4<4000) exit;
else c4+=$4; }
END {printf "c1=[%d];c2=[%d];c3=[%d];c4=[%d]\n",c1,c2,c3,c4}"' file
通过exit在某条件时退出，但是仍执行END操作。
awk '{gsub(/\$/,"");gsub(/,/,"");
if ($4>3000) next;
else c4+=$4; }
END {printf "c4=[%d]\n",c4}"' file
通过next在某条件时跳过该行，对下一行执行操作。


14、awk '{ print FILENAME,$0 }' file1 file2 file3>fileall 把file1、file2、file3的文件内容全部写到fileall中，格式为
打印文件并前置文件名。
15、awk ' $1!=previous { close(previous); previous=$1 }
{print substr($0,index($0," ") +1)>$1}' fileall 把合并后的文件重新分拆为3个文件。并与原文件一致。
16、awk 'BEGIN {"date"|getline d; print d}' 通过管道把date的执行结果送给getline，并赋给变量d，然后打印。
17、awk 'BEGIN {system("echo \"Input your name:\\c\""); getline d;print "\nYour name is",d,"\b!\n"}'
通过getline命令交互输入name，并显示出来。
awk 'BEGIN {FS=":"; while(getline< "/etc/passwd" >0) { if($1~"050[0-9]_") print $1}}'
打印/etc/passwd文件中用户名包含050x_的用户名。

18、awk '{ i=1;while(i awk '{ for(i=1;i type file|awk -F "/" '
{ for(i=1;i { if(i==NF-1) { printf "%s",$i }
else { printf "%s/",$i } }}' 显示一个文件的全路径。
用for和if显示日期
awk 'BEGIN {
for(j=1;j<=12;j++)
{ flag=0;
printf "\n%d月份\n",j;
for(i=1;i<=31;i++)
{
if (j==2&&i>28) flag=1;
if ((j==4||j==6||j==9||j==11)&&i>30) flag=1;
if (flag==0) {printf "%02d%02d ",j,i}
}
}
}'
19、在awk中调用系统变量必须用单引号，如果是双引号，则表示字符串
Flag=abcd
awk '{print '$Flag'}' 结果为abcd
awk '{print "$Flag"}' 结果为$Flag

 

1、删除重复的行 
#awk '!a[]++'

2、将数据文件中的每个词的第一个字母变成大写
cat test
linux is long live!!!
i am a cuer

awk ',1,1); sub(/^./,toupper(first),); print }' test
Linux is long live!!!
I am a cuer

3.awk 范例
a.显示文件第3和5行: awk 'NR==3 || NR ==5 ' /etc/passwd
b.打印前3行和7行以后的: awk 'NR<4 || NR> 7 {print $1}' /etc/passwd
c.打印以root和nobody开始的记录: awk '/^(root|nobody)/' /etc/passwd
d.如果记录以r或p或rp开头，就打印这个记录: awk '/^[rp]/' /etc/passwd
e.显示1-3和5-7行的信息
#awk '(NR<4 && NR>0) || (NR>4 && NR<8) {print}' /etc/passwd

4.含有root的列
#gawk  'BEGIN{FS=":" ;sum=0} $1 == "root" {sum=sum+1} END {print sum}' passwd

5.把合在一起的数字汉字用空格分开。
#sed 's/^[ ]*//;s/^[0-9]*/& /'  file
#awk '{gsub(/[^0-9]+/,"",$1);print $1}' file

6.统计一列数的总数和平均值。
#awk '{sum +=$2} END{print "sum:" sum}' test.txt
#awk '{sum +=$2} END{print "sum:" sum/NR}' test.txt

7.?指定如果零个或一个字符或扩展正则表达式的具体值在字符串中，则字符串匹配
#awk '/smith?/' file

8.显示包含abc或123的字符串
#awk '/abc|123/' file

9.将具有字符串ae或alle或anne或allnne的所有记录打印至标准输出
#awk '/a(ll)?(nn)?e/' file

10.{m}指定如果正好有m个模式的具体值位于字符串中，则字符串匹配.下面显示只包含两个l的字符串
#awk  '/l{2}/'  file

11.{m,}指定如果至少m个模式的具体值在字符串中，则字符串匹配，下面显示至少包含两个t的字符串
#awk '/t{2,}/'  file

12.{m, n}指定如果 m 和 n 之间(包含的 m 和 n)个模式的具体值在字符串中(其中m<= n)，则字符串匹配,下面显示包含1和2个er的字符串
#awk '/er{1,2}/' file

13.将具有zxm后跟以字母顺序从 a 到 h 排列的任何字符的所有记录打印至标准输出
#awk '/zxm[a-h]/'  file

14.[^String]在[ ]和在指定字符串开头的^指明正则表达式与方括号内的任何字符不匹配
#awk '/sm[^a-h]/' file


15. ~或!~表示指定变量与正则表达式匹配或不匹配的条件语句
#awk  '$1 ~ /n/' file

16.将把字符 h 作为第二个字段的第一个字符和最后一个字符的所有记录打印至标准输出
#awk  '$2 ~ /^h/'  file
#awk  '$2 ~ /h$/'  file


17.将具有以两个字符隔开的字符 a 和 e 的所有记录打印至标准输出
#awk  '/a..e/' file


18.将具有以零个或更多字符隔开的字符 a 和 e 打印至标准输出
#awk  '/a*e/' file

19.awk 命令识别大多数用于 C 语言约定中的转义序列，以及 awk 命令本身用作特殊字符的几个转义序列。转义序列是：
    转义序列表示的字符
    \"\"（双引号）
    \//（斜杠）字符
    \ddd其编码由 1、2 或 3 位八进制整数表示的字符，其中 d 表示一个八进制数位
    \\\ (反斜杠) 字符
    \a警告字符
    \b退格字符
    \f换页字符
    \n换行字符
    \r回车字符
    \t跳格字符
    \v垂直跳格


20.要显示长于 72 个字符的文件的行，请输入：
awk 'length($0) >72' file

21.要显示字start和stop之间的所有行，包含“start”和“stop”，请输入：
awk '/start/,/stop/' file

22.在屏幕上打印”What is your name?",并等待用户应答。当一行输入完毕后，getline函数从终端接收该行输入，并把它储存在自定义变量name中。如果第一个域匹配变量name的值，print函数就被执行，END块打印See you和name的值
#awk 'BEGIN{printf "What is your name?"; getline name < "/dev/tty" } $1 ~name {print "Found" name on line ", NR "."} END{print "See you," name "."} test

23.awk将逐行读取文件/etc/passwd的内容，在到达文件末尾前，计数器lc一直增加，当到末尾时，打印lc的值
#awk 'BEGIN{while (getline < "/etc/passwd" > 0) lc++; print lc}'

24.system函数可以在awk中执行linux的命令。如：
#awk 'BEGIN{system("clear")'

25.如果第一个域小于第二个域则打印
#awk '{if ($1 <$2) print $2 "too high"}' test

26.变量的初始值为1，若i小于可等于NF(记录中域的个数),则执行打印语句，且i增加1。直到i的值大于NF.
# awk '{ i = 1; while ( i <= NF ) { print NF,$i; i++}}' test
# awk '{for (i = 1; i<NF; i++) print NF,$i}' test

27.将一个文件的总行数显示出来
#gawk '{nlines++} END {print nlines}'  file

28.显示拥有至少一个字段的所有行。这是一个简单的方法，将一个文件里的所有空白行删除
#gawk 'NF > 0'  file

29.此程序会显示出范围是0 到100 之间的7 个随机数
#gawk 'BEGIN {for (i = 1; i <= 7; i++) print int(101 * rand())}'

30.此程序会显示出所有指定的文件的总字节数
#ls -l files | gawk '{x += $4}; END {print "total bytes: " x}'

31.此程序会将指定文件里最长一行的长度显示出来。expand 会将tab 改成space，所以是用实际的右边界来做长度的比较。
#expand file | gawk '{if (x < length()) x = length()} END {print "maximum line length is " x}'


32.显示所有只有四个字符的字段
#awk 'length($1)==4{print $1}'  file


33.显示所有以一个C或E开头的字段
$ awk -F"[: ]" '$1~/^C|E/{print $1}' datafile

34.在文件的第一行前插入一行
#awk 'BEGIN {print "new line"} {print $0}' file >file1

35.在文件末尾添加一行
#awk 'END {print "THE END"} {print $0}' file >file1


36.awk和cut的相同用法
#awk -F: '{print $1,$2,$3}' file
#cut -d: -f2,3,4,5  file
#cut  -c 1-5 /etc/passwd 显示文件中的前1-5个字符

37.以@或:为分隔符的文件
awk -F[@:] '{print $1}' file

38.结果以$分隔
awk 'BEGIN{FS=":"} {OFS="$"} {if($1~/Mike/) print "",$3,$4,$5}' love

39.把一行竖排的数据转换成横排
awk '{printf("%s,",$1)}' a.txt
awk '{printf ("%s\n",$0)}' a.txt

40.systime函数返回从1970年1月1日开始到当前时间(不计闰年)的整秒数
awk '{ now = systime(); print now }'
awk '{ now=strftime( "%D", systime() ); print now }'
awk '{ now=strftime("%m/%d/%y"); print now }'


41.将时间戳转成日值的awk方法
echo "1180051515"|awk '{print strftime("%F %T",$0)}'

42.打印输入记录的最后一个字段
awk -F/ '{print $NF}' a.txt

43.打印输入记录的第2个字段
awk '{x=2;print $x}' a.txt

44.显示文件a.txt的当前记录号、域数和每一行的第一个和最后一个域和文件名
awk '{print NR,NF,$1,$NF,FILENAME}'  a.txt

45.在awk中调用系统变量必须用单引号，如果是双引号，则表示字符串
Flag=abcd
awk ‘{print ‘$Flag’}’ 结果为abcd
awk ‘{print “$Flag”}’ 结果为$Flag

46.把三个文件的内容追加到一个文件里
awk '{print FILENAME,$0}' a b c >all


47.通过管道把date的执行结果送给getline，并赋给变量d，然后打印。
awk 'BEGIN {"date"|getline d; print d}'

48.通过getline命令交互输入name，并显示出来:
#awk 'BEGIN {system("echo \"Input your name:\\c\""); getline d;print "\nYour name is",d,"\b!\n"}'

49.输出不换行
#awk -F: '{printf  $1}'  /etc/passwd 

50.toupper和tolower函数可用于字符串大小间的转换
#awk '{ print toupper("test"), tolower("TEST") }'

51.split函数可按给定的分隔符把字符串分割为一个数组.如果分隔符没提供,按当前FS值进行分割
#awk '{ split( "20:18:00", time, ":" ); print time[2],time[1],time[3] }'

52.sub函数匹配记录中最大,最靠左边的子字符串的正则表达式,并用替换字符串替换这些字符串.如果没有指定目标字符串就默认使用整个记录.替换只发生在第一次匹配的时候.格式如下:
#awk '{ sub(/test/, "mytest"); print }' testfile
#awk '{ sub(/test/, "mytest"); $1}; print }' testfile

53.system函数可以在awk中执行linux的命令。
#awk 'BEGIN{system("clear")}'

54.执行shell的date命令，并通过管道输出给getline，然后getline从管道中读取并将输入赋值给d，split函数把变量d转化成数组mon，然后打印数组mon的第二个元素
#awk 'BEGIN{"date" | getline d; split(d,mon); print mon[2]}' test

 

# 每行后面增加一行空行
awk '1;{print ""}'
awk 'BEGIN{ORS="\n\n"};1'
# 每行后面增加一行空行。输出文件不会包含连续的两个或两个以上的空行
# 注意：在Unix系统， DOS行包括的 CRLF （\r\n） 通常会被作为非空行对待
# 因此 'NF' 将会返回TRUE。
awk 'NF{print $0 "\n"}'
# 每行后面增加两行空行
awk '1;{print "\n"}'
编号和计算：
# 以文件为单位，在每句行前加上编号 （左对齐）.
# 使用制表符 （\t） 来代替空格可以有效保护页变的空白。
awk '{print FNR "\t" $0}' files*
# 用制表符 （\t） 给所有文件加上连贯的编号。
awk '{print NR "\t" $0}' files*
# number each line of a file （number on left, right-aligned）
# Double the percent signs if typing from the DOS command prompt.
awk '{printf("%5d : %s\n", NR,$0)}'
# 给非空白行的行加上编号
# 记得Unix对于 \r 的处理的特殊之处。（上面已经提到）
awk 'NF{$0=++a " :" $0};{print}'
awk '{print (NF? ++a " :" :"") $0}'
# 计算行数 （模拟 "wc -l"）
awk 'END{print NR}'
# 计算每行每个区域之和
awk '{s=0; for (i=1; i<=NF; i++) s=s+$i; print s}'
# 计算所有行所有区域的总和
awk '{for (i=1; i<=NF; i++) s=s+$i}; END{print s}'
# 打印每行每区域的绝对值
awk '{for (i=1; i<=NF; i++) if ($i < 0) $i = -$i; print }'
awk '{for (i=1; i<=NF; i++) $i = ($i < 0) ? -$i : $i; print }'
# 计算所有行所有区域（词）的个数
awk '{ total = total + NF }; END {print total}' file
# 打印包含 "Beth" 的行数
awk '/Beth/{n++}; END {print n+0}' file
# 打印第一列最大的行
# 并且在行前打印出这个最大的数
awk '$1 > max {max=$1; maxline=$0}; END{ print max, maxline}'
# 打印每行的列数，并在后面跟上此行内容
awk '{ print NF ":" $0 } '
# 打印每行的最后一列
awk '{ print $NF }'
# 打印最后一行的最后一列
awk '{ field = $NF }; END{ print field }'
# 打印列数超过4的行
awk 'NF > 4'
# 打印最后一列大于4的行
awk '$NF > 4'
文本转换和替代：
# 在Unix环境：转换DOS新行 （CR/LF） 为Unix格式
awk '{sub(/\r$/,"");print}' # 假设每行都以Ctrl-M结尾
# 在Unix环境：转换Unix新行 （LF） 为DOS格式
awk '{sub(/$/,"\r");print}
# 在DOS环境：转换Unix新行 （LF） 为DOS格式
awk 1
# 在DOS环境：转换DOS新行 （CR/LF） 为Unix格式
# DOS版本的awk不能运行, 只能用gawk:
gawk -v BINMODE="w" '1' infile >outfile
# 用 "tr" 替代的方法。
tr -d \r <infile >outfile # GNU tr 版本为 1.22 或者更高
# 删除每行前的空白（包括空格符和制表符）
# 使所有文本左对齐
awk '{sub(/^[ \t]+/, ""); print}'
# 删除每行结尾的空白（包括空格符和制表符）
awk '{sub(/[ \t]+$/, "");print}'
# 删除每行开头和结尾的所有空白（包括空格符和制表符）
awk '{gsub(/^[ \t]+|[ \t]+$/,"");print}'
awk '{$1=$1;print}' # 每列之间的空白也被删除
# 在每一行开头处插入5个空格 （做整页的左位移）
awk '{sub(/^/, " ");print}'
# 用79个字符为宽度，将全部文本右对齐
awk '{printf "%79s\n", $0}' file*
# 用79个字符为宽度，将全部文本居中对齐
awk '{l=length();s=int((79-l)/2); printf "%"(s+l)"s\n",$0}' file*
# 每行用 "bar" 查找替换 "foo"
awk '{sub(/foo/,"bar");print}' # 仅仅替换第一个找到的"foo"
gawk '{$0=gensub(/foo/,"bar",4);print}' # 仅仅替换第四个找到的"foo"
awk '{gsub(/foo/,"bar");print}' # 全部替换
# 在包含 "baz" 的行里，将 "foo" 替换为 "bar"
awk '/baz/{gsub(/foo/, "bar")};{print}'
# 在不包含 "baz" 的行里，将 "foo" 替换为 "bar"
awk '!/baz/{gsub(/foo/, "bar")};{print}'
# 将 "scarlet" 或者 "ruby" 或者 "puce" 替换为 "red"
awk '{gsub(/scarlet|ruby|puce/, "red"); print}'
# 倒排文本 （模拟 "tac"）
awk '{a[i++]=$0} END {for (j=i-1; j>=0;) print a[j--] }' file*
# 如果一行结尾为反斜线符，将下一行接到这行后面
# （如果有连续多行后面带反斜线符，将会失败）
awk '/\\$/ {sub(/\\$/,""); getline t; print $0 t; next}; 1' file*
# 排序并打印所有登录用户的姓名
awk -F ":" '{ print $1 | "sort" }' /etc/passwd
# 以相反的顺序打印出每行的前两列
awk '{print $2, $1}' file
# 调换前两列的位置
awk '{temp = $1; $1 = $2; $2 = temp}' file
# 打印每行，并删除第二列
awk '{ $2 = ""; print }'
# 倒置每行并打印
awk '{for (i=NF; i>0; i--) printf("%s ",i);printf ("\n")}' file
# 删除重复连续的行 （模拟 "uniq"）
awk 'a !~ $0; {a=$0}'
# 删除重复的、非连续的行
awk '! a[$0]++' # 最简练
awk '!($0 in a) {a[$0];print}' # 最有效
# 用逗号链接每5行
awk 'ORS=%NR%5?",":"\n"' file #bug awk 'ORS=NR%5?",":"\n"' file
选择性的打印某些行：
# 打印文件的前十行 （模拟 "head"）
awk 'NR < 11'
# 打印文件的第一行 （模拟 "head -1"）
awk 'NR>1{exit};1'
# 打印文件的最后两行 （模拟 "tail -2"）
awk '{y=x "\n" $0; x=$0};END{print y}'
# 打印文件的最后一行 （模拟 "tail -1"）
awk 'END{print}'
# 打印匹配正则表达式的行 （模拟 "grep"）
awk '/regex/'
# 打印不匹配正则表达式的行 （模拟 "grep -v"）
awk '!/regex/'
# 打印匹配正则表达式的前一行，但是不打印当前行
awk '/regex/{print x};{x=$0}'
awk '/regex/{print (x=="" ? "match on line 1" : x)};{x=$0}'
# 打印匹配正则表达式的后一行，但是不打印当前行
awk '/regex/{getline;print}'
# 以任何顺序查找包含 AAA、BBB 和 CCC 的行
awk '/AAA/; /BBB/; /CCC/'
# 以指定顺序查找包含 AAA、BBB 和 CCC 的行
awk '/AAA.*BBB.*CCC/'
# 打印长度大于64个字节的行
awk 'length > 64'
# 打印长度小于64个字节的行
awk 'length < 64'
# 打印从匹配正则起到文件末尾的内容
awk '/regex/,0'
awk '/regex/,EOF'
# 打印指定行之间的内容 （8-12行, 包括第8和第12行）
awk 'NR==8,NR==12'
# 打印第52行
awk 'NR==52'
awk 'NR==52 {print;exit}' # 对于大文件更有效率
# 打印两个正则匹配间的内容 （包括正则的内容）
awk '/Iowa/,/Montana/' # 大小写敏感
选择性的删除某些行：
# 删除所有空白行 （类似于 "grep '.' "）
awk NF
awk '/./'

</code></pre>

