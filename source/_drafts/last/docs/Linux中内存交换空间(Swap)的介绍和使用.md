## 什么是swap
`swap`分区是`Linux`中特殊的一种存储，他在安装系统时被自动创建。  

用于当内存不够时，操作系统会取出内存中不活跃或不常用的一部分放到`swap`分区中，使内存空闲出一部分空间用于执行其他程序。相当于临时充当一部分内存，当操作系统想要访问`swap`空间上的的数据时，`swap`会把数据放回内存中供操作系统使用。

类似`windows`中的虚拟内存。每个操作系统都有这种内存调度处理，当内存不够用，还想跑点东西，可以临时选择增加`swap`空间。以达到`实际使用内存`超过`物理内存`。由于`swap`的读取速度远不如内存，最好的办法还是`加物理内存`。
## swap分区类型
 - **swap 文件**
 - **swap 分区**  

速度比较：物理内存 > swap分区 > swap文件。

由于`swap`分区需要一个新的磁盘，当你的磁盘已经装了不少内容，不方便重新规划的话，也只好使用swap文件了。本文使用swap文件方式。

## 查看swap信息
> 基于Cent OS 7.6 

 使用`swapon -s`命令查看系统中当前`swap`信息：  

```shell
swapon -s

Filename            Type        Size     Used        Priority
/swapfile           file        1048572  199668      -2
``` 
 - **Filename**：swap 分区/文件所在的位置
 - **Type**：swap类型；`partition`为分区类型，`file`为文件类型
 - **Size**：总容量，单位是KB
 - **Used**：已被使用的容量
 - **Priority**：优先级，越高越优先被操作系统使用
 
若查不到 swap 信息，使用`free -h`命令，查看内存使用情况：

```shell
free -h
              total        used        free      shared  buff/cache   available
Mem:           487M         65M        289M         44K        132M        398M
Swap:          1.0G        259M        764M
```
## 操作swap
>

创建`swap文件`， `fallocate`命令可以为文件预分配物理空间，和touch、vim的区别就是可瞬间创建超大的文件。快速创建的原因是通过分配块并将它们标记为未初始化，无需对数据块进行IO，可以快速完成预分配。当使用时再使用物理内存。`-l`后加大小，可以是k、m、g。再后面是保存位置和名称。
   
```shell
# 创建一个1g文件swapfile保存到/
sudo fallocate -l 1G /swapfile
``` 

修改权限，仅允许文件所有者进行读写操作：

```shell
sudo chmod 600 /swapfile
``` 
将这个文件设为 Linux 交换区：

```shell
sudo mkswap /swapfile
``` 
激活交换区：

```shell  
sudo swapon /swapfile
``` 
修改`/etc/fstab`，实现系统重启自动加载swap文件：  

```shell
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```
至此完成swap文件增加。使用`free -h`查看 swap 使用情况：  

```shell
free -h
              total        used        free      shared  buff/cache   available
Mem:           487M         65M        289M         44K        132M        398M
Swap:          1.0G        259M        764M
```
停止所有swap：  

```shell
sudo swapoff -a

# 查看内存使用情况，可以看到 Swap 信息都变成0，说明操作成功了
free -h
              total        used        free      shared  buff/cache   available
Mem:           487M        261M         92M        484K        134M        201M
Swap:     	      0           0           0
```

