# 运行SOPHON-DEMO中的Yolov5模型推理

目标：在 TPU 上使用 YoloV5 模型进行推理

## 环境说明

* CPU: sg2042
* TPU: SC7 HP75
* Chip: BM1684X
* OS: oe2403
* 平台：riscv pcie

需要提前安装好 libsophon、sophon-mv、sophon-sail 包

## 操作步骤

克隆源码

```sh
git clone https://github.com/sophgo/sophon-demo.git
cd sophon-demo/sample/YOLOv5/
```

### 数据准备

项目已在`scripts`目录下提供了相关模型和数据的下载脚本`download.sh`

```sh
chmod -R +x scrips/
./scripts/download.sh
```

下载的模型在./models/目录下，数据在./datasets/目录下

### 模型编译

TODO

### cpp程序

#### 程序编译

1. bmcv

```sh
cd cpp/yolov5_bmcv
mkdir build && cd build
cmake .. 
make
cd ..
```

编译完成后，会在yolov5_bmcv目录下生成yolov5_bmcv.pcie。

2. sail

```sh
cd cpp/yolov5_sail
mkdir build && cd build
cmake ..
make
cd ..
```

编译完成后，会在yolov5_sail目录下生成yolov5_sail.pcie。

#### 推理测试

1. 参数说明

`yolov5_bmcv.pcie`与`yolov5_sail.pcie`参数相同，根据实际情况进行传参

查看可执行程序的默认参数

```sh
./yolov5_bmcv.pcie --help
```

>**注意**： cpp例程传参与python不同，需要用等于号，例如`./yolov5_bmcv.pcie --bmodel=xxx`。cpp可以使用`--use_cpu_opt=true`开启后处理cpu加速，`use_cpu_opt`仅限输出维度为5的模型(一般是3输出，别的输出个数可能需要用户自行修改后处理代码)。

2. 测试图片

图片测试实例如下，支持对整个图片文件夹进行测试。

```sh
./yolov5_bmcv.pcie --input=../../datasets/test --bmodel=../../models/BM1684X/yolov5s_v6.1_3output_fp32_1b.bmodel --dev_id=0 --conf_thresh=0.5 --nms_thresh=0.5 --classnames=../../datasets/coco.names 
```

测试结束后，会将预测的图片保存在`results/images`下，预测的结果保存在`results/yolov5s_v6.1_3output_fp32_1b.bmodel_test_bmcv_cpp_result.json`下，同时会打印预测结果、推理时间等信息。

> **注意**：  
> 1.cpp例程暂时没有在图片上写字。

3. 测试视频

视频测试实例如下，支持对视频流进行测试。

```sh
./yolov5_bmcv.pcie --input=../../datasets/test_car_person_1080P.mp4 --bmodel=../../models/BM1684X/yolov5s_v6.1_3output_fp32_1b.bmodel --dev_id=0 --conf_thresh=0.5 --nms_thresh=0.5 --classnames=../../datasets/coco.names
```

测试结束后，会将预测结果画在图片上并保存在`results/images`中，同时会打印预测结果、推理时间等信息。

## 参考资料

* [sophon-demo: YOLOv5](https://github.com/sophgo/sophon-demo/blob/release/sample/YOLOv5/README.md)
* [C++ 例程](https://github.com/sophgo/sophon-demo/blob/release/sample/YOLOv5/cpp/README.md)
