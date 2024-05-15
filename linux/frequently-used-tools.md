# linux一些常用工具

* __glances__  
  是来自一位法国开发者的系统监控工具 glances ，能够在一个简洁的界面中查看到当前系统的负载、网络、IO 读写、硬件温度、进程详情等信息.  
  `sudo apt install -y glances`

* __htop,iotop,bmon,dstat__  
  如果你追求更小巧一些的工具，传统的老牌神器 htop 自然是不遑多让的，你甚至可以在 htop 中快速的设置每一个进程所能使用的资源最大值，来避免某个进程过分自私，用完整台设备的资源。  
  `sudo apt install -y htop`

  然而因为 htop 只专注于传统 “top” 命令范畴相关的功能，所以网络和存储相关，我们想偷懒，就还得安装下面几个工具  
  `sudo apt install -y iftop iotop bmon dstat`

* __更简单的 Docker 安装__  
  参见[在笔记本上搭建高性价比的 Linux 学习环境：基础篇](https://soulteary.com/2022/06/21/building-a-cost-effective-linux-learning-environment-on-a-laptop-the-basics.html)