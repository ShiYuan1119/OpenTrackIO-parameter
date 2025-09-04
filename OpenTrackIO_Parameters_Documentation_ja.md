# OpenTrackIO パラメータ詳細ドキュメント

SMPTE RiS-OSVP Metadata CamDKit 公式仕様に基づく

## 概要

OpenTrackIOは、SMPTE RiS-OSVPワーキンググループによって開発された無料のオープンソースプロトコルで、バーチャルプロダクション（Virtual Production）環境でのメタデータ交換専用に設計されています。このプロトコルは、カメラ、レンズ、トラッカー、変換情報を含むJSONデータ構造を定義します。

**プロトコルバージョン**: v1.0.0  
**仕様ソース**: [SMPTE RiS-OSVP Metadata CamDKit](https://github.com/SMPTE/ris-osvp-metadata-camdkit)

---

## データ構造概要

OpenTrackIOデータには以下の主要部分が含まれます：

- **`static`**: 静的情報（継続時間、カメラ、レンズ、トラッカーの固定属性）
- **`tracker`**: トラッカー状態情報
- **`timing`**: タイム同期情報
- **`lens`**: レンズ動的パラメータ
- **`protocol`**: プロトコル識別情報
- **`transforms`**: 空間変換情報
- **`globalStage`**: グローバルステージ座標
- **`custom`**: カスタム拡張フィールド

---

## 1. Static 静的情報

### 1.1 Duration (継続時間)

```json
"duration": {
  "num": 1,
  "denom": 25
}
```

**説明**: セグメントの継続時間  
**型**: 厳密正有理数  
**単位**: 秒  
**制約**: 正数でなければならない  
**サンプリング**: 静的

### 1.2 Camera カメラ静的情報

#### 1.2.1 captureFrameRate (キャプチャフレームレート)

```json
"captureFrameRate": {
  "num": 24000,
  "denom": 1001
}
```

**説明**: カメラのキャプチャフレームレート  
**型**: 厳密正有理数  
**単位**: ヘルツ (Hz)  
**制約**: 正数でなければならない  
**説明**: 29.97 fps (30000/1001) などのドロップフレームレートの正確な表現をサポート

#### 1.2.2 activeSensorPhysicalDimensions (センサー物理寸法)

```json
"activeSensorPhysicalDimensions": {
  "height": 24.0,
  "width": 36.0
}
```

**説明**: カメラセンサーアクティブエリアの物理寸法  
**型**: 物理寸法オブジェクト  
**単位**: ミリメートル  
**制約**: 高さと幅は非負実数でなければならない

#### 1.2.3 activeSensorResolution (センサー解像度)

```json
"activeSensorResolution": {
  "height": 2160,
  "width": 3840
}
```

**説明**: カメラセンサーアクティブエリアのピクセル解像度  
**型**: センサー寸法オブジェクト  
**単位**: ピクセル  
**制約**: 高さと幅は [0..2,147,483,647] 範囲の整数でなければならない

#### 1.2.4 デバイス識別情報

```json
"make": "CameraMaker",
"model": "Model20",
"serialNumber": "1234567890A",
"firmwareVersion": "1.2.3",
"label": "A"
```

**説明**: カメラデバイスの識別情報  
**型**: 非空UTF-8文字列  
**制約**: 各フィールドは最大1023文字、空であってはならない

- **make**: カメラ製造業者名
- **model**: カメラモデル
- **serialNumber**: カメラシリアル番号（一意識別）
- **firmwareVersion**: ファームウェアバージョン
- **label**: ユーザー定義のカメラ識別子

#### 1.2.5 anamorphicSqueeze (アナモルフィックレンズ圧縮比)

```json
"anamorphicSqueeze": {
  "num": 1,
  "denom": 1
}
```

**説明**: アナモルフィックレンズの公称圧縮比  
**型**: 厳密正有理数  
**制約**: 正数でなければならない  
**説明**: 軸整列正方形がカメラセンサーに投影される際のアスペクト比を表す

#### 1.2.6 isoSpeed (ISO感度)

```json
"isoSpeed": 4000
```

**説明**: ISO 12232で定義された算術ISO感度  
**型**: 厳密正整数  
**制約**: 正整数でなければならない

#### 1.2.7 fdlLink (FDLリンク)

```json
"fdlLink": "urn:uuid:62ea03ac-ce56-43c6-ab8d-a9ec8a9ee7b3"
```

**説明**: カメラが使用するASCフレーム決定リストを識別するURN  
**型**: UUID URN  
**制約**: 有効なUUID URN形式でなければならない

#### 1.2.8 shutterAngle (シャッター角度)

```json
"shutterAngle": 45.0
```

**説明**: キャプチャフレームレートの分数として表現されるシャッター速度  
**型**: 実数  
**単位**: 度  
**制約**: (0..360] 範囲内でなければならない  
**説明**: シャッター速度（1/s）= パラメータ値 ÷ 360 × キャプチャフレームレート

### 1.3 Lens レンズ静的情報

#### 1.3.1 distortionOverscanMax (歪みオーバースキャン最大値)

```json
"distortionOverscanMax": 1.2
```

**説明**: レンズ歪みの静的最大オーバースキャン係数  
**型**: 1以上の実数  
**制約**: ≥ 1.0 でなければならない  
**説明**: フレームごとに動的オーバースキャン値を提供する代替案

#### 1.3.2 undistortionOverscanMax (歪み除去オーバースキャン最大値)

```json
"undistortionOverscanMax": 1.3
```

**説明**: レンズ歪み除去の静的最大オーバースキャン係数  
**型**: 1以上の実数  
**制約**: ≥ 1.0 でなければならない

#### 1.3.3 レンズデバイス情報

```json
"make": "LensMaker",
"model": "Model15",
"serialNumber": "1234567890A"
```

**説明**: レンズデバイス識別情報  
**型**: 非空UTF-8文字列  
**制約**: 各フィールドは最大1023文字

#### 1.3.4 nominalFocalLength (公称焦点距離)

```json
"nominalFocalLength": 14.0
```

**説明**: レンズの公称焦点距離  
**型**: 厳密正実数  
**単位**: ミリメートル  
**説明**: 固定焦点レンズの側面に記載される値、ズームレンズでは未定義

#### 1.3.5 calibrationHistory (キャリブレーション履歴)

```json
"calibrationHistory": [
  "LensMaker 123",
  "TrackerMaker 123"
]
```

**説明**: レンズキャリブレーション履歴を説明する自由文字列のリスト  
**型**: 文字列配列  
**制約**: 各文字列は1-1023文字

### 1.4 Tracker トラッカー静的情報

```json
"tracker": {
  "make": "TrackerMaker",
  "model": "Tracker",
  "serialNumber": "1234567890A",
  "firmwareVersion": "1.2.3"
}
```

**説明**: トラッキングデバイスの製造業者とデバイス情報  
**型**: 非空UTF-8文字列  
**制約**: 各フィールドは最大1023文字

---

## 2. Tracker トラッカー状態

```json
"tracker": {
  "notes": "Example generated sample.",
  "recording": false,
  "slate": "A101_A_4",
  "status": "Optical Good"
}
```

### 2.1 notes (備考)

**説明**: トラッキングシステムに関する備考情報  
**型**: 非空UTF-8文字列  
**制約**: 最大1023文字  
**サンプリング**: 通常

### 2.2 recording (録画状態)

**説明**: トラッキングシステムがデータを録画しているかどうかを示す  
**型**: ブール値  
**制約**: true または false  
**サンプリング**: 通常

### 2.3 slate (スレート)

**説明**: 録画スレートを説明する文字列  
**型**: 非空UTF-8文字列  
**制約**: 最大1023文字  
**サンプリング**: 通常

### 2.4 status (状態)

**説明**: トラッキングシステムの状態を説明する文字列  
**型**: 非空UTF-8文字列  
**制約**: 最大1023文字  
**サンプリング**: 通常  
**一般的な値**: "Optical Good", "Optical Poor", "IMU Only", "No Data"

---

## 3. Timing タイム同期

### 3.1 mode (タイムモード)

```json
"mode": "internal"
```

**説明**: サンプル伝送メカニズムが固有のタイム同期を提供するかどうかを示す  
**型**: 列挙値  
**選択肢**:
- `"internal"`: 伝送メカニズムに固有の時間がなく、サンプルにPTPタイムスタンプを含める必要がある
- `"external"`: 伝送メカニズムが固有のタイム同期を提供  
**サンプリング**: 通常

### 3.2 recordedTimestamp (録画タイムスタンプ)

```json
"recordedTimestamp": {
  "seconds": 1718806000,
  "nanoseconds": 500000000
}
```

**説明**: データ録画時点のPTPタイムスタンプ  
**型**: タイムスタンプオブジェクト  
**単位**: 秒  
**制約**:
- `seconds`: 48ビット符号なし整数（エポックからの秒数）
- `nanoseconds`: 32ビット符号なし整数  
**サンプリング**: 通常

### 3.3 sampleRate (サンプリングレート)

```json
"sampleRate": {
  "num": 24000,
  "denom": 1001
}
```

**説明**: 有理数として表現されるサンプルフレームレート  
**型**: 厳密正有理数  
**制約**: 正数でなければならない  
**説明**: 29.97などのドロップフレームレートは30000/1001として表現すべき

### 3.4 sampleTimestamp (サンプルタイムスタンプ)

```json
"sampleTimestamp": {
  "seconds": 1718806554,
  "nanoseconds": 500000000
}
```

**説明**: データキャプチャ時点のPTPタイムスタンプ  
**型**: タイムスタンプオブジェクト  
**説明**: データパケット伝送のPTPタイムスタンプとは異なる場合がある

### 3.5 sequenceNumber (シーケンス番号)

```json
"sequenceNumber": 0
```

**説明**: 各サンプルとともに増加する整数  
**型**: 非負整数  
**制約**: ≥ 0  
**サンプリング**: 通常

### 3.6 synchronization (同期情報)

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

#### 同期パラメータ詳細

- **`locked`**: トラッキングデバイスが同期ソースにロックされているかどうか
- **`source`**: 同期ソースタイプ
  - `"genlock"`: 外部ブラック/カラーバースト信号
  - `"videoIn"`: 外部ビデオ信号
  - `"ptp"`: PTPマスタークロック
  - `"ntp"`: NTPサーバー
- **`frequency`**: 同期信号周波数（サンプリングフレームレートと異なる場合がある）
- **`present`**: 同期ソースが存在するかどうか
- **`ptp`**: PTP固有情報
  - **`profile`**: PTP設定プロファイル（例："SMPTE ST2059-2:2021"）
  - **`domain`**: PTPドメイン識別子
  - **`leaderIdentity`**: PTPマスタークロックの一意識別子（通常MACアドレス）
  - **`leaderPriorities`**: マスタークロック優先度
    - **`priority1`**: 管理者設定の静的優先度
    - **`priority2`**: 役割またはクロック品質に基づく動的優先度
  - **`leaderAccuracy`**: 時間オフセット精度（秒）
  - **`leaderTimeSource`**: マスタークロックのタイムソース（GNSS、原子時計、NTP）
  - **`meanPathDelay`**: デバイスとPTPマスタークロック間の平均往復遅延
  - **`vlan`**: PTPトラフィックのVLAN ID

### 3.7 timecode (タイムコード)

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

**説明**: サンプルのSMPTEタイムコード  
**制約**:
- `hours`: 0-23
- `minutes`: 0-59  
- `seconds`: 0-59
- `frames`: 0-119（フレームレートより小さくなければならない）
- `frameRate`: 厳密正有理数

---

## 4. Lens レンズパラメータ

### 4.1 custom (カスタム係数)

```json
"custom": [1.0, 2.0]
```

**説明**: 既存のレンズモデルを拡張するオプションのカスタム係数  
**型**: 浮動小数点数配列  
**説明**: 特定の制作者と消費者間で具体的な意味と応用方法を協議する必要がある

### 4.2 distortion (歪みパラメータ)

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

#### 歪みモデル詳細

- **`model`**: 歪みモデル名
  - `"Brown-Conrady D-U"`: 歪み座標から非歪み座標へのマッピング（デフォルト）
  - `"Brown-Conrady U-D"`: 非歪み座標から歪み座標へのマッピング
- **`radial`**: 径方向歪み係数 (k1-kN)
  - 最低1つの係数を含む必要がある
  - 通常k1, k2, k3などを含む
- **`tangential`**: 接線方向歪み係数 (p1-pN)
  - オプションパラメータ
  - 提供される場合、最低1つの係数を含む必要がある
- **`overscan`**: オーバースキャン係数
  - ≥ 1.0 でなければならない
  - 制作者が提供するか消費者が計算する

### 4.3 distortionOffset (歪みオフセット)

```json
"distortionOffset": {
  "x": 1.0,
  "y": 2.0
}
```

**説明**: バーチャルカメラ歪み中心のXおよびYオフセット  
**型**: 平面オフセットオブジェクト  
**単位**: ミリメートル  
**制約**: XとYは実数でなければならない

### 4.4 encoders (エンコーダー)

```json
"encoders": {
  "focus": 0.1,
  "iris": 0.2,
  "zoom": 0.3
}
```

**説明**: FIZエンコーダーの正規化値  
**型**: 正規化浮動小数点数 (0-1)  
**制約**: focus、iris、zoomの少なくとも1つを含む必要がある  
**方向定義**:
- **`focus`**: 0=無限遠、1=最短距離
- **`iris`**: 0=最大絞り（開放）、1=最小絞り（クローズ）
- **`zoom`**: 0=広角、1=望遠

### 4.5 entrancePupilOffset (入射瞳オフセット)

```json
"entrancePupilOffset": 0.123
```

**説明**: 公称撮像面に対する入射瞳のオフセット  
**型**: 実数  
**単位**: メートル  
**説明**: 正の値は入射瞳が被写体側にあることを示し、負の値は逆を示す

### 4.6 exposureFalloff (露出減衰)

```json
"exposureFalloff": {
  "a1": 1.0,
  "a2": 2.0,
  "a3": 3.0
}
```

**説明**: レンズ露出減衰（ビネッティング）を計算するための係数  
**型**: 露出減衰オブジェクト  
**制約**: 係数は実数でなければならない  
**説明**: a1は必須パラメータ、a2とa3はオプション

### 4.7 fStop (F値)

```json
"fStop": 4.0
```

**説明**: レンズの線形F値  
**型**: 厳密正実数  
**制約**: > 0 でなければならない  
**計算**: 焦点距離 ÷ 入射瞳直径

### 4.8 pinholeFocalLength (ピンホール焦点距離)

```json
"pinholeFocalLength": 24.305
```

**説明**: シンプルCGIピンホールカメラモデルにおけるピンホールと撮像面間の距離  
**型**: 厳密正実数  
**単位**: ミリメートル  
**制約**: > 0 でなければならない

### 4.9 focusDistance (フォーカス距離)

```json
"focusDistance": 10.0
```

**説明**: レンズのフォーカス距離/位置  
**型**: 厳密正実数  
**単位**: メートル  
**制約**: > 0 でなければならない

### 4.10 projectionOffset (投影オフセット)

```json
"projectionOffset": {
  "x": 0.1,
  "y": 0.2
}
```

**説明**: バーチャルカメラ透視投影中心のXおよびYオフセット  
**型**: 平面オフセットオブジェクト  
**単位**: ミリメートル  
**制約**: XとYは実数でなければならない

### 4.11 rawEncoders (生エンコーダー)

```json
"rawEncoders": {
  "focus": 1000,
  "iris": 2000,
  "zoom": 3000
}
```

**説明**: FIZエンコーダーの生の値  
**型**: 非負整数  
**制約**: focus、iris、zoomの少なくとも1つを含む必要がある  
**説明**: エンコーダー解像度に依存し、ゼロ化/レンジング前の値

### 4.12 tStop (T値)

```json
"tStop": 4.1
```

**説明**: レンズの線形T値  
**型**: 厳密正実数  
**制約**: > 0 でなければならない  
**計算**: F値 ÷ √(レンズ透過率)

---

## 5. Protocol プロトコル情報

```json
"protocol": {
  "name": "OpenTrackIO",
  "version": [0, 9, 3]
}
```

### 5.1 name (プロトコル名)

**説明**: プロトコル名識別子  
**型**: 文字列  
**固定値**: "OpenTrackIO"

### 5.2 version (プロトコルバージョン)

**説明**: プロトコルバージョン番号  
**型**: 整数配列  
**形式**: [メジャーバージョン, マイナーバージョン, パッチバージョン]  
**現在のバージョン**: [1, 0, 0]

---

## 6. 識別子

### 6.1 sampleId (サンプルID)

```json
"sampleId": "urn:uuid:e3f34dae-b68c-47d9-99af-e0bfc277723c"
```

**説明**: データ伝送サンプルの一意識別子  
**型**: UUID URN  
**形式**: "urn:uuid:{uuid}"  
**制約**: 有効なUUID形式でなければならない

### 6.2 sourceId (ソースID)

```json
"sourceId": "urn:uuid:889eec71-62ca-4c7f-b1ef-4c2b16f4eacf"
```

**説明**: データ伝送ソースの一意識別子  
**型**: UUID URN  
**形式**: "urn:uuid:{uuid}"

### 6.3 sourceNumber (ソース番号)

```json
"sourceNumber": 1
```

**説明**: ソースからのストリームインデックスを識別する番号  
**型**: 非負整数  
**制約**: ≥ 0  
**説明**: ソースが複数のサンプルストリームを生成する場合に特に重要

### 6.4 relatedSampleIds (関連サンプルID)

```json
"relatedSampleIds": [
  "urn:uuid:b545b989-34b2-452a-9630-cf952d021a3f",
  "urn:uuid:dd8ce967-21f2-444e-a007-5eb33333ca51"
]
```

**説明**: このサンプルに関連するサンプルIDのリスト  
**型**: UUID URN配列  
**説明**: 指定されたsampleIdのサンプルが存在することは保証されない

---

## 7. GlobalStage グローバルステージ座標

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

**説明**: グローバルENUおよび測地座標系におけるステージ原点の位置  
**型**: グローバル位置オブジェクト  
**単位**: メートル（E, N, U, h0）、度（lat0, lon0）  
**説明**: ステージが移動体内にある場合、このパラメータは動的である可能性がある

### 座標系詳細

- **E (East)**: 東方向座標（メートル）
- **N (North)**: 北方向座標（メートル）
- **U (Up)**: 上方向座標（メートル）
- **lat0**: 緯度（度）
- **lon0**: 経度（度）
- **h0**: 高度（メートル）

**参照**: [Local tangent plane coordinates](https://en.wikipedia.org/wiki/Local_tangent_plane_coordinates)

---

## 8. Transforms 空間変換

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

### 8.1 変換合成ルール

**説明**: 順序通りに合成される変換のリスト  
**合成順序**: リストの最初の変換から最後の変換まで  
**最終結果**: ステージ原点に対するカメラセンサーの位置（メートル）と方向（度）

### 8.2 座標系

- **Z軸**: 上向き
- **座標系**: 右手座標系
- **Y軸**: カメラ前方向（pan、tilt、rollがゼロのとき）
- **X軸**: カメラ右方向

### 8.3 translation (平行移動)

**説明**: 3D平行移動ベクトル  
**型**: Vector3オブジェクト  
**単位**: メートル  
**制約**: x、y、zは実数でなければならない

### 8.4 rotation (回転)

**説明**: オイラー角回転  
**型**: Rotator3オブジェクト  
**単位**: 度  
**回転順序**: 内在回転、ZXY軸まわりで測定  
**角度定義**:
- **`pan`**: ヨー角（Z軸まわり）
- **`tilt`**: ピッチ角（X軸まわり）
- **`roll`**: ロール角（Y軸まわり）

**オイラー角説明**:
- 人間が読みやすく、>360°または<0°の循環をサポート
- バーチャルカメラにはジンバルロックの物理的制限がない
- クォータニオンとの変換は簡単で、精度損失は受容可能

### 8.5 scale (スケール)

**説明**: 3Dスケールベクトル（オプション）  
**型**: Vector3オブジェクト  
**制約**: x、y、zは実数でなければならない

### 8.6 id (識別子)

**説明**: 変換の識別子  
**型**: 非空UTF-8文字列  
**一般的な値**: "Camera", "Dolly", "Crane Arm", "Gimbal"

---

## 9. Custom カスタムフィールド

```json
"custom": {
  "pot1": 2435,
  "button1": false
}
```

**説明**: ユーザー定義のカスタムフィールド  
**型**: 任意のJSON値  
**説明**: 特定のアプリケーション要件をサポートするために標準プロトコルを拡張することを許可

---

## 10. 実際の応用例

### 10.1 基本カメラトラッキング

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

### 10.2 多軸システム（クレーン + ジンバル）

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

## 11. データ型規約

### 11.1 有理数表現

```json
{
  "num": 24000,
  "denom": 1001
}
```

**用途**: フレームレート、継続時間などの正確な表現  
**利点**: 浮動小数点精度問題を回避  
**例**: 29.97 fps = 30000/1001

### 11.2 タイムスタンプ形式

```json
{
  "seconds": 1718806554,
  "nanoseconds": 500000000
}
```

**形式**: PTPタイムスタンプ  
**精度**: ナノ秒レベル  
**基準**: Unixエポック（1970-01-01 00:00:00 UTC）

### 11.3 UUID URN形式

```
"urn:uuid:e3f34dae-b68c-47d9-99af-e0bfc277723c"
```

**形式**: RFC 4122 UUID  
**プレフィックス**: "urn:uuid:"  
**用途**: グローバル一意識別子

---

## 12. ベストプラクティス

### 12.1 データ整合性

1. **必須フィールド**: `protocol`フィールドが常に存在することを確認
2. **バージョン互換性**: プロトコルバージョン互換性をチェック
3. **データ検証**: すべての制約条件を検証

### 12.2 パフォーマンス最適化

1. **静的データ**: 不変のパラメータを`static`部分に配置
2. **サンプリングレート**: アプリケーション要件に応じて適切なサンプリングレートを選択
3. **データ圧縮**: ネットワーク伝送時のデータ圧縮を考慮

### 12.3 同期戦略

1. **PTP優先**: サポートされているネットワークでPTPを優先使用
2. **タイムスタンプ一貫性**: すべてのデバイスが統一された時間基準を使用することを確認
3. **遅延補償**: ネットワークと処理遅延を考慮

---

## 13. エラー処理

### 13.1 一般的なエラー

1. **形式エラー**: JSON形式が正しくない
2. **型エラー**: パラメータ型が一致しない
3. **制約違反**: 数値が許可範囲を超過
4. **必須フィールド欠如**: 必要なパラメータが欠如

### 13.2 フォルトトレラント機構

1. **デフォルト値**: オプションパラメータに合理的なデフォルト値を提供
2. **データ降格**: 部分的なデータ損失時に実行を継続
3. **エラー報告**: 詳細なエラー情報を記録

---

## 参考資料

1. [SMPTE RiS-OSVP Metadata CamDKit](https://github.com/SMPTE/ris-osvp-metadata-camdkit)
2. [OpenTrackIO 公式ドキュメント](https://ris-pub.smpte.org/ris-osvp-metadata-camdkit/)
3. [OpenLensIO 仕様](https://ris-pub.smpte.org/ris-osvp-metadata-camdkit/res/OpenLensIO_v1-0-0.pdf)
4. [SMPTE ST2059-2:2021 PTP設定プロファイル](https://ieeexplore.ieee.org/document/9450292)

---

*このドキュメントはSMPTE RiS-OSVP Metadata CamDKitソースコード解析に基づいて生成されました、バージョン: 1.0.0*