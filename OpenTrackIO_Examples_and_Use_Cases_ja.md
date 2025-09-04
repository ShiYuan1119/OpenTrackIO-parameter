# OpenTrackIO 実例と応用シナリオ

## 実際の応用例集

実際のバーチャルプロダクションシナリオに基づくOpenTrackIOパラメータ設定例

---

## シナリオ1: 基本単一カメラトラッキング

### 応用シナリオ
- 単一カメラ
- 光学トラッキングシステム
- LEDウォールバーチャルプロダクション

### 設定例

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

### パラメータ説明

- **カメラ位置**: LEDウォールから4.5メートル、高さ1.6メートル
- **レンズ設定**: 50mm焦点距離、F2.8、フォーカス距離3.2メートル
- **トラッキング状態**: 光学トラッキング良好、録画中
- **タイム同期**: 外部同期、24fps

---

## シナリオ2: 多軸クレーンシステム

### 応用シナリオ
- 電動クレーン + ジンバル
- 高精度モーション制御
- 複雑な運動軌跡

### 設定例

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

### 変換チェーン解析

1. **Dolly**: ベースプラットフォーム、軽微な右移動と回転
2. **Crane_Arm**: クレーンアーム延長、上方と前方への運動
3. **Camera**: 最終カメラ微調整、精密フレーミング

**最終位置**: (2.5, 4.35, 2.85) メートル  
**最終回転**: Pan: -5°, Tilt: 4°, Roll: 1.2°

---

## シナリオ3: 高速撮影トラッキング

### 応用シナリオ
- 高フレームレート撮影
- 高速運動トラッキング
- スポーツイベント放送

### 設定例

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

### 高速トラッキング特徴

- **サンプリングレート**: 120fps高周波サンプリング
- **シャッター設定**: 172.8°シャッター角度（約1/250秒）
- **PTP同期**: ナノ秒レベルの時間精度
- **望遠レンズ**: 200mm遠距離トラッキング

---

## シナリオ4: アナモルフィックレンズ撮影

### 応用シナリオ
- 映画級アナモルフィックレンズ
- ワイドスクリーン形式
- 複雑な歪み補正

### 設定例

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

### アナモルフィックレンズ特徴

- **圧縮比**: 2:1水平圧縮
- **複雑な歪み**: 4次径方向 + 接線方向歪み
- **投影オフセット**: 光学中心オフセット補償
- **露出減衰**: エッジ光量減衰補償

---

## シナリオ5: マルチカメラ同期システム

### 応用シナリオ
- マルチカメラ同時撮影
- 精密タイム同期
- 協調運動制御

### カメラA設定

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

### カメラB設定

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

### 同期要点

- **統一PTPドメイン**: すべてのカメラが同じPTPドメインを使用
- **ソース番号**: 一意のsourceNumber識別
- **タイムベース**: 共有PTPマスタークロック

---

## シナリオ6: 移動プラットフォーム撮影

### 応用シナリオ
- 車載撮影
- 船舶撮影
- 航空機撮影

### 設定例

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

### 移動プラットフォーム特徴

- **GPS座標**: ロサンゼルス市街地座標
- **動的ステージ**: globalStageが車両移動とともに更新
- **GNSS同期**: 衛星時間基準を使用

---

## シナリオ7: レンズ歪み補正

### 応用シナリオ
- 広角レンズ
- 魚眼レンズ
- 精密歪み補正

### 設定例

```json
{
  "lens": {
    "pinholeFocalLength": 14.0,
    "fStop": 2.8,
    "distortion": [{
      "model": "Brown-Conrady D-U",
      "radial": [
        -0.2573,   // k1: 主要樽型歪み
        0.1124,    // k2: 2次補正
        -0.0321,   // k3: 高次補正
        0.0089,    // k4: エッジ補正
        -0.0012,   // k5: 微調整
        0.0001     // k6: 極エッジ
      ],
      "tangential": [
        0.0023,    // p1: 垂直接線
        -0.0015    // p2: 水平接線
      ],
      "overscan": 1.35
    }],
    "distortionOffset": {
      "x": -0.12,  // 光学中心オフセット
      "y": 0.08
    }
  }
}
```

### 歪み補正説明

- **広角歪み**: 負のk1値は樽型歪みを示す
- **高次補正**: 複数の径方向係数で精密モデリング
- **接線補正**: レンズ製造誤差を補償
- **中心オフセット**: 光学中心と幾何中心の不一致

---

## シナリオ8: ズームトラッキング撮影

### 応用シナリオ
- ズームレンズトラッキング
- 動的焦点距離変化
- リアルタイムパラメータ更新

### 設定例

```json
{
  "lens": {
    "encoders": {
      "focus": 0.35,
      "iris": 0.4,
      "zoom": 0.75    // 75% 望遠位置
    },
    "rawEncoders": {
      "focus": 3500,
      "iris": 4000,
      "zoom": 7500
    },
    "pinholeFocalLength": 135.0,  // 現在の焦点距離
    "focusDistance": 8.5,
    "fStop": 4.0,
    "distortion": [{
      "model": "Brown-Conrady D-U",
      "radial": [
        -0.045,   // 望遠端では歪みが小さい
        0.012,
        -0.002
      ],
      "overscan": 1.08
    }]
  }
}
```

### ズーム特徴

- **焦点距離範囲**: 仮定24-200mm、現在135mm
- **エンコーダー対応**: 生の値と正規化値の両方を提供
- **歪み変化**: 異なる焦点距離での歪み係数が異なる

---

## シナリオ9: PTPネットワーク同期

### 応用シナリオ
- 大規模制作ネットワーク
- マルチデバイス精密同期
- プロフェッショナルレベルのタイム管理

### 設定例

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
        "translation": 0.00833,    // 1/120秒オフセット
        "rotation": 0.00417,       // 1/240秒オフセット
        "lensEncoders": 0.0167     // 1/60秒オフセット
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

### PTPネットワーク特徴

- **原子時計基準**: 最高精度タイムソース
- **VLAN分離**: 専用ネットワークセグメント
- **遅延補償**: 異なるデータソースのタイムオフセット
- **ナノ秒精度**: サブミリ秒レベル同期

---

## シナリオ10: カスタム拡張応用

### 応用シナリオ
- 特殊機器統合
- 実験的機能
- ベンダー固有拡張

### 設定例

```json
{
  "lens": {
    "custom": [
      [0.123, 0.456, 0.789],     // カスタムレンズ係数グループ1
      [1.234, 2.345, 3.456]      // カスタムレンズ係数グループ2
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

### 拡張応用説明

- **環境データ**: 温度、湿度がトラッキング精度に影響
- **モーションブラー**: リアルタイムモーションブラーパラメータ
- **クロマキー**: リアルタイム合成パラメータ
- **ベンダー拡張**: 特定デバイス機能

---

## データフロー例

### リアルタイムデータストリーム

```json
// フレーム N
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

// フレーム N+1
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

### モーション補間

```python
def interpolate_transform(t1: Transform, t2: Transform, alpha: float) -> Transform:
    """2つの変換間で線形補間を行う"""
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

## エラー処理例

### データ検証エラー

```python
try:
    clip = Clip.model_validate(json_data)
except ValidationError as e:
    for error in e.errors():
        field = error['loc']
        message = error['msg']
        print(f"フィールド {'.'.join(field)}: {message}")
```

### ネットワーク伝送エラー

```python
def handle_transmission_error(error_code: int, data: dict):
    if error_code == 1001:  # データ損失
        # 補間を使用して欠損フレームを補填
        return interpolate_missing_frame(data)
    elif error_code == 1002:  # タイムスタンプエラー
        # シーケンス番号を使用してタイムスタンプを再構築
        return rebuild_timestamp(data)
    else:
        raise TransmissionError(f"未知のエラー: {error_code}")
```

---

## パフォーマンス最適化例

### データ圧縮

```python
def compress_opentrackio(clip: Clip) -> dict:
    """冗長データを削除して伝送量を減らす"""
    data = clip.model_dump(exclude_none=True)
    
    # デフォルト値を削除
    if data.get('lens', {}).get('distortion', [{}])[0].get('model') == 'Brown-Conrady D-U':
        del data['lens']['distortion'][0]['model']
    
    return data
```

### バッチ処理

```python
def process_batch(clips: list[Clip]) -> list[Clip]:
    """複数のOpenTrackIOサンプルをバッチ処理"""
    # 静的データを抽出し、重複処理を回避
    static_data = clips[0].static if clips else None
    
    processed = []
    for clip in clips:
        # 静的データを再利用
        if static_data:
            clip.static = static_data
        processed.append(clip)
    
    return processed
```

---

## 統合テストケース

### エンドツーエンドテスト

```python
def test_end_to_end_pipeline():
    # 1. テストデータ生成
    clip = generate_test_clip()
    
    # 2. シリアル化
    json_data = clip.to_json()
    
    # 3. ネットワーク伝送シミュレーション
    transmitted = simulate_network_transmission(json_data)
    
    # 4. デシリアル化
    received_clip = Clip.model_validate(json.loads(transmitted))
    
    # 5. データ整合性検証
    assert_clips_equal(clip, received_clip)
```

### ストレステスト

```python
def test_high_frequency_streaming():
    """高周波データストリーム処理をテスト"""
    sample_rate = 240  # 240 fps
    duration = 10      # 10秒
    
    clips = []
    for frame in range(sample_rate * duration):
        clip = generate_frame_data(frame)
        clips.append(clip)
    
    # 処理パフォーマンステスト
    start_time = time.time()
    process_clip_stream(clips)
    end_time = time.time()
    
    processing_time = end_time - start_time
    assert processing_time < duration, "リアルタイム処理要件未満足"
```

---

## デバッグツール

### データ可視化

```python
def visualize_transform_chain(transforms: list[Transform]):
    """変換チェーンを可視化"""
    import matplotlib.pyplot as plt
    from mpl_toolkits.mplot3d import Axes3D
    
    fig = plt.figure()
    ax = fig.add_subplot(111, projection='3d')
    
    # 累積変換
    current_pos = [0, 0, 0]
    positions = [current_pos.copy()]
    
    for transform in transforms:
        current_pos[0] += transform.translation.x
        current_pos[1] += transform.translation.y
        current_pos[2] += transform.translation.z
        positions.append(current_pos.copy())
    
    # 軌跡を描画
    xs, ys, zs = zip(*positions)
    ax.plot(xs, ys, zs, 'b-o')
    ax.set_xlabel('X (メートル)')
    ax.set_ylabel('Y (メートル)')
    ax.set_zlabel('Z (メートル)')
    plt.show()
```

### パラメータ検証ツール

```python
def validate_lens_parameters(lens: Lens) -> list[str]:
    """レンズパラメータの論理一貫性を検証"""
    warnings = []
    
    if lens.f_stop and lens.t_stop:
        if lens.t_stop[0] < lens.f_stop[0]:
            warnings.append("T値はF値より小さくあるべきではない")
    
    if lens.focus_distance and lens.pinhole_focal_length:
        if lens.focus_distance[0] < lens.pinhole_focal_length[0] / 1000:
            warnings.append("フォーカス距離が近すぎる可能性がある")
    
    return warnings
```

---

## 一般的な応用パターン

### 1. リアルタイムストリーミング

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
        # ... その他のデータを入力
        self.sequence += 1
        return clip
```

### 2. 録画再生

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

### 3. データ同期

```python
class MultiSourceSynchronizer:
    def __init__(self):
        self.sources: dict[int, list[Clip]] = {}
    
    def add_sample(self, source_number: int, clip: Clip):
        if source_number not in self.sources:
            self.sources[source_number] = []
        self.sources[source_number].append(clip)
    
    def get_synchronized_frame(self, timestamp: float) -> dict[int, Clip]:
        """タイムスタンプに基づいてすべてのソースの同期フレームを取得"""
        result = {}
        for source_num, clips in self.sources.items():
            # タイムスタンプに最も近いサンプルを検索
            closest_clip = min(clips, 
                key=lambda c: abs(c.timing.sampleTimestamp[0].seconds - timestamp))
            result[source_num] = closest_clip
        return result
```

---

## 展開設定

### ネットワーク設定

```yaml
# PTPネットワーク設定例
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

### システム要件

```yaml
system:
  os: Linux/Windows/macOS
  cpu: x86_64, ARM64
  memory: ≥ 4GB
  network: ギガビットイーサネット
  storage: SSD推奨
  
dependencies:
  python: "≥ 3.8"
  pydantic: "≥ 2.0"
  json: 内蔵
```

---

## 故障診断

### 診断チェックリスト

1. **プロトコルバージョン**: バージョン互換性をチェック
2. **ネットワーク接続**: PTP/UDP接続を検証
3. **データ形式**: JSON構造を検証
4. **タイム同期**: クロック同期状態をチェック
5. **パラメータ範囲**: 数値制約を検証

### ログ例

```
[INFO] OpenTrackIO: プロトコルバージョン 1.0.0
[INFO] PTP: マスタークロック aa:bb:cc:dd:ee:ff に同期
[WARN] Lens: F値 0.5 は小さすぎる可能性がある
[ERROR] Timing: シーケンス番号が不連続 (期待値:1441, 実際:1443)
[DEBUG] Transform: カメラ位置 (1.25, -2.08, 1.81)
```

---

*実例と応用シナリオマニュアルバージョン: 1.0.0*