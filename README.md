# Video Frame Interpolation – Model Sandbox

## ⚠️ Disclaimer

**None of the repositories, models, or code in this directory are my own work.**  
I do **not** claim authorship, ownership, or credit for any of the following projects.

This directory exists **solely for personal experimentation, benchmarking, and learning**.  
All credit belongs to the original authors and maintainers of each repository.

I have collected these projects into **one local workspace** to:
- Test and compare different video frame interpolation approaches
- Experiment with model behavior, performance, and quality
- Explore both research and practical implementations
- Learn how different architectures (flow-based, depth-aware, flow-free, etc.) behave



## Purpose of This Folder

- Centralized testing location
- No redistribution intent
- No modification for publication
- No commercial use
- Strictly personal research & experimentation



## Directory Structure

```
Video Frame Interpolation/
├── .git/
├── 00_WORKING-MODELS/
│   └── RIFE_original_repo/
│
├── DAIN - Depth Aware Interpolation/
│
├── FLAVR - Flow Free Video Interpolation/
│
├── frame interpolation pytorch/
│
├── my_videos/
│
├── Open Source GUI/
│   ├── ComfyUI Frame Interpolation/
│   ├── flowframes/
│   └── Waifu2x Extension GUI/
│
├── RIFE - Real Time Intermediate Flow Estimation/
│   ├── original/
│   └── FORK Repos/
│       ├── rife from vladmandic/
│       └── rife from xychesea/
│
└── README.md
```



## Notes

- Some repositories are **original upstream sources**
- Some are **forks** created by other developers
- GUIs are included for quick testing and visual comparison
- Folder names reflect **model family or usage**, not authorship

If you are the author of any of these projects and have concerns, this setup is **local-only** and can be adjusted or removed.



# NOTES FOR RIFE (ECCV2022-RIFE) — Video FPS Increase + High-Quality Encoding

This README documents a **working end-to-end pipeline** to:
1) Increase video FPS / create slow motion using **RIFE (ECCV2022)**  
2) Re-encode the result with **FFmpeg + x264 `-preset veryslow`** for best quality

Platform tested: **Windows (PowerShell) + Conda**



## 1. Create Conda Environment

```powershell
conda create -n rife python=3.10 -y
conda activate rife
```

(Optional) Jupyter:
```powershell
conda install -c conda-forge notebook ipykernel -y
python -m ipykernel install --user --name rife --display-name "Python (rife)"
```



## 2. Clone Repository

```powershell
git clone https://github.com/hzwer/ECCV2022-RIFE.git
cd ECCV2022-RIFE
```



## 3. Install PyTorch

### NVIDIA GPU (recommended)
Example for CUDA 12.1:
```powershell
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121
```

Verify:
```powershell
python -c "import torch; print(torch.cuda.is_available())"
```



## 4. Install FFmpeg and Requirements

```powershell
conda install -c conda-forge ffmpeg -y
pip install -r requirements.txt
```



## 5. Download Pretrained Model (HD v3.6)

Download:
- `RIFE_trained_model_v3.6.zip`

Extract **contents** into `train_log/`.

Correct structure:
```
ECCV2022-RIFE/
├─ inference_video.py
├─ train_log/
│  ├─ flownet.pkl
│  ├─ IFNet_HDv3.py
│  └─ RIFE_HDv3.py
```



## 6. Add Your Video

Place your input video in the repo root:
```
ECCV2022-RIFE/
├─ inference_video.py
├─ piplup.mp4
└─ train_log/
```



## 7. Run Interpolation (RIFE)

### Best quality default (2× FPS)
```powershell
python inference_video.py --video piplup.mp4 --exp 1 --scale 1
```

### Slower motion (4×)
```powershell
python inference_video.py --video piplup.mp4 --exp 2 --scale 1
```

### Cleanest very-slow motion (recommended)
Run multiple passes:
```powershell
python inference_video.py --video piplup.mp4 --exp 1 --scale 1
python inference_video.py --video piplup_2X_66fps.mp4 --exp 1 --scale 1
```



## 8. Encode with FFmpeg (High Quality)

RIFE improves **motion**. FFmpeg improves **compression quality**.

### Recommended encode
```powershell
ffmpeg -i "piplup_2X_66fps.mp4" `
  -c:v libx264 -preset veryslow -crf 16 `
  -pix_fmt yuv420p `
  -c:a copy `
  "piplup_2X_veryslow.mp4"
```

### Maximum quality
```powershell
ffmpeg -i "piplup_2X_66fps.mp4" `
  -c:v libx264 -preset veryslow -crf 14 `
  -pix_fmt yuv420p `
  -c:a copy `
  "piplup_2X_crf14_veryslow.mp4"
```



## 9. Common Issues

**PowerShell line breaks**
- Use backtick `, not \

**CUDA out of memory**
- Lower scale:
```powershell
--scale 0.5
```

**Audio missing**
```powershell
ffmpeg -i interpolated.mp4 -i piplup.mp4 -map 0:v -map 1:a -c copy final.mp4
```


## 10. Recommended Best-Quality Workflow

1. Interpolate 2× (`--exp 1`)
2. Optional second pass for 4×
3. Encode with x264 `veryslow` + `crf 16`


