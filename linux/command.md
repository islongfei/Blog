* `stat fileName` 查看文件创建更新时间
* `top` ->`1` 查看cpu核数
* `top` KiB Mem 物理内存总量
* `cat /proc/cpuinfo`查看CPU的详细信息 
* `cat /proc/cpuinfo | grep "physical id" | sort | uniq | wc -l` 查看物理CPU个数
* `cat /proc/cpuinfo | grep "processor" | wc -l`查看逻辑CPU个数
