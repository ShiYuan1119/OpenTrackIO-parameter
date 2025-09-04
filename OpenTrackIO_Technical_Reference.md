# OpenTrackIO 技术参考手册

## JSON Schema 结构分析

基于 SMPTE CamDKit 源码的详细技术规范

---

## 数据类型定义

### 基础类型

#### StrictlyPositiveRational (严格正有理数)
```python
class StrictlyPositiveRational:
    num: int  # 分子，必须 > 0
    denom: int  # 分母，必须 > 0
```

**用途**: 帧率、持续时间、采样率  
**验证**: num > 0 且 denom > 0

#### NormalizedFloat (归一化浮点数)
```python
NormalizedFloat = Annotated[float, Field(ge=0.0, le=1.0)]
```

**范围**: [0.0, 1.0]  
**用途**: 编码器位置、透明度等

#### NonNegativeInt (非负整数)
```python
NonNegativeInt = Annotated[int, Field(ge=0)]
```

**范围**: [0, ∞)  
**用途**: 序列号、计数器等

---

## 核心类结构

### Clip 主类

```python
class Clip(CompatibleBaseModel):
    static: Static = Static()
    tracker: Tracker = Tracker()
    timing: Timing = Timing()
    lens: Lens = Lens()
    # ... 其他字段
```

### Static 静态信息类

```python
class Static(CompatibleBaseModel):
    duration: StrictlyPositiveRational | None = None
    camera: StaticCamera = StaticCamera()
    lens: StaticLens = StaticLens()
    tracker: StaticTracker = StaticTracker()
```

---

## 详细参数规范

### Camera 相机参数

#### PhysicalDimensions (物理尺寸)
```python
class PhysicalDimensions(CompatibleBaseModel):
    height: Annotated[float, Field(ge=0.0, strict=True)]
    width: Annotated[float, Field(ge=0.0, strict=True)]
```

**约束**: 高度和宽度必须 ≥ 0.0  
**单位**: 毫米  
**精度**: 浮点数

#### SenselDimensions (传感器分辨率)
```python
class SenselDimensions(CompatibleBaseModel):
    height: Annotated[int, Field(ge=0, le=MAX_INT_32)]
    width: Annotated[int, Field(ge=0, le=MAX_INT_32)]
```

**约束**: [0, 2,147,483,647]  
**单位**: 像素  
**类型**: 32 位整数

#### ShutterAngle (快门角度)
```python
ShutterAngle = Annotated[float, Field(ge=0.0, le=360.0, strict=True)]
```

**范围**: (0.0, 360.0]  
**单位**: 度  
**计算**: 快门速度 = 角度 ÷ 360 × 帧率

### Lens 镜头参数

#### Distortion (畸变模型)
```python
class Distortion(CompatibleBaseModel):
    model: NonBlankUTF8String | None = "Brown-Conrady D-U"
    radial: tuple[float, ...] = Field(min_length=1)
    tangential: tuple[float, ...] | None = Field(min_length=1)
    overscan: UnityOrGreaterFloat | None = None
```

**模型类型**:
- `"Brown-Conrady D-U"`: 畸变→未畸变（默认）
- `"Brown-Conrady U-D"`: 未畸变→畸变

**径向系数**: k1, k2, k3, ... (最少1个)  
**切向系数**: p1, p2, ... (可选，最少1个)  
**过扫描**: ≥ 1.0

#### FizEncoders (FIZ 编码器)
```python
class FizEncoders(CompatibleBaseModel):
    focus: NormalizedFloat | None = None
    iris: NormalizedFloat | None = None
    zoom: NormalizedFloat | None = None
```

**约束**: 至少包含一个非空值  
**范围**: [0.0, 1.0]  
**方向约定**:
- Focus: 0=∞, 1=最近
- Iris: 0=开放, 1=关闭  
- Zoom: 0=广角, 1=长焦

#### RawFizEncoders (原始编码器)
```python
class RawFizEncoders(CompatibleBaseModel):
    focus: NonNegativeInt | None = None
    iris: NonNegativeInt | None = None
    zoom: NonNegativeInt | None = None
```

**约束**: 至少包含一个非空值  
**类型**: 非负整数  
**说明**: 依赖编码器分辨率的原始值

### Timing 时间参数

#### TimingMode (时间模式)
```python
class TimingMode(StrEnum):
    INTERNAL = "internal"
    EXTERNAL = "external"
```

#### Timestamp (时间戳)
```python
class Timestamp(CompatibleBaseModel):
    seconds: NonNegative48BitInt
    nanoseconds: NonNegativeInt
```

**秒字段**: 48 位无符号整数  
**纳秒字段**: 32 位无符号整数  
**基准**: Unix 纪元

#### Timecode (时间码)
```python
class Timecode(CompatibleBaseModel):
    hours: int = Field(ge=0, le=23)
    minutes: int = Field(ge=0, le=59)
    seconds: int = Field(ge=0, le=59)
    frames: int = Field(ge=0, le=119)
    frame_rate: StrictlyPositiveRational
    sub_frame: NonNegativeInt = 0
    dropFrame: bool | None = False
```

**验证规则**: frames < frameRate  
**子帧**: 支持隔行扫描等应用

### Transform 变换参数

#### Vector3 (3D 向量)
```python
class Vector3(CompatibleBaseModel):
    x: float | None = None
    y: float | None = None
    z: float | None = None
```

#### Rotator3 (3D 旋转)
```python
class Rotator3(CompatibleBaseModel):
    pan: float | None = None
    tilt: float | None = None
    roll: float | None = None
```

**旋转顺序**: ZXY 内在旋转  
**单位**: 度

---

## 数据验证规则

### 字段验证

1. **非空字符串**: 1-1023 字符，UTF-8 编码
2. **UUID 格式**: "urn:uuid:{uuid}" 格式
3. **数值范围**: 各类型的特定范围约束
4. **数组长度**: 最小长度要求

### 模型验证

1. **FIZ 编码器**: 至少一个非空值
2. **时间码**: 帧数必须小于帧率
3. **畸变系数**: 径向系数数组不能为空

### 兼容性验证

1. **协议版本**: 检查版本兼容性
2. **字段存在性**: 处理可选字段
3. **向后兼容**: 忽略未知字段

---

## 网络传输考虑

### 数据大小优化

1. **可选字段**: 仅传输必要数据
2. **精度平衡**: 根据应用需求选择精度
3. **压缩**: JSON 压缩传输

### 实时性要求

1. **低延迟**: 最小化处理延迟
2. **缓冲策略**: 处理网络抖动
3. **丢包处理**: 容错机制

---

## 坐标系统详解

### 舞台坐标系

- **原点**: 舞台中心或参考点
- **X 轴**: 舞台右侧（面向前方时）
- **Y 轴**: 舞台前方
- **Z 轴**: 舞台上方
- **单位**: 米

### 相机坐标系

- **原点**: 相机传感器中心
- **X 轴**: 相机右侧
- **Y 轴**: 相机前方（镜头朝向）
- **Z 轴**: 相机上方

### 全局坐标系

- **ENU**: 东-北-上坐标系
- **大地坐标**: WGS84 椭球体
- **参考**: 本地切平面坐标

---

## 镜头畸变模型

### Brown-Conrady 模型

**径向畸变**:
```
r' = r × (1 + k1×r² + k2×r⁴ + k3×r⁶ + ...)
```

**切向畸变**:
```
x' = x + [2×p1×x×y + p2×(r² + 2×x²)]
y' = y + [p1×(r² + 2×y²) + 2×p2×x×y]
```

其中:
- r = √(x² + y²)
- k1, k2, k3, ... 为径向畸变系数
- p1, p2, ... 为切向畸变系数

### 过扫描计算

过扫描系数用于确定渲染时需要的额外图像区域，以补偿去畸变过程中的像素损失。

---

## PTP 同步详解

### PTP 配置文件

1. **IEEE Std 1588-2019**: 通用工业应用
2. **IEEE Std 802.1AS-2020**: 音视频桥接网络
3. **SMPTE ST2059-2:2021**: SMPTE 2110 系统

### 最佳主时钟算法 (BMCA)

优先级顺序:
1. Priority1 (管理员设置)
2. Clock Class (时钟质量)
3. Clock Accuracy (时钟精度)
4. Clock Variance (时钟方差)
5. Priority2 (动态优先级)
6. Clock Identity (时钟标识)

### 延迟测量

**meanPathDelay**: 双向延迟测量  
**计算**: (t4 - t1 - t3 + t2) / 2  
**用途**: 时间同步精度补偿

---

## 扩展机制

### Custom 字段

```json
"custom": {
  "vendorSpecific": "value",
  "experimentalFeature": 123,
  "metadata": {
    "nested": "object"
  }
}
```

**用途**: 厂商特定扩展  
**约束**: 有效 JSON 值  
**建议**: 使用命名空间避免冲突

### Lens Custom 系数

```json
"lens": {
  "custom": [
    [1.0, 2.0, 3.0],  # 第一组自定义系数
    [4.0, 5.0, 6.0]   # 第二组自定义系数
  ]
}
```

**用途**: 扩展镜头模型  
**协商**: 制作方与消费方协商含义

---

## 实现注意事项

### 数据精度

1. **浮点精度**: 使用双精度浮点数
2. **有理数**: 避免浮点精度问题
3. **时间精度**: 纳秒级时间戳

### 内存管理

1. **大数组**: 注意内存使用
2. **缓存策略**: 静态数据缓存
3. **垃圾回收**: 及时释放资源

### 线程安全

1. **并发访问**: 保护共享数据
2. **原子操作**: 关键状态更新
3. **锁策略**: 避免死锁

---

## 测试用例

### 基本验证

```python
def test_basic_opentrackio():
    clip = Clip()
    clip.protocol = [VersionedProtocol(name="OpenTrackIO", version=[1,0,0])]
    clip.sampleId = ["urn:uuid:12345678-1234-5678-9abc-123456789abc"]
    assert clip.model_validate(clip.model_dump())
```

### 边界条件

```python
def test_boundary_conditions():
    # 测试最小值
    lens = Lens()
    lens.fStop = [0.1]  # 最小光圈值
    
    # 测试最大值
    camera = StaticCamera()
    camera.activeSensorResolution = SenselDimensions(2147483647, 2147483647)
```

### 错误处理

```python
def test_validation_errors():
    with pytest.raises(ValueError):
        # 负数帧率应该失败
        Timing(sampleRate=[StrictlyPositiveRational(num=-1, denom=1)])
```

---

## 性能基准

### 解析性能

| 数据大小 | 解析时间 | 内存使用 |
|---------|---------|---------|
| 1KB     | <1ms    | <10KB   |
| 10KB    | <5ms    | <50KB   |
| 100KB   | <50ms   | <500KB  |

### 网络传输

| 采样率 | 数据率 | 带宽需求 |
|-------|-------|---------|
| 24fps | ~2KB/s | <20Kbps |
| 60fps | ~5KB/s | <50Kbps |
| 120fps| ~10KB/s| <100Kbps|

---

## 调试指南

### 常见问题

1. **时间同步失败**
   - 检查 PTP 配置
   - 验证网络延迟
   - 确认时钟源

2. **畸变参数错误**
   - 验证系数数量
   - 检查模型类型
   - 确认过扫描值

3. **变换链错误**
   - 验证变换顺序
   - 检查坐标系统
   - 确认单位一致性

### 调试工具

```python
def debug_clip(clip: Clip):
    """调试 OpenTrackIO 数据的辅助函数"""
    clip._print_non_none()  # 打印所有非空字段
    
    # 验证数据完整性
    try:
        clip.model_validate(clip.model_dump())
        print("✓ 数据验证通过")
    except Exception as e:
        print(f"✗ 验证失败: {e}")
```

---

## 版本兼容性

### 版本号格式

```json
"version": [major, minor, patch]
```

### 兼容性规则

1. **主版本**: 不兼容更改
2. **次版本**: 向后兼容的新功能
3. **修订版本**: 向后兼容的错误修复

### 迁移策略

```python
def migrate_version(data: dict, from_version: list, to_version: list):
    """版本迁移辅助函数"""
    if from_version[0] != to_version[0]:
        raise ValueError("主版本不兼容")
    
    # 处理次版本和修订版本的兼容性
    return data
```

---

## 集成示例

### Python 集成

```python
import json
from camdkit.clip import Clip

# 解析 OpenTrackIO 数据
def parse_opentrackio(json_data: str) -> Clip:
    data = json.loads(json_data)
    return Clip.model_validate(data)

# 生成 OpenTrackIO 数据
def generate_opentrackio(clip: Clip) -> str:
    return json.dumps(clip.model_dump(exclude_none=True), indent=2)
```

### C++ 集成

```cpp
#include "OpenTrackIOParser.h"

// 解析示例
std::string json_data = "...";
auto clip = OpenTrackIOParser::parse(json_data);

// 访问参数
if (clip.has_lens() && clip.lens().has_f_stop()) {
    double fstop = clip.lens().f_stop();
    std::cout << "F-Stop: " << fstop << std::endl;
}
```

---

## 安全考虑

### 输入验证

1. **JSON 解析**: 防止恶意 JSON
2. **数值范围**: 验证所有数值约束
3. **字符串长度**: 防止缓冲区溢出

### 网络安全

1. **加密传输**: 使用 TLS/SSL
2. **身份验证**: 验证数据源
3. **完整性检查**: 数据签名验证

---

## 扩展开发

### 添加新参数

1. **定义类型**: 在相应的 types 文件中添加
2. **更新模型**: 修改主要类定义
3. **添加验证**: 实现约束检查
4. **更新文档**: 补充参数说明

### 自定义畸变模型

```python
class CustomDistortion(Distortion):
    model: str = "Custom-Model"
    custom_coefficients: tuple[float, ...] = None
    
    @model_validator(mode="after")
    def validate_custom_coefficients(self):
        if self.model == "Custom-Model" and not self.custom_coefficients:
            raise ValueError("自定义模型需要提供系数")
        return self
```

---

## 术语表

- **FIZ**: Focus, Iris, Zoom（对焦、光圈、变焦）
- **PTP**: Precision Time Protocol（精确时间协议）
- **ENU**: East-North-Up（东-北-上坐标系）
- **URN**: Uniform Resource Name（统一资源名称）
- **UUID**: Universally Unique Identifier（通用唯一标识符）
- **BMCA**: Best Master Clock Algorithm（最佳主时钟算法）
- **GNSS**: Global Navigation Satellite System（全球导航卫星系统）

---

*技术参考手册版本: 1.0.0*  
*基于 SMPTE RiS-OSVP Metadata CamDKit 源码分析*