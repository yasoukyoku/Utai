<p align="center">
  <a href="README.md">English</a> ·
  <a href="README_zh.md"><b>中文</b></a> ·
  <a href="README_ja.md">日本語</a>
</p>

<p align="center">
  <img src="icon.png" width="96" height="96" alt="Utai">
</p>

<h1 align="center">Utai</h1>

<p align="center">
  <b>AI 变声工作站 & 歌声合成 DAW</b><br>
  <sub>音频分离、变声转换、RVC 训练、AI 歌声合成、VST 宿主</sub>
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

## 开发环境配置

### 前置要求

| 工具 | 版本 | 检查方法 | 安装 |
|------|------|----------|------|
| Python | 3.10.x | `py -3.10 --version` | [python.org/downloads](https://www.python.org/downloads/release/python-31011/)（启用 "py launcher"） |
| Node.js | 20+ | `node --version` | [nodejs.org](https://nodejs.org/) |
| Git | 任意 | `git --version` | [git-scm.com](https://git-scm.com/download/win) |
| NVIDIA GPU | RTX 30/40/50 | `nvidia-smi` | 可选；支持 CPU 回退 |

### 1. 克隆仓库

```bash
git clone https://github.com/SaigyoYuyuko/Utai.git
cd Utai
```

仓库中已包含经过修改的 MSST 和 RVC 源代码，**无需**单独克隆这些项目。

### 2. Python 环境

方式 A：使用配置脚本（交互式 GPU 选择）：

```bash
setup_env.bat
```

方式 B：手动配置（以 CUDA 12.1 为例）：

```bash
py -3.10 -m venv venv
venv\Scripts\activate
pip install --upgrade pip
pip install -r requirements-torch-cu121.txt --index-url https://download.pytorch.org/whl/cu121
pip install -r requirements.txt
```

其他 GPU 选项（同时修改文件名和 `--index-url`）：
- CUDA 12.4：`requirements-torch-cu124.txt` + `--index-url https://download.pytorch.org/whl/cu124`
- CUDA 12.8：`requirements-torch-cu128.txt` + `--index-url https://download.pytorch.org/whl/cu128`
- 仅 CPU：`requirements-torch-cpu.txt`（无需 `--index-url`）

> **fairseq**（工作流页面的 RVC 变声转换需要）必须使用 `--no-deps` 安装，因为其 omegaconf 版本与 MSST 存在冲突：
> ```bash
> venv\Scripts\activate
> pip install --no-deps fairseq>=0.12.2
> pip install hydra-core bitarray sacrebleu
> ```
> 注意：合成页面的 RVC（通过 Score2HuBERT 调用）**不需要** fairseq。

### 3. 下载模型文件

以下二进制模型文件体积过大，无法纳入 git 管理。克隆后需手动下载。

#### ffmpeg

从 [gyan.dev/ffmpeg/builds](https://www.gyan.dev/ffmpeg/builds/)（Windows 构建版）下载，然后：

```bash
mkdir bin
```

将 `ffmpeg.exe` 和 `ffprobe.exe` 放入 `bin/` 目录。同时将 `ffmpeg.exe` 复制一份到 `RVC/RVC20240604Nvidia/`（RVC 需要本地副本）。

#### RVC 预训练模型

从 [huggingface.co/lj1995/VoiceConversionWebUI](https://huggingface.co/lj1995/VoiceConversionWebUI/tree/main) 下载：

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

#### MSST 分离模型

分离模型可通过应用内的模型管理器按需下载。`MSST/pretrain/` 目录已创建并包含 `.gitkeep` 占位文件。

#### DiffSinger 声库（合成页面用）

下载声库并放入 `synth/utai-renderer/Singers/`。可从 [lunaiproject/lunai_singers/releases](https://github.com/lunaiproject/lunai_singers/releases) 获取。

#### UtaiRenderer（合成页面用）

渲染器为 .NET 8 自包含二进制文件。从源码构建：

```bash
:: 需要 .NET 8 SDK：https://dotnet.microsoft.com/download/dotnet/8.0

:: 克隆 OpenUtau 分支
cd synth
git clone --depth 1 https://github.com/keirokeer/OpenUtau-DiffSinger-Lunai.git

:: 构建（如从源码编译，需按 synth/utai-renderer-src/ 的说明修改访问修饰符）
cd utai-renderer-src
dotnet publish -c Release -r win-x64 --self-contained true -o "../utai-renderer"
cd ../..
```

### 4. 前端

```bash
cd app\frontend
npm install
npm run build
cd ..\..

:: 将构建产物复制到后端静态文件目录
xcopy /e /y app\frontend\dist\* app\backend\static\
```

### 5. 启动开发服务器

```bash
run.bat
```

此脚本会激活虚拟环境，将 `bin/` 添加到 PATH，并在 `http://localhost:8765` 启动 FastAPI 后端。

前端热重载开发（在另一个终端中）：

```bash
cd app\frontend
set VITE_API_TARGET=http://127.0.0.1:18090
npm run dev
```

> Vite 开发服务器会将 API 请求代理到后端。如更改了后端端口，请相应设置 `VITE_API_TARGET`。

### 6. Electron（可选）

```bash
cd electron
npm install

:: 开发模式（连接到运行中的后端）：
npm start

:: 构建便携版应用（需先构建前端）：
cd ..
rebuild_utai.bat
```

### 验证配置

完成步骤 1-5 后，在浏览器中打开 `http://localhost:8765`，应能看到 Utai 工作流编辑器。关键检查项：

- **工作流页面**：拖入 Audio Input 节点，加载文件
- **训练页面**：可从侧边栏进入
- **合成页面**：需要 UtaiRenderer + 至少一个 DiffSinger 声库

---

## Score2HuBERT 训练配置

Score2HuBERT 是一个神经网络，能直接从乐谱生成 HuBERT 特征和 F0，取代 DiffSinger 传统的声学模型。训练后的模型驱动合成页面的歌声渲染。源代码位于 `score2hubert/`（v1，探索阶段）和 `score2hubert_v2/`（v2，生产版本）。

> 本节仅在需要重新训练模型时才需要阅读。普通 Utai 开发只需要预训练模型检查点。

### 额外前置要求

| 工具 | 用途 |
|------|------|
| .NET 8 SDK | S2H 本身不需要，但 UtaiRenderer 集成时需要 |
| ~30GB 磁盘空间 | 训练数据（ACE-Opencpop + 日语歌唱数据集） |
| 10GB+ 显存的 NVIDIA GPU | 训练用（原始训练使用 RTX 3080 Ti） |

### 配置

```bash
:: 源代码已在仓库中。为 S2H 设置独立虚拟环境：
cd score2hubert_v2
py -3.10 -m venv venv
venv\Scripts\activate
pip install -r ../score2hubert/requirements.txt
```

### 下载训练数据

数据集未包含在仓库中。完整来源列表和下载链接请参阅 `score2hubert_v2/docs/DATA_CREDITS.md`。

| 数据集 | 语言 | 大小 | 来源 |
|--------|------|------|------|
| ACE-Opencpop | 中文 | ~100h | HuggingFace 上的 `espnet/ace-opencpop-segments` |
| Kiritan Singing DB | 日语 | ~1h | zunko.jp |
| Itako Singing DB | 日语 | ~0.8h | zunko.jp |
| ONIKU KURUMI | 日语 | ~1h | onikuru.info |
| OFUTON P | 日语 | ~0.8h | （同系列作者） |
| Natsume Singing DB | 日语 | ~0.9h | ksdcm1ng.wixsite.com |

将原始数据集放入 `score2hubert/data/raw/`，然后运行预处理：

```bash
cd score2hubert_v2
venv\Scripts\activate

:: 预处理各数据集（以 Kiritan 为例）：
python scripts/preprocess_kiritan_itako.py

:: 各数据集的预处理脚本见 scripts/preprocess_*.py
```

### SOFA 强制对齐（可选）

SOFA 用于精确的音素边界对齐。需单独克隆：

```bash
:: 从项目根目录：
git clone https://github.com/qiuqiao/SOFA.git SOFA
cd SOFA

:: 下载预训练对齐器检查点：
::   - 普通话：ckpt/Mandarin/pretrained_mandarin_singing/
::   - 日语：ckpt/JPN_Test2_Plus/
:: 下载方法见 SOFA 仓库说明。

:: 使用 SOFA-Mandarin 重对齐 ACE 数据：
cd ../score2hubert_v2
bash scripts/run_sofa_mandarin_ace.sh
python scripts/preprocess_sofa_realign_ace.py --workers 8

:: 使用 SOFA-Greenleaf 重对齐日语数据：
python scripts/preprocess_sofa_realign.py
```

### 训练

```bash
cd score2hubert_v2
venv\Scripts\activate

:: 开始训练（Run 9 配置，即最终发布版配置）：
set PYTHONUNBUFFERED=1
set PYTHONIOENCODING=utf-8
python -u scripts/train.py --config configs/train_run9.yaml

:: 从检查点恢复训练：
python -u scripts/train.py --config configs/train_run9.yaml --resume checkpoints/latest.pt
```

在 RTX 3080 Ti 上训练 75K 步约需 12 小时。架构详情和记录全部 9 次训练的会话日志请参阅 `score2hubert_v2/docs/`。

### 评估

```bash
:: 分语言指标：
python scripts/eval_per_lang_v2.py --ckpt checkpoints/best.pt --config configs/train_run9.yaml

:: 合成音频用于听感评估：
python scripts/eval_audio.py --ckpt checkpoints/best.pt --config configs/train_run9.yaml

:: 应用后置静音处理：
python scripts/apply_midi_mute.py --input test_outputs/ --output test_outputs_muted/
```

### 模型架构

- **参数量**：54.63M
- **编码器**：5 层 Conformer（512 维隐藏层，8 头注意力）
- **解码器**：6 块确定性 HuBERT 回归头（768 维输出）
- **F0 预测器**：双模式（歌唱/说唱），基于编码特征条件生成
- **输入**：IPA 音素 + MIDI 音高 + 时值
- **输出**：HuBERT Layer 12 特征（768 维，100fps）+ F0 曲线
- **训练数据**：ACE-Opencpop（中文）+ 5 个日语歌唱数据集
- **Run 9 指标**：ZH cos_sim 0.868，JA cos_sim 0.816，JA F0 MAE 45.4 cents

---

## Utai 是什么？

**Utai**（Uta[歌] + AI）是一款面向 AI 音频处理和歌声合成的桌面应用。它将多个机器学习项目整合到可视化的节点式工作流中，支持音频分离、变声转换和媒体导出，同时内置 DAW，使用 Score2HuBERT 和 RVC 实现 AI 歌声合成。

### 功能特性

| 功能 | 说明 |
|------|------|
| **音频分离** | 使用 113 个 MSST/UVR 模型将音频分离为人声、伴奏、鼓、贝斯等 |
| **变声转换** | 使用 RVC（Retrieval-based Voice Conversion）转换音色 |
| **RVC 训练** | 从音频数据集训练自定义声音模型，全流程自动化 |
| **媒体下载** | 通过 yt-dlp 集成从 URL 下载音频/视频 |
| **歌声合成** | 使用 Score2HuBERT + RVC 管线从 MIDI/USTX 乐谱进行 AI 歌声合成 |
| **DAW** | 多轨钢琴卷帘、编排视图、播放控制栏、轨道混音器、音频效果 |
| **VST 插件宿主** | 通过 JUCE 集成加载和演奏 VST2/VST3 乐器插件 |
| **音频效果** | 逐轨 EQ、混响、压缩器、增益（Web Audio API） |
| **MIDI 导入** | 导入 SMF (.mid) 文件，支持轨道分类和歌词匹配 |
| **USTX 导入** | 导入 OpenUtau 工程文件 (.ustx)，无损保留所有音符 |
| **工程文件** | 工作流保存为 `.utai`，合成工程保存为 `.utaiSynth` |
| **多语言** | 中文、英文、日文界面 |

---

## 架构

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

### 合成渲染管线

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

### 技术栈

| 层级 | 技术 |
|------|------|
| 桌面外壳 | Electron 33 |
| 前端 | React 19, TypeScript, Vite 6 |
| 节点编辑器 | @xyflow/react 12 (React Flow) |
| 状态管理 | Zustand 5 |
| 音频可视化 | WaveSurfer.js 7 |
| 音频引擎 | Web Audio API |
| 歌声合成 | Score2HuBERT (自研) + RVC v2 |
| 插件宿主 | JUCE (VST2/VST3) |
| 后端 | FastAPI 0.115, Python 3.10 |
| 机器学习框架 | PyTorch 2.5.1 (CUDA 12.1) |
| 音频分离 | MSST (Music Source Separation Training) |
| 变声转换 | RVC (Retrieval-based Voice Conversion) |
| 音高变换 | pyworld (WORLD vocoder) |
| 媒体工具 | ffmpeg, yt-dlp |
| 多语言 | i18next (zh/en/ja) |

---

## 项目结构

```
Utai/
├── app/
│   ├── frontend/              # React 应用（Vite + TypeScript）
│   │   ├── src/
│   │   │   ├── nodes/         # 工作流节点 UI 组件
│   │   │   ├── components/    # 共享 UI + synth/（DAW 组件）
│   │   │   ├── lib/synth/     # Canvas 引擎、音频引擎、MIDI 解析器
│   │   │   ├── store/         # Zustand 状态仓库
│   │   │   ├── pages/         # WorkflowPage, TrainingPage, SynthPage
│   │   │   ├── api/           # 后端 API 客户端
│   │   │   ├── i18n/          # 翻译文件（zh/en/ja）
│   │   │   └── styles/        # utai.css（复古深色主题）
│   │   └── package.json
│   │
│   └── backend/               # FastAPI 应用
│       ├── main.py            # 入口文件、CORS、静态文件、子进程补丁
│       ├── config.py          # 路径解析、设备检测
│       ├── core/              # 执行器、图、音频缓冲区、模型管理器
│       ├── nodes/             # 节点实现（分离、RVC、合并等）
│       ├── wrappers/          # ML 工具封装（MSST、RVC、S2H、VST、yt-dlp）
│       ├── api/               # FastAPI 路由处理器
│       └── schemas/           # Pydantic 模型
│
├── MSST/                      # MSST（为 Utai 导入结构修改过）
│   ├── configs/               # 80+ 个模型 YAML 配置 + vr_modelparams/
│   ├── data/models_info.json  # 113 个模型目录及下载地址
│   ├── inference/             # msst_infer.py, vr_infer.py
│   ├── modules/               # 模型架构（roformer, bandit, scnet 等）
│   ├── utils/                 # utils.py, constant.py, logger.py
│   └── pretrain/              # 已下载的模型（gitignore）
│
├── RVC/RVC20240604Nvidia/     # RVC（为 Utai 修改过）
│   ├── infer/                 # 推理、训练、UVR、F0 预测器
│   ├── configs/               # v1/v2 训练配置
│   └── assets/                # 预训练模型（gitignore）、用户权重
│
├── score2hubert/              # Score2HuBERT v1（探索阶段，5 个阶段）
│   ├── src/                   # 模型、数据、训练、推理代码
│   ├── configs/               # 训练配置
│   ├── scripts/               # 训练、评估、预处理、诊断
│   └── docs/                  # 架构文档、阶段报告
│
├── score2hubert_v2/           # Score2HuBERT v2（生产版本，9 次训练）
│   ├── src/                   # 生产模型代码
│   ├── configs/               # Run 8/9 配置
│   ├── scripts/               # 训练、评估、预处理、SOFA 对齐
│   └── docs/                  # 架构、会话日志、数据来源
│
├── synth/                     # 歌声合成运行时（UtaiRenderer、声库）
├── VST/host/                  # JUCE VST 宿主源码（CMake + C++）
├── electron/                  # Electron 桌面外壳
├── installer/                 # 安装器 UI、NSIS 脚本、C# 启动器
├── config/mirrors.json        # 模型下载镜像配置
│
├── requirements.txt           # Python 依赖
├── requirements-torch-*.txt   # PyTorch 变体（CUDA 12.1/12.4/12.8/CPU）
├── setup_env.bat              # Python 环境配置（交互式）
├── run.bat                    # 开发服务器启动器
├── build_installer.bat        # 构建安装包
└── rebuild_utai.bat           # 同步到便携版 Utai/ 目录
```

---

## 打包与分发说明

> 面向参与安装器或便携版构建的开发者。

### 便携版构建

便携版应用（`Utai/` 目录）使用 `python310._pth`（嵌入式 Python 模式）代替标准虚拟环境。`sitecustomize.py` 自动打补丁：
- `subprocess.Popen` → `CREATE_NO_WINDOW`（隐藏控制台窗口）
- `multiprocessing.spawn.is_forking` → 修复 `_pth` 隔离模式的兼容问题

### 关键依赖注意事项

| 包名 | 约束 | 原因 |
|------|------|------|
| `setuptools` | `<71` | v72+ 移除了 `pkg_resources`，导致 `librosa==0.9.2` 无法使用 |
| `fairseq` | `--no-deps` | omegaconf 版本与 MSST 冲突 |
| `faiss-cpu` | 需显式安装 | 安装其他包后容易从虚拟环境中消失 |
| `pesq` | 内置 wheel | 需要 C 编译器；嵌入式 Python 无法编译 |
| `dawdreamer` | 必需 | VST 宿主；缺失会导致插件验证返回 422 |
| `pypinyin` | 必需 | 中文 G2P；缺失会导致合成时出现无声音符 |

### MSST 补丁说明

本仓库中的 MSST 代码是 [ZFTurbo/Music-Source-Separation-Training](https://github.com/ZFTurbo/Music-Source-Separation-Training) 的重构版本。主要差异：

| 组件 | 上游 | Utai |
|------|------|------|
| 推理 | `inference.py`（单文件） | `inference/` 包 |
| 模块 | `models/` | `modules/` |
| 工具 | 不同布局 | `utils/utils.py`, `utils/constant.py` |
| 数据 | 无模型目录 | `data/models_info.json`（113 个模型） |
| 配置 | 通用 | 80+ 个模型 YAML + `vr_modelparams/` |

`from inference.msst_infer import MSSeparator` 和 `from utils.utils import demix` 等导入路径只在 Utai 修改过的代码中有效。

---

## 已知限制

- 仅支持 Windows（尚未支持 macOS/Linux）
- 推荐使用 NVIDIA GPU（CPU 推理可用但速度很慢）
- 未签名的可执行文件会触发 Windows SmartScreen 警告
- DiffSinger 声库需手动下载
- VST 插件支持需要用户自行提供 `.dll`/`.vst3` 文件
- MSST 模型按需下载（每个约 200MB-2GB）
- Score2HuBERT 模型因训练数据许可限制为非商业用途

---

## 常见问题排查

开发环境配置中的常见问题：

| 症状 | 原因 | 解决方法 |
|------|------|----------|
| `torch` 安装了 CPU 版本 | 缺少 `--index-url` 参数 | 安装 PyTorch 时**必须**加 `--index-url https://download.pytorch.org/whl/cu121`（或对应 CUDA 版本） |
| `No module named 'pkg_resources'` | 安装了 `setuptools>=71` | 先固定版本：`pip install "setuptools<71"`，再安装其他包 |
| `omegaconf` 版本冲突 | fairseq 拉取了不兼容版本 | 必须：`pip install --no-deps fairseq>=0.12.2`，然后单独 `pip install hydra-core bitarray sacrebleu` |
| 界面显示 `{"detail":"Not Found"}` | 前端静态文件未构建 | 在 `app/frontend/` 执行 `npm run build`，然后将 `dist/*` 复制到 `app/backend/static/` |
| 音频处理时 `ffmpeg not found` | ffmpeg 不在预期路径 | 将 `ffmpeg.exe` 放在 `bin/`（MSST 用）**和** `RVC/RVC20240604Nvidia/`（RVC 训练用）**两个位置** |
| RVC 训练瞬间完成（0 步） | 缺少预训练模型 | 将全部 12 个 `.pth` 文件下载到 `RVC/RVC20240604Nvidia/assets/pretrained_v2/` |
| RVC 训练 `USE_LIBUV` 报错 | Windows libuv 套接字问题 | 代码中已处理（`USE_LIBUV=0`）。若使用原始 RVC 脚本，运行前设置 `set USE_LIBUV=0` |
| RVC 训练 `MASTER_ADDR` 报错 | Windows 将 `localhost` 解析为主机名 | 代码中已修复（使用 `127.0.0.1`）。若用原始脚本，设置 `set MASTER_ADDR=127.0.0.1` |
| MSST 加载了错误的模型架构 | 配置文件命名不匹配 | MSST 配置文件命名为 `{model}.ckpt.yaml`（不是 `{model}.yaml`）。确保 `MSST/configs/` 的子目录完整 |
| 处理时弹出控制台窗口 | 子进程补丁缺失或被绕过 | `main.py` 全局补丁了 `subprocess.Popen` 加 `CREATE_NO_WINDOW`。勿删除。`asyncio.create_subprocess_exec` 需传 `creationflags=0x08000000` |
| 端口 8765 冲突 | 之前的后端实例仍在运行 | 结束已有 Python 进程，或修改 `app/backend/config.py` 中的 `PORT` |
| `faiss` 从虚拟环境消失 | 被其他包安装覆盖 | 批量安装后重新 `pip install faiss-cpu` |
| Electron 版导出音频 500 错误 | 嵌入式 Python 无 `tkinter` | 已修复 — 导出使用 Electron IPC 文件对话框。开发模式回退到 tkinter |

---

## 致谢

- [MSST](https://github.com/ZFTurbo/Music-Source-Separation-Training) — 音频分离框架
- [RVC](https://github.com/RVC-Project/Retrieval-based-Voice-Conversion-WebUI) — 变声转换与训练
- [OpenUtau](https://github.com/stakira/OpenUtau) — 歌声合成引擎（Core 库）
- [DiffSinger](https://github.com/openvpi/DiffSinger) — 神经网络歌声合成
- [SOFA](https://github.com/qiuqiao/SOFA) — 面向歌唱的强制对齐器
- [JUCE](https://juce.com/) — VST2/VST3 插件宿主框架
- [yt-dlp](https://github.com/yt-dlp/yt-dlp) — 媒体下载器
- [ffmpeg](https://ffmpeg.org/) — 音视频处理
- [React Flow](https://reactflow.dev/) — 节点式 UI 框架
- [WaveSurfer.js](https://wavesurfer.xyz/) — 音频波形可视化
- 训练数据：ACE-Opencpop, zunko.jp (Kiritan/Itako), ONIKU Kurumi, OFUTON P, Natsume, JVS-MuSiC
- 模型作者：aufr33, anvuew, Gabox, kimberley, SYHFT, pcunwa 等

---

<p align="center">
  <sub>基于 FastAPI、React Flow、PyTorch 构建，致敬复古 UI 设计</sub><br>
  <sub>v1.8.3</sub>
</p>
