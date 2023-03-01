# Linux-GREP命令

**打印匹配行的前后5行**

> * grep -5 'parttern' inputfile
> * grep -C 5 'parttern' inputfile

**打印匹配行的后5行**

> grep -A 5 'parttern' inputfile

**打印匹配行的前5行**

> grep -B 5 'parttern' inputfile

**查看某一个文件第5行和第10行**

> sed -n '5,10p' filename