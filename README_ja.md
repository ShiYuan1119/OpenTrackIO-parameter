# OpenTrackIO パラメータ解析ドキュメント

SMPTE RiS-OSVP Metadata CamDKit 公式仕様に基づく詳細解析

## ドキュメント構造

### 📚 主要ドキュメント

1. **[OpenTrackIO_Parameters_Documentation_ja.md](./OpenTrackIO_Parameters_Documentation_ja.md)**
   - 完全なパラメータ定義と説明
   - 各パラメータの詳細な説明、型、制約
   - 日本語コメントと補足説明を含む

2. **[OpenTrackIO_Technical_Reference_ja.md](./OpenTrackIO_Technical_Reference_ja.md)**
   - 技術実装の詳細
   - データ型定義と検証ルール
   - パフォーマンス最適化と統合ガイド

3. **[OpenTrackIO_Parameter_Quick_Reference_ja.md](./OpenTrackIO_Parameter_Quick_Reference_ja.md)**
   - パラメータクイック検索表
   - カテゴリ別に整理されたパラメータリスト
   - 検証チェックリスト

4. **[OpenTrackIO_Examples_and_Use_Cases_ja.md](./OpenTrackIO_Examples_and_Use_Cases_ja.md)**
   - 実際の応用シナリオの例
   - 異なる撮影環境での設定
   - デバッグとトラブルシューティングガイド

### 🔧 ソースコードファイル

- **`camdkit/`**: 公式リポジトリからコピーされたコアPythonパッケージ
  - `clip.py`: メインデータモデル定義
  - `lens_types.py`: レンズパラメータタイプ
  - `camera_types.py`: カメラパラメータタイプ
  - `timing_types.py`: タイム同期タイプ
  - `tracker_types.py`: トラッカータイプ
  - `transform_types.py`: 空間変換タイプ

- **`examples/`**: 公式サンプルJSONファイル
  - `complete_static_example.json`: 完全な静的例
  - `complete_dynamic_example.json`: 完全な動的例
  - `recommended_static_example.json`: 推奨静的例
  - `recommended_dynamic_example.json`: 推奨動的例

## OpenTrackIO 概要

OpenTrackIOは、SMPTE RiS-OSVPワーキンググループによって開発されたオープンソースプロトコルで、バーチャルプロダクション環境でのリアルタイムメタデータ交換に使用されます。このプロトコルは、以下の情報を含むJSONデータ構造を定義します：

### コアコンポーネント

- **🎥 Camera**: カメラハードウェア情報と設定
- **🔍 Lens**: レンズパラメータと光学特性
- **📡 Tracker**: トラッキングシステムの状態と設定
- **⏱️ Timing**: タイム同期とサンプリング情報
- **📐 Transforms**: 空間変換と座標系
- **🌍 GlobalStage**: グローバル座標位置決定
- **⚙️ Custom**: カスタム拡張フィールド

### 主な特徴

- ✅ **精密タイム同期**: PTP、NTPなどの複数のタイムソースをサポート
- ✅ **柔軟なデータ構造**: 静的パラメータと動的パラメータの分離
- ✅ **多様な歪みモデル**: Brown-Conradyなどの標準モデル
- ✅ **変換チェーン合成**: 複雑な多軸運動システムをサポート
- ✅ **拡張性**: ベンダー固有機能のカスタムフィールドサポート
- ✅ **バージョン互換性**: 明確なバージョン管理メカニズム

## クイックスタート

### 基本使用例

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

### Python統合

```python
from camdkit.clip import Clip
import json

# OpenTrackIOデータの解析
data = json.loads(opentrackio_json)
clip = Clip.model_validate(data)

# パラメータへのアクセス
camera_position = clip.transforms[0].translation
lens_fstop = clip.lens.f_stop[0]
```

## 参考資料

- 📖 [OpenTrackIO 公式ドキュメント](https://ris-pub.smpte.org/ris-osvp-metadata-camdkit/)
- 📖 [OpenLensIO 仕様](https://ris-pub.smpte.org/ris-osvp-metadata-camdkit/res/OpenLensIO_v1-0-0.pdf)
- 💻 [SMPTE GitHubリポジトリ](https://github.com/SMPTE/ris-osvp-metadata-camdkit)
- 📋 [SMPTE ST2059-2:2021](https://ieeexplore.ieee.org/document/9450292) (PTP設定プロファイル)

## 貢献

このドキュメントは、SMPTE RiS-OSVP Metadata CamDKitのソースコード解析に基づいて生成され、日本語ユーザーに詳細なOpenTrackIOパラメータリファレンスを提供することを目的としています。

### 更新履歴

- **v1.0.0** (2024): 初期バージョン、CamDKitソースコード解析に基づく
- すべてのコアパラメータ定義をカバー
- 実際の応用例を含む
- 技術実装ガイドを提供

---

*ドキュメント生成時期: 2024年*  
*SMPTE RiS-OSVP Metadata CamDKit v0.9.3 に基づく*