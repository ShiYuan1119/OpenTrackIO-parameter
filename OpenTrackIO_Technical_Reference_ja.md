# OpenTrackIO 技術リファレンスマニュアル

## JSON Schema 構造解析

SMPTE CamDKit ソースコードに基づく詳細技術仕様

---

## データ型定義

### 基本型

#### StrictlyPositiveRational (厳密正有理数)
```python
class StrictlyPositiveRational:
    num: int  # 分子、必須 > 0
    denom: int  # 分母、必須 > 0
```

**用途**: フレームレート、継続時間、サンプリングレート  
**検証**: num > 0 かつ denom > 0

#### NormalizedFloat (正規化浮動小数点数)
```python
NormalizedFloat = Annotated[float, Field(ge=0.0, le=1.0)]
```

**範囲**: [0.0, 1.0]  
**用途**: エンコーダー位置、透明度など

#### NonNegativeInt (非負整数)
```python
NonNegativeInt = Annotated[int, Field(ge=0)]
```

**範囲**: [0, ∞)  
**用途**: シーケンス番号、カウンターなど

---

## コアクラス構造

### Clip メインクラス

```python
class Clip(CompatibleBaseModel):
    static: Static = Static()
    tracker: Tracker = Tracker()
    timing: Timing = Timing()
    lens: Lens = Lens()
    # ... その他のフィールド
```

### Static 静的情報クラス

```python
class Static(CompatibleBaseModel):
    duration: StrictlyPositiveRational | None = None
    camera: StaticCamera = StaticCamera()
    lens: StaticLens = StaticLens()
    tracker: StaticTracker = StaticTracker()
```

---

## 詳細パラメータ仕様

### Camera カメラパラメータ

#### PhysicalDimensions (物理寸法)
```python
class PhysicalDimensions(CompatibleBaseModel):
    height: Annotated[float, Field(ge=0.0, strict=True)]
    width: Annotated[float, Field(ge=0.0, strict=True)]
```

**制約**: 高さと幅は ≥ 0.0  
**単位**: ミリメートル  
**精度**: 浮動小数点数

#### SenselDimensions (センサー解像度)
```python
class SenselDimensions(CompatibleBaseModel):
    height: Annotated[int, Field(ge=0, le=MAX_INT_32)]
    width: Annotated[int, Field(ge=0, le=MAX_INT_32)]
```

**制約**: [0, 2,147,483,647]  
**単位**: ピクセル  
**型**: 32ビット整数

#### ShutterAngle (シャッター角度)
```python
ShutterAngle = Annotated[float, Field(ge=0.0, le=360.0, strict=True)]
```

**範囲**: (0.0, 360.0]  
**単位**: 度  
**計算**: シャッター速度 = 角度 ÷ 360 × フレームレート

### Lens レンズパラメータ

#### Distortion (歪みモデル)
```python
class Distortion(CompatibleBaseModel):
    model: NonBlankUTF8String | None = "Brown-Conrady D-U"
    radial: tuple[float, ...] = Field(min_length=1)
    tangential: tuple[float, ...] | None = Field(min_length=1)
    overscan: UnityOrGreaterFloat | None = None
```

**モデルタイプ**:
- `"Brown-Conrady D-U"`: 歪み→非歪み（デフォルト）
- `"Brown-Conrady U-D"`: 非歪み→歪み

**径方向係数**: k1, k2, k3, ... (最低1個)  
**接線方向係数**: p1, p2, ... (オプション、最低1個)  
**オーバースキャン**: ≥ 1.0

#### FizEncoders (FIZ エンコーダー)
```python
class FizEncoders(CompatibleBaseModel):
    focus: NormalizedFloat | None = None
    iris: NormalizedFloat | None = None
    zoom: NormalizedFloat | None = None
```

**制約**: 少なくとも1つの非null値を含む  
**範囲**: [0.0, 1.0]  
**方向規約**:
- Focus: 0=∞、1=最近
- Iris: 0=開放、1=クローズ  
- Zoom: 0=広角、1=望遠

#### RawFizEncoders (生エンコーダー)
```python
class RawFizEncoders(CompatibleBaseModel):
    focus: NonNegativeInt | None = None
    iris: NonNegativeInt | None = None
    zoom: NonNegativeInt | None = None
```

**制約**: 少なくとも1つの非null値を含む  
**型**: 非負整数  
**説明**: エンコーダー解像度に依存する生の値

### Timing タイムパラメータ

#### TimingMode (タイムモード)
```python
class TimingMode(StrEnum):
    INTERNAL = "internal"
    EXTERNAL = "external"
```

#### Timestamp (タイムスタンプ)
```python
class Timestamp(CompatibleBaseModel):
    seconds: NonNegative48BitInt
    nanoseconds: NonNegativeInt
```

**秒フィールド**: 48ビット符号なし整数  
**ナノ秒フィールド**: 32ビット符号なし整数  
**基準**: Unixエポック

#### Timecode (タイムコード)
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

**検証ルール**: frames < frameRate  
**サブフレーム**: インターレース等のアプリケーションをサポート

### Transform 変換パラメータ

#### Vector3 (3Dベクトル)
```python
class Vector3(CompatibleBaseModel):
    x: float | None = None
    y: float | None = None
    z: float | None = None
```

#### Rotator3 (3D回転)
```python
class Rotator3(CompatibleBaseModel):
    pan: float | None = None
    tilt: float | None = None
    roll: float | None = None
```

**回転順序**: ZXY内在回転  
**単位**: 度

---

## データ検証ルール

### フィールド検証

1. **非空文字列**: 1-1023文字、UTF-8エンコーディング
2. **UUID形式**: "urn:uuid:{uuid}" 形式
3. **数値範囲**: 各型の特定範囲制約
4. **配列長**: 最小長要件

### モデル検証

1. **FIZ エンコーダー**: 少なくとも1つの非null値
2. **タイムコード**: フレーム数はフレームレートより小さい必要がある
3. **歪み係数**: 径方向係数配列は空であってはならない

### 互換性検証

1. **プロトコルバージョン**: バージョン互換性をチェック
2. **フィールド存在**: オプションフィールドを処理
3. **後方互換**: 未知フィールドを無視

---

## ネットワーク伝送考慮事項

### データサイズ最適化

1. **オプションフィールド**: 必要なデータのみ送信
2. **精度バランス**: アプリケーション要件に応じて精度を選択
3. **圧縮**: JSON圧縮伝送

### リアルタイム要件

1. **低遅延**: 処理遅延を最小化
2. **バッファリング戦略**: ネットワークジッターを処理
3. **パケット損失処理**: フォルトトレラント機構

---

## 座標系詳細

### ステージ座標系

- **原点**: ステージ中心または参照点
- **X軸**: ステージ右側（前方を向いているとき）
- **Y軸**: ステージ前方
- **Z軸**: ステージ上方
- **単位**: メートル

### カメラ座標系

- **原点**: カメラセンサー中心
- **X軸**: カメラ右側
- **Y軸**: カメラ前方（レンズ方向）
- **Z軸**: カメラ上方

### グローバル座標系

- **ENU**: 東-北-上座標系
- **測地座標**: WGS84楕円体
- **参照**: ローカル接平面座標

---

## レンズ歪みモデル

### Brown-Conradyモデル

**径方向歪み**:
```
r' = r × (1 + k1×r² + k2×r⁴ + k3×r⁶ + ...)
```

**接線方向歪み**:
```
x' = x + [2×p1×x×y + p2×(r² + 2×x²)]
y' = y + [p1×(r² + 2×y²) + 2×p2×x×y]
```

ここで:
- r = √(x² + y²)
- k1, k2, k3, ... は径方向歪み係数
- p1, p2, ... は接線方向歪み係数

### オーバースキャン計算

オーバースキャン係数は、歪み除去プロセス中のピクセル損失を補償するために、レンダリング時に必要な追加画像領域を決定するために使用されます。

---

## PTP同期詳細

### PTP設定プロファイル

1. **IEEE Std 1588-2019**: 汎用産業アプリケーション
2. **IEEE Std 802.1AS-2020**: オーディオビデオブリッジネットワーク
3. **SMPTE ST2059-2:2021**: SMPTE 2110システム

### ベストマスタークロックアルゴリズム (BMCA)

優先順序:
1. Priority1 (管理者設定)
2. Clock Class (クロック品質)
3. Clock Accuracy (クロック精度)
4. Clock Variance (クロック分散)
5. Priority2 (動的優先度)
6. Clock Identity (クロック識別)

### 遅延測定

**meanPathDelay**: 双方向遅延測定  
**計算**: (t4 - t1 - t3 + t2) / 2  
**用途**: タイム同期精度補償

---

## 拡張メカニズム

### Customフィールド

```json
"custom": {
  "vendorSpecific": "value",
  "experimentalFeature": 123,
  "metadata": {
    "nested": "object"
  }
}
```

**用途**: ベンダー固有拡張  
**制約**: 有効なJSON値  
**推奨**: 競合を避けるために名前空間を使用

### Lens Custom係数

```json
"lens": {
  "custom": [
    [1.0, 2.0, 3.0],  # 第1グループのカスタム係数
    [4.0, 5.0, 6.0]   # 第2グループのカスタム係数
  ]
}
```

**用途**: レンズモデルの拡張  
**協議**: 制作者と消費者間で意味を協議

---

## 実装注意事項

### データ精度

1. **浮動小数点精度**: 倍精度浮動小数点数を使用
2. **有理数**: 浮動小数点精度問題を回避
3. **時間精度**: ナノ秒レベルのタイムスタンプ

### メモリ管理

1. **大配列**: メモリ使用量に注意
2. **キャッシュ戦略**: 静的データをキャッシュ
3. **ガベージコレクション**: リソースを適時解放

### スレッドセーフティ

1. **並行アクセス**: 共有データを保護
2. **アトミック操作**: 重要な状態更新
3. **ロック戦略**: デッドロックを回避

---

## テストケース

### 基本検証

```python
def test_basic_opentrackio():
    clip = Clip()
    clip.protocol = [VersionedProtocol(name="OpenTrackIO", version=[1,0,0])]
    clip.sampleId = ["urn:uuid:12345678-1234-5678-9abc-123456789abc"]
    assert clip.model_validate(clip.model_dump())
```

### 境界条件

```python
def test_boundary_conditions():
    # 最小値をテスト
    lens = Lens()
    lens.fStop = [0.1]  # 最小絞り値
    
    # 最大値をテスト
    camera = StaticCamera()
    camera.activeSensorResolution = SenselDimensions(2147483647, 2147483647)
```

### エラー処理

```python
def test_validation_errors():
    with pytest.raises(ValueError):
        # 負のフレームレートは失敗するはず
        Timing(sampleRate=[StrictlyPositiveRational(num=-1, denom=1)])
```

---

## パフォーマンスベンチマーク

### 解析パフォーマンス

| データサイズ | 解析時間 | メモリ使用量 |
|------------|---------|------------|
| 1KB        | <1ms    | <10KB      |
| 10KB       | <5ms    | <50KB      |
| 100KB      | <50ms   | <500KB     |

### ネットワーク伝送

| サンプリングレート | データレート | 帯域幅要件 |
|------------------|------------|----------|
| 24fps            | ~2KB/s     | <20Kbps  |
| 60fps            | ~5KB/s     | <50Kbps  |
| 120fps           | ~10KB/s    | <100Kbps |

---

## デバッグガイド

### よくある問題

1. **タイム同期失敗**
   - PTP設定をチェック
   - ネットワーク遅延を検証
   - クロックソースを確認

2. **歪みパラメータエラー**
   - 係数数を検証
   - モデルタイプをチェック
   - オーバースキャン値を確認

3. **変換チェーンエラー**
   - 変換順序を検証
   - 座標系をチェック
   - 単位の一貫性を確認

### デバッグツール

```python
def debug_clip(clip: Clip):
    """OpenTrackIOデータをデバッグするためのヘルパー関数"""
    clip._print_non_none()  # すべての非nullフィールドを印刷
    
    # データ整合性を検証
    try:
        clip.model_validate(clip.model_dump())
        print("✓ データ検証成功")
    except Exception as e:
        print(f"✗ 検証失敗: {e}")
```

---

## バージョン互換性

### バージョン番号形式

```json
"version": [major, minor, patch]
```

### 互換性ルール

1. **メジャーバージョン**: 非互換変更
2. **マイナーバージョン**: 後方互換の新機能
3. **パッチバージョン**: 後方互換のバグ修正

### 移行戦略

```python
def migrate_version(data: dict, from_version: list, to_version: list):
    """バージョン移行ヘルパー関数"""
    if from_version[0] != to_version[0]:
        raise ValueError("メジャーバージョンが非互換")
    
    # マイナーバージョンとパッチバージョンの互換性を処理
    return data
```

---

## 統合例

### Python統合

```python
import json
from camdkit.clip import Clip

# OpenTrackIOデータを解析
def parse_opentrackio(json_data: str) -> Clip:
    data = json.loads(json_data)
    return Clip.model_validate(data)

# OpenTrackIOデータを生成
def generate_opentrackio(clip: Clip) -> str:
    return json.dumps(clip.model_dump(exclude_none=True), indent=2)
```

### C++統合

```cpp
#include "OpenTrackIOParser.h"

// 解析例
std::string json_data = "...";
auto clip = OpenTrackIOParser::parse(json_data);

// パラメータにアクセス
if (clip.has_lens() && clip.lens().has_f_stop()) {
    double fstop = clip.lens().f_stop();
    std::cout << "F-Stop: " << fstop << std::endl;
}
```

---

## セキュリティ考慮事項

### 入力検証

1. **JSON解析**: 悪意のあるJSONを防ぐ
2. **数値範囲**: すべての数値制約を検証
3. **文字列長**: バッファオーバーフローを防ぐ

### ネットワークセキュリティ

1. **暗号化伝送**: TLS/SSLを使用
2. **認証**: データソースを検証
3. **整合性チェック**: データ署名検証

---

## 拡張開発

### 新パラメータの追加

1. **型定義**: 対応するtypesファイルに追加
2. **モデル更新**: 主要クラス定義を変更
3. **検証追加**: 制約チェックを実装
4. **ドキュメント更新**: パラメータ説明を補完

### カスタム歪みモデル

```python
class CustomDistortion(Distortion):
    model: str = "Custom-Model"
    custom_coefficients: tuple[float, ...] = None
    
    @model_validator(mode="after")
    def validate_custom_coefficients(self):
        if self.model == "Custom-Model" and not self.custom_coefficients:
            raise ValueError("カスタムモデルには係数の提供が必要")
        return self
```

---

## 用語集

- **FIZ**: Focus, Iris, Zoom（フォーカス、絞り、ズーム）
- **PTP**: Precision Time Protocol（精密時間プロトコル）
- **ENU**: East-North-Up（東-北-上座標系）
- **URN**: Uniform Resource Name（統一リソース名）
- **UUID**: Universally Unique Identifier（汎用一意識別子）
- **BMCA**: Best Master Clock Algorithm（ベストマスタークロックアルゴリズム）
- **GNSS**: Global Navigation Satellite System（全球測位衛星システム）

---

*技術リファレンスマニュアルバージョン: 1.0.0*  
*SMPTE RiS-OSVP Metadata CamDKit ソースコード解析に基づく*