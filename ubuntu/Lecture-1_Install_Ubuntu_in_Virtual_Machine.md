在虚拟机中安装Ubuntu
---

1. 在下载系统镜像之前，需要搞明白两件事情：

   1. 下载`Desktop version`还是`Server version`. 这里选择了`Server version`, 因为接下来的需求中基本不需要GUI界面，即便需要也可以安装。
   2. 宿主机的CPU架构是`x86`还是`arm`,是`32-bit`还是`64-bit`, 这些信息可以在系统信息中得到。这里我使用的平台是Mac mini M1，选择`arm64`版本。

2. 下载系统镜像，下面两种方法都可以

   1. 从官方网站https://cn.ubuntu.com/ 找到目标版本镜像的下载链接

   2. 从国内开源镜像源下载（速度较快）

      1. https://developer.aliyun.com/mirror/
      2. https://mirrors.huaweicloud.com/home
      3. https://mirror.nju.edu.cn/
      4. https://mirrors.ustc.edu.cn/

      前两个是商业公司的镜像站，后两个是大学的镜像站，都很不错。

3. 安装

   1. 这里使用Parallels Desktop安装，VMware/VirtualBox都可以，大同小异。其中VirtualBox是免费的。
   2. 



