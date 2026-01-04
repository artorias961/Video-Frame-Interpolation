# My notes



```bash
# Activate environment
conda activate rife



# Pass 1 (interpolation)
python inference_video.py --video piplup.mp4 --exp 1 --scale 1

# Final Encode (best quality)
ffmpeg -i "piplup_2X_66fps.mp4" -c:v libx264 -preset veryslow -crf 14 -pix_fmt yuv420p -c:a copy "piplup_2X_crf14_veryslow.mp4"


# Option 2 final encode:
ffmpeg -i "piplup_2X_66fps.mp4" -c:v libx264 -preset veryslow -crf 16 -pix_fmt yuv420p -c:a copy "piplup_2X_veryslow.mp4"


# Option 3 simple encode
ffmpeg -i interpolated.mp4 -i piplup.mp4 -map 0:v -map 1:a -c copy final.mp4
```