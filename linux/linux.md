
### tar
```
tar -cvf examples.tar files
-c, --create  create a new archive 创建一个归档文件
-v, --verbose verbosely list files processed 显示创建归档文件的进程
-f, --file=ARCHIVE use archive file or device ARCHIVE  后面要立刻接被处理的档案名,比如--file=examples.tar

tar -xvf examples.tar （解压至当前目录下）
tar -xvf examples.tar  -C /path (/path 解压至其它路径)
-x, --extract, extract files from an archive 从一个归档文件中提取文件

tar -zcvf examples.tgz examples
tar -zxvf examples.tar （解压至当前执行目录下）
tar -zxvf examples.tar  -C /path (/path 解压至其它路径)
-z, --gzip filter the archive through gzip 通过gzip压缩的形式对文件进行归档

tar -jcvf examples.tar.bz2 examples   (examples为当前执行路径下的目录)
tar -jcvf file.tar.bz2 dir #dir目录
tar -jxvf examples.tar.bz2 （解压至当前执行目录下）
tar -jxvf examples.tar.bz2  -C /path (/path 解压至其它路径)
-j, --bzip2 filter the archive through bzip2 通过bzip2压缩的形式对文件进行归档
```

### ssh-keygen
`ssh-keygen -t rsa` 生成密钥，生成的密钥存储于`~/.ssh`中

### date
`Sun Mar 10 23:09:31 CST 2019`
显示日期

### cal
```
March 2019
Su Mo Tu We Th Fr Sa
           1  2
3  4  5  6  7  8  9
10 11 12 13 14 15 16
17 18 19 20 21 22 23
24 25 26 27 28 29 30
31
```
显示日历

### bc
计算器

### sync
内存中数据写入磁盘

### shutdown/reboot/halt/poweroff
关机

### init
执行等级
--run level 0 :关机  
--run level 3 :纯文本模式  
--run level 5 :含有图形接口模式  
--run level 6 :重新启动  

### chgrp group [-R] dirname/filename
改变文件所属群组

### chown user:group [-R] dir/file
改变文件所属用户

### head [-n num] file
只看头几行

### tail [-n num] file
只看尾几行

### od [-t type] file
二进制方式查看

### file
查看文件类型

### which [-a] cmd
查找cmd位置，-a表示由path可以找到的所有指令均列出
