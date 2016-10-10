# Linux 语句集锦
***
## Linux Shell 文本处理工具集锦
本文将介绍Linux下使用Shell处理文本时最常用的工具：  
find、grep、xargs、sort、uniq、tr、cut、paste、wc、sed、awk；  
提供的例子和参数都是最常用和最为实用的；  
我对shell脚本使用的原则是命令单行书写，尽量不要超过2行；  
如果有更为复杂的任务需求，还是考虑python吧；  
#### find 文件查找
* 查找txt和pdf文件  
> find . ( -name "\*.txt" -o -name "\*.pdf" ) -print

* 正则方式查找.txt和pdf文件
> find . -regex  ".\*(.txt|.pdf)$"  

 -iregex： 忽略大小写的正则
* 否定参数查找所有非txt文件
> find . ! -name "\*.txt" -print  

* 指定搜索深度打印出当前目录的文件（深度为1）
> find . -maxdepth 1 -type f  

* 从根目录开始查找所有扩展名为.log的文本文件，并找出包含”ERROR”的行  
```
find / -type f -name "*.log" | xargs grep "ERROR"  
```

* 从当前目录开始查找所有扩展名为.in的文本文件，并找出包含”thermcontact”的行  
```
find . -name "*.in" | xargs grep "thermcontact"   
```

##### 定制搜索
* 按类型搜索  
> find . -type d -print  //只列出所有目录  

 -type f 文件 / l 符号链接  
* 按时间搜索：-atime 访问时间 (单位是天，分钟单位则是-amin，以下类似）-mtime 修改时间 （内容被修改）-ctime 变化时间 （元数据或权限变化）  
    最近7天被访问过的所有文件：  
> find . -atime 7 -type f -print

* 按大小搜索：w字 k M G寻找大于2k的文件
> find . -type f -size +2k

* 按权限查找
> find . -type f -perm 644 -print //找具有可执行权限的所有文件

* 按用户查找
> find . -type f -user weber -print// 找用户weber所拥有的文件

##### 找到后的后续动作  
* 删除：删除当前目录下所有的swp文件：
> find . -type f -name ".swp" -delete  

* 执行动作（强大的exec）
> find . -type f -user root -exec chown weber {} ; //将当前目录下的所有权变更为weber

 注：{}是一个特殊的字符串，对于每一个匹配的文件，{}会被替换成相应的文件名；  
 eg：将找到的文件全都copy到另一个目录：
> find . -type f -mtime +10 -name "\*.txt" -exec cp {} OLD ;

* 结合多个命令tips: 如果需要后续执行多个命令，可以将多个命令写成一个脚本。然后 -exec 调用时执行脚本即可；   
> -exec ./commands.sh {} \;

##### -print的定界符  
默认使用’n’作为文件的定界符；  
-print0 使用”作为文件的定界符，这样就可以搜索包含空格的文件；  
### grep 文本搜索
grep match_patten file // 默认访问匹配行  
* 常用参数-o 只输出匹配的文本行 VS -v 只输出没有匹配的文本行-c 统计文件中包含文本的次数  
> grep -c "text" filename  

 -n 打印匹配的行号  
 -i 搜索时忽略大小写  
 -l 只打印文件名    
* 在多级目录中对文本递归搜索(程序员搜代码的最爱)：  
> grep "class" . -R -n  

* 匹配多个模式  
> grep -e "class" -e "vitural" file  

* grep输出以作为结尾符的文件名：（-z）  
> grep "test" file\* -lZ| xargs -0 rm  

### xargs 命令行参数转换  
xargs 能够将输入数据转化为特定命令的命令行参数；这样，可以配合很多命令来组合使用。比如grep，比如find；  
* 将多行输出转化为单行输出cat file.txt| xargsn 是多行文本间的定界符  
* 将单行转化为多行输出cat single.txt | xargs -n 3-n：指定每行显示的字段数  
##### xargs参数说明  
-d 定义定界符 （默认为空格 多行的定界符为 n）  
-n 指定输出为多行  
-I {} 指定替换字符串，这个字符串在xargs扩展时会被替换掉,用于待执行的命令需要多个参数时  
eg：  
> cat file.txt | xargs -I {} ./command.sh -p {} -1  

 -0：指定为输入定界符  
 eg：统计程序行数  
> find source_dir/ -type f -name "\*.cpp" -print0 |xargs -0 wc -l

### sort 排序  
字段说明：  
-n 按数字进行排序 VS -d 按字典序进行排序  
-r 逆序排序  
-k N 指定按第N列排序  
eg：  
> sort -nrk 1 data.txt  
> ort -bd data // 忽略像空格之类的前导空白字符  

### uniq 消除重复行  
* 消除重复行  
> sort unsort.txt | uniq

* 统计各行在文件中出现的次数  
> sort unsort.txt | uniq -c  

* 找出重复行  
> sort unsort.txt | uniq -d

 可指定每行中需要比较的重复内容：-s 开始位置 -w 比较字符数  

### 用tr进行转换  
通过使用 tr，可以非常容易地实现 sed 的许多最基本功能

### 查看网络IO
sar -n DEV -u 2

### 网卡流量监控  
```
watch -n 1 "/sbin/ifconfig eno16777736 | grep bytes"  
```

### top命令查看python进程  
```
top -p `pgrep python | tr "\\n" "," | sed 's/,$//'`  
```

### nohup命令查看  
```
nohup command > myout.file 2>&1  &  
```
使用 jobs 查看任务    
使用 fg %n关闭  

### 清理缓存  
```
echo 3 > /proc/sys/vm/drop_caches  
```

### crontab使用  
```
$ vim /etc/crontab  
10  *  *  *  * root bash /home/sparkwork/scheduler_timer_crontab.sh >> /home/sparkwork/logs/scheduler_monitor.log 2>1&  
```

### wc命令  
该命令各选项含义如下：
* -c 统计字节数  
* -l 统计行数  
* -w 统计字数  
举例：
1. 统计demo目录下，js文件数量  
```
find demo/ -name "*.js" |wc -l  
```

2. 统计demo目录下所有js文件代码行数  
```
find demo/ -name "*.js" |xargs cat|wc -l 或 wc -l `find ./ -name "*.js"`|tail -n1  
```

3. 统计demo目录下所有js文件代码行数，过滤了空行  
```
find /demo -name "*.js" |xargs cat|grep -v ^$|wc -l  
```

### wget  
有时候我们需要wget一个文件下载到指定的目录下，或者重命名成指定的名字  
```
wget -r -p -np -k -P ~/tmp/ http://java-er.com  
```
参数：
* -P 表示下载到哪个目录
* -r 表示递归下载
* -np 表示不下载旁站连接.不下载父目录
* -k 表示将下载的网页里的链接修改为本地链接.
* -p 获得所有显示网页所需的元素
* -c 断点续传
* -nd 递归下载时不创建一层一层的目录，把所有的文件下载到当前目录
* -L 递归时不进入其它主机，如wget -c -r www.xxx.org/
* -A 指定要下载的文件样式列表，多个样式用逗号分隔
* -i 后面跟一个文件，文件内指明要下载的URL
* 使用代理下载
```
wget -Y on -p -k https://sourceforge.net/projects/wvware/
```
代理可以在环境变量或wgetrc文件中设定

### tar解压  
1. 对于.tar结尾的文件 
   tar -xf all.tar 
2. 对于.gz结尾的文件 
   gzip -d all.gz 
   gunzip all.gz 
3. 对于.tgz或.tar.gz结尾的文件 
   tar -xzf all.tar.gz 
   tar -xzf all.tgz 
4. 对于.bz2结尾的文件 
   bzip2 -d all.bz2 
   bunzip2 all.bz2 
5. 对于tar.bz2结尾的文件 
   tar -xjf all.tar.bz2 
6. 对于.Z结尾的文件 
   uncompress all.Z 
7. 对于.tar.Z结尾的文件 
   tar -xZf all.tar.z 
8. Windows下的zip
   unzip all.zip

