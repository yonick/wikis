# FFMPEG  Building Tutorial

### libx264
#### getting code
git clone https://git.videolan.org/git/x264.git

#### compiling code
cd x264 && ./configure --prefix=$HOME/ffmpeg-build --enable-static --enable-shared && make && make install



### libx265
#### getting code
git clone https://github.com/videolan/x265.git
#### compiling code
cd x265/build/msys

vim make-Makefiles.sh // adding
cmake -D CMAKE_INSTALL_PREFIX="$HOME/ffmpeg-build"

./make-Makefiles.sh && make



### fdk-aac
#### getting code
git clone git://github.com/mstorsjo/fdk-aac.git
#### compiling code
cd fdk-aac && autoreconf -fiv

./configure --prefix=$HOME/ffmpeg-build && make && make install



### libmfx
#### getting code
git clone https://github.com/lu-zero/mfx_dispatch.git
#### compiling code
cd mfx_dispatch && autoreconf -i && ./configure --prefix=$HOME/ffmpeg-build && make && make install



### openh264
#### getting code
git clone -b openh264v1.5.1 https://github.com/cisco/openh264.git
#### compiling code
cd openh264/build

vim platform-mingw_nt.mk // x86_64-w64-mingw32-ar --> x86_64-w64-mingw32-gcc-ar

make && make install

move the header&libs to $HOME/ffmpeg-build, modify pkg-config



### libvpx
#### getting code
git clone https://github.com/webmproject/libvpx
#### compiling code
cd libvpx && ./configure --prefix=$HOME/ffmpeg-build && make && make install



### vid.stab
#### getting code
git clone https://github.com/georgmartius/vid.stab.git
#### compiling code
cd vid.stab && ./configure --prefix=$HOME/ffmpeg-build && make && make install



### opencl
#### getting sdk
http://registrationcenter-download.intel.com/akdlm/irc_nas/9302/intel_sdk_for_opencl_setup_6.1.0.1600.exe
#### reimp lib
http://wyw.dcweb.cn/download.asp?path=&file=reimp_new.zip

build reimp_new

reimp C:\Program Files (x86)\Intel\OpenCL SDK\6.1\lib\x64\OpenCL.lib

copy libopencl.lib and headers to $HOME/ffmpeg-build



### ffmpeg
#### getting code
git clone https://git.ffmpeg.org/ffmpeg.git
#### compiling code
cd ffmpeg

PKG_CONFIG_PATH=$HOME/ffmpeg-build/lib/pkgconfig ./configure --prefix=$HOME/ffmpeg-build --enable-small --disable-debug --disable-doc --arch=x86_64 --enable-cross-compile --target-os=mingw32 --enable-libvidstab --enable-libfdk-aac --enable-libx264 --enable-libx265 --enable-libmfx --enable-libopenh264 --enable-libvpx --enable-opencl --extra-cflags=-I$HOME/ffmpeg-build/include --extra-ldflags=-L$HOME/ffmpeg-build/lib --disable-static --enable-shared --enable-version3 --enable-gpl --enable-nonfree


