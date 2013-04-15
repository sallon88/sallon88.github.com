---
layout: post
title: centos6.3下编译hhvm
category : tech
tags : [php]
---

#### 服务器配置
centos6.3 64位

#### 安装hhvm必须的依赖包
	sudo yum install git cpp make autoconf automake libtool patch memcached gcc-c++ cmake wget boost-devel mysql-devel pcre-devel gd-devel libxml2-devel expat-devel libicu-devel bzip2-devel oniguruma-devel openldap-devel readline-devel libc-client-devel libcap-devel binutils-devel pam-devel elfutils-libelf-devel

centos的yum源中并没有libmcrypt包，所以必须从第三方下载
	cd /usr/local/src/
	wget http://mirror.nus.edu.sg/Fedora/epel/6/x86_64/libmcrypt-2.5.8-9.el6.x86_64.rpm
	wget http://mirror.nus.edu.sg/Fedora/epel/6/x86_64/libmcrypt-devel-2.5.8-9.el6.x86_64.rpm
	rpm -i libmcrypt-*.rpm

#### 升级gcc到4.6
centos中的gcc版本为4.4，必须升级到4.6才能正常编译hhvm, 
运行以下脚本
{% highlight bash %}
#!/bin/sh
# install gcc4.6.1

cd /usr/local/src
# build & install gmp
wget http://ftp.gnu.org/gnu/gmp/gmp-4.3.2.tar.bz2
tar jxf gmp-4.3.2.tar.bz2 &&cd gmp-4.3.2/
./configure --prefix=/usr/local/gmp
make &&make install
cd ..

# build & install mpfr
wget http://ftp.gnu.org/gnu/mpfr/mpfr-2.4.2.tar.bz2
tar jxf mpfr-2.4.2.tar.bz2 ;cd mpfr-2.4.2/
./configure --prefix=/usr/local/mpfr -with-gmp=/usr/local/gmp
make &&make install
cd ..

#build & install mpc
wget http://ftp.gnu.org/gnu/mpc/mpc-1.0.1.tar.gz
tar xzf mpc-1.0.1.tar.gz ;cd mpc-1.0.1
./configure --prefix=/usr/local/mpc -with-mpfr=/usr/local/mpfr -with-gmp=/usr/local/gmp
make &&make install
cd ..

#build & install gcc4.6.1
wget http://ftp.gnu.org/gnu/gcc/gcc-4.6.1/gcc-4.6.1.tar.bz2
tar jxf gcc-4.6.1.tar.bz2 ;cd gcc-4.6.1
./configure --prefix=/usr/local/gcc -enable-threads=posix -disable-checking -disable-multilib -enable-languages=c,c++ -with-gmp=/usr/local/gmp -with-mpfr=/usr/local/mpfr/ -with-mpc=/usr/local/mpc/

if [ $? -eq 0 ];then
echo "gcc configure succeed"
else
echo "gcc configure failed"
exit 1
fi

export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/mpc/lib:/usr/local/gmp/lib:/usr/local/mpfr/lib/
make && make install

[ $? -eq 0 ] && echo install success
cd ..

#set gcc evn
echo '/usr/local/gcc/lib/' > /etc/ld.so.conf.d/gcc.4.6.1.conf
echo '/usr/local/mpc/lib/' >> /etc/ld.so.conf.d/gcc.4.6.1.conf
echo '/usr/local/gmp/lib/' >> /etc/ld.so.conf.d/gcc.4.6.1.conf
echo '/usr/local/mpfr/lib/' >> /etc/ld.so.conf.d/gcc.4.6.1.conf
ldconfig
mv /usr/bin/gcc  /usr/bin/gcc_old
mv /usr/bin/g++  /usr/bin/g++_old
mv /usr/bin/c++  /usr/bin/c++_old
ln -s -f /usr/local/gcc/bin/gcc  /usr/bin/gcc
ln -s -f /usr/local/gcc/bin/g++  /usr/bin/g++
ln -s -f /usr/local/gcc/bin/c++  /usr/bin/c++

cp /usr/local/gcc/lib64/libstdc++.so.6.0.16 /usr/lib64/.
mv /usr/lib64/libstdc++.so.6 /usr/lib64/libstdc++.so.6.bak
ln -s -f /usr/lib64/libstdc++.so.6.0.16 /usr/lib64/libstdc++.so.6
{% endhighlight %}

#### 升级boost库到1.50
首先翻墙到以下地址下载源码包:
	http://sourceforge.net/projects/boost/files/boost/1.50.0/boost_1_50_0.tar.gz/download
然后安装
{% highlight bash %}
cd /usr/local/src
tar -zxvf boost_1_50_0.tar.gz
cd boost_1_50_0/
./bootstrap.sh --prefix=/usr --libdir=/usr/lib
./bjam --layout=system install
export Boost_LIBRARYDIR=/usr/include/boost/
{% endhighlight %}

#### 获取hhvm源码，设置编译环境
{% highlight bash %}
#!/bin/sh
cd /usr/local/src
git clone git://github.com/facebook/hiphop-php.git
cd hiphop-php
export CMAKE_PREFIX_PATH=/usr/local
export HPHP_HOME=`/bin/pwd`
export HPHP_LIB=`/bin/pwd`/bin
export USE_HHVM=1
cd ..
{% endhighlight %}

#### 安装hhvm所需的第三方包

{% highlight bash %}
#!/bin/sh
# install hhvm_depends

# install libevent
git clone git://github.com/libevent/libevent.git
cd libevent
git checkout release-1.4.14b-stable
cat $HPHP_HOME/src/third_party/libevent-1.4.14.fb-changes.diff | patch -p1
./autogen.sh
./configure --prefix=$CMAKE_PREFIX_PATH
make
make install
cd ..

# install libCurl
git clone git://github.com/bagder/curl.git
cd curl
cat $HPHP_HOME/src/third_party/libcurl-7.22.1.fb-changes.diff | patch -p1
./buildconf
./configure --prefix=$CMAKE_PREFIX_PATH
make
make install
cd ..

# install libunwind
wget 'http://download.savannah.gnu.org/releases/libunwind/libunwind-1.0.tar.gz'
tar -zxf libunwind-1.0.tar.gz
cd libunwind-1.0
autoreconf -i -f
./configure --prefix=$CMAKE_PREFIX_PATH
make
make install
cd ..

# install libmemcached
wget 'http://launchpad.net/libmemcached/1.0/0.49/+download/libmemcached-0.49.tar.gz'
tar -xzvf libmemcached-0.49.tar.gz
cd libmemcached-0.49
./configure --prefix=$CMAKE_PREFIX_PATH
make
make install
cd ..

# install tbb
wget 'http://threadingbuildingblocks.org/sites/default/files/software_releases/source/tbb40_20120613oss_src.tgz'
tar -zxf tbb40_20120613oss_src.tgz
cd tbb40_20120613oss

make > make.log
awk 'END {print}' make.log |sed -e 's/`/ /' -e "s/'//" |awk {'print $4'} > tmpname
TBB_NAME=`cat tmpname`
rm -f tmpname

mkdir -p /usr/include/serial
cp -a include/serial/* /usr/include/serial/
mkdir -p /usr/include/tbb
cp -a include/tbb/* /usr/include/tbb/

cp $TBB_NAME/libtbb.so.2 /usr/lib64/
ln -s /usr/lib64/libtbb.so.2 /usr/lib64/libtbb.so
cd ..

# install libdwarf
git clone git://libdwarf.git.sourceforge.net/gitroot/libdwarf/libdwarf
cd libdwarf/libdwarf
./configure
make
cp libdwarf.a $CMAKE_PREFIX_PATH/lib64/
cp libdwarf.h $CMAKE_PREFIX_PATH/include/
cp dwarf.h $CMAKE_PREFIX_PATH/include/
cd ../..
{% endhighlight %}

#### 编译安装hhvm
{% highlight bash %}
cd /usr/local/src/hiphop-php
export HPHP_HOME=`/bin/pwd`
export HPHP_LIB=`/bin/pwd`/bin
cmake .
make
{% endhighlight %}
编译时间会非常非常长, 如果出现以下错误
'const struct HPHP::VM::Transl::RegRIP' has no user-provided default constructor这类错误，需要编辑文件 hiphop-php/src/CMake/HPHPSetup.cmake 118行左右，在`-std=gnu++0x` 后加入 ` -fpermissive`

编译好后的程序hhvm位于 $HPHP_HOME/src/hhvm/, 为方便使用可以建立一个软链接
	ln -s $HPHP_HOME/src/hhvm/hhvm /usr/bin/hhvm

#### 运行
hhvm的运行，需要两个全局变量, 可以在~/.bashrc中加入以下两条命令
	export HPHP_HOME=/usr/local/src/hiphop-php
	export HPHP_LIB=/usr/local/src/hiphop-php/bin

更新下设置后(. ~/.bashrc)后，运行php脚本
	hhvm hello.php
如果加参数-m server -p 端口，会启动一个内置的web server
	hhvm -m server 
此时可以通过浏览器http://host/hello.php访问

下面以通过hhvm运行codeigniter框架为例，演示一下更通用的用法

首先创建一个配置文件:
vi hhvm.hdf
{% highlight bash %}
Server {
	Port = 8000
	SourceRoot = /var/www/html/codeigniter/
}

Eval {
	Jit = true
}

Log {
	Level = Error
	UseLogFile = true
	File = /tmp/hhvm/error.log
	Access {
		* {
			File = /tmp/hhvm/access.log
			Format = %h %l %u %t \"%r\" %>s %b
		}
	}
}

VirtualHost {
	* {
		Pattern = .*
		RewriteRules {
			dirindex {
				pattern = ^(.*)/$
				to = $1/index.php
				qsa = true
			}
		}
	}
}

StaticFile {
	  FilesMatch {
		  	* {
			  	pattern = .*\.(dll|exe)
			  	headers {
				  * = Content-Disposition: attachment
				}
			}
	  }
	  Extensions {
		  css = text/css
		  gif = image/gif
		  html = text/html
		  jpe = image/jpeg
		  jpeg = image/jpeg
		  jpg = image/jpeg
		  png = image/png
		  tif = image/tiff
		  tiff = image/tiff
		  txt = text/plain
	  }
}
{% endhighlight %}

然后运行
	hhvm -m daemon -u sallon --config hhvm.hdf

-m daemon 表示是后台进程方式运行
-u sallon 表示以用户sallon的身份运行
--config hhvm.hdf 则指明了刚刚创建的配置文件

此时访问http://localhost:8000/index.php会显示内部错误，原因是
hhvm对一些php函数的解释和php本身的解析有细微的不同，就codeignter中发现的问题有两处:

* 如果is_dir()的参数为空，hhvm中会返回true, 而php官方会返回false;
* phpinfo($_ci_view, PATHINFO_EXTENSION), 如果$_ci_view为空，hhvm会返回NULL， 而php官方会返回空字符串

所以需要修改两处codeigniter源文件:

* 入口文件index.php, 大约252行，在if (($_temp = realpath($view_folder)) !== FALSE) 的上一行，加入 $view_folder = APPPATH.'views';
* 核心类文件/system/core/Loader.php, 823行左右，把$_ci_ext === '' 改为 $_ci_ext == ''

此时即可通过浏览正常访问

根据实际的压力测试结果, hhvm和官方的解释器(未开启apc)相比，反应速度提升约5倍速左右。内存占用会更少，貌似无论运行神马程序，通过memery_get_usage看到的内存占用都是0.03M.

参考:  
[hiphop wiki](https://github.com/facebook/hiphop-php/wiki)  
[Building and installing HHVM on CentOS 6.3](https://github.com/facebook/hiphop-php/wiki/Building-and-installing-HHVM-on-CentOS-6.3)  
[hiphop for php](http://www.hiphop-php.com/wp/?p=113)  
