# CaptureFrameRate 参数详细解析

基于 SMPTE RiS-OSVP Metadata CamDKit 源码分析

---

## 1. CaptureFrameRate 参数概述

### 1.1 基本定义

`captureFrameRate` 参数表示相机的捕获帧率，定义了相机每秒钟捕获的图像帧数。它是一个**严格正有理数**（StrictlyPositiveRational），以赫兹（Hz）为单位。

```json
"captureFrameRate": {
  "num": 24000,
  "denom": 1001
}
```

### 1.2 源码定义

**文件位置**: `camdkit/camera_types.py:60-65`

```python
capture_frame_rate: Annotated[StrictlyPositiveRational | None,
  Field(alias="captureFrameRate",
      json_schema_extra={"units": HERTZ,
                         "clip_property": "capture_frame_rate",
                         "constraints": STRICTLY_POSITIVE_RATIONAL})] = None
"""Capture frame rate of the camera"""
```

### 1.3 技术特性

- **数据类型**: StrictlyPositiveRational
- **单位**: 赫兹 (Hz)
- **采样类型**: 静态 (Static)
- **所属部分**: camera 对象
- **别名**: "captureFrameRate"

---

## 2. 标准帧率设定值详解

### 2.1 常见帧率对照表

基于提供的设定值表格和源码分析：

| FPS | num | denom | 精确值 | 用途说明 |
|-----|-----|-------|--------|----------|
| 60 | 60000 | 1000 | 60.000 | 高帧率视频，游戏录制 |
| 59.94 | 60000 | 1001 | 59.940 | NTSC 高清标准 |
| 50 | 50000 | 1000 | 50.000 | PAL 高帧率 |
| 29.97 | 30000 | 1001 | 29.970 | NTSC 标准清晰度 |
| 25 | 25000 | 1000 | 25.000 | PAL 标准，电影制作 |
| 24 | 24000 | 1000 | 24.000 | 电影标准帧率 |
| 23.98 | 24000 | 1001 | 23.976 | 电影 NTSC 兼容 |

### 2.2 NTSC 分数帧率的技术背景

**为什么使用 1001 作为分母？**

NTSC 标准中，为了避免彩色副载波与音频载波的干扰，将原本的整数帧率降低了 0.1%：
- 30 fps → 30000/1001 ≈ 29.97 fps
- 24 fps → 24000/1001 ≈ 23.976 fps
- 60 fps → 60000/1001 ≈ 59.94 fps

**源码支持**: `camdkit/utils.py:12`

```python
_WELL_KNOWN_FRACTIONAL_FPS = set([
    Fraction(24000, 1001),   # 23.976 fps
    Fraction(30000, 1001),   # 29.970 fps  
    Fraction(60000, 1001),   # 59.940 fps
    Fraction(120000, 1001)   # 119.880 fps
])
```

---

## 3. 帧率自动识别机制

### 3.1 FPS 猜测算法

**文件位置**: `camdkit/utils.py:15-40`

```python
def guess_fps(fps: numbers.Real) -> Fraction:
    """Heuristically determines an exact fps value from an approximate one using
    well-known fps values."""
    
    if fps is None:
        raise ValueError
    
    if isinstance(fps, numbers.Integral):
        return Fraction(fps)
    
    if not isinstance(fps, numbers.Real):
        raise TypeError
    
    if isinstance(fps, numbers.Rational) and fps.denominator == 1:
        return fps
    
    approx_fps = float(fps)
    
    # 检查是否接近整数
    if abs(int(approx_fps) - approx_fps) / approx_fps < _FPS_THRESHOLD:
        return Fraction(int(approx_fps))
    
    # 检查是否匹配已知分数帧率
    for wkfps in _WELL_KNOWN_FRACTIONAL_FPS:
        if round(float(wkfps), 2) == round(approx_fps, 2):
            return wkfps
    
    raise ValueError("Not a valid FPS")
```

### 3.2 阈值设定

```python
_FPS_THRESHOLD = 0.01  # 1% 的误差容忍度
```

**算法逻辑**:
1. **整数检测**: 如果输入接近整数（误差<1%），返回整数帧率
2. **已知分数匹配**: 检查是否匹配预定义的分数帧率
3. **精度匹配**: 使用四舍五入到小数点后2位进行匹配

---

## 4. 与其他参数的关系

### 4.1 与 Duration 的协作

Duration 和 captureFrameRate 共同决定了片段包含的总帧数：

```
Total Frames = Duration × captureFrameRate
```

**实例计算**:
```json
{
  "duration": {"num": 1, "denom": 25},        // 0.04 秒
  "captureFrameRate": {"num": 24000, "denom": 1001}  // 23.976 fps
}
```

总帧数 = (1/25) × (24000/1001) = 24000/25025 ≈ 0.959 帧

### 4.2 与 timing.sampleRate 的区别

| 参数 | 作用域 | 含义 |
|------|--------|------|
| captureFrameRate | 相机静态属性 | 相机硬件的图像捕获频率 |
| timing.sampleRate | 动态采样 | 跟踪数据的采样频率 |

**典型场景**:
- 相机以 24 fps 拍摄画面
- 跟踪系统以 120 Hz 采样位置数据
- 两者可以不同，但需要时间同步

### 4.3 与 timecode.frameRate 的协调

**文件位置**: `examples/complete_static_example.json:100-103`

```json
"timecode": {
  "frameRate": {
    "num": 24000,
    "denom": 1001
  }
}
```

通常情况下，`captureFrameRate` 和 `timecode.frameRate` 应该保持一致，确保时码与实际图像帧的对应关系。

---

## 5. 实际应用场景

### 5.1 电影制作标准配置

```json
{
  "static": {
    "camera": {
      "captureFrameRate": {
        "num": 24000,
        "denom": 1001
      }
    }
  }
}
```

**应用**: 数字电影院标准，与 NTSC 系统兼容

### 5.2 电视广播标准配置

#### PAL 制式
```json
{
  "captureFrameRate": {
    "num": 25000,
    "denom": 1000
  }
}
```

#### NTSC 制式
```json
{
  "captureFrameRate": {
    "num": 30000,
    "denom": 1001
  }
}
```

### 5.3 高帧率应用

```json
{
  "captureFrameRate": {
    "num": 60000,
    "denom": 1001
  }
}
```

**应用**: 体育转播、慢动作回放、VR 内容制作

---

## 6. 源码实现细节

### 6.1 数据验证

**文件位置**: `camdkit/camera_types.py:141-144`

```python
@field_validator("capture_frame_rate", "anamorphic_squeeze", mode="before")
@classmethod
def coerce_camera_type_to_strictly_positive_rational(cls, v):
    return rationalize_strictly_and_positively(v)
```

### 6.2 JSON 序列化别名

```python
Field(alias="captureFrameRate")
```

**作用**: 确保 JSON 输出使用驼峰命名 `captureFrameRate`，而 Python 代码使用下划线命名 `capture_frame_rate`

### 6.3 示例代码生成

**文件位置**: `camdkit/examples.py:106`

```python
clip.capture_frame_rate = Fraction(24000, 1001)
```

这会自动转换为标准的 JSON 格式。

---

## 7. 性能与精度考虑

### 7.1 有理数优势

1. **精确表示**: 避免浮点数的精度损失
2. **标准兼容**: 完美表示 NTSC 等标准的分数帧率
3. **计算稳定**: 数学运算保持精度

### 7.2 数值范围限制

- **分子范围**: 1 ≤ num ≤ 2,147,483,647
- **分母范围**: 1 ≤ denom ≤ 4,294,967,295
- **实际帧率范围**: 约 2.33×10⁻¹⁰ Hz 到 2.1×10⁹ Hz

### 7.3 常见错误处理

```python
# 错误示例
{"num": 0, "denom": 1}        # 分子不能为0
{"num": -24, "denom": 1}      # 不能为负数
{"num": 24, "denom": 0}       # 分母不能为0

# 正确示例  
{"num": 24000, "denom": 1001} # ✓ 标准 23.976 fps
{"num": 25, "denom": 1}       # ✓ 标准 25 fps
```

---

## 8. 最佳实践建议

### 8.1 帧率选择原则

1. **遵循行业标准**: 优先使用标准帧率 (23.976, 24, 25, 29.97, 30, 50, 59.94, 60)
2. **考虑后期流程**: 确保与后期制作流程兼容
3. **区域标准**: 根据目标市场选择 PAL 或 NTSC 标准

### 8.2 虚拟制作配置

```json
{
  "static": {
    "duration": {"num": 1, "denom": 24},
    "camera": {
      "captureFrameRate": {"num": 24000, "denom": 1001}
    }
  },
  "timing": {
    "sampleRate": {"num": 24000, "denom": 1001}
  }
}
```

**建议**: 保持 duration、captureFrameRate 和 sampleRate 的一致性

### 8.3 多相机同步

在多相机系统中，所有相机的 `captureFrameRate` 应该保持一致：

```json
{
  "camera_A": {"captureFrameRate": {"num": 24000, "denom": 1001}},
  "camera_B": {"captureFrameRate": {"num": 24000, "denom": 1001}},
  "camera_C": {"captureFrameRate": {"num": 24000, "denom": 1001}}
}
```

---

## 9. 故障排查指南

### 9.1 常见问题

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 帧率不匹配 | captureFrameRate 与 timecode 不一致 | 统一帧率设置 |
| 精度丢失 | 使用浮点数表示 | 改用有理数格式 |
| 验证失败 | 数值超出范围 | 检查 num/denom 范围 |

### 9.2 调试技巧

```python
# 验证帧率转换
from fractions import Fraction
fps = Fraction(24000, 1001)
print(f"FPS: {float(fps):.6f}")  # 输出: 23.976024

# 检查是否为标准帧率
from camdkit.utils import guess_fps
try:
    standard_fps = guess_fps(23.976)
    print(f"Standard FPS: {standard_fps}")
except ValueError:
    print("非标准帧率")
```

---

## 总结

`captureFrameRate` 参数是 OpenTrackIO 协议中的核心相机属性，它通过精确的有理数表示确保了各种标准帧率的准确性。正确配置此参数对于确保虚拟制作流程中的时间同步和多设备协作至关重要。在实际应用中，应根据制作需求和技术标准选择合适的帧率设置，并确保与其他时间相关参数的一致性。