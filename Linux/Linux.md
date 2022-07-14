# Linux

## 主机规划与磁盘分区

### 1、各个硬件设备在Linux中的文件名

在Linux中，没个设备都被当成一个文件来对待，比如STAT接口的硬盘文件名问/dev/sd[a-d],括号内的字母为a-d当中的任意一个， 亦即有/dev/sda, /dev/sdb, /dev/sdc, 及 /dev/sdd这四个文件的意思。在Linux这个系统当中，几乎所有的硬件设备文件都在/dev这个目录 内，

