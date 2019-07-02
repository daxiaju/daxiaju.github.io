---
title: install_twisted_on_windows
date: 2019-07-01 12:13:28
tags:
---

Twisted库是比较流行的一个异步操作库，实现了大量的网络协议，scrapy就是基于Twisted实现异步操作和网络通讯。文末有彩蛋。

## 方法1.源代码安装

Twisted默认源代码安装需要在本地编译c代码，需要安装VC

	pip install twisted

## 方法2.第三方预编译包
https://www.lfd.uci.edu网站上提供了大非官方提供的windows平台二进制python包，很多无法直接安装的库可以在这里找到平台专用包。

打开该页面，Ctrl+F打开搜索框，输入要查找的包名称即可快速找到安装包。

twisted的版本如下。

https://www.lfd.uci.edu/~gohlke/pythonlibs/#twisted

截至发稿最新版本Twisted19.2.1的安装包连接如下

### python27 Win64

    pip install https://download.lfd.uci.edu/pythonlibs/t4jqbe6o/Twisted-19.2.1-cp27-cp27m-win_amd64.whl

### python27 Win32 

    
	pip install https://download.lfd.uci.edu/pythonlibs/t4jqbe6o/Twisted-19.2.1-cp27-cp27m-win32.whl

### python37 Win64
  
    pip install https://download.lfd.uci.edu/pythonlibs/t4jqbe6o/Twisted-19.2.1-cp37-cp37m-win_amd64.whl

### python37 Win32

    pip install https://download.lfd.uci.edu/pythonlibs/t4jqbe6o/Twisted-19.2.1-cp37-cp37m-win32.whl

其他版本可以自行查找下载。

## 方法3(推荐）

官方最新提供了支持pip安装方式，如下

    pip install Twisted[windows_platform]

一键安装，从此无烦恼。


## 小彩蛋
通过twisted可以快速建立各种简易的网络服务器，比如启动一个本地测试用的FTP服务器代码如下：

ftpserver.py:

    # Copyright (c) Twisted Matrix Laboratories.
	# See LICENSE for details.
	
	"""
	An example FTP server with minimal user authentication.
	"""
	
	from twisted.protocols.ftp import FTPFactory, FTPRealm, BaseFTPRealm
	from twisted.cred.portal import Portal
	from twisted.cred.checkers import AllowAnonymousAccess, FilePasswordDB
	from twisted.internet import reactor
	from twisted.python import log, failure, filepath
	
	"""
	An example FTP server with minimal user authentication.
	"""
	
	#
	# First, set up a portal (twisted.cred.portal.Portal). This will be used
	# to authenticate user logins, including anonymous logins.
	#
	# Part of this will be to establish the "realm" of the server - the most
	# important task in this case is to establish where anonymous users will
	# have default access to. In a real world scenario this would typically
	# point to something like '/pub' but for this example it is pointed at the
	# current working directory.
	#
	# The other important part of the portal setup is to point it to a list of
	# credential checkers. In this case, the first of these is used to grant
	# access to anonymous users and is relatively simple; the second is a very
	# primitive password checker.  This example uses a plain text password file
	# that has one username:password pair per line. This checker *does* provide
	# a hashing interface, and one would normally want to use it instead of
	# plain text storage for anything remotely resembling a 'live' network. In
	# this case, the file "pass.dat" is used, and stored in the same directory
	# as the server. BAD.
	#
	# Create a pass.dat file which looks like this:
	#
	# =====================
	#   jeff:bozo
	#   grimmtooth:bozo2
	# =====================
	class LocalFTPRealm(BaseFTPRealm):
	    def __init__(self):
	        self.userHome = filepath.FilePath('./HOME')
	           
	    def getHomeDirectory(self, avatarId):
	        return self.userHome
	p = Portal(LocalFTPRealm(),
	           [AllowAnonymousAccess(), FilePasswordDB('./auth', cache=True)])
	#
	# Once the portal is set up, start up the FTPFactory and pass the portal to
	# it on startup. FTPFactory will start up a twisted.protocols.ftp.FTP()
	# handler for each incoming OPEN request. Business as usual in Twisted land.
	#
	f = FTPFactory(p)
	
	#
	# You know this part. Point the reactor to port 21 coupled with the above factory,
	# and start the event loop.
	#
	reactor.listenTCP(21, f)
	reactor.run()

ftp同一文件夹下创建一个auth文件，每行一个用户，用户名和密码用冒号:分割。

auth:

    abc:123

FTP根目录对应当前文件夹下的HOME文件夹，Enjoy.