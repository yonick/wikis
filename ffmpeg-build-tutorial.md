# FFMPEG Building Tutorial
---
### `libx264`
#### 1. Get the code
```sh
> git clone https://git.videolan.org/git/x264.git
```

#### 2. Compile
```sh
> cd x264
> ./configure --prefix=$HOME/ffmpeg-build --enable-static --enable-shared
> make && make install
```
### `libx265`
#### 1. Get the code
```sh
> git clone https://github.com/videolan/x265.git
```
#### 2. Compile
```sh
> cd x265/build/msys
```
#### 3. Modify make-Makefiles.sh
```sh
cmake -D CMAKE_INSTALL_PREFIX="$HOME/ffmpeg-build"
```
#### 4. Compile
```sh
> ./make-Makefiles.sh && make
```

### `fdk-aac`
#### 1. Get the code
```sh
> git clone git://github.com/mstorsjo/fdk-aac.git
```
#### 2. Compile
```sh
> cd fdk-aac && autoreconf -fiv
>./configure --prefix=$HOME/ffmpeg-build && make && make install
```
### `libmfx`
#### 1. Get the code
```sh
> git clone https://github.com/lu-zero/mfx_dispatch.git
```
##### 2. Compile
```sh
> cd mfx_dispatch && autoreconf -i
> ./configure --prefix=$HOME/ffmpeg-build
> make && make install
```
### `openh264`
#### 1. Get the code
```sh
> git clone -b openh264v1.5.1 https://github.com/cisco/openh264.git
```
#### 2. Modify openh264/build/platform-mingw_nt.mk
```sh
x86_64-w64-mingw32-ar --> x86_64-w64-mingw32-gcc-ar
```
#### 3. Modify openh264/Makefile
```sh
PREFIX=$HOME/ffmpeg-build
```
#### 4. Compile
```sh
> make && make install
```
### `libvpx`
#### 1. Get the code
```sh
> git clone https://github.com/webmproject/libvpx
```
#### 2. Compile
```sh
> cd libvpx
> ./configure --prefix=$HOME/ffmpeg-build --enable-shared
> make && make install
```
### `vid.stab`
#### 1. Get the code
```sh
> git clone https://github.com/georgmartius/vid.stab.git
```
#### 2. Compile
```sh
> cd vid.stab
> ./configure --prefix=$HOME/ffmpeg-build
> make && make install
```
### `opencl`
#### 1. Get the sdk
```sh
> wget http://registrationcenter-download.intel.com/akdlm/irc_nas/9302/intel_sdk_for_opencl_setup_6.1.0.1600.exe
```
#### 2. reimp lib
```sh
> wget http://wyw.dcweb.cn/download.asp?path=&file=reimp_new.zip
> cd reimp_new && make
> reimp C:\Program Files (x86)\Intel\OpenCL SDK\6.1\lib\x64\OpenCL.lib
```
#### 3. Copy libopencl.lib and headers to $HOME/ffmpeg-build

### `ffmpeg`
#### 1. Get the code
```sh
> git clone https://git.ffmpeg.org/ffmpeg.git
```
#### 2. Compile
```sh
> cd ffmpeg
> PKG_CONFIG_PATH=$HOME/ffmpeg-build/lib/pkgconfig ./configure --prefix=$HOME/ffmpeg-build --enable-small --disable-debug --disable-doc --arch=x86_64 --enable-cross-compile --target-os=mingw32 --enable-libvidstab --enable-libfdk-aac --enable-libx264 --enable-libx265 --enable-libmfx --enable-libopenh264 --enable-libvpx --enable-opencl --extra-cflags=-I$HOME/ffmpeg-build/include --extra-ldflags=-L$HOME/ffmpeg-build/lib --disable-static --enable-shared --enable-version3 --enable-gpl --enable-nonfree
```


