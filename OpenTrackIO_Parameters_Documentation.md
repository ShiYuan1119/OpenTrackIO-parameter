# OpenTrackIO 参数详细文档

基于 SMPTE RiS-OSVP Metadata CamDKit 官方规范

## 概述

OpenTrackIO 是 SMPTE RiS-OSVP 工作组开发的免费开源协议，专为虚拟制作（Virtual Production）环境中的元数据交换而设计。该协议定义了包含相机、镜头、跟踪器和变换信息的 JSON 数据结构。

**协议版本**: v1.0.0  
**规范来源**: [SMPTE RiS-OSVP Metadata CamDKit](https://github.com/SMPTE/ris-osvp-metadata-camdkit)

---

## 数据结构总览

OpenTrackIO 数据包含以下主要部分：

- **`static`**: 静态信息（持续时间、相机、镜头、跟踪器的固定属性）
- **`tracker`**: 跟踪器状态信息
- **`timing`**: 时间同步信息
- **`lens`**: 镜头动态参数
- **`protocol`**: 协议标识信息
- **`transforms`**: 空间变换信息
- **`globalStage`**: 全局舞台坐标
- **`custom`**: 自定义扩展字段

---

## 1. Static 静态信息

### 1.1 Duration (持续时间)

```json
"duration": {
  "num": 1,
  "denom": 25
}
```

**描述**: 片段的持续时间  
**类型**: 严格正有理数  
**单位**: 秒  
**约束**: 必须为正数  
**采样**: 静态

### 1.2 Camera 相机静态信息

#### 1.2.1 captureFrameRate (捕获帧率)

```json
"captureFrameRate": {
  "num": 24000,
  "denom": 1001
}
```

**描述**: 相机的捕获帧率  
**类型**: 严格正有理数  
**单位**: 赫兹 (Hz)  
**约束**: 必须为正数  
**说明**: 支持精确表示诸如 29.97 fps (30000/1001) 等掉帧帧率

#### 1.2.2 activeSensorPhysicalDimensions (传感器物理尺寸)

```json
"activeSensorPhysicalDimensions": {
  "height": 24.0,
  "width": 36.0
}
```

**描述**: 相机传感器活动区域的物理尺寸  
**类型**: 物理尺寸对象  
**单位**: 毫米  
**约束**: 高度和宽度必须为非负实数  

#### 1.2.3 activeSensorResolution (传感器分辨率)

```json
"activeSensorResolution": {
  "height": 2160,
  "width": 3840
}
```

**描述**: 相机传感器活动区域的像素分辨率  
**类型**: 传感器尺寸对象  
**单位**: 像素  
**约束**: 高度和宽度必须为 [0..2,147,483,647] 范围内的整数

#### 1.2.4 设备标识信息

```json
"make": "CameraMaker",
"model": "Model20",
"serialNumber": "1234567890A",
"firmwareVersion": "1.2.3",
"label": "A"
```

**描述**: 相机设备的标识信息  
**类型**: 非空 UTF-8 字符串  
**约束**: 每个字段最大 1023 字符，不能为空

- **make**: 相机制造商名称
- **model**: 相机型号
- **serialNumber**: 相机序列号（唯一标识）
- **firmwareVersion**: 固件版本
- **label**: 用户定义的相机标识符

#### 1.2.5 anamorphicSqueeze (变形镜头压缩比)

```json
"anamorphicSqueeze": {
  "num": 1,
  "denom": 1
}
```

**描述**: 变形镜头的名义压缩比  
**类型**: 严格正有理数  
**约束**: 必须为正数  
**说明**: 表示轴对齐正方形在相机传感器上成像的高宽比

#### 1.2.6 isoSpeed (ISO 感光度)

```json
"isoSpeed": 4000
```

**描述**: ISO 12232 定义的算术 ISO 感光度  
**类型**: 严格正整数  
**约束**: 必须为正整数

#### 1.2.7 fdlLink (FDL 链接)

```json
"fdlLink": "urn:uuid:62ea03ac-ce56-43c6-ab8d-a9ec8a9ee7b3"
```

**描述**: 标识相机使用的 ASC 帧决策列表的 URN  
**类型**: UUID URN  
**约束**: 必须为有效的 UUID URN 格式

#### 1.2.8 shutterAngle (快门角度)

```json
"shutterAngle": 45.0
```

**描述**: 快门速度，表示为捕获帧率的分数  
**类型**: 实数  
**单位**: 度  
**约束**: 必须在 (0..360] 范围内  
**说明**: 快门速度（1/s）= 参数值 ÷ 360 × 捕获帧率

### 1.3 Lens 镜头静态信息

#### 1.3.1 distortionOverscanMax (畸变过扫描最大值)

```json
"distortionOverscanMax": 1.2
```

**描述**: 镜头畸变的静态最大过扫描系数  
**类型**: 大于等于 1 的实数  
**约束**: 必须 ≥ 1.0  
**说明**: 替代每帧提供动态过扫描值的方案

#### 1.3.2 undistortionOverscanMax (去畸变过扫描最大值)

```json
"undistortionOverscanMax": 1.3
```

**描述**: 镜头去畸变的静态最大过扫描系数  
**类型**: 大于等于 1 的实数  
**约束**: 必须 ≥ 1.0

#### 1.3.3 镜头设备信息

```json
"make": "LensMaker",
"model": "Model15",
"serialNumber": "1234567890A"
```

**描述**: 镜头设备标识信息  
**类型**: 非空 UTF-8 字符串  
**约束**: 每个字段最大 1023 字符

#### 1.3.4 nominalFocalLength (标称焦距)

```json
"nominalFocalLength": 14.0
```

**描述**: 镜头的标称焦距  
**类型**: 严格正实数  
**单位**: 毫米  
**说明**: 定焦镜头侧面标注的数值，变焦镜头则未定义

#### 1.3.5 calibrationHistory (校准历史)

```json
"calibrationHistory": [
  "LensMaker 123",
  "TrackerMaker 123"
]
```

**描述**: 描述镜头校准历史的自由字符串列表  
**类型**: 字符串数组  
**约束**: 每个字符串 1-1023 字符

### 1.4 Tracker 跟踪器静态信息

```json
"tracker": {
  "make": "TrackerMaker",
  "model": "Tracker",
  "serialNumber": "1234567890A",
  "firmwareVersion": "1.2.3"
}
```

**描述**: 跟踪设备的制造商和设备信息  
**类型**: 非空 UTF-8 字符串  
**约束**: 每个字段最大 1023 字符

---

## 2. Tracker 跟踪器状态

```json
"tracker": {
  "notes": "Example generated sample.",
  "recording": false,
  "slate": "A101_A_4",
  "status": "Optical Good"
}
```

### 2.1 notes (备注)

**描述**: 关于跟踪系统的备注信息  
**类型**: 非空 UTF-8 字符串  
**约束**: 最大 1023 字符  
**采样**: 常规

### 2.2 recording (录制状态)

**描述**: 指示跟踪系统是否正在录制数据  
**类型**: 布尔值  
**约束**: true 或 false  
**采样**: 常规

### 2.3 slate (场记板)

**描述**: 描述录制场记板的字符串  
**类型**: 非空 UTF-8 字符串  
**约束**: 最大 1023 字符  
**采样**: 常规

### 2.4 status (状态)

**描述**: 描述跟踪系统状态的字符串  
**类型**: 非空 UTF-8 字符串  
**约束**: 最大 1023 字符  
**采样**: 常规  
**常见值**: "Optical Good", "Optical Poor", "IMU Only", "No Data"

---

## 3. Timing 时间同步

### 3.1 mode (时间模式)

```json
"mode": "internal"
```

**描述**: 指示样本传输机制是否提供固有时间同步  
**类型**: 枚举值  
**可选值**:
- `"internal"`: 传输机制缺乏固有时间，样本必须包含 PTP 时间戳
- `"external"`: 传输机制提供固有时间同步  
**采样**: 常规

### 3.2 recordedTimestamp (录制时间戳)

```json
"recordedTimestamp": {
  "seconds": 1718806000,
  "nanoseconds": 500000000
}
```

**描述**: 数据录制时刻的 PTP 时间戳  
**类型**: 时间戳对象  
**单位**: 秒  
**约束**:
- `seconds`: 48 位无符号整数（自纪元开始的秒数）
- `nanoseconds`: 32 位无符号整数  
**采样**: 常规

### 3.3 sampleRate (采样率)

```json
"sampleRate": {
  "num": 24000,
  "denom": 1001
}
```

**描述**: 样本帧率，表示为有理数  
**类型**: 严格正有理数  
**约束**: 必须为正数  
**说明**: 掉帧率如 29.97 应表示为 30000/1001

### 3.4 sampleTimestamp (样本时间戳)

```json
"sampleTimestamp": {
  "seconds": 1718806554,
  "nanoseconds": 500000000
}
```

**描述**: 数据捕获时刻的 PTP 时间戳  
**类型**: 时间戳对象  
**说明**: 可能与数据包传输的 PTP 时间戳不同

### 3.5 sequenceNumber (序列号)

```json
"sequenceNumber": 0
```

**描述**: 随每个样本递增的整数  
**类型**: 非负整数  
**约束**: ≥ 0  
**采样**: 常规

### 3.6 synchronization (同步信息)

```json
"synchronization": {
  "locked": true,
  "source": "ptp",
  "frequency": {
    "num": 24000,
    "denom": 1001
  },
  "present": true,
  "ptp": {
    "profile": "SMPTE ST2059-2:2021",
    "domain": 1,
    "leaderIdentity": "00:11:22:33:44:55",
    "leaderPriorities": {
      "priority1": 128,
      "priority2": 128
    },
    "leaderAccuracy": 5e-08,
    "leaderTimeSource": "GNSS",
    "meanPathDelay": 0.000123,
    "vlan": 100
  }
}
```

#### 同步参数详解

- **`locked`**: 跟踪设备是否锁定到同步源
- **`source`**: 同步源类型
  - `"genlock"`: 外部黑场/色同步信号
  - `"videoIn"`: 外部视频信号
  - `"ptp"`: PTP 主时钟
  - `"ntp"`: NTP 服务器
- **`frequency`**: 同步信号频率（可能与采样帧率不同）
- **`present`**: 同步源是否存在
- **`ptp`**: PTP 特定信息
  - **`profile`**: PTP 配置文件（如 "SMPTE ST2059-2:2021"）
  - **`domain`**: PTP 域标识符
  - **`leaderIdentity`**: PTP 主时钟的唯一标识符（通常为 MAC 地址）
  - **`leaderPriorities`**: 主时钟优先级
    - **`priority1`**: 管理员设置的静态优先级
    - **`priority2`**: 基于角色或时钟质量的动态优先级
  - **`leaderAccuracy`**: 时间偏移精度（秒）
  - **`leaderTimeSource`**: 主时钟的时间源（GNSS、原子钟、NTP）
  - **`meanPathDelay`**: 设备与 PTP 主时钟间的平均往返延迟
  - **`vlan`**: PTP 流量的 VLAN ID

### 3.7 timecode (时间码)

```json
"timecode": {
  "hours": 1,
  "minutes": 2,
  "seconds": 3,
  "frames": 4,
  "frameRate": {
    "num": 24000,
    "denom": 1001
  }
}
```

**描述**: 样本的 SMPTE 时间码  
**约束**:
- `hours`: 0-23
- `minutes`: 0-59  
- `seconds`: 0-59
- `frames`: 0-119（必须小于帧率）
- `frameRate`: 严格正有理数

---

## 4. Lens 镜头参数

### 4.1 custom (自定义系数)

```json
"custom": [1.0, 2.0]
```

**描述**: 扩展现有镜头模型的可选自定义系数  
**类型**: 浮点数数组  
**说明**: 特定制作方和消费方之间需要协商具体含义和应用方式

### 4.2 distortion (畸变参数)

```json
"distortion": [
  {
    "model": "Brown-Conrady U-D",
    "radial": [1.0, 2.0, 3.0, 4.0, 5.0, 6.0],
    "tangential": [1.0, 2.0],
    "overscan": 3.0
  }
]
```

#### 畸变模型详解

- **`model`**: 畸变模型名称
  - `"Brown-Conrady D-U"`: 从畸变坐标映射到未畸变坐标（默认）
  - `"Brown-Conrady U-D"`: 从未畸变坐标映射到畸变坐标
- **`radial`**: 径向畸变系数 (k1-kN)
  - 最少包含 1 个系数
  - 通常包含 k1, k2, k3 等
- **`tangential`**: 切向畸变系数 (p1-pN)
  - 可选参数
  - 如果提供，最少包含 1 个系数
- **`overscan`**: 过扫描系数
  - 必须 ≥ 1.0
  - 可由制作方提供或消费方计算

### 4.3 distortionOffset (畸变偏移)

```json
"distortionOffset": {
  "x": 1.0,
  "y": 2.0
}
```

**描述**: 虚拟相机畸变中心的 X 和 Y 偏移  
**类型**: 平面偏移对象  
**单位**: 毫米  
**约束**: X 和 Y 必须为实数

### 4.4 encoders (编码器)

```json
"encoders": {
  "focus": 0.1,
  "iris": 0.2,
  "zoom": 0.3
}
```

**描述**: FIZ 编码器的归一化值  
**类型**: 归一化浮点数 (0-1)  
**约束**: 至少包含 focus、iris、zoom 中的一个  
**方向定义**:
- **`focus`**: 0=无穷远，1=最近距离
- **`iris`**: 0=最大光圈（开放），1=最小光圈（关闭）
- **`zoom`**: 0=广角，1=长焦

### 4.5 entrancePupilOffset (入瞳偏移)

```json
"entrancePupilOffset": 0.123
```

**描述**: 入瞳相对于名义成像平面的偏移  
**类型**: 实数  
**单位**: 米  
**说明**: 正值表示入瞳位于朝向物体的一侧，负值表示相反

### 4.6 exposureFalloff (曝光衰减)

```json
"exposureFalloff": {
  "a1": 1.0,
  "a2": 2.0,
  "a3": 3.0
}
```

**描述**: 计算镜头曝光衰减（晕影）的系数  
**类型**: 曝光衰减对象  
**约束**: 系数必须为实数  
**说明**: a1 为必需参数，a2 和 a3 为可选

### 4.7 fStop (光圈值)

```json
"fStop": 4.0
```

**描述**: 镜头的线性 F 值  
**类型**: 严格正实数  
**约束**: 必须 > 0  
**计算**: 焦距 ÷ 入瞳直径

### 4.8 pinholeFocalLength (针孔焦距)

```json
"pinholeFocalLength": 24.305
```

**描述**: 简单 CGI 针孔相机模型中针孔与成像平面的距离  
**类型**: 严格正实数  
**单位**: 毫米  
**约束**: 必须 > 0

### 4.9 focusDistance (对焦距离)

```json
"focusDistance": 10.0
```

**描述**: 镜头的对焦距离/位置  
**类型**: 严格正实数  
**单位**: 米  
**约束**: 必须 > 0

### 4.10 projectionOffset (投影偏移)

```json
"projectionOffset": {
  "x": 0.1,
  "y": 0.2
}
```

**描述**: 虚拟相机透视投影中心的 X 和 Y 偏移  
**类型**: 平面偏移对象  
**单位**: 毫米  
**约束**: X 和 Y 必须为实数

### 4.11 rawEncoders (原始编码器)

```json
"rawEncoders": {
  "focus": 1000,
  "iris": 2000,
  "zoom": 3000
}
```

**描述**: FIZ 编码器的原始值  
**类型**: 非负整数  
**约束**: 至少包含 focus、iris、zoom 中的一个  
**说明**: 依赖于编码器分辨率，归零/测距前的值

### 4.12 tStop (T 值)

```json
"tStop": 4.1
```

**描述**: 镜头的线性 T 值  
**类型**: 严格正实数  
**约束**: 必须 > 0  
**计算**: F 值 ÷ √(镜头透射率)

---

## 5. Protocol 协议信息

```json
"protocol": {
  "name": "OpenTrackIO",
  "version": [0, 9, 3]
}
```

### 5.1 name (协议名称)

**描述**: 协议名称标识符  
**类型**: 字符串  
**固定值**: "OpenTrackIO"

### 5.2 version (协议版本)

**描述**: 协议版本号  
**类型**: 整数数组  
**格式**: [主版本, 次版本, 修订版本]  
**当前版本**: [1, 0, 0]

---

## 6. 标识符

### 6.1 sampleId (样本ID)

```json
"sampleId": "urn:uuid:e3f34dae-b68c-47d9-99af-e0bfc277723c"
```

**描述**: 数据传输样本的唯一标识符  
**类型**: UUID URN  
**格式**: "urn:uuid:{uuid}"  
**约束**: 必须为有效的 UUID 格式

### 6.2 sourceId (源ID)

```json
"sourceId": "urn:uuid:889eec71-62ca-4c7f-b1ef-4c2b16f4eacf"
```

**描述**: 数据传输源的唯一标识符  
**类型**: UUID URN  
**格式**: "urn:uuid:{uuid}"

### 6.3 sourceNumber (源编号)

```json
"sourceNumber": 1
```

**描述**: 标识来自源的流索引的编号  
**类型**: 非负整数  
**约束**: ≥ 0  
**说明**: 当源产生多个样本流时特别重要

### 6.4 relatedSampleIds (相关样本ID)

```json
"relatedSampleIds": [
  "urn:uuid:b545b989-34b2-452a-9630-cf952d021a3f",
  "urn:uuid:dd8ce967-21f2-444e-a007-5eb33333ca51"
]
```

**描述**: 与此样本相关的样本 ID 列表  
**类型**: UUID URN 数组  
**说明**: 不保证给定 sampleId 的样本存在

---

## 7. GlobalStage 全局舞台坐标

```json
"globalStage": {
  "E": 100.0,
  "N": 200.0,
  "U": 300.0,
  "lat0": 100.0,
  "lon0": 200.0,
  "h0": 300.0
}
```

**描述**: 舞台原点在全局 ENU 和大地坐标系中的位置  
**类型**: 全局位置对象  
**单位**: 米（E, N, U, h0），度（lat0, lon0）  
**说明**: 如果舞台在移动载具内，此参数可能是动态的

### 坐标系统详解

- **E (East)**: 东向坐标（米）
- **N (North)**: 北向坐标（米）
- **U (Up)**: 向上坐标（米）
- **lat0**: 纬度（度）
- **lon0**: 经度（度）
- **h0**: 高度（米）

**参考**: [Local tangent plane coordinates](https://en.wikipedia.org/wiki/Local_tangent_plane_coordinates)

---

## 8. Transforms 空间变换

```json
"transforms": [
  {
    "translation": {
      "x": 1.0,
      "y": 2.0,
      "z": 3.0
    },
    "rotation": {
      "pan": 180.0,
      "tilt": 90.0,
      "roll": 45.0
    },
    "scale": {
      "x": 1.0,
      "y": 2.0,
      "z": 3.0
    },
    "id": "Camera"
  }
]
```

### 8.1 变换组合规则

**描述**: 变换列表，按顺序组合  
**组合顺序**: 从列表第一个变换开始，到最后一个变换结束  
**最终结果**: 相机传感器相对于舞台原点的位置（米）和方向（度）

### 8.2 坐标系统

- **Z 轴**: 向上
- **坐标系**: 右手坐标系
- **Y 轴**: 相机前向方向（当 pan、tilt、roll 为零时）
- **X 轴**: 相机右向方向

### 8.3 translation (平移)

**描述**: 3D 平移向量  
**类型**: Vector3 对象  
**单位**: 米  
**约束**: x、y、z 必须为实数

### 8.4 rotation (旋转)

**描述**: 欧拉角旋转  
**类型**: Rotator3 对象  
**单位**: 度  
**旋转顺序**: 内在旋转，绕 ZXY 轴测量  
**角度定义**:
- **`pan`**: 偏航角（绕 Z 轴）
- **`tilt`**: 俯仰角（绕 X 轴）
- **`roll`**: 滚转角（绕 Y 轴）

**欧拉角说明**:
- 人类可读，支持 >360° 或 <0° 的循环
- 虚拟相机不存在万向锁的物理限制
- 与四元数的转换简单，精度损失可接受

### 8.5 scale (缩放)

**描述**: 3D 缩放向量（可选）  
**类型**: Vector3 对象  
**约束**: x、y、z 必须为实数

### 8.6 id (标识符)

**描述**: 变换的标识符  
**类型**: 非空 UTF-8 字符串  
**常见值**: "Camera", "Dolly", "Crane Arm", "Gimbal"

---

## 9. Custom 自定义字段

```json
"custom": {
  "pot1": 2435,
  "button1": false
}
```

**描述**: 用户定义的自定义字段  
**类型**: 任意 JSON 值  
**说明**: 允许扩展标准协议以支持特定应用需求

---

## 10. 实际应用示例

### 10.1 基本相机跟踪

```json
{
  "tracker": {
    "status": "Optical Good",
    "recording": true,
    "slate": "Scene_01_Take_03"
  },
  "timing": {
    "mode": "external",
    "sampleRate": {"num": 24, "denom": 1}
  },
  "lens": {
    "fStop": 2.8,
    "focusDistance": 3.5,
    "pinholeFocalLength": 35.0
  },
  "transforms": [{
    "translation": {"x": 0.0, "y": -2.0, "z": 1.8},
    "rotation": {"pan": 0.0, "tilt": -5.0, "roll": 0.0},
    "id": "Camera"
  }],
  "protocol": {
    "name": "OpenTrackIO",
    "version": [1, 0, 0]
  }
}
```

### 10.2 多轴系统（摇臂 + 云台）

```json
{
  "transforms": [
    {
      "translation": {"x": 1.0, "y": 2.0, "z": 3.0},
      "rotation": {"pan": 45.0, "tilt": 0.0, "roll": 0.0},
      "id": "Dolly"
    },
    {
      "translation": {"x": 0.0, "y": 1.5, "z": 2.0},
      "rotation": {"pan": -30.0, "tilt": 15.0, "roll": 0.0},
      "id": "Crane Arm"
    },
    {
      "translation": {"x": 0.0, "y": 0.2, "z": 0.1},
      "rotation": {"pan": 10.0, "tilt": -10.0, "roll": 2.0},
      "id": "Camera"
    }
  ]
}
```

---

## 11. 数据类型约定

### 11.1 有理数表示

```json
{
  "num": 24000,
  "denom": 1001
}
```

**用途**: 精确表示帧率、持续时间等  
**优势**: 避免浮点数精度问题  
**示例**: 29.97 fps = 30000/1001

### 11.2 时间戳格式

```json
{
  "seconds": 1718806554,
  "nanoseconds": 500000000
}
```

**格式**: PTP 时间戳  
**精度**: 纳秒级  
**基准**: Unix 纪元（1970-01-01 00:00:00 UTC）

### 11.3 UUID URN 格式

```
"urn:uuid:e3f34dae-b68c-47d9-99af-e0bfc277723c"
```

**格式**: RFC 4122 UUID  
**前缀**: "urn:uuid:"  
**用途**: 全局唯一标识符

---

## 12. 最佳实践

### 12.1 数据完整性

1. **必需字段**: 确保 `protocol` 字段始终存在
2. **版本兼容**: 检查协议版本兼容性
3. **数据验证**: 验证所有约束条件

### 12.2 性能优化

1. **静态数据**: 将不变的参数放在 `static` 部分
2. **采样率**: 根据应用需求选择合适的采样率
3. **数据压缩**: 考虑网络传输时的数据压缩

### 12.3 同步策略

1. **PTP 优先**: 在支持的网络中优先使用 PTP
2. **时间戳一致性**: 确保所有设备使用统一的时间基准
3. **延迟补偿**: 考虑网络和处理延迟

---

## 13. 错误处理

### 13.1 常见错误

1. **格式错误**: JSON 格式不正确
2. **类型错误**: 参数类型不匹配
3. **约束违反**: 数值超出允许范围
4. **必需字段缺失**: 缺少必需的参数

### 13.2 容错机制

1. **默认值**: 为可选参数提供合理默认值
2. **数据降级**: 在部分数据丢失时继续运行
3. **错误报告**: 详细记录错误信息

---

## 参考资料

1. [SMPTE RiS-OSVP Metadata CamDKit](https://github.com/SMPTE/ris-osvp-metadata-camdkit)
2. [OpenTrackIO 官方文档](https://ris-pub.smpte.org/ris-osvp-metadata-camdkit/)
3. [OpenLensIO 规范](https://ris-pub.smpte.org/ris-osvp-metadata-camdkit/res/OpenLensIO_v1-0-0.pdf)
4. [SMPTE ST2059-2:2021 PTP 配置文件](https://ieeexplore.ieee.org/document/9450292)

---

*此文档基于 SMPTE RiS-OSVP Metadata CamDKit 源码分析生成，版本: 1.0.0*