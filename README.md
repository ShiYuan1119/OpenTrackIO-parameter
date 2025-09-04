# OpenTrackIO 参数分析文档

基于 SMPTE RiS-OSVP Metadata CamDKit 官方规范的详细分析

## 文档结构

### 📚 主要文档

1. **[OpenTrackIO_Parameters_Documentation.md](./OpenTrackIO_Parameters_Documentation.md)**
   - 完整的参数定义和说明
   - 每个参数的详细描述、类型、约束
   - 包含中文注释和补充说明

2. **[OpenTrackIO_Technical_Reference.md](./OpenTrackIO_Technical_Reference.md)**
   - 技术实现细节
   - 数据类型定义和验证规则
   - 性能优化和集成指南

3. **[OpenTrackIO_Parameter_Quick_Reference.md](./OpenTrackIO_Parameter_Quick_Reference.md)**
   - 参数快速查询表
   - 按分类整理的参数列表
   - 验证检查清单

4. **[OpenTrackIO_Examples_and_Use_Cases.md](./OpenTrackIO_Examples_and_Use_Cases.md)**
   - 实际应用场景示例
   - 不同拍摄环境的配置
   - 调试和故障排除指南

### 🔧 源码文件

- **`camdkit/`**: 从官方仓库复制的核心 Python 包
  - `clip.py`: 主要数据模型定义
  - `lens_types.py`: 镜头参数类型
  - `camera_types.py`: 相机参数类型
  - `timing_types.py`: 时间同步类型
  - `tracker_types.py`: 跟踪器类型
  - `transform_types.py`: 空间变换类型

- **`examples/`**: 官方示例 JSON 文件
  - `complete_static_example.json`: 完整静态示例
  - `complete_dynamic_example.json`: 完整动态示例
  - `recommended_static_example.json`: 推荐静态示例
  - `recommended_dynamic_example.json`: 推荐动态示例

## OpenTrackIO 概述

OpenTrackIO 是由 SMPTE RiS-OSVP 工作组开发的开源协议，用于虚拟制作环境中的实时元数据交换。该协议定义了包含以下信息的 JSON 数据结构：

### 核心组件

- **🎥 Camera**: 相机硬件信息和设置
- **🔍 Lens**: 镜头参数和光学特性
- **📡 Tracker**: 跟踪系统状态和配置
- **⏱️ Timing**: 时间同步和采样信息
- **📐 Transforms**: 空间变换和坐标系统
- **🌍 GlobalStage**: 全局坐标定位
- **⚙️ Custom**: 自定义扩展字段

### 主要特性

- ✅ **精确时间同步**: 支持 PTP、NTP 等多种时间源
- ✅ **灵活数据结构**: 静态和动态参数分离
- ✅ **多种畸变模型**: Brown-Conrady 等标准模型
- ✅ **变换链组合**: 支持复杂的多轴运动系统
- ✅ **扩展性**: 自定义字段支持厂商特定功能
- ✅ **版本兼容**: 明确的版本管理机制

## 快速开始

### 基本使用示例

```json
{
  "protocol": {
    "name": "OpenTrackIO",
    "version": [1, 0, 0]
  },
  "tracker": {
    "status": "Optical Good",
    "recording": true
  },
  "timing": {
    "mode": "external",
    "sampleRate": {"num": 24, "denom": 1}
  },
  "lens": {
    "fStop": 2.8,
    "focusDistance": 3.5,
    "pinholeFocalLength": 50.0
  },
  "transforms": [{
    "translation": {"x": 0.0, "y": -2.0, "z": 1.8},
    "rotation": {"pan": 0.0, "tilt": -5.0, "roll": 0.0},
    "id": "Camera"
  }]
}
```

### Python 集成

```python
from camdkit.clip import Clip
import json

# 解析 OpenTrackIO 数据
data = json.loads(opentrackio_json)
clip = Clip.model_validate(data)

# 访问参数
camera_position = clip.transforms[0].translation
lens_fstop = clip.lens.f_stop[0]
```

## 参考资料

- 📖 [OpenTrackIO 官方文档](https://ris-pub.smpte.org/ris-osvp-metadata-camdkit/)
- 📖 [OpenLensIO 规范](https://ris-pub.smpte.org/ris-osvp-metadata-camdkit/res/OpenLensIO_v1-0-0.pdf)
- 💻 [SMPTE GitHub 仓库](https://github.com/SMPTE/ris-osvp-metadata-camdkit)
- 📋 [SMPTE ST2059-2:2021](https://ieeexplore.ieee.org/document/9450292) (PTP 配置文件)

## 贡献

本文档基于 SMPTE RiS-OSVP Metadata CamDKit 源码分析生成，旨在为中文用户提供详细的 OpenTrackIO 参数参考。

### 更新日志

- **v1.0.0** (2024): 初始版本，基于 CamDKit 源码分析
- 涵盖所有核心参数定义
- 包含实际应用示例
- 提供技术实现指南

---

*文档生成时间: 2024年*  
*基于 SMPTE RiS-OSVP Metadata CamDKit v0.9.3*