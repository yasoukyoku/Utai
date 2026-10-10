<p align="center">
  <a href="README.md"><b>English</b></a> ·
  <a href="README_zh.md">中文</a> ·
  <a href="README_ja.md">日本語</a>
</p>

<p align="center">
  <img src="icon.png" width="96" height="96" alt="Utai">
</p>

<h1 align="center">Utai</h1>

<p align="center">
  <b>AI Voice Conversion Workstation & Singing Synthesis DAW</b><br>
  <sub>Audio separation, voice conversion, RVC training, AI singing synthesis, VST hosting</sub>
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

## Development Setup

### Prerequisites

| Tool | Version | How to check | Install |
|------|---------|-------------|---------|
| Python | 3.10.x | `py -3.10 --version` | [python.org/downloads](https://www.python.org/downloads/release/python-31011/) (enable "py launcher") |
| Node.js | 20+ | `node --version` | [nodejs.org](https://nodejs.org/) |
| Git | any | `git --version` | [git-scm.com](https://git-scm.com/download/win) |
| NVIDIA GPU | RTX 30/40/50 | `nvidia-smi` | Optional; CPU fallback available |

### 1. Clone the Repository

```bash
git clone https://github.com/SaigyoYuyuko/Utai.git
cd Utai
```

The repo includes patched versions of MSST and RVC source code. You do **not** need to clone those separately.

### 2. Python Environment

Option A: use the setup script (interactive GPU selection):

```bash
setup_env.bat
```

Option B: manual (CUDA 12.1 example):

```bash
py -3.10 -m venv venv
venv\Scripts\activate
pip install --upgrade pip
pip install -r requirements-torch-cu121.txt --index-url https://download.pytorch.org/whl/cu121
pip install -r requirements.txt
```

Other GPU options (change both the file and `--index-url`):
- CUDA 12.4: `requirements-torch-cu124.txt` + `--index-url https://download.pytorch.org/whl/cu124`
- CUDA 12.8: `requirements-torch-cu128.txt` + `--index-url https://download.pytorch.org/whl/cu128`
- CPU only: `requirements-torch-cpu.txt` (no `--index-url` needed)

> **fairseq** (needed for RVC workflow-page voice conversion) must be installed with `--no-deps` due to omegaconf version conflict with MSST:
> ```bash
> venv\Scripts\activate
> pip install --no-deps fairseq>=0.12.2
> pip install hydra-core bitarray sacrebleu
> ```
> Note: the synth page's RVC (via Score2HuBERT) does NOT need fairseq.

### 3. Download Model Files

These binary model files are too large for git. Download them manually after cloning.

#### ffmpeg

Download from [gyan.dev/ffmpeg/builds](https://www.gyan.dev/ffmpeg/builds/) (Windows builds), then:

```bash
mkdir bin
```

Place `ffmpeg.exe` and `ffprobe.exe` into `bin/`. Also place a copy of `ffmpeg.exe` into `RVC/RVC20240604Nvidia/` (RVC needs it locally).

#### RVC Pretrained Models

Download from [huggingface.co/lj1995/VoiceConversionWebUI](https://huggingface.co/lj1995/VoiceConversionWebUI/tree/main):

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

#### MSST Separation Models

Separation models are downloaded on-demand through the in-app Model Manager. The `MSST/pretrain/` directories are already created with `.gitkeep` files.

#### DiffSinger Voice Banks (for Synth Page)

Download voice banks and place into `synth/utai-renderer/Singers/`. Available at [lunaiproject/lunai_singers/releases](https://github.com/lunaiproject/lunai_singers/releases).

#### UtaiRenderer (for Synth Page)

The renderer is a .NET 8 self-contained binary. To build from source:

```bash
:: Requires .NET 8 SDK: https://dotnet.microsoft.com/download/dotnet/8.0

:: Clone the OpenUtau fork
cd synth
git clone --depth 1 https://github.com/keirokeer/OpenUtau-DiffSinger-Lunai.git

:: Build (apply access modifier patches per synth/utai-renderer-src/ if building from source)
cd utai-renderer-src
dotnet publish -c Release -r win-x64 --self-contained true -o "../utai-renderer"
cd ../..
```

### 4. Frontend

```bash
cd app\frontend
npm install
npm run build
cd ..\..

:: Copy built frontend to backend static serving directory
xcopy /e /y app\frontend\dist\* app\backend\static\
```

### 5. Run the Dev Server

```bash
run.bat
```

This activates the venv, adds `bin/` to PATH, and starts the FastAPI backend at `http://localhost:8765`.

For frontend hot-reload development (separate terminal):

```bash
cd app\frontend
set VITE_API_TARGET=http://127.0.0.1:18090
npm run dev
```

> The Vite dev server proxies API requests to the backend. Set `VITE_API_TARGET` to the backend's actual port if you changed it.

### 6. Electron (Optional)

```bash
cd electron
npm install

:: Dev mode (connects to running backend):
npm start

:: Build the portable app (after building frontend):
cd ..
rebuild_utai.bat
```

### Verify the Setup

After completing steps 1-5, open `http://localhost:8765` in your browser. You should see the Utai workflow editor. Key checks:

- **Workflow page**: drag an Audio Input node, load a file
- **Training page**: accessible from the sidebar
- **Synth page**: requires UtaiRenderer + at least one DiffSinger voice bank

---

## Score2HuBERT Training Setup

Score2HuBERT is a neural network that generates HuBERT features + F0 directly from musical scores, replacing DiffSinger's traditional acoustic model. The trained model powers the synth page's vocal rendering. Source code lives in `score2hubert/` (v1, exploration) and `score2hubert_v2/` (v2, production).

> This section is only needed if you want to retrain the model. For normal Utai development, the pre-trained model checkpoint is all you need.

### Additional Prerequisites

| Tool | Purpose |
|------|---------|
| .NET 8 SDK | Not needed for S2H itself, but for UtaiRenderer integration |
| ~30GB disk | Training data (ACE-Opencpop + Japanese singing datasets) |
| NVIDIA GPU with 10GB+ VRAM | Training (RTX 3080 Ti used for original training) |

### Setup

```bash
:: The source code is already in the repo. Set up a separate venv for S2H:
cd score2hubert_v2
py -3.10 -m venv venv
venv\Scripts\activate
pip install -r ../score2hubert/requirements.txt
```

### Download Training Data

Datasets are NOT included in the repo. See `score2hubert_v2/docs/DATA_CREDITS.md` for full source list and download links.

| Dataset | Language | Size | Source |
|---------|----------|------|--------|
| ACE-Opencpop | Chinese | ~100h | `espnet/ace-opencpop-segments` on HuggingFace |
| Kiritan Singing DB | Japanese | ~1h | zunko.jp |
| Itako Singing DB | Japanese | ~0.8h | zunko.jp |
| ONIKU KURUMI | Japanese | ~1h | onikuru.info |
| OFUTON P | Japanese | ~0.8h | (same author family) |
| Natsume Singing DB | Japanese | ~0.9h | ksdcm1ng.wixsite.com |

Place raw datasets under `score2hubert/data/raw/`, then run preprocessing:

```bash
cd score2hubert_v2
venv\Scripts\activate

:: Preprocess each dataset (example for Kiritan):
python scripts/preprocess_kiritan_itako.py

:: See scripts/preprocess_*.py for each dataset's preprocessor
```

### SOFA Forced Alignment (Optional)

SOFA is used for precise phoneme boundary alignment. Clone it separately:

```bash
:: From project root:
git clone https://github.com/qiuqiao/SOFA.git SOFA
cd SOFA

:: Download pretrained aligner checkpoints:
::   - Mandarin: ckpt/Mandarin/pretrained_mandarin_singing/
::   - Japanese: ckpt/JPN_Test2_Plus/
:: See SOFA repo for download instructions.

:: Realign ACE data with SOFA-Mandarin:
cd ../score2hubert_v2
bash scripts/run_sofa_mandarin_ace.sh
python scripts/preprocess_sofa_realign_ace.py --workers 8

:: Realign Japanese data with SOFA-Greenleaf:
python scripts/preprocess_sofa_realign.py
```

### Training

```bash
cd score2hubert_v2
venv\Scripts\activate

:: Start training (Run 9 config, the shipped configuration):
set PYTHONUNBUFFERED=1
set PYTHONIOENCODING=utf-8
python -u scripts/train.py --config configs/train_run9.yaml

:: Resume from checkpoint:
python -u scripts/train.py --config configs/train_run9.yaml --resume checkpoints/latest.pt
```

Training takes ~12 hours on an RTX 3080 Ti for 75K steps. See `score2hubert_v2/docs/` for architecture details and session logs documenting all 9 training runs.

### Evaluation

```bash
:: Per-language metrics:
python scripts/eval_per_lang_v2.py --ckpt checkpoints/best.pt --config configs/train_run9.yaml

:: Audio synthesis for listening evaluation:
python scripts/eval_audio.py --ckpt checkpoints/best.pt --config configs/train_run9.yaml

:: Apply post-mute processing:
python scripts/apply_midi_mute.py --input test_outputs/ --output test_outputs_muted/
```

### Model Architecture

- **Parameters**: 54.63M
- **Encoder**: 5-layer Conformer (512-dim hidden, 8 heads)
- **Decoder**: 6-block deterministic HuBERT regression head (768-dim output)
- **F0 Predictor**: Dual-mode (singing/rap), conditional on encoded features
- **Input**: IPA phonemes + MIDI pitch + duration
- **Output**: HuBERT Layer 12 features (768-dim, 100fps) + F0 contour
- **Training data**: ACE-Opencpop (Chinese) + 5 Japanese singing datasets
- **Run 9 metrics**: ZH cos_sim 0.868, JA cos_sim 0.816, JA F0 MAE 45.4 cents

---

## What is Utai?

**Utai** (Uta[歌] + AI) is a desktop application for AI-powered audio processing and singing synthesis. It integrates multiple ML projects into a visual node-based workflow for audio separation, voice conversion, and media export, plus a built-in DAW for AI singing voice synthesis using Score2HuBERT and RVC.

### Features

| Feature | Description |
|---------|-------------|
| **Audio Separation** | Split audio into vocals, instrumentals, drums, bass, etc. using 113 MSST/UVR models |
| **Voice Conversion** | Convert voice timbre using RVC (Retrieval-based Voice Conversion) |
| **RVC Training** | Train custom voice models from audio datasets with full pipeline automation |
| **Media Download** | Download audio/video from URLs via yt-dlp integration |
| **Singing Synthesis** | AI singing from MIDI/USTX scores using Score2HuBERT + RVC pipeline |
| **DAW** | Multi-track piano roll, arrangement view, transport bar, track mixer, audio effects |
| **VST Plugin Host** | Load and play VST2/VST3 instrument plugins via JUCE integration |
| **Audio Effects** | Per-track EQ, reverb, compressor, gain (Web Audio API) |
| **MIDI Import** | Import SMF (.mid) files with track classification and lyric pairing |
| **USTX Import** | Import OpenUtau project files (.ustx) with lossless note preservation |
| **Project Files** | Save/share workflows as `.utai` and synth projects as `.utaiSynth` |
| **i18n** | Chinese, English, Japanese interface |

---

## Architecture

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

### Synth Render Pipeline

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

### Technology Stack

| Layer | Technology |
|-------|-----------|
| Desktop Shell | Electron 33 |
| Frontend | React 19, TypeScript, Vite 6 |
| Node Editor | @xyflow/react 12 (React Flow) |
| State | Zustand 5 |
| Audio Visualization | WaveSurfer.js 7 |
| Audio Engine | Web Audio API |
| Singing Synthesis | Score2HuBERT (custom) + RVC v2 |
| Plugin Host | JUCE (VST2/VST3) |
| Backend | FastAPI 0.115, Python 3.10 |
| ML Framework | PyTorch 2.5.1 (CUDA 12.1) |
| Separation | MSST (Music Source Separation Training) |
| Voice Conversion | RVC (Retrieval-based Voice Conversion) |
| Pitch Transform | pyworld (WORLD vocoder) |
| Media Tools | ffmpeg, yt-dlp |
| i18n | i18next (zh/en/ja) |

---

## Project Structure

```
Utai/
├── app/
│   ├── frontend/              # React application (Vite + TypeScript)
│   │   ├── src/
│   │   │   ├── nodes/         # Workflow node UI components
│   │   │   ├── components/    # Shared UI + synth/ (DAW components)
│   │   │   ├── lib/synth/     # Canvas engines, audio engine, MIDI parser
│   │   │   ├── store/         # Zustand stores
│   │   │   ├── pages/         # WorkflowPage, TrainingPage, SynthPage
│   │   │   ├── api/           # Backend API client
│   │   │   ├── i18n/          # Translations (zh/en/ja)
│   │   │   └── styles/        # utai.css (retro dark theme)
│   │   └── package.json
│   │
│   └── backend/               # FastAPI application
│       ├── main.py            # Entry point, CORS, static serving, subprocess patches
│       ├── config.py          # Path resolution, device detection
│       ├── core/              # Executor, graph, audio buffer, model manager
│       ├── nodes/             # Node implementations (separation, RVC, merge, etc.)
│       ├── wrappers/          # ML tool wrappers (MSST, RVC, S2H, VST, yt-dlp)
│       ├── api/               # FastAPI route handlers
│       └── schemas/           # Pydantic models
│
├── MSST/                      # MSST (patched for Utai's import structure)
│   ├── configs/               # 80+ per-model YAML configs + vr_modelparams/
│   ├── data/models_info.json  # 113 model catalog with download URLs
│   ├── inference/             # msst_infer.py, vr_infer.py
│   ├── modules/               # Model architectures (roformer, bandit, scnet, etc.)
│   ├── utils/                 # utils.py, constant.py, logger.py
│   └── pretrain/              # Downloaded models (gitignored)
│
├── RVC/RVC20240604Nvidia/     # RVC (patched for Utai)
│   ├── infer/                 # Inference, training, UVR, F0 predictors
│   ├── configs/               # v1/v2 training configs
│   └── assets/                # Pretrained models (gitignored), user weights
│
├── score2hubert/              # Score2HuBERT v1 (exploration, 5 phases)
│   ├── src/                   # Model, data, training, inference code
│   ├── configs/               # Training configs
│   ├── scripts/               # Train, eval, preprocess, diagnostics
│   └── docs/                  # Architecture docs, phase reports
│
├── score2hubert_v2/           # Score2HuBERT v2 (production, 9 runs)
│   ├── src/                   # Production model code
│   ├── configs/               # Run 8/9 configs
│   ├── scripts/               # Training, eval, preprocessing, SOFA alignment
│   └── docs/                  # Architecture, session logs, data credits
│
├── synth/                     # Singing synthesis runtime (UtaiRenderer, voice banks)
├── VST/host/                  # JUCE VST host source (CMake + C++)
├── electron/                  # Electron desktop wrapper
├── installer/                 # Installer UI, NSIS script, C# launcher
├── config/mirrors.json        # Model download mirror configuration
│
├── requirements.txt           # Python dependencies
├── requirements-torch-*.txt   # PyTorch variants (CUDA 12.1/12.4/12.8/CPU)
├── setup_env.bat              # Python environment setup (interactive)
├── run.bat                    # Dev server launcher
├── build_installer.bat        # Build the installer package
└── rebuild_utai.bat           # Sync to portable Utai/ directory
```

---

## Packaging & Distribution Notes

> For developers working on the installer or portable build.

### Portable Build

The portable app (`Utai/` directory) uses `python310._pth` (embedded Python mode) instead of a standard venv. `sitecustomize.py` auto-patches:
- `subprocess.Popen` → `CREATE_NO_WINDOW` (suppress console windows)
- `multiprocessing.spawn.is_forking` → fix for `_pth` isolated mode

### Key Dependency Gotchas

| Package | Constraint | Reason |
|---------|-----------|--------|
| `setuptools` | `<71` | v72+ removes `pkg_resources`, breaks `librosa==0.9.2` |
| `fairseq` | `--no-deps` | omegaconf version conflict with MSST |
| `faiss-cpu` | explicit install | Tends to disappear from venvs after other installs |
| `pesq` | bundled wheel | Requires C compiler; embedded Python can't compile |
| `dawdreamer` | required | VST host; missing causes 422 on plugin validation |
| `pypinyin` | required | Chinese G2P; missing causes silent notes in synth |

### MSST Patching

The MSST code in this repo is a restructured version of [ZFTurbo/Music-Source-Separation-Training](https://github.com/ZFTurbo/Music-Source-Separation-Training). Key differences:

| Component | Upstream | Utai |
|-----------|----------|------|
| Inference | `inference.py` (single file) | `inference/` package |
| Modules | `models/` | `modules/` |
| Utils | Different layout | `utils/utils.py`, `utils/constant.py` |
| Data | No model catalog | `data/models_info.json` (113 models) |
| Configs | Generic | 80+ per-model YAMLs + `vr_modelparams/` |

Import paths like `from inference.msst_infer import MSSeparator` and `from utils.utils import demix` only work with Utai's patched code.

---

## Known Limitations

- Windows only (macOS/Linux not yet supported)
- NVIDIA GPU recommended (CPU inference works but is very slow)
- Unsigned executable triggers Windows SmartScreen warning
- DiffSinger voice banks must be manually downloaded
- VST plugin support requires users to provide their own `.dll`/`.vst3` files
- MSST models are downloaded on-demand (~200MB-2GB each)
- Score2HuBERT model is non-commercial due to training data licenses

---

## Troubleshooting

Common issues encountered during development setup:

| Symptom | Cause | Fix |
|---------|-------|-----|
| `torch` installs CPU-only version | Missing `--index-url` flag | **Must** add `--index-url https://download.pytorch.org/whl/cu121` (or your CUDA version) when running `pip install -r requirements-torch-*.txt` |
| `No module named 'pkg_resources'` | `setuptools>=71` installed | Pin setuptools first: `pip install "setuptools<71"` before other packages |
| `omegaconf` version conflict | fairseq pulls incompatible version | Always: `pip install --no-deps fairseq>=0.12.2`, then `pip install hydra-core bitarray sacrebleu` separately |
| UI shows `{"detail":"Not Found"}` | Frontend static files not built | Run `npm run build` in `app/frontend/`, then copy `dist/*` → `app/backend/static/` |
| `ffmpeg not found` during audio processing | ffmpeg not in expected paths | Place `ffmpeg.exe` in **both** `bin/` (for MSST) **and** `RVC/RVC20240604Nvidia/` (for RVC training) |
| RVC training completes instantly (0 steps) | Missing pretrained models | Download all 12 `.pth` files into `RVC/RVC20240604Nvidia/assets/pretrained_v2/` |
| RVC training crashes with `USE_LIBUV` error | Windows libuv socket issue | Already handled in code (`USE_LIBUV=0`). If using raw RVC scripts, set `set USE_LIBUV=0` before running |
| RVC training crashes with `MASTER_ADDR` error | Windows resolves `localhost` to hostname | Already fixed in code (uses `127.0.0.1`). If raw scripts, set `set MASTER_ADDR=127.0.0.1` |
| MSST loads wrong model architecture | Config file naming mismatch | MSST configs are named `{model}.ckpt.yaml` (not `{model}.yaml`). Ensure `MSST/configs/` has all subdirectories intact |
| Console windows flash during processing | Subprocess patch missing or bypassed | `main.py` globally patches `subprocess.Popen` with `CREATE_NO_WINDOW`. Never remove this. For `asyncio.create_subprocess_exec`, pass `creationflags=0x08000000` |
| Port 8765 conflict | Previous backend instance still running | Kill existing Python processes, or change `PORT` in `app/backend/config.py` |
| `faiss` disappears from venv | Overwritten by other package installs | Re-run `pip install faiss-cpu` after bulk installs |
| Export audio 500 error in Electron build | `tkinter` unavailable in embedded Python | Already fixed — export uses Electron IPC for file dialogs. Dev mode falls back to tkinter |

---

## Credits

- [MSST](https://github.com/ZFTurbo/Music-Source-Separation-Training) — Audio separation framework
- [RVC](https://github.com/RVC-Project/Retrieval-based-Voice-Conversion-WebUI) — Voice conversion & training
- [OpenUtau](https://github.com/stakira/OpenUtau) — Singing synthesis engine (Core library)
- [DiffSinger](https://github.com/openvpi/DiffSinger) — Neural singing voice synthesis
- [SOFA](https://github.com/qiuqiao/SOFA) — Singing-Oriented Forced Aligner
- [JUCE](https://juce.com/) — VST2/VST3 plugin host framework
- [yt-dlp](https://github.com/yt-dlp/yt-dlp) — Media downloader
- [ffmpeg](https://ffmpeg.org/) — Audio/video processing
- [React Flow](https://reactflow.dev/) — Node-based UI framework
- [WaveSurfer.js](https://wavesurfer.xyz/) — Audio waveform visualization
- Training data: ACE-Opencpop, zunko.jp (Kiritan/Itako), ONIKU Kurumi, OFUTON P, Natsume, JVS-MuSiC
- Model authors: aufr33, anvuew, Gabox, kimberley, SYHFT, pcunwa, and others

---

<p align="center">
  <sub>Built with FastAPI, React Flow, PyTorch, and a love for retro UI design</sub><br>
  <sub>v1.8.3</sub>
</p>
