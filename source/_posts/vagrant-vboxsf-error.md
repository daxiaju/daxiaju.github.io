---
title: Vagrant vboxsf error的解决方法
date: 2019-06-28 16:39:44
tags: 写代码
---

# 出错情况
以前一直用的一个vagrant环境，突然启动出现问题。
vagrant 运行在windows上，vagrant 启动centos7虚拟机，vagrant up时提示错误，无法将host上的文件夹映射到虚拟机里。

错误信息如下:

	Vagrant was unable to mount VirtualBox shared folders. This is usually
	because the filesystem "vboxsf" is not available. This filesystem is
	made available via the VirtualBox Guest Additions and kernel module.
	Please verify that these guest additions are properly installed in the
	guest. This is not a bug in Vagrant and is usually caused by a faulty
	Vagrant box. For context, the command attempted was:
	
	mount -t vboxsf -o uid=1000,gid=1000 vagrant /vagrant
	
	The error output from the command was:
	
	/sbin/mount.vboxsf: mounting failed with the error: No such device

## 处理方式

先ssh连进guest

    vagrant ssh

然后

	cd /opt/VBoxGuestAdditions-*/init  
	sudo ./vboxadd setup

提示信息

	VirtualBox Guest Additions: Building the VirtualBox Guest Additions kernel modules.  This may take a while.
	grep: Unmatched ) or \)
	This system is currently not set up to build kernel modules.
	Please install the Linux kernel "header" files matching the current kernel
	for adding new hardware support to the system.
	The distribution packages containing the headers are probably:
	    kernel-devel kernel-devel-3.10.0-957.21.3.el7.x86_64
	VirtualBox Guest Additions: Starting.
	VirtualBox Guest Additions: Building the VirtualBox Guest Additions kernel modules.  This may take a while.
	grep: Unmatched ) or \)
	This system is currently not set up to build kernel modules.
	Please install the Linux kernel "header" files matching the current kernel
	for adding new hardware support to the system.
	The distribution packages containing the headers are probably:
	    kernel-devel kernel-devel-3.10.0-957.21.3.el7.x86_64
	modprobe vboxguest failed
	The log file /var/log/vboxadd-setup.log may contain further information.

安装kernel

    sudo yum install kernel-devel kernel-devel-3.10.0-957.21.3.el7.x86_64

重新运行 ./vboxadd setup 提示错误

    VirtualBox Guest Additions: Building the VirtualBox Guest Additions kernel modules.  This may take a while.
	grep: Unmatched ) or \)
	VirtualBox Guest Additions: Look at /var/log/vboxadd-setup.log to find out what went wrong
	VirtualBox Guest Additions: Starting.

/var/log/vboxadd-setup.log:
  
	Building the main Guest Additions module.
	Building the shared folder support module.
	Building the graphics driver module.
	Error building the module:
	Could not find the X.Org or XFree86 Window System, skipping.
    
在虚拟机中安装xorg

    yum -y install xorg-x11-drivers xorg-x11-utils

在host上安装vbguest插件

	vagrant plugin install vagrant-vbguest

重新启动虚拟机，问题解决
	
	vagrant reload