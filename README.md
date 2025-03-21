# TPU开发文档 - riscv pcie mode

## 介绍

记录在SG2042+openEuler riscv环境下安装、使用算能TPU卡的过程

## 主要内容

环境配置

* [安装TPU过程](./setup_tpu.md)
* [开发环境安装指导](./environment_install_guide.md)
* [openEuler2403+TPU riscv pcie mode使用问题解决](./oe2403_libsophon_issue.md)

LLM-TPU项目

* [编译llama3.1 bmodel模型指导](./build_llama3.1_bmodel_guide.md)：包含了初始docker容器环境设置
* [运行llama3.1 bmodel模型指导](./run_llama3.1_bmodel_guide.md)
* [运行DeepSeek-R1蒸馏模型14b版本双芯bmodel指导](./run_deepseek-r1-distill-qwen-14b-2dev_bmodel_guide.md)
* [运行DeepSeek-R1蒸馏模型32b版本双芯bmodel指导](./run_deepseek-r1-distill-qwen-32b-2dev_bmodel_guide.md)
* [编译DeepSeek-R1蒸馏模型32b版本双芯bmodel指导](./build_deepseek-r1-distill-qwen-32b-2dev_bmodel_guide.md)

sophon-demo项目

* [yolov5 onnx模型导出指导](./yolov5_onnx_export_guide.md)
* [编译yolov5 bmodel模型指导](./build_yolov5_bmodel_guide.md)
* [运行yolov5 bmodel模型指导](./run_yolov5_bmodel_guide.md)
* [编译Qwen及DeepSeek-R1蒸馏bmodel模型指导](/build_qwen_bmodel_guide.md)
* [运行Qwen及DeepSeek-R1蒸馏bmodel模型指导](./run_qwen_bmodel_guide.md)
* [运行Qwen及DeepSeek-R1蒸馏bmodel模型CPP例程指导](./run_qwen_bmodel_cpp_guild.md)
* [尝试编译运行DeepSeek-R1蒸馏模型32b版本bmodel指导](./build_run_deepseek-r1-distill-qwen-32b_bmodel_guide.md)
