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


# Nice to Know

## Core Principle (Read First)

**Interpolation adds frames — it does NOT decide playback speed.**

Playback speed is decided by:
- Timeline FPS
- Export FPS
- Player / display refresh rate

If these do not match, **sync problems are guaranteed**.

## Baseline: Original Video

| Property | Value |
|-------|------|
| FPS | ~29.97 / 30 |
| Duration | Reference |
| Purpose | Source material |


## Workflow A — High‑FPS (≈131 fps)

### How it is created

```powershell
python inference_video.py --video piplup.mp4 --exp 2 --scale 1
```

### Expected output

| Property | Value |
|-------|------|
| FPS | ~120–132 fps |
| Duration | Same as original |
| Motion | Extremely smooth |
| File size | Large |
| Editing stability | Low |

Example output:
```
piplup_4X_132fps.mp4
```

### What this workflow is GOOD for

- Research / benchmarking
- Demonstrating interpolation quality
- Slow‑motion generation
- High‑refresh (120–144 Hz) displays

### What this workflow is BAD for

- Mixed‑FPS timelines
- DaVinci Resolve comparisons
- Delivery to most platforms
- Stable editing

⚠️ Editors must **drop frames unevenly**, causing visual desync.


## Workflow B — Locked‑FPS (30 or 60 fps)

### Method 1 — Interpolate then lock (recommended)

```powershell
python inference_video.py --video piplup.mp4 --exp 2 --scale 1
ffmpeg -i piplup_4X_132fps.mp4 -vf fps=60 piplup_60fps_master.mp4
```

### Method 2 — Direct 2× interpolation

```powershell
python inference_video.py --video piplup.mp4 --exp 1 --scale 1
```

### Expected output

| Target FPS | Result |
|---------|--------|
| 30 fps | Original timing, smoother motion |
| 60 fps | Best balance of smoothness + stability |

### Why locking FPS works better

- Editors map frames **1:1**
- No uneven frame dropping
- No retime artifacts
- Predictable playback
- Easier comparison

This is why **professional pipelines always lock FPS before editing**.



## Direct Comparison

| Feature | ≈131 fps | Locked 30/60 fps |
|------|---------|----------------|
| Smoothness potential | Very high | High |
| Visual stability | Medium | Very high |
| Editor sync | ❌ Poor | ✅ Stable |
| Comparison shots | ❌ Difficult | ✅ Easy |
| Real‑world delivery | ❌ Rare | ✅ Standard |
| File size | Very large | Reasonable |


## Common Mistakes (DO NOT DO THIS)

### ❌ Forcing FPS on import

```powershell
ffmpeg -r 30 -i input.mp4
```

This causes:
- Timing distortion
- Slow motion
- Uneven frame spacing

### ❌ Mixing FPS in an editor

- 30 fps + 60 fps + 132 fps on one timeline
- Optical Flow on interpolated clips
- Auto‑conform timelines

This guarantees desync.


## DaVinci Resolve Rule (Critical)

> **Timeline FPS is the master clock.**

All clips must:
- Match timeline FPS
- Have retiming disabled
- Avoid Optical Flow



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

## 1. Create Environment

```powershell
conda create -n rife python=3.10 -y
conda activate rife
conda install -c conda-forge ffmpeg -y
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121
```

Verify GPU:
```powershell
python -c "import torch; print(torch.cuda.is_available())"
```



## 2. Run RIFE – Frame Interpolation

### 2× Interpolation (Recommended)

```powershell
python inference_video.py --video piplup.mp4 --exp 1 --scale 1
```

**Expected result**
| Property | Value |
|------|------|
| FPS | ~60–66 fps |
| Duration | Same as original |
| Purpose | Smooth motion |

Output example:
```
piplup_2X_66fps.mp4
```



### 4× Interpolation (Very Smooth / Heavy)

```powershell
python inference_video.py --video piplup.mp4 --exp 2 --scale 1
```

**Expected result**
| Property | Value |
|------|------|
| FPS | ~120–132 fps |
| Duration | Same as original |
| Purpose | Extreme smoothness |

Output example:
```
piplup_4X_132fps.mp4
```



## 3. Verify FPS (Critical Step)

Always verify before encoding:

```powershell
ffmpeg -i piplup_4X_132fps.mp4
```

Correct output example:
```
Video: ..., 131.9 fps
Duration: unchanged
```

If FPS is lower than expected → **stop**.



## 4. Encode Correctly (Preserve Smoothness)

### ✅ Correct Encoding (DO THIS)

```powershell
ffmpeg -i piplup_4X_132fps.mp4 `
  -c:v libx264 `
  -preset veryslow `
  -crf 16 `
  -pix_fmt yuv420p `
  -profile:v high `
  -level 5.1 `
  -movflags +faststart `
  -c:a copy `
  piplup_4X_smooth_x264.mp4
```

**Why this works**
- FPS preserved automatically
- No timing changes
- Stable playback at high FPS



## 5. Commands You MUST NOT Use

### ❌ DO NOT FORCE FPS

```powershell
ffmpeg -r 30 -i input.mp4 ...
```

**Why this breaks smoothness**
- Forces high‑FPS video to play at low FPS
- Causes slow motion
- Makes interpolation pointless



### ❌ DO NOT USE OLD CODECS

Avoid:
```
-c:v mpeg4
```

Reasons:
- Poor motion handling
- Frame pacing jitter
- Bad high‑FPS playback

Always use:
```
-c:v libx264
```


## 6. Playback (Important)

Use:
- **mpv**
- **VLC (hardware acceleration enabled)**
- **MPC-HC**

Avoid:
- Windows “Movies & TV”
- Browser playback for evaluation



## 7. Summary Table

| Step | Result |
|----|----|
| RIFE `--exp 1` | ~66 fps smooth |
| RIFE `--exp 2` | ~132 fps very smooth |
| FFmpeg (no `-r`) | Preserves smoothness |
| FFmpeg with `-r` | Causes slow motion |
| x264 | Stable playback |
| mpeg4 | Choppy / misleading |



## Final Takeaway

If your video:
- Has higher FPS
- Same duration
- Plays at that FPS

Then you have achieved **true smoothness**.

If duration increases → you made slow motion.

