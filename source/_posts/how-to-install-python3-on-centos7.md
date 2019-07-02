---
title: 怎样在Centos7上安装Python3.6
date: 2019-07-02 15:30:45
tags: 服务器
---

## 1.安装ius源
[ius](https://ius.io/)是一个专门提供经过验证RPM包的第三方社区，很多官方RPM库没有提供的新功能可以在该网站上找到。

    sudo yum install -y https://centos7.iuscommunity.org/ius-release.rpm
    
## 2.更新YUM

	sudo yum update


## 3.安装python3包

	sudo yum install -y python36u python36u-libs python36u-devel python36u-pip

然后就可以尽情使用python3了

    python3.6 -V

	# Python 3.6.8

    pip3 -V

	# pip 9.0.1 from /usr/lib/python3.6/site-packages (python 3.6)

添加软链接

    ln -s /usr/bin/python3.6 /usr/bin/python3
	ln -s /usr/bin/pip3.6 /usr/bin/pip3

现在可以直接使用python3和pip3命令了
    