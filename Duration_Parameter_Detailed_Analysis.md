# Duration 参数详细解析

基于 SMPTE RiS-OSVP Metadata CamDKit 源码分析

---

## 1. Duration 参数概述

### 1.1 基本定义

`Duration` 参数表示 OpenTrackIO 数据片段的持续时间长度，是一个**严格正有理数**（StrictlyPositiveRational）。

```json
"duration": {
  "num": 1,
  "denom": 25
}
```

### 1.2 源码定义

**文件位置**: `camdkit/clip.py:42-46`

```python
duration: Annotated[StrictlyPositiveRational | None,
  Field(json_schema_extra={"clip_property": "duration",
                           "constraints": STRICTLY_POSITIVE_RATIONAL,
                           "units": SECOND})] = None
"""Duration of the clip"""
```

### 1.3 数据类型详解

#### StrictlyPositiveRational 类结构

**文件位置**: `camdkit/numeric_types.py:92-122`

```python
class StrictlyPositiveRational(CompatibleBaseModel):
    num: int = Field(ge=1, le=MAX_INT_32, strict=True)      # 分子
    denom: int = Field(ge=1, le=MAX_UINT_32, strict=True)   # 分母
```

**约束条件**:
- `num` (分子): 范围 [1..2,147,483,647]，必须为正整数
- `denom` (分母): 范围 [1..4,294,967,295]，必须为正整数
- 两者都必须严格为正数（≥1）

---

## 2. Duration 与 captureFrameRate 的关系分析

### 2.1 captureFrameRate 参数定义

**文件位置**: `camdkit/camera_types.py:60-65`

```python
capture_frame_rate: Annotated[StrictlyPositiveRational | None,
  Field(alias="captureFrameRate",
      json_schema_extra={"units": HERTZ,
                         "clip_property": "capture_frame_rate",
                         "constraints": STRICTLY_POSITIVE_RATIONAL})] = None
"""Capture frame rate of the camera"""
```

### 2.2 常见帧率设定值

**文件位置**: `camdkit/utils.py:12`

```python
_WELL_KNOWN_FRACTIONAL_FPS = set([
    Fraction(24000, 1001),   # 23.976 fps (23.98 fps)
    Fraction(30000, 1001),   # 29.970 fps (29.97 fps)  
    Fraction(60000, 1001),   # 59.940 fps (59.94 fps)
    Fraction(120000, 1001)   # 119.880 fps (119.88 fps)
])
```

### 2.3 Duration 计算公式

```
Duration (秒) = num / denom
Frame Count = Duration × captureFrameRate
```

### 2.4 实际应用示例

根据提供的 captureFrameRate 设定值表：

| FPS | num | denom | Duration (秒) | Frame Count |
|-----|-----|-------|---------------|-------------|
| 60 | 60000 | 1000 | 60.0 | 3600 |
| 59.94 | 60000 | 1001 | 59.94 | 3596.4 |
| 50 | 50000 | 1000 | 50.0 | 2500 |
| 29.97 | 30000 | 1001 | 29.97 | 899.1 |
| 25 | 25000 | 1000 | 25.0 | 625 |
| 24 | 24000 | 1000 | 24.0 | 576 |
| 23.98 | 24000 | 1001 | 23.976 | 575.424 |

**计算示例**:
- Duration: `{"num": 1, "denom": 25}` = 0.04 秒
- captureFrameRate: `{"num": 24000, "denom": 1001}` = 23.976 fps
- Frame Count = 0.04 × 23.976 ≈ 0.959 帧

---

## 3. 源码实现机制

### 3.1 数据验证器

**文件位置**: `camdkit/clip.py:49-52`

```python
@field_validator("duration", mode="before")
@classmethod
def coerce_duration_to_strictly_positive_rational(cls, v):
    return rationalize_strictly_and_positively(v)
```

### 3.2 有理数转换函数

**文件位置**: `camdkit/numeric_types.py:123-133`

```python
def rationalize_strictly_and_positively(x: Any) -> StrictlyPositiveRational:
    if x:
        if not isinstance(x, StrictlyPositiveRational):
            if isinstance(x, int) and x <= MAX_INT_32:
                return StrictlyPositiveRational(x, 1)
            elif isinstance(x, numbers.Rational):
                return StrictlyPositiveRational(int(x.numerator), int(x.denominator))
            elif isinstance(x, dict) and len(x) == 2 and "num" in x and "denom" in x:
                return StrictlyPositiveRational(int(x["num"]), int(x["denom"]))
            raise ValueError(f"could not convert input of type {type(x)} to a StrictlyPositiveRational")
    return x
```

**支持的输入格式**:
1. 整数: `25` → `{"num": 25, "denom": 1}`
2. Python Fraction: `Fraction(1, 25)` → `{"num": 1, "denom": 25}`
3. 字典格式: `{"num": 1, "denom": 25}` → 直接使用

---

## 4. 实际使用场景

### 4.1 静态片段示例

**文件位置**: `examples/complete_static_example.json:3-6`

```json
"static": {
  "duration": {
    "num": 1,
    "denom": 25
  }
}
```

**解释**: 
- 片段持续时间: 1/25 = 0.04 秒
- 如果 captureFrameRate 为 25 fps，则包含 1 帧数据
- 如果 captureFrameRate 为 24000/1001 fps (≈23.976)，则包含约 0.959 帧数据

### 4.2 代码生成示例

**文件位置**: `camdkit/examples.py:107`

```python
clip.duration = Fraction(1,25)
```

这会自动转换为:
```json
{
  "num": 1,
  "denom": 25
}
```

---

## 5. 技术约束与限制

### 5.1 数值范围限制

- **分子 (num)**: 1 ≤ num ≤ 2,147,483,647 (32位有符号整数最大值)
- **分母 (denom)**: 1 ≤ denom ≤ 4,294,967,295 (32位无符号整数最大值)
- **最小值**: 1/4,294,967,295 ≈ 2.33 × 10⁻¹⁰ 秒
- **最大值**: 2,147,483,647/1 = 2,147,483,647 秒 ≈ 68 年

### 5.2 精度考虑

使用有理数表示可以：
- **精确表示**常见的分数时间值 (如 1/24, 1/25, 1/30)
- **避免浮点数**精度误差
- **支持复杂分数**如 24000/1001 这样的 NTSC 标准帧率

### 5.3 性能优化

**文件位置**: `camdkit/numeric_types.py:115-121`

```python
def __mul__(self, other: Any):
    wrapped = StrictlyPositiveRational._canonicalize(other)
    return StrictlyPositiveRational(self.num * wrapped.num, self.denom * wrapped.denom)

def __rtruediv__(self, other: Any):
    wrapped = StrictlyPositiveRational._canonicalize(other)
    return StrictlyPositiveRational(self.num * wrapped.denom, self.denom * wrapped.num)
```

支持基本的数学运算，便于时间计算。

---

## 6. 最佳实践建议

### 6.1 Duration 设定原则

1. **与帧率匹配**: Duration 应该是 captureFrameRate 的整数倍，以确保包含完整帧数
2. **使用标准值**: 优先使用常见的分数值 (1/24, 1/25, 1/30 等)
3. **避免过大分母**: 虽然支持大分母，但应避免不必要的复杂分数

### 6.2 常见 Duration 设定

| 用途 | Duration | 说明 |
|------|----------|------|
| 单帧 | `{"num": 1, "denom": 25}` | 25fps 下的单帧 |
| 1秒钟 | `{"num": 1, "denom": 1}` | 标准1秒 |
| 半秒 | `{"num": 1, "denom": 2}` | 0.5秒 |
| NTSC帧 | `{"num": 1001, "denom": 30000}` | 29.97fps 下的单帧 |

### 6.3 错误处理

常见错误及解决方案：

```python
# 错误：零值或负值
{"num": 0, "denom": 1}     # 抛出验证错误
{"num": -1, "denom": 1}    # 抛出验证错误

# 错误：超出范围
{"num": 2147483648, "denom": 1}  # 超出32位整数范围

# 正确：标准格式
{"num": 1, "denom": 25}    # ✓ 有效
```

---

## 7. 与其他参数的协作

### 7.1 与 Timing 的关系

Duration 定义了整个片段的时长，而 `timing.sampleRate` 定义了采样频率：

```json
{
  "static": {
    "duration": {"num": 1, "denom": 1}  // 1秒片段
  },
  "timing": {
    "sampleRate": {"num": 24000, "denom": 1001}  // 23.976 fps采样
  }
}
```

**总采样点数** = Duration × sampleRate = 1 × (24000/1001) ≈ 23.976 个采样点

### 7.2 与 Transform 数组的关系

Duration 决定了 transforms 数组应包含多少个变换数据点：

```python
expected_transform_count = duration * sample_rate
```

这确保了时间轴上的数据完整性和一致性。

---

## 总结

Duration 参数作为 OpenTrackIO 协议的核心时间参数，通过严格的有理数表示确保了时间精度和数据一致性。其设计充分考虑了影视制作中各种帧率标准的需求，特别是 NTSC 等复杂分数帧率的精确表示。正确理解和使用 Duration 参数对于确保虚拟制作流程中的时间同步至关重要。