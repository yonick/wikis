# Debug FFMPEG with MSVC

## 1. prepare env
### 1.1 download msys2 and get it ready
pacman -Syu  
pacman -S make  
pacman -S gcc  
pacman -S diffutils  
pacman -S coreutils  
pacman -S pkg-config  
pacman -S tar  
pacman -S yasm  
### 1.2 install vs2015 community
### 1.3 prepare building env
Ensure which "cl" && "link" exists and points to MSVC:  
mv \usr\bin\link.exe \usr\bin\link.exe.bak  
start VS2015 x264 Native Tools Command Prompt go to your msys2 directory and type:  
msys2_shell.cmd -mingw64 -full-path  
## 2. compile static libs
### 2.1 libx264
git clone https://git.videolan.org/git/x264.git  
cd x264 && CC=cl ./configure  --enable-static  --prefix=$HOME/ffmpeg-build && make && make install  
### 2.2 ffmpeg
git clone https://git.ffmpeg.org/ffmpeg.git  
cd ffmpeg  
PKG_CONFIG_PATH=$HOME/ffmpeg-build/lib/pkg-config ./configure --prefix=/home/liuml/ffmpeg-build --enable-libx264 --extra-cflags=-I$HOME/ffmpeg-build/include --extra-ldflags=-$HOME/ffmpeg-build/lib --enable-static --toolchain=msvc --enable-debug --enable-gpl  
## 3. setup ffmpeg project
git clone https://github.com/yonick/ffmpeg-msvc-debug  
* libavcodec.a
libavformat.a
libavutil.a
libavfilter.a
libavdevice.a
libpostproc.a
libswresample.a
libswscale.a
libx264.lib
WS2_32.lib
strmiids.lib
Vfw32.lib
Shlwapi.lib
Secur32.lib  
* #define HAVE_STRUCT_POLLFD 1  // config.h
* disable PRINT_LIB_INFO(avresample, AVRESAMPLE, flags, level); // cmdutils.c

