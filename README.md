# Virtual Try-On with DensePose & Stable Diffusion

**A research-grade pipeline to transfer garments onto human images using DensePose + diffusion models, with a React/Next.js front-end.**

---

## 🔍 Table of Contents

1. [Project Overview](#project-overview)  
2. [Directory Structure](#directory-structure)  
3. [Methodology](#methodology)  
4. [Installation](#installation)  
5. [Usage](#usage)  
   - [Backend (API)](#backend-api)  
   - [Frontend (Web UI)](#frontend-web-ui)  
6. [Configuration](#configuration)  
7. [Examples](#examples)  
8. [Testing & CI](#testing--ci)  
9. [Contributing](#contributing)  
10. [License](#license)  

---

## 📖 Project Overview

This repository implements a **virtual try-on** system that:

1. Detects human body landmarks & segments the person using DensePose.  
2. Extracts, warps, and masks garments.  
3. Fuses the garment onto the person via a Stable Diffusion-based pipeline for photorealism.  
4. Provides a **React/Next.js** frontend for live demo and batch processing.

---

## 🗂 Directory Structure

```text
.
├── FrontEnd/                # Next.js + TailwindCSS app
│   ├── app/                 # page & API routes
│   ├── components/          # UI components
│   └── utils/               # client-side helpers
├── densepose/               # DensePose integration
├── model/                   # custom model code & pipeline
├── data/                    # dataset loaders & transforms
├── engine/                  # training & evaluation scripts
├── evaluation/              # metrics, alignment, etc.
├── utils/                   # misc helpers (logging, db, IO)
├── requirements.txt         # Python deps
├── Dockerfile               # optional containerization
└── README.md
```

---

## ⚙️ Methodology

1. **Person Detection & DensePose**  
   - Load input image → run Detectron2’s person detector.  
   - Estimate surface UV coordinates via DensePose.  

2. **Garment Extraction & Warping**  
   - Crop out the garment from a reference image.  
   - Compute UV-to-image mapping → generate a mask.  
   - Optionally apply horizontal flip / augmentations.

3. **Mask & Chart Processing**  
   - Convert DensePose outputs (charts, IUV) into segmentation masks.  
   - Align garment mask to target UV chart.

4. **Diffusion-Based Rendering**  
   - Pass person image, garment mask, and UV chart into Stable Diffusion (through HuggingFace’s Diffusers).  
   - Use `peft` fine-tuning for better cloth adherence.  
   - Postprocess output: blend seamlessly with original background.

5. **Web Interface**  
   - Next.js API endpoints proxy calls to the Python backend.  
   - Real-time previews with Gradio/React.  
   - Slider controls for blend strength, color adjustments.

---

## 🚀 Installation

### 1. Clone & Setup
```bash
git clone https://github.com/yourusername/virtual-try-on.git
cd virtual-try-on
```

### 2. Python Environment
```powershell
python -m venv .venv
.\.venv\Scripts\Activate
pip install -r requirements.txt
```

> *Alternatively: use `conda env create -f environment.yml`.*

### 3. Frontend Dependencies
```bash
cd FrontEnd
npm install      # or pnpm install
```

---

## 🖥 Usage

### Backend (API)
```powershell
# from project root
cd .
python engine/trainer.py     # for training (optional)
python engine/inference.py    # start REST API on http://localhost:8000
```

Endpoints:
- `POST /api/tryon`  
  • Body: `{ "person": "<base64-img>", "garment": "<base64-img>" }`  
  • Returns: `{ "output": "<base64-img>" }`

### Frontend (Web UI)
```bash
cd FrontEnd
npm run dev           # starts on http://localhost:3000
```
Open your browser → pick person & cloth → click “Try On.”

---

## ⚙️ Configuration

All runtime params live in `densepose/config.py` &  
`FrontEnd/.env.local`:
```env
BACKEND_URL=http://localhost:8000
MODEL_PATH=/path/to/checkpoint.pth
```

Use OmegaConf YAML files under `/densepose/configs/` to override training/inference settings.

---

## 📸 Examples

| Input Person        | Input Garment        | Output Try-On      |
|:-------------------:|:--------------------:|:------------------:|
| ![](.../docs/person1.jpg) | ![](.../docs/cloth1.png) | ![](.../docs/output1.jpg) |

---

## 🔧 Testing & CI

- Run Python lint & tests:
  ```bash
  pytest --maxfail=1 --disable-warnings -q
  ```
- Frontend lint:
  ```bash
  cd FrontEnd && npm run lint
  ```
- GitHub Actions in `.github/workflows/ci.yml` automates checks on each PR.

---

## 🤝 Contributing

1. Fork → create feature branch  
2. Write tests + docs  
3. Open a PR with clear description  
4. Agree to the [Code of Conduct](./CODE_OF_CONDUCT.md)

---

## 📄 License

This project is licensed under the **MIT License**. See [LICENSE](./LICENSE) for details.
