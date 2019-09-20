# 官方地址：
http://www.opencv.org.cn/opencvdoc/2.3.2/html/doc/tutorials/introduction/linux_install/linux_install.html#linux-installation

前提条件：依赖安装
``` shell
GCC 4.x or later # 要求Gcc版本 >4
CMake 2.6 or higher # 要求Cmake >2.6
Subversion (SVN) client # 要求Svn客户端
GTK+2.x or higher, including headers # 要求Gtk >2，并且包括头文件
pkgconfig # 要求pkgconfig
libpng, zlib, libjpeg, libtiff, libjasper with development files (e.g. libpjeg-dev)  # 要求基本库,png,zlib,jpeg等，并且均包括开发包
Python 2.3 or later with developer packages (e.g. python-dev)
# 要求python >2.3，并且包括python-dev
SWIG 1.3.30 or later (only for versions prior to OpenCV 2.3)
# 要求swig >1.3.30
libavcodec # 要求libavcodec
libdc1394 2.x #要求libdc1394 >2

编译：build
a,下载源码包，例如opencv3_4.zip
cd /usr/local/
unzip opencv3_4.zip .
cd opencv3_4
mkdir build 
# 创建一个临时目录，随便取名即可
cmake ..
# 使用cmake编译
sudo make -j8
# 一般服务器都有多个核，所以make可以多进程执行，后面的-j8表示8进程执行
# 请在make前面要加上sudo，否则会报错，，这是个坑，下面个也是要加上sudo
sudo make install
