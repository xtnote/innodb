
### tar
tar -cvf examples.tar files
```
-c, --create  create a new archive 创建一个归档文件
-v, --verbose verbosely list files processed 显示创建归档文件的进程
-f, --file=ARCHIVE use archive file or device ARCHIVE  后面要立刻接被处理的档案名,比如--file=examples.tar
```

tar -xvf examples.tar （解压至当前目录下）
tar -xvf examples.tar  -C /path (/path 解压至其它路径)
`-x, --extract, extract files from an archive 从一个归档文件中提取文件`

tar -zcvf examples.tgz examples
tar -zxvf examples.tar （解压至当前执行目录下）
tar -zxvf examples.tar  -C /path (/path 解压至其它路径)
`-z, --gzip filter the archive through gzip 通过gzip压缩的形式对文件进行归档`

tar -jcvf examples.tar.bz2 examples   (examples为当前执行路径下的目录)
tar -jcvf file.tar.bz2 dir #dir目录
tar -jxvf examples.tar.bz2 （解压至当前执行目录下）
tar -jxvf examples.tar.bz2  -C /path (/path 解压至其它路径)
`-j, --bzip2 filter the archive through bzip2 通过bzip2压缩的形式对文件进行归档`
