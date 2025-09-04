# OpenTrackIO 实例与应用场景

## 实际应用示例集

基于真实虚拟制作场景的 OpenTrackIO 参数配置示例

---

## 场景1: 基础单机位跟踪

### 应用场景
- 单台摄像机
- 光学跟踪系统
- LED 墙虚拟制作

### 配置示例

```json
{
  "static": {
    "camera": {
      "make": "Sony",
      "model": "FX9",
      "serialNumber": "SN123456",
      "label": "Camera_A",
      "activeSensorPhysicalDimensions": {
        "width": 35.7,
        "height": 18.8
      },
      "activeSensorResolution": {
        "width": 4096,
        "height": 2160
      },
      "captureFrameRate": {
        "num": 24,
        "denom": 1
      }
    },
    "lens": {
      "make": "Canon",
      "model": "CN-E24-105mm",
      "nominalFocalLength": 50.0
    },
    "tracker": {
      "make": "OptiTrack",
      "model": "Prime X 13",
      "firmwareVersion": "2.3.1"
    }
  },
  "tracker": {
    "status": "Optical Good",
    "recording": true,
    "slate": "EP01_SC05_T01"
  },
  "timing": {
    "mode": "external",
    "sampleRate": {
      "num": 24,
      "denom": 1
    },
    "timecode": {
      "hours": 10,
      "minutes": 30,
      "seconds": 15,
      "frames": 12,
      "frameRate": {
        "num": 24,
        "denom": 1
      }
    }
  },
  "lens": {
    "fStop": 2.8,
    "focusDistance": 3.2,
    "pinholeFocalLength": 50.0,
    "encoders": {
      "focus": 0.65,
      "iris": 0.35,
      "zoom": 0.5
    }
  },
  "transforms": [{
    "translation": {
      "x": 0.0,
      "y": -4.5,
      "z": 1.6
    },
    "rotation": {
      "pan": 0.0,
      "tilt": -2.5,
      "roll": 0.0
    },
    "id": "Camera"
  }],
  "protocol": {
    "name": "OpenTrackIO",
    "version": [1, 0, 0]
  },
  "sampleId": "urn:uuid:12345678-1234-5678-9abc-123456789abc",
  "sourceId": "urn:uuid:87654321-4321-8765-cba9-987654321abc",
  "sourceNumber": 1
}
```

### 参数说明

- **相机位置**: 距离 LED 墙 4.5 米，高度 1.6 米
- **镜头设置**: 50mm 焦距，F2.8，对焦距离 3.2 米
- **跟踪状态**: 光学跟踪良好，正在录制
- **时间同步**: 外部同步，24fps

---

## 场景2: 多轴摇臂系统

### 应用场景
- 电动摇臂 + 云台
- 高精度运动控制
- 复杂运动轨迹

### 配置示例

```json
{
  "tracker": {
    "status": "Motion Control Active",
    "recording": true,
    "slate": "COMMERCIAL_HERO_T05"
  },
  "timing": {
    "mode": "external",
    "sampleRate": {
      "num": 60,
      "denom": 1
    },
    "synchronization": {
      "locked": true,
      "source": "genlock",
      "present": true,
      "frequency": {
        "num": 60,
        "denom": 1
      }
    }
  },
  "lens": {
    "fStop": 1.4,
    "focusDistance": 2.8,
    "pinholeFocalLength": 85.0,
    "encoders": {
      "focus": 0.72,
      "iris": 0.15,
      "zoom": 0.8
    },
    "distortion": [{
      "model": "Brown-Conrady D-U",
      "radial": [0.05, -0.12, 0.03],
      "tangential": [0.01, -0.005],
      "overscan": 1.15
    }]
  },
  "transforms": [
    {
      "translation": {
        "x": 2.5,
        "y": 1.0,
        "z": 0.0
      },
      "rotation": {
        "pan": 15.0,
        "tilt": 0.0,
        "roll": 0.0
      },
      "id": "Dolly"
    },
    {
      "translation": {
        "x": 0.0,
        "y": 3.2,
        "z": 2.8
      },
      "rotation": {
        "pan": -25.0,
        "tilt": 12.0,
        "roll": 0.0
      },
      "id": "Crane_Arm"
    },
    {
      "translation": {
        "x": 0.0,
        "y": 0.15,
        "z": 0.05
      },
      "rotation": {
        "pan": 5.0,
        "tilt": -8.0,
        "roll": 1.2
      },
      "id": "Camera"
    }
  ],
  "protocol": {
    "name": "OpenTrackIO",
    "version": [1, 0, 0]
  }
}
```

### 变换链分析

1. **Dolly**: 基础平台，轻微右移和旋转
2. **Crane_Arm**: 摇臂延伸，向上和前方运动
3. **Camera**: 最终相机微调，精确构图

**最终位置**: (2.5, 4.35, 2.85) 米  
**最终旋转**: Pan: -5°, Tilt: 4°, Roll: 1.2°

---

## 场景3: 高速摄影跟踪

### 应用场景
- 高帧率拍摄
- 快速运动跟踪
- 体育赛事转播

### 配置示例

```json
{
  "static": {
    "camera": {
      "captureFrameRate": {
        "num": 120,
        "denom": 1
      },
      "shutterAngle": 172.8,
      "isoSpeed": 1600
    }
  },
  "timing": {
    "mode": "external",
    "sampleRate": {
      "num": 120,
      "denom": 1
    },
    "synchronization": {
      "locked": true,
      "source": "ptp",
      "present": true,
      "ptp": {
        "profile": "SMPTE ST2059-2:2021",
        "domain": 127,
        "leaderIdentity": "aa:bb:cc:dd:ee:ff",
        "leaderAccuracy": 1e-09,
        "leaderTimeSource": "GNSS",
        "meanPathDelay": 0.000085
      }
    }
  },
  "lens": {
    "fStop": 5.6,
    "pinholeFocalLength": 200.0,
    "focusDistance": 15.0,
    "encoders": {
      "focus": 0.45,
      "iris": 0.8,
      "zoom": 0.9
    }
  }
}
```

### 高速跟踪特点

- **采样率**: 120fps 高频采样
- **快门设置**: 172.8° 快门角度（约1/250s）
- **PTP 同步**: 纳秒级时间精度
- **长焦镜头**: 200mm 远距离跟踪

---

## 场景4: 变形镜头拍摄

### 应用场景
- 电影级变形镜头
- 宽银幕格式
- 复杂畸变校正

### 配置示例

```json
{
  "static": {
    "camera": {
      "anamorphicSqueeze": {
        "num": 2,
        "denom": 1
      }
    },
    "lens": {
      "make": "Cooke",
      "model": "Anamorphic/i 40mm",
      "nominalFocalLength": 40.0
    }
  },
  "lens": {
    "fStop": 2.0,
    "pinholeFocalLength": 40.0,
    "focusDistance": 4.5,
    "distortion": [{
      "model": "Brown-Conrady U-D",
      "radial": [0.08, -0.25, 0.15, -0.05],
      "tangential": [0.02, -0.01],
      "overscan": 1.4
    }],
    "projectionOffset": {
      "x": 0.05,
      "y": -0.02
    },
    "exposureFalloff": {
      "a1": 0.95,
      "a2": 0.05,
      "a3": 0.01
    }
  }
}
```

### 变形镜头特点

- **压缩比**: 2:1 水平压缩
- **复杂畸变**: 4 阶径向 + 切向畸变
- **投影偏移**: 补偿光学中心偏移
- **曝光衰减**: 边缘光量衰减补偿

---

## 场景5: 多机位同步系统

### 应用场景
- 多机位同时拍摄
- 精确时间同步
- 协调运动控制

### 机位A配置

```json
{
  "sourceNumber": 1,
  "timing": {
    "synchronization": {
      "locked": true,
      "source": "ptp",
      "ptp": {
        "profile": "SMPTE ST2059-2:2021",
        "domain": 100,
        "leaderIdentity": "12:34:56:78:9a:bc"
      }
    }
  },
  "transforms": [{
    "translation": {"x": -3.0, "y": 0.0, "z": 1.8},
    "rotation": {"pan": 30.0, "tilt": 0.0, "roll": 0.0},
    "id": "Camera_A"
  }]
}
```

### 机位B配置

```json
{
  "sourceNumber": 2,
  "timing": {
    "synchronization": {
      "locked": true,
      "source": "ptp",
      "ptp": {
        "profile": "SMPTE ST2059-2:2021",
        "domain": 100,
        "leaderIdentity": "12:34:56:78:9a:bc"
      }
    }
  },
  "transforms": [{
    "translation": {"x": 3.0, "y": 0.0, "z": 1.8},
    "rotation": {"pan": -30.0, "tilt": 0.0, "roll": 0.0},
    "id": "Camera_B"
  }]
}
```

### 同步要点

- **统一PTP域**: 所有机位使用相同 PTP 域
- **源编号**: 唯一的 sourceNumber 标识
- **时间基准**: 共享 PTP 主时钟

---

## 场景6: 移动平台拍摄

### 应用场景
- 车载拍摄
- 船载拍摄
- 飞行器拍摄

### 配置示例

```json
{
  "globalStage": {
    "E": 1250.5,
    "N": 2340.8,
    "U": 125.2,
    "lat0": 34.052235,
    "lon0": -118.243683,
    "h0": 125.2
  },
  "timing": {
    "synchronization": {
      "locked": true,
      "source": "ptp",
      "ptp": {
        "leaderTimeSource": "GNSS"
      }
    }
  },
  "transforms": [
    {
      "translation": {"x": 0.0, "y": 2.5, "z": 1.2},
      "rotation": {"pan": 0.0, "tilt": 0.0, "roll": 0.0},
      "id": "Vehicle"
    },
    {
      "translation": {"x": 0.0, "y": 0.8, "z": 0.3},
      "rotation": {"pan": 15.0, "tilt": -5.0, "roll": 0.0},
      "id": "Camera"
    }
  ]
}
```

### 移动平台特点

- **GPS 坐标**: 洛杉矶市区坐标
- **动态舞台**: globalStage 随车辆移动更新
- **GNSS 同步**: 使用卫星时间基准

---

## 场景7: 镜头畸变校正

### 应用场景
- 广角镜头
- 鱼眼镜头
- 精确畸变校正

### 配置示例

```json
{
  "lens": {
    "pinholeFocalLength": 14.0,
    "fStop": 2.8,
    "distortion": [{
      "model": "Brown-Conrady D-U",
      "radial": [
        -0.2573,   // k1: 主要桶形畸变
        0.1124,    // k2: 二阶校正
        -0.0321,   // k3: 高阶校正
        0.0089,    // k4: 边缘校正
        -0.0012,   // k5: 精细调整
        0.0001     // k6: 极边缘
      ],
      "tangential": [
        0.0023,    // p1: 垂直切向
        -0.0015    // p2: 水平切向
      ],
      "overscan": 1.35
    }],
    "distortionOffset": {
      "x": -0.12,  // 光学中心偏移
      "y": 0.08
    }
  }
}
```

### 畸变校正说明

- **广角畸变**: 负 k1 值表示桶形畸变
- **高阶校正**: 多个径向系数精确建模
- **切向校正**: 补偿镜头制造误差
- **中心偏移**: 光学中心与几何中心不重合

---

## 场景8: 变焦跟踪拍摄

### 应用场景
- 变焦镜头跟踪
- 动态焦距变化
- 实时参数更新

### 配置示例

```json
{
  "lens": {
    "encoders": {
      "focus": 0.35,
      "iris": 0.4,
      "zoom": 0.75    // 75% 长焦位置
    },
    "rawEncoders": {
      "focus": 3500,
      "iris": 4000,
      "zoom": 7500
    },
    "pinholeFocalLength": 135.0,  // 当前焦距
    "focusDistance": 8.5,
    "fStop": 4.0,
    "distortion": [{
      "model": "Brown-Conrady D-U",
      "radial": [
        -0.045,   // 长焦端畸变较小
        0.012,
        -0.002
      ],
      "overscan": 1.08
    }]
  }
}
```

### 变焦特点

- **焦距范围**: 假设 24-200mm，当前 135mm
- **编码器对应**: 原始值与归一化值同时提供
- **畸变变化**: 不同焦距的畸变系数不同

---

## 场景9: PTP 网络同步

### 应用场景
- 大型制作网络
- 多设备精确同步
- 专业级时间管理

### 配置示例

```json
{
  "timing": {
    "mode": "external",
    "synchronization": {
      "locked": true,
      "source": "ptp",
      "frequency": {
        "num": 59940,
        "denom": 1000
      },
      "present": true,
      "offsets": {
        "translation": 0.00833,    // 1/120 秒偏移
        "rotation": 0.00417,       // 1/240 秒偏移
        "lensEncoders": 0.0167     // 1/60 秒偏移
      },
      "ptp": {
        "profile": "SMPTE ST2059-2:2021",
        "domain": 127,
        "leaderIdentity": "aa:bb:cc:dd:ee:ff",
        "leaderPriorities": {
          "priority1": 128,
          "priority2": 128
        },
        "leaderAccuracy": 5e-09,
        "leaderTimeSource": "Atomic clock",
        "meanPathDelay": 0.000045,
        "vlan": 100
      }
    },
    "recordedTimestamp": {
      "seconds": 1703980800,
      "nanoseconds": 123456789
    },
    "sampleTimestamp": {
      "seconds": 1703980800,
      "nanoseconds": 140123456
    }
  }
}
```

### PTP 网络特点

- **原子钟基准**: 最高精度时间源
- **VLAN 隔离**: 专用网络段
- **延迟补偿**: 不同数据源的时间偏移
- **纳秒精度**: 亚毫秒级同步

---

## 场景10: 自定义扩展应用

### 应用场景
- 特殊设备集成
- 实验性功能
- 厂商特定扩展

### 配置示例

```json
{
  "lens": {
    "custom": [
      [0.123, 0.456, 0.789],     // 自定义镜头系数组1
      [1.234, 2.345, 3.456]      // 自定义镜头系数组2
    ]
  },
  "custom": {
    "vendorName": "CustomVendor",
    "deviceType": "ExperimentalTracker",
    "calibrationData": {
      "temperature": 23.5,
      "humidity": 45.2,
      "pressure": 1013.25
    },
    "motionBlur": {
      "enabled": true,
      "strength": 0.75
    },
    "specialEffects": {
      "chromaKey": {
        "enabled": true,
        "color": [0, 255, 0],
        "tolerance": 0.1
      }
    }
  }
}
```

### 扩展应用说明

- **环境数据**: 温度、湿度影响跟踪精度
- **运动模糊**: 实时运动模糊参数
- **色键**: 实时抠像参数
- **厂商扩展**: 特定设备功能

---

## 数据流示例

### 实时数据流

```json
// 帧 N
{
  "timing": {
    "sequenceNumber": 1440,
    "sampleTimestamp": {
      "seconds": 1703980800,
      "nanoseconds": 41666667
    }
  },
  "transforms": [{
    "translation": {"x": 1.2, "y": -2.1, "z": 1.8},
    "rotation": {"pan": 15.5, "tilt": -3.2, "roll": 0.1}
  }]
}

// 帧 N+1
{
  "timing": {
    "sequenceNumber": 1441,
    "sampleTimestamp": {
      "seconds": 1703980800,
      "nanoseconds": 83333334
    }
  },
  "transforms": [{
    "translation": {"x": 1.25, "y": -2.08, "z": 1.81},
    "rotation": {"pan": 15.8, "tilt": -3.1, "roll": 0.12}
  }]
}
```

### 运动插值

```python
def interpolate_transform(t1: Transform, t2: Transform, alpha: float) -> Transform:
    """在两个变换之间进行线性插值"""
    return Transform(
        translation=Vector3(
            x=t1.translation.x + alpha * (t2.translation.x - t1.translation.x),
            y=t1.translation.y + alpha * (t2.translation.y - t1.translation.y),
            z=t1.translation.z + alpha * (t2.translation.z - t1.translation.z)
        ),
        rotation=Rotator3(
            pan=interpolate_angle(t1.rotation.pan, t2.rotation.pan, alpha),
            tilt=interpolate_angle(t1.rotation.tilt, t2.rotation.tilt, alpha),
            roll=interpolate_angle(t1.rotation.roll, t2.rotation.roll, alpha)
        )
    )
```

---

## 错误处理示例

### 数据验证错误

```python
try:
    clip = Clip.model_validate(json_data)
except ValidationError as e:
    for error in e.errors():
        field = error['loc']
        message = error['msg']
        print(f"字段 {'.'.join(field)}: {message}")
```

### 网络传输错误

```python
def handle_transmission_error(error_code: int, data: dict):
    if error_code == 1001:  # 数据丢失
        # 使用插值填补缺失帧
        return interpolate_missing_frame(data)
    elif error_code == 1002:  # 时间戳错误
        # 使用序列号重建时间戳
        return rebuild_timestamp(data)
    else:
        raise TransmissionError(f"未知错误: {error_code}")
```

---

## 性能优化示例

### 数据压缩

```python
def compress_opentrackio(clip: Clip) -> dict:
    """移除冗余数据以减少传输量"""
    data = clip.model_dump(exclude_none=True)
    
    # 移除默认值
    if data.get('lens', {}).get('distortion', [{}])[0].get('model') == 'Brown-Conrady D-U':
        del data['lens']['distortion'][0]['model']
    
    return data
```

### 批量处理

```python
def process_batch(clips: list[Clip]) -> list[Clip]:
    """批量处理多个 OpenTrackIO 样本"""
    # 提取静态数据，避免重复处理
    static_data = clips[0].static if clips else None
    
    processed = []
    for clip in clips:
        # 复用静态数据
        if static_data:
            clip.static = static_data
        processed.append(clip)
    
    return processed
```

---

## 集成测试用例

### 端到端测试

```python
def test_end_to_end_pipeline():
    # 1. 生成测试数据
    clip = generate_test_clip()
    
    # 2. 序列化
    json_data = clip.to_json()
    
    # 3. 网络传输模拟
    transmitted = simulate_network_transmission(json_data)
    
    # 4. 反序列化
    received_clip = Clip.model_validate(json.loads(transmitted))
    
    # 5. 验证数据完整性
    assert_clips_equal(clip, received_clip)
```

### 压力测试

```python
def test_high_frequency_streaming():
    """测试高频数据流处理"""
    sample_rate = 240  # 240 fps
    duration = 10      # 10 秒
    
    clips = []
    for frame in range(sample_rate * duration):
        clip = generate_frame_data(frame)
        clips.append(clip)
    
    # 测试处理性能
    start_time = time.time()
    process_clip_stream(clips)
    end_time = time.time()
    
    processing_time = end_time - start_time
    assert processing_time < duration, "实时处理要求未满足"
```

---

## 调试工具

### 数据可视化

```python
def visualize_transform_chain(transforms: list[Transform]):
    """可视化变换链"""
    import matplotlib.pyplot as plt
    from mpl_toolkits.mplot3d import Axes3D
    
    fig = plt.figure()
    ax = fig.add_subplot(111, projection='3d')
    
    # 累积变换
    current_pos = [0, 0, 0]
    positions = [current_pos.copy()]
    
    for transform in transforms:
        current_pos[0] += transform.translation.x
        current_pos[1] += transform.translation.y
        current_pos[2] += transform.translation.z
        positions.append(current_pos.copy())
    
    # 绘制轨迹
    xs, ys, zs = zip(*positions)
    ax.plot(xs, ys, zs, 'b-o')
    ax.set_xlabel('X (米)')
    ax.set_ylabel('Y (米)')
    ax.set_zlabel('Z (米)')
    plt.show()
```

### 参数验证工具

```python
def validate_lens_parameters(lens: Lens) -> list[str]:
    """验证镜头参数的逻辑一致性"""
    warnings = []
    
    if lens.f_stop and lens.t_stop:
        if lens.t_stop[0] < lens.f_stop[0]:
            warnings.append("T值不应小于F值")
    
    if lens.focus_distance and lens.pinhole_focal_length:
        if lens.focus_distance[0] < lens.pinhole_focal_length[0] / 1000:
            warnings.append("对焦距离可能过近")
    
    return warnings
```

---

## 常见应用模式

### 1. 实时流媒体

```python
class OpenTrackIOStreamer:
    def __init__(self, sample_rate: int):
        self.sample_rate = sample_rate
        self.sequence = 0
    
    def create_sample(self, tracking_data: dict) -> Clip:
        clip = Clip()
        clip.timing = [Timing(
            sequenceNumber=[self.sequence],
            sampleRate=[StrictlyPositiveRational(num=self.sample_rate, denom=1)]
        )]
        # ... 填充其他数据
        self.sequence += 1
        return clip
```

### 2. 录制回放

```python
class OpenTrackIORecorder:
    def __init__(self):
        self.clips: list[Clip] = []
    
    def record(self, clip: Clip):
        self.clips.append(clip)
    
    def playback(self, start_frame: int, end_frame: int) -> list[Clip]:
        return self.clips[start_frame:end_frame]
    
    def export(self, filename: str):
        with open(filename, 'w') as f:
            json.dump([clip.model_dump() for clip in self.clips], f, indent=2)
```

### 3. 数据同步

```python
class MultiSourceSynchronizer:
    def __init__(self):
        self.sources: dict[int, list[Clip]] = {}
    
    def add_sample(self, source_number: int, clip: Clip):
        if source_number not in self.sources:
            self.sources[source_number] = []
        self.sources[source_number].append(clip)
    
    def get_synchronized_frame(self, timestamp: float) -> dict[int, Clip]:
        """根据时间戳获取所有源的同步帧"""
        result = {}
        for source_num, clips in self.sources.items():
            # 查找最接近时间戳的样本
            closest_clip = min(clips, 
                key=lambda c: abs(c.timing.sampleTimestamp[0].seconds - timestamp))
            result[source_num] = closest_clip
        return result
```

---

## 部署配置

### 网络配置

```yaml
# PTP 网络配置示例
network:
  ptp:
    profile: "SMPTE ST2059-2:2021"
    domain: 127
    vlan: 100
    priority: high
  
  multicast:
    address: "239.192.1.1"
    port: 4001
    ttl: 64
```

### 系统要求

```yaml
system:
  os: Linux/Windows/macOS
  cpu: x86_64, ARM64
  memory: ≥ 4GB
  network: Gigabit Ethernet
  storage: SSD 推荐
  
dependencies:
  python: "≥ 3.8"
  pydantic: "≥ 2.0"
  json: 内置
```

---

## 故障诊断

### 诊断清单

1. **协议版本**: 检查版本兼容性
2. **网络连接**: 验证 PTP/UDP 连接
3. **数据格式**: 验证 JSON 结构
4. **时间同步**: 检查时钟同步状态
5. **参数范围**: 验证数值约束

### 日志示例

```
[INFO] OpenTrackIO: 协议版本 1.0.0
[INFO] PTP: 同步到主时钟 aa:bb:cc:dd:ee:ff
[WARN] Lens: F值 0.5 可能过小
[ERROR] Timing: 序列号不连续 (期望:1441, 实际:1443)
[DEBUG] Transform: 相机位置 (1.25, -2.08, 1.81)
```

---

*实例与应用场景手册版本: 1.0.0*