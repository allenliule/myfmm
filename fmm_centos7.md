### 说明

    fmm是一个优秀的地图匹配工具，详情参见https://github.com/cyang-kth/fmm.git
    
    源码提供了Ubuntu、iOS、Windows的安装方法，其中Ubuntu作者上传了镜像，部署最方便。
    Centos7下，默认的软件包的版本均比较旧，自己尝试整理一个编译安装的方法，供参考。

### 1. 新建镜像
```shell script
systemctl daemon-reload && systemctl restart docker
docker pull centos:7
docker run -it centos:7 /bin/bash
```

### 2. 基础依赖
```shell script
yum -y install sudo wget lrzsz

# install aliyun
cd /etc/yum.repos.d/
sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-*
sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*
wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
yum clean all
yum makecache

sudo yum -y group install "Development Tools"
sudo yum -y install make cmake swig 
sudo yum -y install boost boost-devel
sudo yum -y install epel-release
sudo yum -y install geos-devel gdal gdal-devel proj-devel bzip2-devel expat-devel python-devel -y

sudo yum -y install centos-release-scl
sudo yum -y install devtoolset-11-gcc*
mv /usr/bin/gcc /usr/bin/gcc-4.8.5
ln -s /opt/rh/devtoolset-11/root/bin/gcc /usr/bin/gcc
mv /usr/bin/g++ /usr/bin/g++-4.8.5
ln -s /opt/rh/devtoolset-11/root/bin/g++ /usr/bin/g++
mv /usr/bin/c++ /usr/bin/c++-4.8.5
ln -s /opt/rh/devtoolset-11/root/bin/c++ /usr/bin/c++
```

### 3. 依赖包的编译安装，注意有的需要手动修改配置
```shell script
# cmake 3.22.1
sudo yum -y install openssl openssl-devel

wget https://cmake.org/files/v3.22/cmake-3.22.1.tar.gz
tar zxvf cmake-3.22.1.tar.gz
cd cmake-3.22.1
./configure --prefix=/usr/local/cmake
make -j4 && make install
mv /usr/bin/cmake /usr/bin/cmake.del
ln -s /usr/local/cmake/bin/cmake /usr/bin/cmake
sudo ldconfig

# swig 4.0.2
sudo yum -y install pcre-devel

wget http://prdownloads.sourceforge.net/swig/swig-4.0.2.tar.gz --no-check-certificate
tar xzf swig-4.0.2.tar.gz
cd swig-4.0.2
./configure --without-pcre --prefix=/usr/local/swig
make -j4
make install
mv /usr/bin/swig /usr/bin/swig2
ln -s /usr/local/bin/swig /usr/bin/swig

# boost 1.74
sudo yum -y libicu libicu-devel zlib zlib-devel bzip2 bzip2-devel

wget http://ufpr.dl.sourceforge.net/project/boost/boost/1.74.0/boost_1_74_0.tar.bz2
tar --bzip2 -xf boost_1_74_0.tar.bz2
cd boost_1_74_0
./bootstrap.sh --prefix=/usr/local
sudo ./b2 install

sudo mkdir /usr/local/boost
mkdir /usr/local/boost/include
mkdir /usr/local/boost/lib64
cp -rf boost /usr/local/boost/include
cp -rf stage/lib/* /usr/local/boost/lib64

# 手动修改
# vim /etc/ld.so.conf
# /usr/local/boost/lib64/
sudo ldconfig


# gdal 3.4.3
yum -y install sqlite-devel

wget https://download.osgeo.org/proj/proj-6.2.1.tar.gz  --no-check-certificate
tar xzf proj-6.2.1.tar.gz
cd proj-6.2.1
./configure --prefix=/usr/local
make -j4
make install 
mv /usr/bin/proj /usr/bin/proj4
ln -s /usr/local/bin/proj /usr/bin/proj

wget http://s3.amazonaws.com/etc-data.koordinates.com/gdal-travisci/install-libkml-r864-64bit.tar.gz
tar xzf install-libkml-r864-64bit.tar.gz 
sudo cp -r install-libkml/include/* /usr/local/include
sudo cp -r install-libkml/lib/* /usr/local/lib

wget http://download.osgeo.org/gdal/3.4.3/gdal-3.4.3.tar.gz
tar xzf gdal-3.4.3.tar.gz
cd gdal-3.4.3
./configure --with-proj=/usr/local --with-libkml 
make -j4
make install
sudo ldconfig
# gdalinfo --version


# python3
wget https://registry.npmmirror.com/-/binary/python/3.10.12/Python-3.10.12.tgz
tar -zxvf Python-3.10.12.tgz  
cd Python-3.10.12
./configure prefix=/usr/local/python3
make -j4 && make install
mv /usr/bin/python /usr/bin/python2.old
ln -s /usr/local/python3/bin/python3.10 /usr/bin/python

mv /usr/bin/pip /usr/bin/pip2
ln -s /usr/local/python3/bin/pip3.10 /usr/bin/pip

export CPLUS_INCLUDE_PATH=/usr/local/python3/include/python3.10:$CPLUS_INCLUDE_PATH
export LIBRARY_PATH=/usr/local/python3/lib/python3.10:$LIBRARY_PATH

# 手动修改
# vim /usr/bin/yum
# vi /usr/libexec/urlgrabber-ext-down
# 第一行修改为：#!/usr/bin/python2

sudo yum -y install python3-devel

```

### 4. fmm下载、编译、测试
```shell script
mkdir fmm
cd fmm
git clone git@github.com:cyang-kth/fmm.git .

mkdir build
cd build
cmake ..
make -j4
sudo make install

cd ../example/python/
python fmm_test.py
```
