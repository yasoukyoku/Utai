<p align="center">
  <a href="README.md">English</a> ·
  <a href="README_zh.md">中文</a> ·
  <a href="README_ja.md"><b>日本語</b></a>
</p>

<p align="center">
  <img src="icon.png" width="96" height="96" alt="Utai">
</p>

<h1 align="center">Utai</h1>

<p align="center">
  <b>AI ボイスチェンジャー＆歌声合成 DAW</b><br>
  <sub>音声分離、ボイスチェンジ、RVC トレーニング、AI 歌声合成、VST ホスティング</sub>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/version-1.8.3-7b68ee?style=flat-square" alt="version">
  <img src="https://img.shields.io/badge/platform-Windows-0078d4?style=flat-square" alt="platform">
  <img src="https://img.shields.io/badge/GPU-NVIDIA%2030%2F40%2F50-76b900?style=flat-square" alt="gpu">
  <img src="https://img.shields.io/badge/license-apache2.0-green?style=flat-square" alt="license">
  <img src="https://img.shields.io/badge/i18n-zh%20%7C%20en%20%7C%20ja-e8b84b?style=flat-square" alt="languages">
</p>

<p align="center">
  <a href="https://utaisynthesizer.net"><img src="https://img.shields.io/badge/Website-utaisynthesizer.net-ff69b4?style=for-the-badge&logo=google-chrome&logoColor=white" alt="Website"></a>
</p>

---

## 開発環境セットアップ

### 前提条件

| ツール | バージョン | 確認方法 | インストール |
|--------|-----------|---------|-------------|
| Python | 3.10.x | `py -3.10 --version` | [python.org/downloads](https://www.python.org/downloads/release/python-31011/)（「py launcher」を有効にすること） |
| Node.js | 20+ | `node --version` | [nodejs.org](https://nodejs.org/) |
| Git | 任意 | `git --version` | [git-scm.com](https://git-scm.com/download/win) |
| NVIDIA GPU | RTX 30/40/50 | `nvidia-smi` | 任意。CPU フォールバックあり |

### 1. リポジトリのクローン

```bash
git clone https://github.com/SaigyoYuyuko/Utai.git
cd Utai
```

リポジトリには MSST と RVC のパッチ済みソースコードが含まれています。それらを別途クローンする必要は**ありません**。

### 2. Python 環境

方法 A: セットアップスクリプトを使用（対話式 GPU 選択）：

```bash
setup_env.bat
```

方法 B: 手動（CUDA 12.1 の例）：

```bash
py -3.10 -m venv venv
venv\Scripts\activate
pip install --upgrade pip
pip install -r requirements-torch-cu121.txt --index-url https://download.pytorch.org/whl/cu121
pip install -r requirements.txt
```

その他の GPU オプション（ファイル名と `--index-url` の両方を変更すること）：
- CUDA 12.4: `requirements-torch-cu124.txt` + `--index-url https://download.pytorch.org/whl/cu124`
- CUDA 12.8: `requirements-torch-cu128.txt` + `--index-url https://download.pytorch.org/whl/cu128`
- CPU のみ: `requirements-torch-cpu.txt`（`--index-url` 不要）

> **fairseq**（ワークフローページの RVC ボイスチェンジに必要）は、MSST との omegaconf バージョン競合のため `--no-deps` でインストールする必要があります：
> ```bash
> venv\Scripts\activate
> pip install --no-deps fairseq>=0.12.2
> pip install hydra-core bitarray sacrebleu
> ```
> 注意: シンセページの RVC（Score2HuBERT 経由）は fairseq を必要としません。

### 3. モデルファイルのダウンロード

以下のバイナリモデルファイルは git に含めるには大きすぎるため、クローン後に手動でダウンロードしてください。

#### ffmpeg

[gyan.dev/ffmpeg/builds](https://www.gyan.dev/ffmpeg/builds/)（Windows ビルド）からダウンロードし、以下を実行：

```bash
mkdir bin
```

`ffmpeg.exe` と `ffprobe.exe` を `bin/` に配置してください。また、`ffmpeg.exe` のコピーを `RVC/RVC20240604Nvidia/` にも配置してください（RVC がローカルで必要とするため）。

#### RVC 事前学習モデル

[huggingface.co/lj1995/VoiceConversionWebUI](https://huggingface.co/lj1995/VoiceConversionWebUI/tree/main) からダウンロード：

```
RVC/RVC20240604Nvidia/assets/
  pretrained_v2/
    f0G32k.pth, f0G40k.pth, f0G48k.pth    (Generator with F0)
    f0D32k.pth, f0D40k.pth, f0D48k.pth    (Discriminator with F0)
    G32k.pth, G40k.pth, G48k.pth          (Generator without F0)
    D32k.pth, D40k.pth, D48k.pth          (Discriminator without F0)
  hubert/
    hubert_base.pt
  rmvpe/
    rmvpe.pt
```

#### MSST 分離モデル

分離モデルはアプリ内のモデルマネージャーからオンデマンドでダウンロードされます。`MSST/pretrain/` ディレクトリは `.gitkeep` ファイルとともに作成済みです。

#### DiffSinger ボイスバンク（シンセページ用）

ボイスバンクをダウンロードして `synth/utai-renderer/Singers/` に配置してください。[lunaiproject/lunai_singers/releases](https://github.com/lunaiproject/lunai_singers/releases) で入手できます。

#### UtaiRenderer（シンセページ用）

レンダラーは .NET 8 自己完結型バイナリです。ソースからビルドする場合：

```bash
:: .NET 8 SDK が必要: https://dotnet.microsoft.com/download/dotnet/8.0

:: OpenUtau フォークをクローン
cd synth
git clone --depth 1 https://github.com/keirokeer/OpenUtau-DiffSinger-Lunai.git

:: ビルド（ソースからビルドする場合は synth/utai-renderer-src/ のアクセス修飾子パッチを適用）
cd utai-renderer-src
dotnet publish -c Release -r win-x64 --self-contained true -o "../utai-renderer"
cd ../..
```

### 4. フロントエンド

```bash
cd app\frontend
npm install
npm run build
cd ..\..

:: ビルド済みフロントエンドをバックエンドの静的配信ディレクトリにコピー
xcopy /e /y app\frontend\dist\* app\backend\static\
```

### 5. 開発サーバーの起動

```bash
run.bat
```

venv をアクティベートし、`bin/` を PATH に追加して、FastAPI バックエンドを `http://localhost:8765` で起動します。

フロントエンドのホットリロード開発（別のターミナルで実行）：

```bash
cd app\frontend
set VITE_API_TARGET=http://127.0.0.1:18090
npm run dev
```

> Vite 開発サーバーは API リクエストをバックエンドにプロキシします。バックエンドのポートを変更した場合は `VITE_API_TARGET` を設定してください。

### 6. Electron（任意）

```bash
cd electron
npm install

:: 開発モード（実行中のバックエンドに接続）：
npm start

:: ポータブルアプリのビルド（フロントエンドビルド後）：
cd ..
rebuild_utai.bat
```

### セットアップの確認

手順 1〜5 の完了後、ブラウザで `http://localhost:8765` を開いてください。Utai のワークフローエディタが表示されます。主な確認項目：

- **ワークフローページ**: Audio Input ノードをドラッグしてファイルを読み込む
- **トレーニングページ**: サイドバーからアクセス可能
- **シンセページ**: UtaiRenderer + 最低 1 つの DiffSinger ボイスバンクが必要

---

## Score2HuBERT トレーニングセットアップ

Score2HuBERT は、楽譜から HuBERT 特徴量と F0 を直接生成するニューラルネットワークであり、DiffSinger の従来の音響モデルを置き換えるものです。学習済みモデルはシンセページのボーカルレンダリングに使用されます。ソースコードは `score2hubert/`（v1、実験版）と `score2hubert_v2/`（v2、本番版）にあります。

> このセクションはモデルを再学習する場合のみ必要です。通常の Utai 開発では、事前学習済みモデルのチェックポイントだけで十分です。

### 追加の前提条件

| ツール | 用途 |
|--------|------|
| .NET 8 SDK | S2H 自体には不要だが、UtaiRenderer 統合に必要 |
| ディスク空き容量 ~30GB | 学習データ（ACE-Opencpop + 日本語歌唱データセット） |
| VRAM 10GB 以上の NVIDIA GPU | 学習用（オリジナルの学習には RTX 3080 Ti を使用） |

### セットアップ

```bash
:: ソースコードはリポジトリに含まれている。S2H 用に別の venv を作成：
cd score2hubert_v2
py -3.10 -m venv venv
venv\Scripts\activate
pip install -r ../score2hubert/requirements.txt
```

### 学習データのダウンロード

データセットはリポジトリに含まれていません。完全なソースリストとダウンロードリンクは `score2hubert_v2/docs/DATA_CREDITS.md` を参照してください。

| データセット | 言語 | サイズ | ソース |
|-------------|------|--------|--------|
| ACE-Opencpop | 中国語 | ~100h | HuggingFace の `espnet/ace-opencpop-segments` |
| Kiritan Singing DB | 日本語 | ~1h | zunko.jp |
| Itako Singing DB | 日本語 | ~0.8h | zunko.jp |
| ONIKU KURUMI | 日本語 | ~1h | onikuru.info |
| OFUTON P | 日本語 | ~0.8h | (同系列の作者) |
| Natsume Singing DB | 日本語 | ~0.9h | ksdcm1ng.wixsite.com |

生データセットを `score2hubert/data/raw/` に配置し、前処理を実行：

```bash
cd score2hubert_v2
venv\Scripts\activate

:: 各データセットの前処理（きりたんの例）：
python scripts/preprocess_kiritan_itako.py

:: 各データセットのプリプロセッサは scripts/preprocess_*.py を参照
```

### SOFA 強制アライメント（任意）

SOFA は正確な音素境界アライメントに使用されます。別途クローンしてください：

```bash
:: プロジェクトルートから：
git clone https://github.com/qiuqiao/SOFA.git SOFA
cd SOFA

:: 事前学習済みアライナーチェックポイントをダウンロード：
::   - 中国語: ckpt/Mandarin/pretrained_mandarin_singing/
::   - 日本語: ckpt/JPN_Test2_Plus/
:: ダウンロード手順は SOFA リポジトリを参照。

:: ACE データを SOFA-Mandarin で再アライメント：
cd ../score2hubert_v2
bash scripts/run_sofa_mandarin_ace.sh
python scripts/preprocess_sofa_realign_ace.py --workers 8

:: 日本語データを SOFA-Greenleaf で再アライメント：
python scripts/preprocess_sofa_realign.py
```

### 学習

```bash
cd score2hubert_v2
venv\Scripts\activate

:: 学習を開始（Run 9 設定、出荷済み構成）：
set PYTHONUNBUFFERED=1
set PYTHONIOENCODING=utf-8
python -u scripts/train.py --config configs/train_run9.yaml

:: チェックポイントから再開：
python -u scripts/train.py --config configs/train_run9.yaml --resume checkpoints/latest.pt
```

学習は RTX 3080 Ti で 75K ステップにつき約 12 時間かかります。アーキテクチャの詳細と全 9 回の学習ランのセッションログは `score2hubert_v2/docs/` を参照してください。

### 評価

```bash
:: 言語別メトリクス：
python scripts/eval_per_lang_v2.py --ckpt checkpoints/best.pt --config configs/train_run9.yaml

:: リスニング評価用のオーディオ合成：
python scripts/eval_audio.py --ckpt checkpoints/best.pt --config configs/train_run9.yaml

:: ポストミュート処理の適用：
python scripts/apply_midi_mute.py --input test_outputs/ --output test_outputs_muted/
```

### モデルアーキテクチャ

- **パラメータ数**: 54.63M
- **エンコーダ**: 5 層 Conformer（512 次元隠れ層、8 ヘッド）
- **デコーダ**: 6 ブロック決定論的 HuBERT 回帰ヘッド（768 次元出力）
- **F0 予測器**: デュアルモード（歌唱/ラップ）、エンコーダ特徴量に条件付け
- **入力**: IPA 音素 + MIDI ピッチ + 持続時間
- **出力**: HuBERT Layer 12 特徴量（768 次元、100fps） + F0 コンター
- **学習データ**: ACE-Opencpop（中国語） + 日本語歌唱データセット 5 種
- **Run 9 メトリクス**: ZH cos_sim 0.868、JA cos_sim 0.816、JA F0 MAE 45.4 cents

---

## Utai とは？

**Utai**（Uta[歌] + AI）は、AI を活用したオーディオ処理と歌声合成のためのデスクトップアプリケーションです。複数の ML プロジェクトを視覚的なノードベースのワークフローに統合し、音声分離、ボイスチェンジ、メディアエクスポートを実現するほか、Score2HuBERT と RVC を使用した AI 歌声合成のための内蔵 DAW を備えています。

### 機能一覧

| 機能 | 説明 |
|------|------|
| **音声分離** | 113 の MSST/UVR モデルを使用して、オーディオをボーカル、伴奏、ドラム、ベースなどに分離 |
| **ボイスチェンジ** | RVC（Retrieval-based Voice Conversion）を使用した声質変換 |
| **RVC トレーニング** | オーディオデータセットからカスタム音声モデルを学習（パイプライン全自動化） |
| **メディアダウンロード** | yt-dlp 統合による URL からのオーディオ/動画ダウンロード |
| **歌声合成** | Score2HuBERT + RVC パイプラインによる MIDI/USTX 楽譜からの AI 歌声生成 |
| **DAW** | マルチトラックピアノロール、アレンジメントビュー、トランスポートバー、トラックミキサー、オーディオエフェクト |
| **VST プラグインホスト** | JUCE 統合による VST2/VST3 インストゥルメントプラグインの読み込みと再生 |
| **オーディオエフェクト** | トラック単位の EQ、リバーブ、コンプレッサー、ゲイン（Web Audio API） |
| **MIDI インポート** | トラック分類と歌詞ペアリング付きの SMF（.mid）ファイルインポート |
| **USTX インポート** | ノートをロスレスで保持した OpenUtau プロジェクトファイル（.ustx）のインポート |
| **プロジェクトファイル** | ワークフローを `.utai`、シンセプロジェクトを `.utaiSynth` として保存・共有 |
| **i18n** | 中国語、英語、日本語インターフェース |

---

## アーキテクチャ

```
┌─────────────────────────────────────────────────────────┐
│  Electron Shell (Utai.exe)                              │
│  ┌───────────────────────────────────────────────────┐  │
│  │  React Frontend (React Flow + Canvas DAW)         │  │
│  │  http://127.0.0.1:{port}                          │  │
│  └────────────────────┬──────────────────────────────┘  │
│                       │ HTTP/WS                         │
│  ┌────────────────────┴──────────────────────────────┐  │
│  │  Python Backend (FastAPI + Uvicorn)               │  │
│  │  ├─ Node Executor (topological sort, async)       │  │
│  │  ├─ MSST Wrapper (113 separation models)          │  │
│  │  ├─ RVC Wrapper (inference + training)            │  │
│  │  ├─ Score2HuBERT Wrapper (singing synthesis)      │  │
│  │  │  └─ UtaiRenderer.exe (.NET 8 / OpenUtau Core)  │  │
│  │  ├─ VST Wrapper (JUCE host bridge)                │  │
│  │  ├─ Audio Buffer Manager (disk-backed)            │  │
│  │  └─ WebSocket (real-time progress)                │  │
│  └───────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

### シンセレンダリングパイプライン

```
Score (MIDI/USTX notes + lyrics)
  → G2P (lyrics → IPA phonemes, auto language detection)
  → Score2HuBERT (IPA + pitch + duration → HuBERT features + F0)
  → Affine Pitch Transform (shift to RVC model's optimal range)
  → RVC Decoder (HuBERT + F0 → audio waveform)
  → WORLD Inverse Transform (restore original pitch)
  → Post-mute (fade silence tails)
  → WAV output
```

### 技術スタック

| レイヤー | 技術 |
|---------|------|
| デスクトップシェル | Electron 33 |
| フロントエンド | React 19, TypeScript, Vite 6 |
| ノードエディタ | @xyflow/react 12 (React Flow) |
| 状態管理 | Zustand 5 |
| オーディオ可視化 | WaveSurfer.js 7 |
| オーディオエンジン | Web Audio API |
| 歌声合成 | Score2HuBERT (custom) + RVC v2 |
| プラグインホスト | JUCE (VST2/VST3) |
| バックエンド | FastAPI 0.115, Python 3.10 |
| ML フレームワーク | PyTorch 2.5.1 (CUDA 12.1) |
| 音声分離 | MSST (Music Source Separation Training) |
| ボイスチェンジ | RVC (Retrieval-based Voice Conversion) |
| ピッチ変換 | pyworld (WORLD vocoder) |
| メディアツール | ffmpeg, yt-dlp |
| i18n | i18next (zh/en/ja) |

---

## プロジェクト構成

```
Utai/
├── app/
│   ├── frontend/              # React アプリケーション (Vite + TypeScript)
│   │   ├── src/
│   │   │   ├── nodes/         # ワークフローノード UI コンポーネント
│   │   │   ├── components/    # 共有 UI + synth/ (DAW コンポーネント)
│   │   │   ├── lib/synth/     # Canvas エンジン、オーディオエンジン、MIDI パーサー
│   │   │   ├── store/         # Zustand ストア
│   │   │   ├── pages/         # WorkflowPage, TrainingPage, SynthPage
│   │   │   ├── api/           # バックエンド API クライアント
│   │   │   ├── i18n/          # 翻訳ファイル (zh/en/ja)
│   │   │   └── styles/        # utai.css (レトロダークテーマ)
│   │   └── package.json
│   │
│   └── backend/               # FastAPI アプリケーション
│       ├── main.py            # エントリーポイント、CORS、静的配信、subprocess パッチ
│       ├── config.py          # パス解決、デバイス検出
│       ├── core/              # エグゼキューター、グラフ、オーディオバッファ、モデルマネージャー
│       ├── nodes/             # ノード実装（分離、RVC、マージなど）
│       ├── wrappers/          # ML ツールラッパー（MSST, RVC, S2H, VST, yt-dlp）
│       ├── api/               # FastAPI ルートハンドラ
│       └── schemas/           # Pydantic モデル
│
├── MSST/                      # MSST（Utai のインポート構造に合わせてパッチ済み）
│   ├── configs/               # 80 以上のモデル別 YAML 設定 + vr_modelparams/
│   ├── data/models_info.json  # 113 モデルカタログ（ダウンロード URL 付き）
│   ├── inference/             # msst_infer.py, vr_infer.py
│   ├── modules/               # モデルアーキテクチャ (roformer, bandit, scnet 等)
│   ├── utils/                 # utils.py, constant.py, logger.py
│   └── pretrain/              # ダウンロード済みモデル（gitignore 対象）
│
├── RVC/RVC20240604Nvidia/     # RVC（Utai 向けにパッチ済み）
│   ├── infer/                 # 推論、学習、UVR、F0 予測器
│   ├── configs/               # v1/v2 学習設定
│   └── assets/                # 事前学習モデル（gitignore 対象）、ユーザー重み
│
├── score2hubert/              # Score2HuBERT v1（実験版、5 フェーズ）
│   ├── src/                   # モデル、データ、学習、推論コード
│   ├── configs/               # 学習設定
│   ├── scripts/               # 学習、評価、前処理、診断スクリプト
│   └── docs/                  # アーキテクチャドキュメント、フェーズレポート
│
├── score2hubert_v2/           # Score2HuBERT v2（本番版、9 ラン）
│   ├── src/                   # 本番モデルコード
│   ├── configs/               # Run 8/9 設定
│   ├── scripts/               # 学習、評価、前処理、SOFA アライメント
│   └── docs/                  # アーキテクチャ、セッションログ、データクレジット
│
├── synth/                     # 歌声合成ランタイム（UtaiRenderer、ボイスバンク）
├── VST/host/                  # JUCE VST ホストソース (CMake + C++)
├── electron/                  # Electron デスクトップラッパー
├── installer/                 # インストーラー UI、NSIS スクリプト、C# ランチャー
├── config/mirrors.json        # モデルダウンロードミラー設定
│
├── requirements.txt           # Python 依存パッケージ
├── requirements-torch-*.txt   # PyTorch バリアント (CUDA 12.1/12.4/12.8/CPU)
├── setup_env.bat              # Python 環境セットアップ（対話式）
├── run.bat                    # 開発サーバーランチャー
├── build_installer.bat        # インストーラーパッケージのビルド
└── rebuild_utai.bat           # ポータブル Utai/ ディレクトリへの同期
```

---

## パッケージング・配布に関する注意事項

> インストーラーまたはポータブルビルドに取り組む開発者向け。

### ポータブルビルド

ポータブルアプリ（`Utai/` ディレクトリ）は、標準の venv の代わりに `python310._pth`（組み込み Python モード）を使用します。`sitecustomize.py` が以下を自動パッチします：
- `subprocess.Popen` → `CREATE_NO_WINDOW`（コンソールウィンドウの抑制）
- `multiprocessing.spawn.is_forking` → `_pth` 分離モードの修正

### 主な依存関係の注意点

| パッケージ | 制約 | 理由 |
|-----------|------|------|
| `setuptools` | `<71` | v72 以降で `pkg_resources` が削除され、`librosa==0.9.2` が動作しなくなる |
| `fairseq` | `--no-deps` | MSST との omegaconf バージョン競合 |
| `faiss-cpu` | 明示的にインストール | 他パッケージのインストール後に venv から消えることがある |
| `pesq` | 同梱ホイール | C コンパイラが必要。組み込み Python ではコンパイル不可 |
| `dawdreamer` | 必須 | VST ホスト。未インストール時にプラグインバリデーションで 422 エラー |
| `pypinyin` | 必須 | 中国語 G2P。未インストール時にシンセで無音ノートが発生 |

### MSST パッチ

このリポジトリの MSST コードは [ZFTurbo/Music-Source-Separation-Training](https://github.com/ZFTurbo/Music-Source-Separation-Training) を再構成したバージョンです。主な違い：

| コンポーネント | アップストリーム | Utai |
|--------------|---------------|------|
| 推論 | `inference.py`（単一ファイル） | `inference/` パッケージ |
| モジュール | `models/` | `modules/` |
| ユーティリティ | 異なるレイアウト | `utils/utils.py`, `utils/constant.py` |
| データ | モデルカタログなし | `data/models_info.json`（113 モデル） |
| 設定 | 汎用 | 80 以上のモデル別 YAML + `vr_modelparams/` |

`from inference.msst_infer import MSSeparator` や `from utils.utils import demix` のようなインポートパスは、Utai のパッチ済みコードでのみ動作します。

---

## 既知の制限事項

- Windows のみ対応（macOS/Linux は未サポート）
- NVIDIA GPU 推奨（CPU 推論は動作するが非常に低速）
- 未署名の実行ファイルのため Windows SmartScreen 警告が表示される
- DiffSinger ボイスバンクは手動ダウンロードが必要
- VST プラグインのサポートにはユーザー自身の `.dll`/`.vst3` ファイルが必要
- MSST モデルはオンデマンドダウンロード（各 ~200MB〜2GB）
- Score2HuBERT モデルは学習データのライセンスにより非商用

---

## トラブルシューティング

開発環境セットアップ時によくある問題：

| 症状 | 原因 | 解決方法 |
|------|------|----------|
| `torch` が CPU 版のみインストールされる | `--index-url` フラグの欠落 | PyTorch インストール時に**必ず** `--index-url https://download.pytorch.org/whl/cu121`（またはご使用の CUDA バージョン）を付加 |
| `No module named 'pkg_resources'` | `setuptools>=71` がインストール済み | 先にバージョン固定：`pip install "setuptools<71"` してから他パッケージをインストール |
| `omegaconf` バージョン競合 | fairseq が非互換バージョンを取得 | 必ず：`pip install --no-deps fairseq>=0.12.2`、次に `pip install hydra-core bitarray sacrebleu` を個別実行 |
| UI に `{"detail":"Not Found"}` と表示 | フロントエンド静的ファイル未ビルド | `app/frontend/` で `npm run build` を実行し、`dist/*` → `app/backend/static/` にコピー |
| オーディオ処理で `ffmpeg not found` | ffmpeg が想定パスにない | `ffmpeg.exe` を `bin/`（MSST 用）**と** `RVC/RVC20240604Nvidia/`（RVC 学習用）の**両方**に配置 |
| RVC 学習が即座に完了（0 ステップ） | 事前学習モデルの不在 | 12 個すべての `.pth` ファイルを `RVC/RVC20240604Nvidia/assets/pretrained_v2/` にダウンロード |
| RVC 学習で `USE_LIBUV` エラー | Windows libuv ソケット問題 | コードで対処済み（`USE_LIBUV=0`）。生の RVC スクリプト使用時は `set USE_LIBUV=0` を設定 |
| RVC 学習で `MASTER_ADDR` エラー | Windows が `localhost` をホスト名に解決 | コードで修正済み（`127.0.0.1` を使用）。生スクリプトの場合は `set MASTER_ADDR=127.0.0.1` |
| MSST が間違ったモデルアーキテクチャを読み込む | 設定ファイル名の不一致 | MSST 設定ファイルは `{model}.ckpt.yaml`（`{model}.yaml` ではない）。`MSST/configs/` のサブディレクトリが完全か確認 |
| 処理中にコンソールウィンドウが点滅 | subprocess パッチの欠落またはバイパス | `main.py` は `subprocess.Popen` を `CREATE_NO_WINDOW` でグローバルパッチ。削除禁止。`asyncio.create_subprocess_exec` には `creationflags=0x08000000` を渡す |
| ポート 8765 の競合 | 前回のバックエンドインスタンスがまだ実行中 | 既存の Python プロセスを終了するか、`app/backend/config.py` の `PORT` を変更 |
| `faiss` が venv から消える | 他パッケージのインストールで上書き | 一括インストール後に `pip install faiss-cpu` を再実行 |
| Electron ビルドでオーディオエクスポート 500 エラー | 組み込み Python に `tkinter` なし | 修正済み — エクスポートは Electron IPC のファイルダイアログを使用。開発モードは tkinter にフォールバック |

---

## クレジット

- [MSST](https://github.com/ZFTurbo/Music-Source-Separation-Training) — 音声分離フレームワーク
- [RVC](https://github.com/RVC-Project/Retrieval-based-Voice-Conversion-WebUI) — ボイスチェンジ＆学習
- [OpenUtau](https://github.com/stakira/OpenUtau) — 歌声合成エンジン（Core ライブラリ）
- [DiffSinger](https://github.com/openvpi/DiffSinger) — ニューラル歌声合成
- [SOFA](https://github.com/qiuqiao/SOFA) — 歌唱向け強制アライナー
- [JUCE](https://juce.com/) — VST2/VST3 プラグインホストフレームワーク
- [yt-dlp](https://github.com/yt-dlp/yt-dlp) — メディアダウンローダー
- [ffmpeg](https://ffmpeg.org/) — オーディオ/ビデオ処理
- [React Flow](https://reactflow.dev/) — ノードベース UI フレームワーク
- [WaveSurfer.js](https://wavesurfer.xyz/) — オーディオ波形可視化
- 学習データ: ACE-Opencpop, zunko.jp (Kiritan/Itako), ONIKU Kurumi, OFUTON P, Natsume, JVS-MuSiC
- モデル作者: aufr33, anvuew, Gabox, kimberley, SYHFT, pcunwa 他

---

<p align="center">
  <sub>FastAPI、React Flow、PyTorch で構築 — レトロ UI デザインへの愛を込めて</sub><br>
  <sub>v1.8.3</sub>
</p>
