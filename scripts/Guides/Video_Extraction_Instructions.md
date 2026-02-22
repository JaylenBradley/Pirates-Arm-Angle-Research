# Video Frame Extraction Script Setup

## Usage

```bash
# Make sure you're in the project directory, ex:
cd /Users/jaylen/Developer/Research/Pirates_Arm_Angle

# Process all videos in ~/Desktop/baseball_vids (AUTOMATICALLY DELETES videos after extraction)
python scripts/extract_video_frames.py

# Force reprocess already-processed videos
python scripts/extract_video_frames.py --force

# Use a custom videos directory
python scripts/extract_video_frames.py --videos-dir /path/to/your/videos

# Keep original video files after extraction (don't delete them)
python scripts/extract_video_frames.py --keep-videos
```

## What the Script Does

1. **Finds all MP4 videos** in the `baseball_vids` folder on your Desktop
2. **For each video** (e.g., `12650F12-7313-4C8D-A62B-032F5D5C1276.mp4`):
   - Creates a subdirectory: `baseball_vids/12650F12-7313-4C8D-A62B-032F5D5C1276/`
   - Extracts all frames to: `baseball_vids/12650F12-7313-4C8D-A62B-032F5D5C1276/all_frames/`
   - Creates empty directory: `baseball_vids/12650F12-7313-4C8D-A62B-032F5D5C1276/release_frames/`
   - **AUTOMATICALLY DELETES the original video file** to save disk space
3. **Skips videos** that have already been processed (unless `--force` is used)
4. **Uses JPG format** with naming `frame_0001.jpg`, `frame_0002.jpg`, etc.
5. **Works portably** - detects the Desktop automatically so multiple people can use it
6. **Includes 5-minute timeout** to handle corrupted or oversized videos

## ⚠️ Important: Automatic Video Deletion

**By default, the script DELETES the original video files after successfully extracting frames.**

This helps save disk space since video files are large. The frames (JPG images) take up much less space.

**If you want to keep the original videos:**
- Use the `--keep-videos` flag
- Example: `python scripts/extract_video_frames.py --keep-videos`

**Videos are only deleted if:**
- Frame extraction succeeds
- You don't use the `--keep-videos` flag

**Videos are NOT deleted if:**
- Frame extraction fails
- The video times out (>5 minutes)
- You use the `--keep-videos` flag

## Directory Structure After Running

### Default behavior (videos deleted):
```
~/Desktop/baseball_vids/
├── 12650F12-7313-4C8D-A62B-032F5D5C1276/
│   ├── all_frames/
│   │   ├── frame_0001.jpg
│   │   ├── frame_0002.jpg
│   │   └── ...
│   └── release_frames/  (empty - for manual selection)
├── another-video/
│   ├── all_frames/
│   └── release_frames/
└── ...
```

### With --keep-videos flag:
```
~/Desktop/baseball_vids/
├── 12650F12-7313-4C8D-A62B-032F5D5C1276.mp4  ← Video preserved
├── 12650F12-7313-4C8D-A62B-032F5D5C1276/
│   ├── all_frames/
│   │   ├── frame_0001.jpg
│   │   ├── frame_0002.jpg
│   │   └── ...
│   └── release_frames/  (empty - for manual selection)
├── another-video.mp4  ← Video preserved
├── another-video/
│   ├── all_frames/
│   └── release_frames/
└── ...
```

## Summary Output

The script provides a summary showing:
- **Processed**: Videos successfully extracted
- **Skipped**: Videos already processed (unless --force used)
- **Failed**: Videos that failed to extract
- **Deleted**: Number of original videos deleted (when not using --keep-videos)

Example output:
```
==================================================
SUMMARY
==================================================
Processed: 3
Skipped:   0
Failed:    0
Deleted:   3
Total:     3
==================================================
```

## Troubleshooting

**"FFmpeg not found"**
- Install ffmpeg: `brew install ffmpeg`

**"FFmpeg timed out"**
- Video file may be corrupted or too large (>5 minute timeout)
- Video is NOT deleted in this case

**"Failed to delete video"**
- Frames were extracted successfully but video deletion failed
- Check file permissions
- Video directory structure is still created properly
