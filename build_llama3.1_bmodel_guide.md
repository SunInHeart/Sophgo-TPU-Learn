# 编译Llama3.1模型

目标：在docker中编译Llama3.1模型

## 环境说明

* x86主机docker中完成
* AI编译器：tpu-mlir
* 内存：64GB

## TPU-MLIR环境搭建

使用TPU-MLIR编译BModel，通常需要在x86主机上安装TPU-MLIR环境，本次操作x86主机安装的操作系统为openSUSE Leap 15.6，运行内存64GB。安装步骤：

### 安装docker

```sh
zypper install docker
systemctl start docker
systemctl enable docker
groupadd docker
usermod -aG docker $USER
newgrp docker
```

可以从DockerHub中下载镜像，或直接拉取镜像：

```sh
$ docker pull sophgo/tpuc_dev:v3.2
```

可能会下载失败，可直接下载镜像文件：

```sh
$ wget https://sophon-file.sophon.cn/sophon-prod-s3/drive/24/06/14/12/sophgo-tpuc_dev-v3.2_191a433358ad.tar.gz
$ docker load -i sophgo-tpuc_dev-v3.2_191a433358ad.tar.gz
```

若下载镜像文件，则需要确保镜像文件在当前目录，并在当前目录创建容器如下:

```sh
$ docker run --privileged --name myname -v $PWD:/workspace -it sophgo/tpuc_dev:v3.2
```

其中，`myname` 为容器名称, 可以自定义； `$PWD` 为当前目录，与容器的 `/workspace` 目录同步。

后文假定用户已经处于 docker 里面的 `/workspace` 目录。

**容器管理：**

* `ctrl+d`：可退出并停止容器
* `docker ps -a`：显示所有容器，包括停止的
* `docker start -i tpu1`：启动并进入容器，-i代表以交互模式启动容器
* `docker rm tpu1`：移除容器

### 安装tpu_mlir

直接下载并安装

```sh
pip install tpu_mlir -i https://pypi.tuna.tsinghua.edu.cn/simple
```

安装额外的依赖

```sh
pip install tpu_mlir[onnx,torch]
```

## 模型编译

克隆源码LLM-TPU

```sh
git clone https://github.com/sophgo/LLM-TPU.git
cd LLM-TPU
```

本次是尝试编译Llama3_1模型，因此进入以下目录

```sh
cd models/Llama3_1/
```

### 下载模型

```sh
pip install modelscope -i https://pypi.tuna.tsinghua.edu.cn/simple
modelscope download --model LLM-Research/Meta-Llama-3.1-8B-Instruct --local_dir ./Meta-Llama-3.1-8B-Instruct
```

### 对齐模型环境

```sh
pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple
cp ./compile/files/Meta-Llama-3.1-8B-Instruct/modeling_llama.py /usr/local/lib/python3.10/dist-packages/transformers/models/llama/modeling_llama.py
```

同时将./compile/files/Meta-Llama-3.1-8B-Instruct/config.json 替换下载好的Llama-3.1-8B-Instruct路径下的同名文件，命令如下

```sh
cp ./compile/files/Meta-Llama-3.1-8B-Instruct/config.json ./Meta-Llama-3.1-8B-Instruct/
```

### 生成onnx文件

```sh
cd compile
python export_onnx.py --model_path your_model_path --seq_length 512
```

* PS1：your_model_path 指的是原模型下载后的地址, 这里是:"../Meta-Llama-3.1-8B-Instruct"。
* PS2：默认导出sequence length为512的模型
* PS3：导出过程内存消耗上限大概为**50GB**，确保内存足够

### 生成bmodel文件

生成单芯模型

```sh
./compile.sh --mode int8 --name llama3.1-8b --seq_length 512 # same as int4
```

* PS1：生成bmodel耗时大概3小时以上，建议64G内存以及200G以上硬盘空间，不然很可能OOM或者no space left
* PS2：如果想要编译llama3.1-8b，则--name必须为llama3.1-8b
* PS3：目前给定的lib_pcie和lib_soc部分仅包含单芯的动态库，多芯部分会在后续更新
* PS4：步骤三到步骤六可以通过运行compile文件夹下的run_compile.sh完成，具体命令是：

```sh
./run_compile.sh --model_name llama3.1-8b --seq_length 512 --model_path your_model_path --tpu_mlir_path your_tpu_mlir_path
```

如果没有填写model_path，脚本会从modelscope下载模型，如果没有填写tpu_mlir_path，脚本会通过dfss下载对应的tpu_mlir压缩包并解压

运行命令会在当前compile/目录生成目标文件`llama3.1-8b_int8_1dev_512.bmodel`
