# Build Facebook's folly & wangle

### Build folly
```sh
> git clone https://github.com/facebook/folly.git
> cd folly
> ./confiure --prefix=/home/matt/usrsdk --disable-shared
> make && make install
```

### Build wangle
#### Get the code:
```sh
> git clone https://github.com/facebook/wangle.git
> cd wangle
```
#### Modify cmake/FindFolly.cmake:
```sh
#  Copyright (c) 2014, Facebook, Inc.
#  All rights reserved.
#
#  This source code is licensed under the BSD-style license found in the
#  LICENSE file in the root directory of this source tree. An additional grant
#  of patent rights can be found in the PATENTS file in the same directory.
#
# - Try to find folly
# This will define
# FOLLY_FOUND
# FOLLY_INCLUDE_DIR
# FOLLY_LIBRARIES

CMAKE_MINIMUM_REQUIRED(VERSION 2.8.7 FATAL_ERROR)

INCLUDE(FindPackageHandleStandardArgs)

set(MY_FOLLY_LIBRARYDIR "/home/matt/usrsdk/lib")
set(MY_FOLLY_INCLUDEDIR "/home/matt/usrsdk/include")

FIND_LIBRARY(FOLLY_LIBRARY folly HINTS ${MY_FOLLY_LIBRARYDIR})
FIND_PATH(FOLLY_INCLUDE_DIR "folly/String.h" HINTS ${MY_FOLLY_INCLUDEDIR})

SET(FOLLY_LIBRARIES ${FOLLY_LIBRARY})

FIND_PACKAGE_HANDLE_STANDARD_ARGS(Folly
  REQUIRED_ARGS FOLLY_INCLUDE_DIR FOLLY_LIBRARIES)
```
#### Modify CMakeList.txt:
```sh
set(CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake/ “./”)
```
#### cmake && make:
```sh
> mkdir build && cd build
> cmake .. -DCMAKE_INSTALL_PREFIX=/home/matt/usrsdk -DCMAKE_BUILD_TYPE=Debug -DBUILD_TESTS=OFF
> make && make install
```
