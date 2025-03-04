# 编译yolov5 bmodel

目标：利用导出的onnx模型使用TPU-MLIR编译得到yolov5 bmodel

## 环境说明

* x86主机docker镜像中进行
* AI编译器：TPU-MLIR

## 操作步骤

需要提前将导出的onnx模型（如：yolov5s_v6.1_3output.onnx）放到sophon-demo/sample/YOLOv5/models/onnx/目录下

* 生成FP32 BModel

​本例程在scripts目录下提供了TPU-MLIR编译FP32 BModel的脚本，请注意修改gen_fp32bmodel_mlir.sh中的onnx模型路径、生成模型目录和输入大小shapes等参数，并在执行时指定BModel运行的目标平台（支持BM1684/BM1684X/BM1688/CV186X），如：

```sh
./scripts/gen_fp32bmodel_mlir.sh bm1684x # bm1684/bm1688/cv186x
```

执行上述命令会在models/BM1684X等文件夹下生成yolov5s_v6.1_3output_fp32_1b.bmodel文件，即转换好的FP32 BModel。

* 生成FP16 BModel

```sh
./scripts/gen_fp16bmodel_mlir.sh bm1684x
```

执行上述命令会在models/BM1684X文件夹下生成yolov5s_v6.1_3output_fp16_1b.bmodel文件，即转换好的FP16 BModel。

* 生成INT8 BModel

```sh
./scripts/gen_int8bmodel_mlir.sh bm1684x
```

上述脚本会在models/BM1684X文件夹下生成yolov5s_v6.1_3output_int8_1b.bmodel等文件，即转换好的INT8 BModel。

## 参考资料

* [sophon-demo: Yolov5模型编译](https://github.com/sophgo/sophon-demo/blob/release/sample/YOLOv5/README.md#32-%E6%A8%A1%E5%9E%8B%E7%BC%96%E8%AF%91)
