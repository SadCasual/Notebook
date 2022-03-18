# 我在云上的日子--Suricata安装部署（保姆级教学）

> 环境与版本不写清楚的安装都是耍流氓！！！

## 0x00 任务目标

在CentOS 7.6版本的阿里云服务器上部署Suricata 6.0.4版本，其中重要组件使用的版本如下：

* Suricata: 6.0.4
* Luajit: 2.0.5
* PF_RING: 8.1.0
* Hyperscan: 5.4.0
  * boost: 1_60_0
  * ragel: 6.10

跟着本文安装好之后的文件结构是这样子的~~

![image-20220318105728197](C:\Users\20924\AppData\Roaming\Typora\typora-user-images\image-20220318105728197.png)

大家一会也可以根据自身需求调整嗷

## 0x01 环境准备

我的大白兔和灰小兔早早已经遁入虚空了（太贵了实在是续不起费哭...）...被迫入了一台新的小世界--常青~

* 阿里云实例：常青

* 系统：CentOS 7.6 64位系统

* 配置：2C4G

吐槽一下，阿里云的服务器用的是真没腾讯云的舒服哎呀呀呀

---

这是我经过试验的所有需要安装的依赖项及版本，不是说别的版本不行，但这些版本是亲身测试过保证不踩雷的嗷！

| 组件      | 依赖包                   | 命令                                    | 依赖包版本       |
| --------- | ------------------------ | --------------------------------------- | ---------------- |
| Suricata  | deltarpm                 | yum install deltarpm -y                 | 3.6-3.el7        |
| Suricata  | gcc                      | yum install gcc -y                      | 4.8.5-44.el7     |
| Suricata  | libpcap-devel            | yum install libpcap-devel -y            | 14:1.5.3-12.el7  |
| Suricata  | pcre-devel               | yum install pcre-devel -y               | 8.32-17.el7      |
| Suricata  | libyaml-devel            | yum install libyaml-devel -y            | 0.1.4-11.el7_0   |
| Suricata  | file-devel               | yum install file-devel -y               | 5.11-37.el7      |
| Suricata  | zlib-devel               | yum install zlib-devel -y               | 1.2.7-19.el7_9   |
| Suricata  | jansson-devel            | yum install jansson-devel -y            | 2.10-1.el7       |
| Suricata  | nss-devel                | yum install nss-devel -y                | 3.67.0-4.el7_9   |
| Suricata  | libcap-ng-devel          | yum install libcap-ng-devel -y          | 0.7.5-4.el7      |
| Suricata  | libnet-devel             | yum install libnet-devel -y             | 1.1.6-7.el7      |
| Suricata  | tar                      | yum install tar -y                      | 1.26-35.el7      |
| Suricata  | make                     | yum install make -y                     | 1:3.82-24.el7    |
| Suricata  | libnetfilter_queue-devel | yum install libnetfilter_queue-devel -y | 1.0.2-2.el7_2    |
| Suricata  | lua-devel                | yum install lua-devel -y                | 5.1.4-15.el7     |
| Suricata  | PyYAML                   | yum install PyYAML -y                   | 3.10-11.el7      |
| Suricata  | libmaxminddb-devel       | yum install libmaxminddb-devel -y       | 1.2.0-6.el7      |
| Suricata  | rustc                    | yum install rustc -y                    | 1.59.0-2.el7     |
| Suricata  | cargo                    | yum install cargo -y                    | 1.59.0-2.el7     |
| Suricata  | lz4-devel                | yum install lz4-devel -y                | 1.8.3-1.el7      |
| PF_RING   | numactl-devel            | yum install numactl-devel -y            | 2.0.12-5.el7     |
| PF_RING   | git                      | yum install git -y                      | 1.8.3.1-23.el7_8 |
| PF_RING   | bison-devel              | yum install bison-devel -y              | 3.0.4-2.el7      |
| PF_RING   | bison                    | yum install bison -y                    | 3.0.4-2.el7      |
| PF_RING   | flex                     | yum install flex -y                     | 2.5.37-6.el7     |
| Hyperscan | cmake                    | yum install cmake -y                    | 2.8.12.2-2.el7   |
| Hyperscan | ragel                    | yum install ragel -y                    | 7.0.0.9-2.el7    |
| Hyperscan | libtool                  | yum install libtool -y                  | 2.4.2-22.el7_3   |
| Hyperscan | python-devel             | yum install python-devel -y             | 2.7.5-90.el7     |
| Hyperscan | boost                    | yum install boost -y                    | 1.53.0-28.el7    |
| Hyperscan | boost-devel              | yum install boost-devel -y              | 1.53.0-28.el7    |
| Hyperscan | boost-doc                | yum install boost-doc -y                | 1.53.0-28.el7    |
| Hyperscan | libquadmath              | yum install libquadmath -y              | 4.8.5-44.el7     |
| Hyperscan | libquadmath-devel        | yum install libquadmath-devel -y        | 4.8.5-44.el7     |
| Hyperscan | bzip2-devel              | yum install bzip2-devel -y              | 1.0.6-13.el7     |
| Hyperscan | gcc-c++.x86_64           | yum install "gcc-c++.x86_64" -y         | 4.8.5-44.el7     |
| Hyperscan | autoconf                 | yum install autoconf -y                 | 2.69-11.el7      |
| Hyperscan | libicu                   | yum install libicu -y                   | 50.2-4.el7_7     |
| Hyperscan | libicu-devel             | yum install libicu-devel -y             | 50.2-4.el7_7     |

## 0x02 Luajit

> 官网地址：www.luajit.org

Luajit是Lua的一个Just-In-Time也就是运行时**编译器**，类似于GCC之于C/C++

Suricata需要Luajit去加载和运行Lua脚本~~

这里选用的版本还是目前最新的稳定版，也就是2017-05-01发行的**LuaJIT-2.0.5**

### 1. 编译安装

```shell
cd /opt
wget http://luajit.org/download/LuaJIT-2.0.5.tar.gz
tar -zxf LuaJIT-2.0.5.tar.gz
mv LuaJIT-2.0.5/ luajit/
cd luajit/
sudo make && make install
```

编译安装完成后需要更新一下动态库：

```shell
vim /etc/ld.so.conf
```

另起一行添加下面的路径：

```shell
/usr/local/lib
```

保存退出，然后运行命令加载：

```shell
sudo /sbin/ldonfig
```

### 2.验证安装成功

输入命令`ldconfig -p | grep lua`，显示如下即为安装成功：

![image-20220318111454299](C:\Users\20924\AppData\Roaming\Typora\typora-user-images\image-20220318111454299.png)

## 0x03 PF_RING

PF_RING是Luca研究出来的基于Linux内核级的高效数据包捕获技术。简单来说PF_RING 是一个高速数据包捕获库,通过它可以实现将通用 PC 计算机变成一个有效且便宜的网络测量工具箱,进行数据包和现网流量的分析和操作。同时支持调用用户级别的API来创建更有效的应用程序。

PF_RING是拥有一套完整开发接口的高速数据包捕捉库，与我们熟知的libpcap十分相似，但其性能要优于libpcap

> Linux内核协议栈在数据包的收发过程中，内存拷贝操作的时间开销占了整个处理过程时间开销的65%，此外层间传递的系统调用时间也占据了8%～10%，PF_RING就是通过允许用户空间内存直接和rx_buffer做mmap来减少内存拷贝次数，提高性能~，详细说明参考：http://www.javashuo.com/article/p-nanvzxlf-nx.html

这里我用的是VPS，虚拟网卡，如果是使用物理网卡的话，需要先卸载网卡驱动（感谢我周哥诶嘿！）

！！！卸载网卡驱动可能会造成网络无法连接，所以需要现场操作，不能SSH远程

！！！虚拟网卡请跳过这步！！！

```shell
ethtool -i eth0 #这个地方写自己的网卡名嗷
lsmod | grep igb
rmmod igb
```

### 1.编译安装

```shell
cd /opt
git clone https://github.com/ntop/PF_RING/
mv PF_RING/ pf_ring/
make #直接在跟目录下面make,进行全部编译
cd /opt/pf_ring/kernel
make
sudo make install

#最好设置一下，官方解释是2的性能最好，但是有大神测试后发现差别并不是很大
insmod pf_ring.ko transparent_mode=2

#编译安装PF_RING所需依赖库
cd /opt/pf_ring/userland/lib
./configure
make
sudo make install
```

如果需要使用libpcap抓包分析，请卸载之前安装的libpcap，然后进入userland/目录下配置、编译和安装驱动

```shell
#卸载原来的libpqcap
rpm -qa libpcap #查看安装的libpcap，如果有libpcap则强制卸载
rpm --nodeps -e libpcap

#安装PF_RING的libpcap
cd PF_RING-dev/userland/libpcap
./configure
make
sudo make install

#编译网卡驱动
cd PF_RING/drivers/intel/igb/igb-5.3.5.18-zc/src
sudo insmod igb.ko #安装pf_ring网卡驱动
```

---

如果这个地方有这样子的报错的话（没报错请跳过这一趴）

```shell
$ sudo insmod igb.ko
insmod: ERROR: could not insert module igb.ko: Unknown symbol in module
```

![image-20220318143428377](C:\Users\20924\AppData\Roaming\Typora\typora-user-images\image-20220318143428377.png)

可以看下依赖包有没有正确载入

```shell
modinfo igb.ko | grep depends
```

未正确载入的重新载入即可

```
modprobe ptp
modprobe i2c-algo-bit
```

---

最后载入模块：

```shell
sudo modprobe igb
```

### 2.验证安装成功

测试下网卡eth0的接受包数：

```shell
cd /opt/pf_ring//userland/example
make
./pfcount -i eth0 #测试捕获eth0的数据报文
```

![image-20220318144027840](C:\Users\20924\AppData\Roaming\Typora\typora-user-images\image-20220318144027840.png)

显示这样即安装成功！！

## 0x04 Hyperscan

> 官网地址：https://www.hyperscan.io/

Hyperscan是一个高性能的多重正则表达式匹配库。在Suricata中它可以用来执行多模式匹配，它是基于X86平台以PCRE为原型而开发的，在支持PCRE的大部分语法的前提下，Hyperscan增加了特定的语法和工作模式来保证其在真实网络场景下的实用性。

这里有两个依赖boost和ragle都需要编译安装嗷~

### 1.boost依赖编译安装

```shell
cd /opt

# 下载
wget http://downloads.sourceforge.net/project/boost/boost/1.60.0/boost_1_60_0.tar.gz

#安装
tar xvzf boost_1_60_0.tar.gz
mv boost_1_60_0/ boost/
cd boost/
./bootstrap.sh
sudo ./b2 --with-iostreams --with-random install
ldconfig
```

---

如果在执行`./bootstrap.sh`报错的话，请这样调整：

![image-20220318151934736](C:\Users\20924\AppData\Roaming\Typora\typora-user-images\image-20220318151934736.png)

```shell
提示信息【[Unicode]/ICU support for Boost.Regex?... not found.】，安装依赖包【 libicu libicu-devel 】
提示信息【error: no command provided, default command 'g++' not found】，安装依赖包【 gcc-c++ 】
提示信息【- zlib: no】，安装依赖包【 zlib zlib-devel 】
提示信息【- bzip2: no】，安装依赖包【 bzip2 bzip2-devel 】
```

然后一定要重新解压！重新编译！！

---

### 2.ragle依赖编译安装

```shell
# 下载
wget http://www.colm.net/files/ragel/ragel-6.10.tar.gz

# 安装
tar zxvf ragel-6.10.tar.gz
mv ragel-6.10/ ragel/
cd ragel/
./configure
make
sudo make install
ldconfig
```

### 3.Hyperscan编译安装

```shell
cd /opt
git clone https://github.com/intel/hyperscan/
cd /hyperscan/
mkdir build
cd build
cmake -DBUILD_SHARED_LIBS=on -DCMAKE_BUILD_TYPE=Release .. -DBOOST_ROOT=/opt/boost
make -j8
make install
ldconfig
```

再次更新动态库！！

```shell
vim /etc/ld.so.conf
```

另起一行添加下面的路径：

```shell
/usr/local/lib64
```

保存退出，然后运行命令加载：

```shell
sudo ldonfig
```

### 4.验证安装成功

输入命令`ldconfig -p | grep hs`，显示如下即为安装成功：

![image-20220318153056003](C:\Users\20924\AppData\Roaming\Typora\typora-user-images\image-20220318153056003.png)

## 0x05 Suricata

### 1.编译安装

```shell
cd /opt
# 下载
wget https://www.openinfosecfoundation.org/download/suricata-6.0.4.tar.gz
tar zxvf suricata-6.0.4.tar.gz
cd suricata-6.0.4/

# 编译
./configure --prefix=/usr --sysconfdir=/etc --localstatedir=/var --enable-pfring --with-libpfring-includes=/usr/local/include --with-libpfring-libraries=/usr/local/lib --enable-geoip  --enable-luajit --with-libluajit-includes=/usr/local/include/luajit-2.0/ --with-libluajit-libraries=/usr/local/lib/ --with-libhs-includes=/usr/local/include/hs/ --with-libhs-libraries=/usr/local/lib/

# 安装
make
sudo make install
sudo make install-conf
```

### 2.验证之前的三个组件安装成功

Luajit: `suricata --build-info | grep lua`

![image-20220318153816530](C:\Users\20924\AppData\Roaming\Typora\typora-user-images\image-20220318153816530.png)

PF_RING: `suricata --build-info | grep PF_RING`

![image-20220318153813854](C:\Users\20924\AppData\Roaming\Typora\typora-user-images\image-20220318153813854.png)

Hyperscan: `suricata --build-info | grep Hyperscan`

![image-20220318153810861](C:\Users\20924\AppData\Roaming\Typora\typora-user-images\image-20220318153810861.png)

## 0x05 规则配置

安装suricata-update插件

```shell
pip install --upgrade suricata-update
```

这里安装的版本是suricata-update-1.2.2

在crontab配置定时更新规则集：

```shell
10 0 * * * /usr/bin/suricata-update --no-test && /usr/bin/suricatasc -c ruleset-reload
```

## 0x06 启动！！

```shell
suricata --pfring-int=eth0 --pfring-cluster-id=99 --pfring-cluster-type=cluster_flow -c /etc/suricata/suricata.yaml
```

