# musl-deepflow

这个项目用于构建musl-deeplow依赖库的构建，我们把一些依赖库文件单独从开源源码中提取出来进行musl版本的编译。

## 准备

- deepflow编译镜像
  ```
  docker pull hub.deepflow.yunshan.net/public/rust-build:latest-arm64 (arm)
  docker pull hub.deepflow.yunshan.net/public/rust-build:latest (x86)
  ```
  进入docker编译环境：

  ```
  cd deepflow/agent/src
  sudo docker run --rm  -it  --net=host --privileged --workdir /work  -v $(pwd):/work hub.deepflow.yunshan.net/public/rust-build:latest bash
  ```
- 下载已经制作好的musl交叉编译工具链（C，C++编译，存放到编译容器里面解压以供编译使用）
  - wget http://more.musl.cc/x86_64-linux-musl/x86_64-linux-musl-cross.tgz (x86_64 use gcc 11.2.1)
  - wget http://more.musl.cc/10.2.1/x86_64-linux-musl/aarch64-linux-musl-cross.tgz (aarch64 use gcc 10.2.1 这里要和arm编译镜像环境对应)
  - 这部分参考
    - https://www.openwall.com/lists/musl/2019/01/21/5
    - http://musl.cc/
    - http://more.musl.cc/x86_64-linux-musl/

- 升级deepflow编译镜像GCC版本到11.2.1
  - 为什么要升级？
    - 因为x86_64-linux-musl版本是gcc 11.2.1，我们需要做到统一，否则会出莫名错误。

    ```
    yum install centos-release-scl
    yum install devtoolset-11-gcc*
    source /opt/rh/devtoolset-11/enable
    ```
- 注意三个路径（后面编译Makefile文件里面做相应的修改）
  - BCC源文件路径（bcc-src-with-submodule.tar.gz解压后的路径）
  - x86_64-linux-musl-cross（交叉编译路径）
  - 编译镜像里面原来已经有的musl（这个不存在C++编译工具，我们编译好的静态库最终存放在这里）

## bcc编译

这部分编译包含`libbcc.a`和`libbcc-bpf.a`这两个静态库的编译。

### 下载

版本：0.25.0

下载地址：https://github.com/iovisor/bcc/releases/tag/v0.25.0 （bcc-src-with-submodule.tar.gz）

bcc-src-with-submodule.tar.gz 放到编译docker里面编译会依赖它。

### 编译

注意：arm静态库的编译是在x86_64环境中使用aarch64交叉编译工具来编译的。


```bash
cd libbcc
#Makefile 相关路径做调整, 注意arm编译时要指定aarch64的交叉编译路径
make

cd libbcc-bpf
#Makefile 相关路径做调整，注意arm编译时要指定aarch64的交叉编译路径
make
```

## libelf编译


```bash
cd libelf
#Makefile 相关路径做调整，注意arm编译时要指定aarch64的交叉编译路径
make
```

## zlib编译


```bash
cd zlib
#Makefile 相关路径做调整，注意arm编译时要指定aarch64的交叉编译路径
make
```


## 更新编译镜像测试

```bash
#! /bin/bash

CC=musl-gcc CLANG=musl-clang make clean
source /opt/rh/devtoolset-11/enable

# 编译工具链的基本库文件更新到x86_64-linux-musl
cp /work/x86_64-linux-musl-cross/x86_64-linux-musl/lib/libstdc++.a /usr/x86_64-linux-musl/lib64/
cp /work/x86_64-linux-musl-cross/x86_64-linux-musl/lib/libc.a      /usr/x86_64-linux-musl/lib64/

# 依赖的Musl版本库文件Musl
cp /work/zlib/libz.a             /usr/x86_64-linux-musl/lib64/
cp /work/libelf-musl/libelf.a    /usr/x86_64-linux-musl/lib64/
cp /work/libbcc_src/libbcc.a     /usr/x86_64-linux-musl/lib64/
cp /work/libbcc-bpf/libbcc_bpf.a /usr/x86_64-linux-musl/lib64/

export CARGO_CFG_TARGET_ENV=musl
CC=musl-gcc CLANG=musl-clang make
CC=musl-gcc CLANG=musl-clang make profiler 
```
