# 编译Qwen bmodel

目标：导出onnx模型并使用TPU-MLIR编译得到Qwen bmodel

## 环境说明

* x86主机docker镜像中进行
* AI编译器：TPU-MLIR
* 内存占用：48G以上（推荐64GB）
* 存储需求：100G以上

## 操作步骤

### TPU-MLIR环境搭建

#### 下载并解压TPU-MLIR

从sftp上获取TPU-MLIR压缩包

```sh
cd /workspace
pip3 install dfss --upgrade
python3 -m dfss --url=open@sophgo.com:sophon-demo/Qwen/tpu-mlir_v1.10.beta.0-31-g896b42e8c-20240809.tar.gz
tar -xf tpu-mlir_v1.10.beta.0-31-g896b42e8c-20240809.tar.gz
#1684X导出Deepseek-R1-Distill-Qwen专用
python3 -m dfss --url=open@sophgo.com:sophon-demo/Qwen/tpu-mlir_v1.14.beta.0-25-gbca81b22c-20250107.tar.gz
tar -xf tpu-mlir_v1.14.beta.0-25-gbca81b22c-20250107.tar.gz
```

### 获取onnx

#### 下载Qwen官方代码

```sh
cd /workspace/sophon-demo/sample/Qwen
```

注：

* Qwen1.5-1.8B官方库3.7G左右
* Qwen1.5-7B官方库15G左右
* Qwen2.5-7B官方库15G左右
* Deepseek-R1-Distill-Qwen-1.5B官方库3.5G左右
* Deepseek-R1-Distill-Qwen-7B官方库15G左右

>由仓库config.json文件发现数据类型dtype为bfloat16，也能明白为什么模型占用存储与模型参数量大致为两倍关系

在下载之前，要确认自己有huggingface官网的access token或者SSH key。

```sh
git lfs install
git clone https://huggingface.co/Qwen/Qwen1.5-7B-Chat
git clone https://huggingface.co/Qwen/Qwen1.5-1.8B-Chat
git clone https://huggingface.co/Qwen/Qwen2.5-7B-Instruct
git clone https://huggingface.co/deepseek-ai/DeepSeek-R1-Distill-Qwen-1.5B
git clone https://huggingface.co/deepseek-ai/DeepSeek-R1-Distill-Qwen-7B
```

如果git clone完代码之后出现卡住，可以尝试ctrl+c中断，然后进入仓库运行git lfs pull。

推荐采用以下方式下载

```sh
pip install modelscope
modelscope download --model Qwen/Qwen1.5-7B-Chat --local_dir Qwen1.5-7B-Chat
modelscope download --model Qwen/Qwen1.5-1.8B-Chat --local_dir Qwen1.5-1.8B-Chat
modelscope download --model Qwen/Qwen2.5-7B-Instruct --local_dir Qwen2.5-7B-Instruct
modelscope download --model deepseek-ai/DeepSeek-R1-Distill-Qwen-1.5B --local_dir DeepSeek-R1-Distill-Qwen-1.5B
modelscope download --model deepseek-ai/DeepSeek-R1-Distill-Qwen-7B --local_dir DeepSeek-R1-Distill-Qwen-7B
```

#### 修改官方代码

本例程的`tools`目录下提供了修改好之后的`config.json`和`modeling_qwen.py`。可以直接替换掉原仓库的文件：

Qwen1.5

推荐使用虚拟环境从而避免转换其他模型时依赖冲突

```sh
mkdir envs && cd envs
python -m venv --system-site-packages qwen1.5
source qwen1.5/bin/activate
cd ..
```

安装依赖并替换文件

```sh
pip install transformers_stream_generator einops tiktoken accelerate transformers==4.37.0
cp tools/Qwen1.5-7B-Chat/config.json Qwen1.5-7B-Chat/
# 系统环境
# cp tools/Qwen1.5-7B-Chat/modeling_qwen2.py /usr/local/lib/python3.10/dist-packages/transformers/models/qwen2/
# 虚拟环境
cp tools/Qwen1.5-7B-Chat/modeling_qwen2.py /workspace/sophon-demo/sample/Qwen/envs/qwen1.5/lib/python3.10/site-packages/transformers/models/qwen2/

cp tools/Qwen1.5-1.8B-Chat/config.json Qwen1.5-1.8B-Chat/
cp tools/Qwen1.5-1.8B-Chat/modeling_qwen2.py /workspace/sophon-demo/sample/Qwen/envs/qwen1.5/lib/python3.10/site-packages/transformers/models/qwen2/
```

* PS: 此处修改了transformers包所在位置，建议替换前先pip show transformers查看一下路径

退出虚拟环境

```sh
deactivate
```

其余模型操作类似

Qwen2.5

```sh
pip install transformers_stream_generator einops tiktoken accelerate torch==2.0.1+cpu torchvision==0.15.2 transformers==4.45.2
cp tools/Qwen2.5-7B-Instruct/modeling_qwen2.py /workspace/sophon-demo/sample/Qwen/envs/qwen2.5/lib/python3.10/site-packages/transformers/models/qwen2/
```

Deepseek-R1-Distill-Qwen-1.5B/7B (BM1684X，BM1688都需要)

```sh
pip install transformers_stream_generator einops tiktoken accelerate torch==2.0.1+cpu torchvision==0.15.2 transformers==4.45.2
cp tools/DeepSeek_R1_Distill_Qwen2.5-1.5B-Instruct/modeling_qwen2.py /workspace/sophon-demo/sample/Qwen/envs/deepseek-r1/lib/python3.10/site-packages/transformers/models/qwen2/
```

#### 导出onnx

* 导出所有onnx模型，如果过程中提示缺少某些组件，直接pip install组件即可

Qwen1.5

```sh
# bm1684x 单芯
python3 tools/export_onnx_qwen1.5.py --model_path ./Qwen1.5-7B-Chat --seq_length 512 

# bm1684x 多芯
python3 tools/export_onnx_qwen1.5.py --model_path ./Qwen1.5-7B-Chat --seq_length 512 --lm_head_with_topk 1
```

Qwen2.5

```sh
# bm1684x 单芯
python3 tools/export_onnx_qwen2_5.py --model_path ./Qwen2.5-7B-Instruct --seq_length 512 
```

此时有大量onnx模型被导出到本例程中Qwen/models/onnx的目录。

### bmodel编译

首先需要在mlir工具下激活环境

```sh
cd tpu-mlir_v1.10.beta.0-31-g896b42e8c-20240809
source envsetup.sh
```

目前TPU-MLIR支持1684x对Qwen进行BF16,INT8和INT4量化，使用如下命令生成bmodel。

Qwen1.5

```sh
# bm1684x 单芯
./scripts/gen_bmodel.sh --target bm1684x --mode int4 --name qwen1.5-7b --seq_length 512 --addr_mode io_alone

# bm1684x 多芯
./scripts/gen_bmodel.sh --target bm1684x --mode int4 --name qwen1.5-7b --seq_length 512 --addr_mode io_alone --num_device 2 --dynamic 1
```

Qwen2.5

```sh
# bm1684x 单芯
./scripts/gen_bmodel.sh --target bm1684x --mode int4 --name qwen2.5-7b --seq_length 512 --addr_mode io_alone
```

其中，mode可以指定bf16/int8/int4，编译成功之后，BM1684X模型将会存放在models/BM1684X目录下

#### BM1684X编译 Deepseek-R1-Distill-Qwen bmodel

```sh
# bm1684x 单芯
# 请注意1684X导出Deepseek-R1-Distill-Qwen bmodel仅需要运行该脚本，无需转onnx，请使用专用版本tpu-mlir
python tools/model_export_BM1684X_DS_qwen.py --quantize w4bf16 --tpu_mlir_path /workspace/tpu-mlir/tpu-mlir_v1.14.beta.0-25-gbca81b22c-20250107 --torch_path ./DeepSeek-R1-Distill-Qwen-1.5B --seq_length 1024  --out_dir deepseek-r1-distill-qwen-1.5b-1024
```

其中，tpu_mlir_path指定tpu-mlir地址，torch_path指定下载模型位置，seq_length也可指定为8192，编译成功之后，BM1684X模型将会存放在out_dir指定目录下，该目录下还会存在onnx和bmodel两个中间文件夹可以删除。

## 参考资料

* [Qwen模型导出与编译](https://github.com/sophgo/sophon-demo/blob/release/sample/Qwen/docs/Qwen_Export_Guide.md)
