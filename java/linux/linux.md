# 例子

## 根据输入执行相应操作

```shell
#!/bin/bash
string="Bigdata process framework is Hadoop,Hadoop is an open source project"

function print_tips
{
	echo "***********************************"
	echo "(1) 打印string长度"
	echo "(2) 删除字符串中所有的Hadoop"
	echo "(3) 替换第一个Hadoop为Mapreduce"
	echo "(4) 替换全部Hadoop为Mapreduce"
	echo "***********************************"
}

function len_of_string
{
	echo "${#string}"
}

function del_hadoop 
{
	echo "${string//Hadoop/}"
}

function rep_haddop_mapreduce_first 
{
	echo "${string/Hadoop/Mapreduce}"
}

function rep_hadoop_mapreduce_all
{
	echo "${string//Hadoop/Mapreduce}"
}

while true
do
	echo "【string=${string}】"
	echo
	print_tips
	read -p "Pls input your choice(1|2|3|4|q|Q)：" choice

	case $choice in
		1)
			len_of_string
			;;
		2)
			del_hadoop
			;;
		3)
			rep_haddop_mapreduce_first
			;;
		4)
			rep_hadoop_mapreduce_all
			;;
		q|Q)
			exit
			;;
		*)
			echo "Error,input only in {1|2|3|4|q|Q}"
			;;
	esac
done
```

## 获取系统的所有用户并输出

```shell
# -d 指定分隔符；-f 选择第几个片段
#!/bin/bash
index=1

for user in `cat /etc/passwd | cut -d ":" -f 1`
do 
	echo "This is $index user：$user"
	index=$(($index+1))
done	
```

## 系统时间计算

```shell
date +%Y
echo "This is $(date +%Y) year"
echo "This is $(($(date +%Y)+1)) year"
echo "This year have passed $(date +%j) days"
echo "This year have passwd $(($(date +%j) / 7))"
# date_test.sh:行5: 365 - 091: 数值太大不可为算数进制的基 （错误符号是 "091"）,需转成10进制
# echo "This year have passed $((10#$(date +%j) /7)) week"
```

## 判断进程是否存在

```shell
# 判定nginx进程是否存在，若不存在则自动拉起该进程
#!/bin/bash
# 获取nginx进程个数，-v过滤掉，wc -l统计行数，-w只显示字数
nginx_process_num=$(ps -ef | grep nginx | grep -v grep | wc -l)
if [$nginx_process_num -eq 0];then
	systemctl start nginx
fi	
```

```shell
# 写一个监控nginx的脚本；如果Nginx服务宕掉，则该脚本可以检测到并将进程启动
# echo $? 为 0 表示上条命令正确执行
# ps -ef | grep nginx | grep -v grep
# 执行的脚本也会作为一个子进程在系统中执行，所以如果脚本名中含有nginx，则ps -ef|grep nginx就不正确
# echo $$   # 返回执行脚本的子进程ID
#!/bin/bash
this_pid=$$
while true
do
# 过滤掉子进程
ps -ef | grep nginx | grep -v grep | grep -v $this_pid &> /dev/null
if [ $? -eq 0 ];then
	echo "Nginx is running well"
	sleep 3
else
	systemctl start nginx
	echo "Nginx is down,Start it..."
fi	
done
# 后台启动
nohup sh nginx_daemon.sh &
tail -f nohup.out
# netstat -tnlp | grep :80
```

## 输入正整数并计算累加值

```shell
# 提示用户输入一个正整数num，然后计算1+2+3+...+sum的值;必须对num是否为正整数做判断，不符合应当允许再次输入
#sum.sh
#!/bin/bash
while true
do
	read -p "Please input a positive number：" num
	# 如果不是整数和1相加，则会报错
	expr $num + 1 &> /dev/null
	# 上一步命令执行不正确的话返回的是非0数
	if [ $? -eq 0 ];then
	# 判断是否是正整数
		if [ `expr $num \> 0` -eq 1 ];then
			for((i=1;i<=$num;i++))
			do
				sum=`expr $sum + $i`
			done
			echo "1+2+...+$num = $sum"
			exit
		fi
	fi
	echo "error,input ellegal"
	continue
done
```

## 数学运算之bc

```shell
# bc_test.sh
#!/bin/bash
# bc是bash内建的运算器，支持浮点数运算,内建变量scale可以设置，默认为0
read -p "num1:" num1
read -p "num2:" num2
# echo "scale=4;$num1 / $num2" | bc
num3=`echo "scale=4;$num1 / $num2" | bc`
echo "$num1 / $num2 = $num3"
```

# 常用命令

```shell
# 后台运行java程序
nohup java -jar xxx.jar >temp.txt &
# 通过jobs命令查看后台运行任务
jobs
# 将某个作业调回前台控制，只需要fg+编号即可
fg 23
```

* 命令替换

  ```shell
  # 在bash中，$(command)与`command`（反引号）都是用来作命令替换,来重组命令行，先完成引号里的命令行，然后将其结果替换出来，再重组成新的命令行
  [root@localhost ~]# echo today is $(date "+%Y-%m-%d")
  today is 2017-11-07
  [root@localhost ~]# echo today is `date "+%Y-%m-%d"`
  today is 2017-11-07
  # ``和$()两者是等价的，但推荐初学者使用$()，易于掌握；缺点是极少数UNIX可能不支持，但``都支持
  [root@localhost ~]#  echo Linux `echo Shell `echo today is `date "+%Y-%m-%d"```
  Linux Shellecho today is 2017-11-07     #过多使用``会有问题
  [root@localhost ~]# echo Linux `echo Shell $(echo today is $(date "+%Y-%m-%d"))`
  Linux Shell today is 2017-11-07    ``和$()混合使用
  [root@localhost ~]# echo Linux $(echo Shell $(echo today is $(date "+%Y-%m-%d")))
  Linux Shell today is 2017-11-07    #多个$()同时使用也不会有问题
  
  # $(())主要用来进行【整数运算】，包括加减乘除，引用变量前面可以加$，也可以不加$
  num5=$(((100+20) / 30))
  echo $num5
  
  num1=20
  num2=30
  
  ((num1++))
  ((num2--))
  
  echo $num1  # 21
  echo $num2  # 29
  
  num3=$((num1+num2))
  echo $num3  # 50
  num4=$((num1+num2*2))
  echo $num4	# 79
  ```

* 变量替换

  ```shell
  #一般情况下，$var与${var}是没有区别的，但是用${ }会比较精确的界定变量名称的范围
  [root@localhost ~]# A=Linux
  [root@localhost ~]# echo $AB    #表示变量AB
  [root@localhost ~]# echo ${A}B    #表示变量A后连接着B
  LinuxB
  ```

# 变量

```shell
var1="I love you,Do you love me"

# 【替换】
# 变量内容符合旧字符串，则【第一个】旧字符串会被新字符串取代
# ${变量名/旧字符串/新字符串}
var6=${var1/love/LOVE}  # I LOVE you,Do you love me

# 变量内容符合旧字符串，则【全部的】旧字符串会被新字符串取代
# ${变量名//旧字符串/新字符串}
var7=${var1//love/LOVE} # I LOVE you,Do you LOVE me

# 【计算长度】
echo ${#var1}
echo `expr length "$var1"`

# 【抽取子串】
# ${string:position},从string中的position开始，索引下标从0开始
echo ${var1:0}									# I love you,Do you love me
# ${string:position:length},从position开始，匹配长度为length
echo ${var1:0:4}								# I lo
# ${string: -position},从右边开始匹配，注意有空格
echo ${var1: -2}								# me
```

