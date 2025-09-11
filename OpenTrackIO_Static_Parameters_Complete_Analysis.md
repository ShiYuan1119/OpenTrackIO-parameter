# OpenTrackIO Static 参数完整解析

基于 SMPTE RiS-OSVP Metadata CamDKit 源码深度分析

---

## 概述

`static` 部分包含了 OpenTrackIO 数据片段中的所有静态信息，这些信息在整个片段持续时间内保持不变。static 部分包含四个主要组件：

1. **duration** - 片段持续时间
2. **camera** - 相机静态属性
3. **lens** - 镜头静态属性  
4. **tracker** - 跟踪器静态属性

**源码定义**: `camdkit/clip.py:41-56`

```python
class Static(CompatibleBaseModel):
    duration: Annotated[StrictlyPositiveRational | None,
      Field(json_schema_extra={"clip_property": "duration",
                               "constraints": STRICTLY_POSITIVE_RATIONAL,
                               "units": SECOND})] = None
    """Duration of the clip"""

    camera: StaticCamera = StaticCamera()
    lens: StaticLens = StaticLens()
    tracker: StaticTracker = StaticTracker()
```

---

## 1. Duration 参数

### 1.1 基本定义

**源码位置**: `camdkit/clip.py:42-46`

```json
"duration": {
  "num": 1,
  "denom": 25
}
```

**技术规格**:
- **类型**: StrictlyPositiveRational
- **单位**: 秒 (SECOND)
- **约束**: 严格正有理数
- **采样**: 静态

### 1.2 数据结构

```python
class StrictlyPositiveRational(CompatibleBaseModel):
    num: int = Field(ge=1, le=MAX_INT_32, strict=True)      # 分子: [1..2,147,483,647]
    denom: int = Field(ge=1, le=MAX_UINT_32, strict=True)   # 分母: [1..4,294,967,295]
```

### 1.3 验证机制

**源码位置**: `camdkit/clip.py:49-52`

```python
@field_validator("duration", mode="before")
@classmethod
def coerce_duration_to_strictly_positive_rational(cls, v):
    return rationalize_strictly_and_positively(v)
```

**支持输入格式**:
- 整数: `25` → `{"num": 25, "denom": 1}`
- Python Fraction: `Fraction(1, 25)` → `{"num": 1, "denom": 25}`
- 字典: `{"num": 1, "denom": 25}` → 直接使用

---

## 2. Camera 静态参数

**源码位置**: `camdkit/camera_types.py:59-144`

### 2.1 captureFrameRate (捕获帧率)

```json
"captureFrameRate": {
  "num": 24000,
  "denom": 1001
}
```

**技术规格**:
- **类型**: StrictlyPositiveRational
- **单位**: 赫兹 (HERTZ)
- **JSON别名**: "captureFrameRate"
- **约束**: 严格正有理数

**常见设定值**:

| FPS | num | denom | 实际值 | 用途 |
|-----|-----|-------|--------|------|
| 23.98 | 24000 | 1001 | 23.976024 | 电影NTSC兼容 |
| 24 | 24000 | 1000 | 24.000 | 电影标准 |
| 25 | 25000 | 1000 | 25.000 | PAL标准 |
| 29.97 | 30000 | 1001 | 29.970030 | NTSC标准清晰度 |
| 59.94 | 60000 | 1001 | 59.940060 | NTSC高清 |
| 60 | 60000 | 1000 | 60.000 | 高帧率 |

### 2.2 activeSensorPhysicalDimensions (传感器物理尺寸)

```json
"activeSensorPhysicalDimensions": {
  "height": 24.0,
  "width": 36.0
}
```

**源码位置**: `camdkit/camera_types.py:31-41`

```python
class PhysicalDimensions(CompatibleBaseModel):
    """Height and width of the active area of the camera sensor in millimeters"""
    height: Annotated[float, Field(ge=0.0, strict=True)]
    width: Annotated[float, Field(ge=0.0, strict=True)]
```

**技术规格**:
- **类型**: PhysicalDimensions
- **单位**: 毫米 (MILLIMETER)
- **约束**: height, width ≥ 0.0
- **用途**: 定义相机传感器活动区域的物理尺寸

**常见传感器尺寸**:
- **全画幅**: 36×24 mm
- **APS-C**: 23.6×15.6 mm  
- **Micro 4/3**: 17.3×13.0 mm
- **Super 35**: 24.89×18.66 mm

### 2.3 activeSensorResolution (传感器分辨率)

```json
"activeSensorResolution": {
  "height": 2160,
  "width": 3840
}
```

**源码位置**: `camdkit/camera_types.py:44-53`

```python
class SenselDimensions(CompatibleBaseModel):
    """Photosite resolution of the active area of the camera sensor in pixels"""
    height: Annotated[int, Field(ge=0, le=MAX_INT_32)]
    width: Annotated[int, Field(ge=0, le=MAX_INT_32)]
```

**技术规格**:
- **类型**: SenselDimensions
- **单位**: 像素 (PIXEL)
- **约束**: height, width ∈ [0..2,147,483,647]
- **用途**: 定义传感器像素分辨率

**常见分辨率**:
- **4K UHD**: 3840×2160
- **HD**: 1920×1080
- **2K**: 2048×1080
- **8K**: 7680×4320

### 2.4 设备标识信息

#### make (制造商)
```json
"make": "CameraMaker"
```

**技术规格**:
- **类型**: NonBlankUTF8String
- **约束**: 非空UTF-8字符串，最大1023字符
- **用途**: 相机制造商名称

#### model (型号)
```json
"model": "Model20"
```

**技术规格**:
- **类型**: NonBlankUTF8String
- **约束**: 非空UTF-8字符串，最大1023字符
- **用途**: 相机型号标识

#### serialNumber (序列号)
```json
"serialNumber": "1234567890A"
```

**技术规格**:
- **类型**: NonBlankUTF8String
- **JSON别名**: "serialNumber"
- **约束**: 非空UTF-8字符串，最大1023字符
- **用途**: 唯一标识相机设备

#### firmwareVersion (固件版本)
```json
"firmwareVersion": "1.2.3"
```

**技术规格**:
- **类型**: NonBlankUTF8String
- **JSON别名**: "firmwareVersion"
- **约束**: 非空UTF-8字符串，最大1023字符
- **用途**: 相机固件版本标识

#### label (标签)
```json
"label": "A"
```

**技术规格**:
- **类型**: NonBlankUTF8String
- **约束**: 非空UTF-8字符串，最大1023字符
- **用途**: 用户自定义的相机标识符

### 2.5 anamorphicSqueeze (变形压缩比)

```json
"anamorphicSqueeze": {
  "num": 2,
  "denom": 1
}
```

**技术规格**:
- **类型**: StrictlyPositiveRational
- **JSON别名**: "anamorphicSqueeze"
- **约束**: 严格正有理数
- **用途**: 轴对齐正方形的高宽比，用于去压缩变形镜头图像

**常见压缩比**:
- **1.33:1** (4:3) → `{"num": 4, "denom": 3}`
- **2:1** → `{"num": 2, "denom": 1}`
- **1:1** (无变形) → `{"num": 1, "denom": 1}`

### 2.6 isoSpeed (ISO感光度)

```json
"isoSpeed": 4000
```

**源码位置**: `camdkit/camera_types.py:118-122`

```python
iso: Annotated[StrictlyPositiveInt | None,
  Field(alias="isoSpeed",
        json_schema_extra={"clip_property": "iso",
                           "constraints": STRICTLY_POSITIVE_INTEGER})] = None
"""Arithmetic ISO scale as defined in ISO 12232"""
```

**技术规格**:
- **类型**: StrictlyPositiveInt
- **JSON别名**: "isoSpeed"
- **约束**: [1..4,294,967,295]
- **标准**: ISO 12232算术ISO标度

**常见ISO值**: 100, 200, 400, 800, 1600, 3200, 6400, 12800, 25600

### 2.7 fdlLink (FDL链接)

```json
"fdlLink": "urn:uuid:62ea03ac-ce56-43c6-ab8d-a9ec8a9ee7b3"
```

**技术规格**:
- **类型**: UUIDURN
- **JSON别名**: "fdlLink"
- **约束**: UUID URN格式
- **用途**: 标识相机使用的ASC框架决策列表

### 2.8 shutterAngle (快门角度)

```json
"shutterAngle": 45.0
```

**源码位置**: `camdkit/camera_types.py:56, 130-138`

```python
ShutterAngle = Annotated[float, Field(ge=0.0, le=360.0, strict=True)]

shutter_angle: Annotated[ShutterAngle | None,
Field(alias="shutterAngle",
      json_schema_extra={"units": DEGREE,
                         "clip_property": "shutter_angle",
                         "constraints": "The parameter shall be a real number in the range (0..360]."})] = None
"""Shutter speed as a fraction of the capture frame rate."""
```

**技术规格**:
- **类型**: float
- **单位**: 度 (DEGREE)
- **约束**: (0..360]
- **计算**: 快门速度(1/s) = (快门角度/360) × 捕获帧率

**常见快门角度**:
- **180°**: 标准电影快门，1/(2×帧率)
- **90°**: 1/(4×帧率)，运动较清晰
- **270°**: 1/(1.33×帧率)，运动较模糊
- **360°**: 1/帧率，最大运动模糊

---

## 3. Lens 静态参数

**源码位置**: `camdkit/lens_types.py:27-86`

### 3.1 distortionOverscanMax (畸变过扫描最大值)

```json
"distortionOverscanMax": 1.2
```

**源码位置**: `camdkit/lens_types.py:28-36`

```python
distortion_overscan_max: Annotated[UnityOrGreaterFloat | None,
  Field(alias="distortionOverscanMax",
        json_schema_extra={"clip_property": "lens_distortion_overscan_max",
                           "constraints": REAL_AT_LEAST_UNITY})] = None
```

**技术规格**:
- **类型**: UnityOrGreaterFloat
- **约束**: ≥ 1.0
- **用途**: 镜头畸变的静态最大过扫描因子
- **说明**: 替代每帧提供动态过扫描值

### 3.2 undistortionOverscanMax (去畸变过扫描最大值)

```json
"undistortionOverscanMax": 1.3
```

**技术规格**:
- **类型**: UnityOrGreaterFloat
- **约束**: ≥ 1.0
- **用途**: 镜头去畸变的静态最大过扫描因子

### 3.3 设备标识信息

#### make (制造商)
```json
"make": "LensMaker"
```

#### model (型号)
```json
"model": "Model15"
```

#### serialNumber (序列号)
```json
"serialNumber": "1234567890A"
```

#### firmwareVersion (固件版本)
```json
"firmwareVersion": "1.0.0"
```

**技术规格**: 与相机设备标识信息相同，均为NonBlankUTF8String类型

### 3.4 nominalFocalLength (标称焦距)

```json
"nominalFocalLength": 14.0
```

**源码位置**: `camdkit/lens_types.py:70-77`

```python
nominal_focal_length: Annotated[StrictlyPositiveFloat | None,
  Field(alias="nominalFocalLength",
        json_schema_extra={"units": MILLIMETER,
                           "clip_property": "lens_nominal_focal_length",
                           "constraints": STRICTLY_POSITIVE_REAL})] = None
"""Nominal focal length of the lens. The number printed on the side
of a prime lens, e.g. 50 mm, and undefined in the case of a zoom lens."""
```

**技术规格**:
- **类型**: StrictlyPositiveFloat
- **单位**: 毫米 (MILLIMETER)
- **约束**: > 0.0
- **用途**: 定焦镜头侧面印制的焦距值，变焦镜头时未定义

**常见焦距**:
- **超广角**: 8-24mm
- **广角**: 24-35mm
- **标准**: 35-85mm
- **长焦**: 85-300mm
- **超长焦**: >300mm

### 3.5 calibrationHistory (校准历史)

```json
"calibrationHistory": [
  "LensMaker 123",
  "TrackerMaker 123"
]
```

**源码位置**: `camdkit/lens_types.py:78-85`

```python
calibration_history: Annotated[tuple[str, ...] | None,
  Field(alias="calibrationHistory",
        json_schema_extra={"clip_property": "lens_calibration_history",
                           "type": "array",
                           "items":
                           { "type": "string", "minLength": 1, "maxLength": 1023 }
                           })] = None
"""List of free strings that describe the history of calibrations of the lens."""
```

**技术规格**:
- **类型**: tuple[str, ...]
- **约束**: 每个字符串长度1-1023字符
- **用途**: 描述镜头校准历史的自由文本列表

---

## 4. Tracker 静态参数

**源码位置**: `camdkit/tracker_types.py:19-40`

### 4.1 设备标识信息

#### make (制造商)
```json
"make": "TrackerMaker"
```

**源码位置**: `camdkit/tracker_types.py:20-23`

```python
make: Annotated[NonBlankUTF8String | None,
  Field(json_schema_extra={"clip_property": "tracker_make",
                           "constraints": NONBLANK_UTF8_MAX_1023_CHARS})] = None
"""Non-blank string naming tracking device manufacturer"""
```

#### model (型号)
```json
"model": "Tracker"
```

**源码位置**: `camdkit/tracker_types.py:25-28`

```python
model: Annotated[NonBlankUTF8String | None,
  Field(json_schema_extra={"clip_property": "tracker_model",
                           "constraints": NONBLANK_UTF8_MAX_1023_CHARS})] = None
"""Non-blank string identifying tracking device model"""
```

#### serialNumber (序列号)
```json
"serialNumber": "1234567890A"
```

**源码位置**: `camdkit/tracker_types.py:30-34`

```python
serial_number: Annotated[NonBlankUTF8String | None,
  Field(alias="serialNumber",
        json_schema_extra={"clip_property": "tracker_serial_number",
                           "constraints": NONBLANK_UTF8_MAX_1023_CHARS})] = None
"""Non-blank string uniquely identifying the tracking device"""
```

#### firmwareVersion (固件版本)
```json
"firmwareVersion": "1.2.3"
```

**源码位置**: `camdkit/tracker_types.py:36-40`

```python
firmware: Annotated[NonBlankUTF8String | None,
  Field(alias="firmwareVersion",
        json_schema_extra={"clip_property": "tracker_firmware",
                           "constraints": NONBLANK_UTF8_MAX_1023_CHARS})] = None
"""Non-blank string identifying tracking device firmware version"""
```

**技术规格**: 所有标识信息均为NonBlankUTF8String类型，约束为非空UTF-8字符串，最大1023字符

---

## 5. 数据验证机制

### 5.1 自动转换验证器

**源码位置**: `camdkit/camera_types.py:141-144`

```python
@field_validator("capture_frame_rate", "anamorphic_squeeze", mode="before")
@classmethod
def coerce_camera_type_to_strictly_positive_rational(cls, v):
    return rationalize_strictly_and_positively(v)
```

**支持的转换**:
1. **整数** → StrictlyPositiveRational: `24` → `{"num": 24, "denom": 1}`
2. **Python Fraction** → StrictlyPositiveRational: `Fraction(24000, 1001)` → `{"num": 24000, "denom": 1001}`
3. **字典** → StrictlyPositiveRational: `{"num": 24000, "denom": 1001}` → 直接使用

### 5.2 约束验证

所有参数都有严格的类型和数值约束：

| 约束类型 | 说明 | 示例 |
|----------|------|------|
| StrictlyPositiveRational | 严格正有理数 | num≥1, denom≥1 |
| StrictlyPositiveInt | 严格正整数 | [1..4,294,967,295] |
| StrictlyPositiveFloat | 严格正浮点数 | > 0.0 |
| UnityOrGreaterFloat | ≥1.0的浮点数 | ≥ 1.0 |
| NonBlankUTF8String | 非空UTF-8字符串 | 长度1-1023字符 |

---

## 6. 实际应用示例

### 6.1 完整的Static配置

```json
{
  "static": {
    "duration": {
      "num": 1,
      "denom": 25
    },
    "camera": {
      "captureFrameRate": {
        "num": 24000,
        "denom": 1001
      },
      "activeSensorPhysicalDimensions": {
        "height": 24.0,
        "width": 36.0
      },
      "activeSensorResolution": {
        "height": 2160,
        "width": 3840
      },
      "make": "CameraMaker",
      "model": "Model20",
      "serialNumber": "1234567890A",
      "firmwareVersion": "1.2.3",
      "label": "A",
      "anamorphicSqueeze": {
        "num": 1,
        "denom": 1
      },
      "isoSpeed": 4000,
      "fdlLink": "urn:uuid:62ea03ac-ce56-43c6-ab8d-a9ec8a9ee7b3",
      "shutterAngle": 45.0
    },
    "lens": {
      "distortionOverscanMax": 1.2,
      "undistortionOverscanMax": 1.3,
      "make": "LensMaker",
      "model": "Model15",
      "serialNumber": "1234567890A",
      "nominalFocalLength": 14.0,
      "calibrationHistory": [
        "LensMaker 123",
        "TrackerMaker 123"
      ]
    },
    "tracker": {
      "make": "TrackerMaker",
      "model": "Tracker",
      "serialNumber": "1234567890A",
      "firmwareVersion": "1.2.3"
    }
  }
}
```

### 6.2 计算关系示例

**总帧数计算**:
```
Duration = 1/25 = 0.04秒
captureFrameRate = 24000/1001 ≈ 23.976 fps
总帧数 = 0.04 × 23.976 ≈ 0.959帧
```

**快门速度计算**:
```
shutterAngle = 45°
captureFrameRate = 23.976 fps
快门速度 = (45/360) × 23.976 = 2.997 fps = 1/2.997秒 ≈ 1/3秒
```

---

## 7. 最佳实践建议

### 7.1 参数一致性

1. **设备标识统一**: 确保make, model, serialNumber在所有相关设备中保持一致的命名规范
2. **帧率匹配**: captureFrameRate应与timing.sampleRate和timecode.frameRate保持一致
3. **物理参数合理**: 传感器物理尺寸应与分辨率形成合理的像素密度

### 7.2 数据完整性

1. **必要参数**: 虽然大部分参数为可选，但建议至少提供duration和captureFrameRate
2. **校准信息**: 对于精密应用，应提供完整的calibrationHistory
3. **设备追踪**: 通过serialNumber确保设备的唯一性和可追踪性

### 7.3 性能优化

1. **有理数精度**: 使用标准分数值避免精度损失
2. **字符串长度**: 控制标识字符串长度，避免过长的描述
3. **可选参数**: 只提供必要的参数，减少数据传输量

---

## 总结

OpenTrackIO的static部分提供了完整的静态设备和配置信息框架。通过严格的类型约束和验证机制，确保了数据的准确性和一致性。正确配置这些参数对于虚拟制作流程中的设备管理、时间同步和数据质量控制至关重要。

在实际应用中，应根据具体的制作需求和设备能力，合理选择和配置相关参数，确保整个虚拟制作流程的稳定性和可靠性。