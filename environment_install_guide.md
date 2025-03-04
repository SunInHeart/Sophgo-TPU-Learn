# TPU开发环境安装

在算能 SC7 HP75 上运行 sophon-demo 与 LLM-TPU 中模型环境配置简要说明，参考 [sophon-demo环境安装指南](https://github.com/sophgo/sophon-demo/blob/release/docs/Environment_Install_Guide.md#6-riscv-pcie%E5%B9%B3%E5%8F%B0%E7%9A%84%E5%BC%80%E5%8F%91%E5%92%8C%E8%BF%90%E8%A1%8C%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA)。

**环境说明**：

* 操作系统：openEuler riscv 2309
* CPU：SG2042
* TPU：SC7 HP75
* Chip：bm1684X
* SDK版本：24.04.01

## 说明

Sophon Demo 主要依赖的环境

* x86 TPU-MLIR docker环境：用于编译和量化模型
* 开发环境：用于编译 C++ 程序
* 运行环境：用于部署程序

## TPU-MLIR环境搭建

1. 安装 docker，已安装则跳过

```sh
# 如果您的docker环境损坏，可以先卸载docker
sudo apt-get remove docker docker.io containerd runc

# 安装依赖
sudo apt-get update
sudo apt-get install \
        ca-certificates \
        curl \
        gnupg \
        lsb-release

# 获取密钥
sudo mkdir -p /etc/apt/keyrings
curl -fsSL \
    https://download.docker.com/linux/ubuntu/gpg | \
    gpg --dearmor -o docker.gpg && \
    sudo mv -f docker.gpg /etc/apt/keyrings/

# 添加 docker 软件包
echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
    https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 安装 docker
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin

# docker命令免root权限执行
# 创建docker用户组，若已有docker组会报错，没关系可忽略
sudo groupadd docker
# 将当前用户加入docker组
sudo usermod -aG docker $USER
# 切换当前会话到新group或重新登录重启X会话
newgrp docker​ 
```

>提示：需要logout系统然后重新登录，再使用docker就不需要sudo了。

2. 拉取镜像创建并进入容器

直接拉取镜像

```sh
docker pull sophgo/tpuc_dev:latest
```

或者下载所需镜像文件，然后加载

```sh
$ wget https://sophon-file.sophon.cn/sophon-prod-s3/drive/24/06/14/12/sophgo-tpuc_dev-v3.2_191a433358ad.tar.gz
$ docker load -i sophgo-tpuc_dev-v3.2_191a433358ad.tar.gz
```

创建并启动容器

```sh
# 这里将本级目录映射到docker内的/workspace目录,用户需要根据实际情况将demo的目录映射到docker里面
# myname只是举个名字的例子, 请指定成自己想要的容器的名字
docker run --privileged --name myname --network host -v $PWD:/workspace -it sophgo/tpuc_dev:latest
# 此时已经进入docker，并在/workspace目录下
```

3. 安装 TPU-MLIR

目前支持五种配置: onnx, torch, tensorflow, caffe, paddle。可使用一条命令安装多个配置，也可直接安装全部依赖环境:

```sh
pip install tpu_mlir[onnx,torch,caffe]
pip install tpu_mlir[all]
```

建议TPU-MLIR的镜像仅用于编译和量化模型，程序编译和运行请在开发和运行环境中进行。更多TPU-MLIR的教程请参考算能官网的[《TPU-MLIR快速入门手册》](https://doc.sophgo.com/sdk-docs/v24.04.01/docs_latest_release/docs/tpu-mlir/quick_start/html/index.html)和[《TPU-MLIR开发参考手册》](https://doc.sophgo.com/sdk-docs/v24.04.01/docs_latest_release/docs/tpu-mlir/developer_manual/html/index.html)。

## riscv PCIe平台的开发和运行环境搭建

如果您在riscv平台安装了PCIe加速卡，开发环境与运行环境可以是统一的，您可以直接在宿主机上搭建开发和运行环境。 这里提供SG2042服务器的环境安装方法，其他类型机器具体请参考官网开发手册。

### 安装libsophon

从算能官网上下载符合环境依赖的SOPHONSDK，解压后在libsophon_{date}_{time}文件夹下面有安装包。

如果采用的系统为openEuler riscv 2309，使用以下方法 [安装驱动](./setup_tpu.md#安装驱动)。参考[《LIBSOPHON使用手册.pdf》](https://doc.sophgo.com/sdk-docs/v24.04.01/docs_latest_release/docs/libsophon/guide/html/1_install.html#linux)。

如果是openEuler riscv 2403，需要进行一些处理，参考[《oe2403_libsophon_issue》](./oe2403_libsophon_issue.md)解决。

### 安装sophon-ffmpeg和sophon-opencv

从算能官网上下载符合环境依赖的SOPHONSDK，解压后在sophon-mw_{date}_{time}文件夹下面有安装包，

安装需要 sophon-mw_0.10.0_riscv_64.tar.gz 文件

```sh
tar -xzvf sophon-mw_0.10.0_riscv_64.tar.gz
sudo cp -r sophon-mw_0.10.0_riscv_64/* /
sudo ln -s /opt/sophon/sophon-ffmpeg_0.10.0 /opt/sophon/sophon-ffmpeg-latest
sudo ln -s /opt/sophon/sophon-opencv_0.10.0 /opt/sophon/sophon-opencv-latest
sudo ln -s /opt/sophon/sophon-sample_0.10.0 /opt/sophon/sophon-sample-latest

sudo sed -i "s/usr\/local/opt\/sophon\/sophon-ffmpeg-latest/g"  /opt/sophon/sophon-ffmpeg-latest/lib/pkgconfig/*.pc
sudo sed -i "s/^prefix=.*$/prefix=\/opt\/sophon\/sophon-opencv-latest/g"  /opt/sophon/sophon-opencv-latest/lib/pkgconfig/opencv4.pc
```

安装 bz2 libc6 libgcc 依赖库

```sh
dnf install bzip2 glibc libgcc
```

然后是一些配置工作：

```sh
添加库和可执行文件路径：
sudo cp /opt/sophon/sophon-ffmpeg-latest/data/01_sophon-ffmpeg.conf   /etc/ld.so.conf.d/
sudo cp /opt/sophon/sophon-opencv-latest/data/02_sophon-opencv.conf   /etc/ld.so.conf.d/
sudo ldconfig

sudo cp /opt/sophon/sophon-ffmpeg-latest/data/sophon-ffmpeg-autoconf.sh   /etc/profile.d/
sudo cp /opt/sophon/sophon-opencv-latest/data/sophon-opencv-autoconf.sh   /etc/profile.d/
sudo cp /opt/sophon/sophon-sample-latest/data/sophon-sample-autoconf.sh   /etc/profile.d/
source /etc/profile
```

卸载方式：

```sh
sudo rm -f /etc/ld.so.conf.d/01_sophon-ffmpeg.conf
sudo rm -f /etc/ld.so.conf.d/02_sophon-opencv.conf
sudo ldconfig
sudo rm -f /etc/profile.d/sophon-ffmpeg-autoconf.sh
sudo rm -f /etc/profile.d/sophon-opencv-autoconf.sh
sudo rm -f /etc/profile.d/sophon-sample-autoconf.sh
sudo rm -f /opt/sophon/sophon-ffmpeg-latest
sudo rm -f /opt/sophon/sophon-opencv-latest
sudo rm -f /opt/sophon/sophon-sample-latest
sudo rm -rf /opt/sophon/sophon-ffmpeg_0.10.0
sudo rm -rf /opt/sophon/sophon-opencv_0.10.0
sudo rm -rf /opt/sophon/sophon-sample_0.10.0
sudo rm -rf /opt/sophon/opencv-bmcpu_0.10.0
```

注意事项：

* 如果需要用 sophon-opencv 的 python 接口，手动设置环境变量：
export PYTHONPATH=$PYTHONPATH:/opt/sophon/sophon-opencv-latest/opencv-python

具体请参考[《MULTIMEDIA使用手册》](http://doc.sophgo.com/sdk-docs/v24.04.01/docs_latest_release/docs/sophon-mw/manual/html/index.html)、[《MULTIMEDIA开发参考手册》](https://doc.sophgo.com/sdk-docs/v24.04.01/docs_latest_release/docs/sophon-mw/guide/html/index.html)。

### 编译安装sophon-sail

如果例程依赖sophon-sail则需要编译和安装sophon-sail，否则可跳过本章节。

采用交叉编译方式编译sophon-sail。

#### 编译可被C++接口调用的动态库及头文件

安装需要源码文件 sophon-sail_3.8.0.tar.gz

**.获取交叉编译需要使用的libsophon,sophon-ffmpeg,sophon-opencv**
此章节所有的编译操作都是在x86主机上,使用交叉编译的方式进行编译。下面示例中选择libsophon的版本为0.5.1, sophon-ffmpeg的版本为0.10.0,sophon-opencv的版本为0.10.0。

1. 获取‘libsophon_0.5.1_riscv64.tar.gz’,并解压

```sh
tar -xvf libsophon_0.4.0_riscv64.tar.gz
```

解压后libsophon的目录为‘libsophon_0.5.1_riscv64/opt/sophon/libsophon-0.5.1’

2. 获取‘sophon-mw_0.10.0_riscv_64.tar.gz’,并解压

```sh
tar -xvf sophon-mw_0.10.0_riscv_64.tar.gz
```

解压后sophon-ffmpeg的目录为‘sophon-mw_0.10.0_riscv_64/opt/sophon/sophon-ffmpeg_0.10.0’。

解压后sophon-opencv的目录为‘sophon-mw_0.10.0_riscv_64/opt/sophon/sophon-opencv_0.10.0’。

**.安装g++-riscv64-linux-gnu工具链**
如果已经安装,可忽略此步骤

```sh
sudo apt-get install gcc-riscv64-linux-gnu g++-riscv64-linux-gnu
```

**.典型编译方式一**
通过交叉编译的方式,编译出包含bmcv,sophon-ffmpeg,sophon-opencv的SAIL。

1. 下载SOPHON-SAIL源码,解压后进入其源码目录

```sh
tar -zxvf sophon-sail_3.8.0.tar.gz
cd sophon-sail
```

2. 创建编译文件夹build,并进入build文件夹

```sh
mkdir build && cd build
```

3. 执行编译命令

```sh
cmake -DBUILD_TYPE=riscv  \
    -DBUILD_PYSAIL=OFF \
    -DCMAKE_TOOLCHAIN_FILE=../cmake/BM168x_RISCV/ToolChain_riscv64_linux.cmake \
    -DLIBSOPHON_BASIC_PATH=/root/sophon_sdk/libsophon_0.5.1_riscv64/opt/sophon/libsophon-0.5.1 \
    -DFFMPEG_BASIC_PATH=/root/sophon_sdk/sophon-mw_0.10.0_riscv_64/opt/sophon/sophon-ffmpeg_0.10.0 \
    -DOPENCV_BASIC_PATH=/root/sophon_sdk/sophon-mw_0.10.0_riscv_64/opt/sophon/sophon-opencv_0.10.0 ..
make sail
```

4. 安装SAIL动态库及头文件,程序将自动在源码目录下创建‘build_riscv’,编译结果将安装在‘build_riscv’下面

```sh
sudo make install
```

5. 将‘build_riscv’文件夹下的‘sophon-sail’拷贝至目标RISCV主机的‘/opt/sophon’目录下,即可在目标机器上面进行调用。

**.典型编译方式二**
通过交叉编译的方式,编译出不包含bmcv,sophon-ffmpeg,sophon-opencv的SAIL。

通过此方式编译出来的SAIL无法使用其Decoder、Bmcv等多媒体相关接口。

1. 下载SOPHON-SAIL源码,解压后进入其源码目录

2. 创建编译文件夹build,并进入build文件夹

```sh
mkdir build && cd build
```

3. 执行编译命令

```sh
cmake -DBUILD_TYPE=riscv  \
    -DONLY_RUNTIME=ON \
    -DBUILD_PYSAIL=OFF \
    -DCMAKE_TOOLCHAIN_FILE=../cmake/BM168x_RISCV/ToolChain_riscv64_linux.cmake \
    -DLIBSOPHON_BASIC_PATH=/root/sophon_sdk/libsophon_0.5.1_riscv64/opt/sophon/libsophon-0.5.1 ..
make sail
```

4. 安装SAIL动态库及头文件,程序将自动在源码目录下创建‘build_riscv’,编译结果将安装在‘build_riscv’下面

```sh
sudo make install
```

5. 将‘build_riscv’文件夹下的‘sophon-sail’拷贝至目标RISCV主机的‘/opt/sophon’目录下,即可在目标机器上面进行调用。

具体请参考：

* [《编译可被C++接口调用的动态库及头文件》](https://doc.sophgo.com/sdk-docs/v24.04.01/docs_latest_release/docs/sophon-sail/docs/zh/html/1_build.html#riscv-mode)

#### 编译可被Python3接口调用的Wheel文件

**.典型编译方式一**
使用指定版本的python3(和目标RISCV服务器上的python3保持一致),通过交叉编译的方式,编译出包含bmcv,sophon-ffmpeg,sophon-opencv的SAIL, python3的安装方式可通过python官方网站获取, 也可以根据[获取在X86主机上进行交叉编译的Python3]获取已经编译好的python3。 本示例使用的python3路径为‘python_3.11.0/bin/python3’,python3的动态库目录‘python_3.11.0/lib’。

1. 下载SOPHON-SAIL源码,解压后进入其源码目录

2. 创建编译文件夹build,并进入build文件夹

```sh
mkdir build && cd build
```

3. 执行编译命令

```sh
cmake -DBUILD_TYPE=riscv  \
    -DCMAKE_TOOLCHAIN_FILE=../cmake/BM168x_RISCV/ToolChain_riscv64_linux.cmake \
    -DPYTHON_EXECUTABLE=/root/miniconda3/envs/sail-python/bin/python3 \
    -DCUSTOM_PY_LIBDIR=/root/miniconda3/envs/sail-python/lib \
    -DLIBSOPHON_BASIC_PATH=../../libsophon_0.5.1_riscv64/opt/sophon/libsophon-0.5.1 \
    -DFFMPEG_BASIC_PATH=../../sophon-mw_0.10.0_riscv_64/opt/sophon/sophon-ffmpeg_0.10.0 \
    -DOPENCV_BASIC_PATH=../../sophon-mw_0.10.0_riscv_64/opt/sophon/sophon-opencv_0.10.0 ..
make pysail
```

4. 打包生成python wheel,生成的wheel包的路径为‘python/riscv/dist’,文件名为‘sophon_riscv64-3.8.0-py3-none-any.whl’

```sh
cd ../python/riscv
chmod +x sophon_riscv_whl.sh
./sophon_riscv_whl.sh
```

5. 安装python wheel

将dist/目录下的‘sophon_riscv64-3.8.0-py3-none-any.whl’拷贝到目标RISCV服务器上,然后执行如下安装命令

```sh
pip3 install sophon_riscv64-3.8.0-py3-none-any.whl --force-reinstall
```

**.典型编译方式二**
使用指定版本的python3(和目标RISCV服务器上的python3保持一致),通过交叉编译的方式,编译出不包含bmcv,sophon-ffmpeg,sophon-opencv的SAIL, python3的安装方式可通过python官方网站获取, 也可以根据[获取在X86主机上进行交叉编译的Python3]获取已经编译好的python3。 本示例使用的python3路径为‘python_3.11.0/bin/python3’,python3的动态库目录‘python_3.11.0/lib’。

通过此方式编译出来的SAIL无法使用其Decoder、Bmcv等多媒体相关接口。

1. 下载SOPHON-SAIL源码,解压后进入其源码目录

2. 创建编译文件夹build,并进入build文件夹

```sh
mkdir build && cd build
```

3. 执行编译命令

```sh
cmake -DBUILD_TYPE=riscv  \
    -DONLY_RUNTIME=ON \
    -DCMAKE_TOOLCHAIN_FILE=../cmake/BM168x_RISCV/ToolChain_riscv64_linux.cmake \
    -DPYTHON_EXECUTABLE=/root/miniconda3/envs/sail-python/bin/python3 \
    -DCUSTOM_PY_LIBDIR=/root/miniconda3/envs/sail-python/lib \
    -DLIBSOPHON_BASIC_PATH=/root/sophon_sdk/libsophon_0.5.1_riscv64/opt/sophon/libsophon-0.5.1 ..
make pysail
```

4. 打包生成python wheel,生成的wheel包的路径为‘python/riscv/dist’,文件名为‘sophon_riscv64-3.8.0-py3-none-any.whl’

```sh
cd ../python/riscv
chmod +x sophon_riscv_whl.sh
./sophon_riscv_whl.sh
```

5. 安装python wheel

将‘sophon_riscv64-3.8.0-py3-none-any.whl’拷贝到目标RISCV服务器上,然后执行如下安装命令

```sh
pip3 install sophon_riscv64-3.8.0-py3-none-any.whl --force-reinstall
```

具体参考：

* [《编译可被Python3接口调用的Wheel文件》](https://doc.sophgo.com/sdk-docs/v24.04.01/docs_latest_release/docs/sophon-sail/docs/zh/html/1_build.html#id8)
