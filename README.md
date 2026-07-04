# SMJE-YOLOv8s 航拍小目标检测系统

基于YOLOv8改进的轻量化无人机航拍小目标检测算法，支持边缘设备实时部署。

## 📋 项目简介

本项目针对无人机航拍图像中**背景复杂、目标尺度小、密集遮挡严重、机载边缘设备算力受限**等问题，提出了 **SMJE-YOLOv8s** 改进算法：

- **SFFM模块**：多尺度自适应特征融合，增强小目标特征提取能力
- **BFSM模块**：轻量级特征注意力，抑制复杂背景干扰
- **TensorRT量化**：模型轻量化，适配Jetson Orin NX边缘部署
- **PyQt5可视化界面**：支持实时检测、图片检测、视频检测

## 🎯 核心指标

| 指标 | 数值 |
|------|------|
| **mAP@0.5** | **42.20%**（相比基线YOLOv8s提升 **9.3%**） |
| **模型参数量** | **10.5M**（比YOLOv8s减少 **1.63M**） |
| **推理帧率** | **15-20 FPS**（Jetson Orin NX，输入800×800） |
| **模型体积** | **19M**（TensorRT FP16标化后） |

## 🛠️ 技术栈

- **深度学习框架**：PyTorch 1.13.0, Ultralytics YOLOv8
- **目标检测**：YOLOv8s + SFFM + BFSM 改进
- **GUI开发**：PyQt5
- **边缘部署**：TensorRT 10.7, ONNX
- **图像处理**：OpenCV 4.7.0
- **硬件平台**：NVIDIA RTX 3060Ti（训练）/ Jetson Orin NX（部署）

## 📁 项目结构

```
SMJE-YOLOv8s/
├── models/                 # 模型定义
│   ├── yolov8_smj5.py     # SMJE改进网络结构
│   ├── sffm.py            # 多尺度特征融合模块
│   └── bfsm.py            # 轻量级注意力模块
├── gui/                    # 可视化界面
│   ├── main_window.py     # 主窗口
│   ├── detector.py        # 检测核心
│   └── utils.py           # 工具函数
├── deploy/                 # 边缘部署
│   ├── export_onnx.py     # ONNX导出
│   ├── tensorrt_convert.py # TensorRT转换
│   └── jetson_infer.py    # Jetson推理
├── data/                   # 数据集配置
│   └── visdrone.yaml      # VisDrone2019配置
├── utils/                  # 通用工具
│   ├── augment.py         # 数据增强
│   ├── metrics.py         # 评估指标
│   └── plots.py           # 可视化
├── train.py               # 训练脚本
├── val.py                 # 验证脚本
├── detect.py              # 检测脚本
├── requirements.txt       # 依赖包
└── README.md              # 项目说明
```

## 🚀 快速开始

### 环境配置

```bash
# 克隆项目
git clone https://github.com/JinIilaiQiaoMen/SMJE-YOLOv8s.git
cd SMJE-YOLOv8s

# 创建虚拟环境
python -m venv venv
source venv/bin/activate  # Linux/Mac
# 或 venv\Scripts\activate  # Windows

# 安装依赖
pip install -r requirements.txt
```

### 模型训练

```bash
# 训练SMJE-YOLOv8s
python train.py \
  --model models/yolov8_smje.py \
  --data data/visdrone.yaml \
  --epochs 200 \
  --batch 4 \
  --imgsz 800 \
  --device 0
```

### 模型验证

```bash
# 验证模型性能
python val.py \
  --weights runs/train/exp/weights/best.pt \
  --data data/visdrone.yaml \
  --imgsz 800
```

### 启动GUI检测工具

```bash
# 启动可视化检测界面
python gui/main_window.py
```

支持三种检测模式：
- **摄像头实时检测**：支持内罯/外接摄像头
- **图片文件检测**：支持.jpg, .png, .bmp等格式
- **视频文件检测**：支持.mp4, .avi, .mov等格式

### 边缘部署（Jetson Orin NX）

```bash
# 1. 导出ONNX模型
python deploy/export_onnx.py \
  --weights runs/train/exp/weights/best.pt \
  --imgsz 800

# 2. TensorRT转换
python deploy/tensorrt_convert.py \
  --onnx model.onnx \
  --fp16

# 3. Jetson推理
python deploy/jetson_infer.py \
  --engine model.engine \
  --source 0  # 摄像头
```

## 📊 数据集

使用 [VisDrone2019-DET](https://github.com/VisDrone/VisDrone-Dataset) 航拍数据集：

| 类别 | 说明 |
|------|------|
| pedestrian | 行人 |
| person | 人 |
| bicycle | 自行车 |
| car | 汽车 |
| van | 厢式货车 |
| truck | 卡车 |
| tricycle | 三轮车 |
| awning-tricycle | 遮阳三轮车 |
| bus | 公交车 |
| motor | 摩托车 |

数据集划分：训练集6471张 / 验证集548张 / 测试集1610张

## 🔬 实验结果

### 对比实验

| 算法 | Backbone | mAP@50 | 参数量(M) |
|------|----------|--------|-----------|
| Faster-RCNN | ResNet50 | 32.2% | 41.18 |
| YOLOv5s | CSPDarkNet | 31.4% | 10.0 |
| YOLOv8s | CSPDarkNet | 32.9% | 12.13 |
| **SMJE-YOLOv8s** | **改进CSPDarkNet** | **42.2%** | **10.5** |

### 各类别AP

| 类别 | AP |
|------|-----|
| car | 81.5% |
| bus | 57.7% |
| motor | 48.0% |
| van | 47.2% |
| pedestrian | 45.0% |

## 🎬 演示视频

项目演示视频展示了PyQt5可视化检测工具的实际运行效果：

- ✅ 多类别目标实时检测
- ✅ 置信度分数动态显示
- ✅ 检测框实时跟踪
- ✅ 支持视频文件输入

> 演示视频已上传至项目Release，可在 [Releases](https://github.com/JinIilaiQiaoMen/SMJE-YOLOv8s/releases) 页面查看。

## 📦 模型下载

| 模型 | 说明 | 下载 |
|------|------|------|
| SMJE-YOLOv8s (PyTorch) | 训练好的权重文件 | [Release v1.0](https://github.com/JinIilaiQiaoMen/SMJE-YOLOv8s/releases) |
| SMJE-YOLOv8s (ONNX) | ONNX格式 | [Release v1.0](https://github.com/JinIilaiQiaoMen/SMJE-YOLOv8s/releases) |
| SMJE-YOLOv8s (TensorRT) | TensorRT引擎 | [Release v1.0](https://github.com/JinIilaiQiaoMen/SMJE-YOLOv8s/releases) |

## 🖥️ 系统界面

### GUI检测工具功能

| 功能模块 | 主要功能描述 |
|----------|-------------|
| 模型加载 | 支持.pt、.pth格式模型加载，自动解析与设备适配 |
| 检测参数调整 | 支持置信度阈值、IOU阈值、输入尺寸调节 |
| 检测来源选择 | 支持摄像头实时检测、图片检测、视频文件检测 |
| 实时推理与标注 | 完成目标检测、框选标注、结果显示 |
| 模型一键部署 | 支VOLO系列训练模型向边缘设备的快速部署 |
| 数据与结果展示 | 检测结果可视化输出，支持检测信息统计 |

## 🔧 核心改进

### SFFM 多尺度特征融合模块

通过并行融合不同分辨率的特征图，增强模型检测多尺度目标的能力：

```python
SFFM(x) = SiLU(BN(Conv([x, x₂, x₃]))) + x
```

其中 x₂、x₃ 是经过下采样后获得的特征，通过多尺度信息互补解决单一尺度特征困难。

### BFSM 轻量级特征注意力模块

利用1×1卷积生成注意力图，通过Sigmoid函数归一化，自动抑制背景区域响应值：

```python
BFSM = x ⊙ σ(Conv(x)) + x
```

## 📚 引用

如果您在研究中使用了本项目，请引用：

```bibtex
@article{ren2026smj5,
  title={基于YOLOv8的航拍小目标检测算法的研究与部署},
  author={任晨光},
  journal={福州外语外贸学院毕业论文},
  year={2026}
}
```

## 📄 许可证

本项目采用 [MIT License](LICENSE) 开源许可证。

## 👤 作者


## 🙏 致谢

- [Ultralytics YOLOv8](https://github.com/ultralytics/ultralytics)
- [VisDrone Dataset](https://github.com/VisDrone/VisDrone-Dataset)
- [TensorRT](https://developer.nvidia.com/tensorrt)

---

> **注意**：本项目为毕业设计研究成果，论文内容不上传GitHub。如需了解详细算法原理和实验分析，请参考相关学术文献。
