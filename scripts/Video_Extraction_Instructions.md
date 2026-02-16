# Video Frame Extraction Script Setup

## Quick Setup

Run these commands in your terminal:

```bash
cd /Users/jaylen/Developer/Research/Pirates_Arm_Angle/ViTPose

# Create scripts directory
mkdir -p scripts

# Create the script file (copy the entire command below)
cat > scripts/extract_video_frames.py << 'EOF'
#!/usr/bin/env python3
# Copyright (c) OpenMMLab. All rights reserved.
"""
Video Frame Extraction Script for Baseball Pitching Analysis

This script processes all MP4 videos in a baseball_vids directory on the Desktop.
For each video, it:
1. Creates a subdirectory named after the video (without extension)
2. Extracts all frames to a 'all_frames' subdirectory using ffmpeg
3. Creates an empty 'release_frames' subdirectory for manual frame selection

Usage:
    python scripts/extract_video_frames.py [--videos-dir PATH]
"""

import os
import subprocess
import sys
from pathlib import Path
from argparse import ArgumentParser


def get_desktop_path():
    """Get the user's Desktop path in a portable way."""
    home = Path.home()
    desktop = home / "Desktop"
    
    if not desktop.exists():
        raise FileNotFoundError(f"Desktop directory not found at: {desktop}")
    
    return desktop


def get_video_files(videos_dir):
    """Get all MP4 files in the videos directory."""
    video_files = list(Path(videos_dir).glob("*.mp4"))
    video_files.extend(list(Path(videos_dir).glob("*.MP4")))
    return sorted(video_files)


def check_ffmpeg_installed():
    """Check if ffmpeg is installed and accessible."""
    try:
        result = subprocess.run(
            ["ffmpeg", "-version"],
            capture_output=True,
            text=True,
            check=True
        )
        print(f"✓ FFmpeg found: {result.stdout.split()[2]}")
        return True
    except (subprocess.CalledProcessError, FileNotFoundError):
        print("✗ FFmpeg not found. Please install ffmpeg:")
        print("  brew install ffmpeg")
        return False


def is_already_processed(video_path, videos_dir):
    """Check if a video has already been processed."""
    video_name = video_path.stem
    all_frames_dir = Path(videos_dir) / video_name / "all_frames"
    
    if all_frames_dir.exists():
        frame_files = list(all_frames_dir.glob("frame_*.png"))
        if frame_files:
            return True, len(frame_files)
    
    return False, 0


def extract_frames(video_path, output_dir):
    """Extract all frames from a video using ffmpeg."""
    output_dir.mkdir(parents=True, exist_ok=True)
    output_pattern = str(output_dir / "frame_%04d.png")
    
    cmd = ["ffmpeg", "-i", str(video_path), output_pattern]
    
    try:
        print(f"  Running: ffmpeg -i {video_path.name} {output_pattern}")
        result = subprocess.run(cmd, capture_output=True, text=True, check=True)
        
        frame_files = list(output_dir.glob("frame_*.png"))
        frame_count = len(frame_files)
        
        return True, frame_count, None
    except subprocess.CalledProcessError as e:
        error_msg = f"FFmpeg error: {e.stderr}"
        return False, 0, error_msg


def process_videos(videos_dir, force=False):
    """Process all videos in the videos directory."""
    videos_dir = Path(videos_dir)
    
    if not videos_dir.exists():
        print(f"✗ Videos directory not found: {videos_dir}")
        print(f"  Please create the directory and add your video files.")
        return
    
    video_files = get_video_files(videos_dir)
    
    if not video_files:
        print(f"✗ No MP4 video files found in: {videos_dir}")
        return
    
    print(f"\n{'='*70}")
    print(f"Found {len(video_files)} video(s) to process")
    print(f"{'='*70}\n")
    
    processed_count = 0
    skipped_count = 0
    failed_count = 0
    
    for i, video_path in enumerate(video_files, 1):
        video_name = video_path.stem
        print(f"[{i}/{len(video_files)}] Processing: {video_path.name}")
        
        already_processed, existing_frames = is_already_processed(video_path, videos_dir)
        
        if already_processed and not force:
            print(f"  ⊙ Skipping (already processed: {existing_frames} frames)")
            skipped_count += 1
            print()
            continue
        
        video_dir = videos_dir / video_name
        all_frames_dir = video_dir / "all_frames"
        release_frames_dir = video_dir / "release_frames"
        
        all_frames_dir.mkdir(parents=True, exist_ok=True)
        release_frames_dir.mkdir(parents=True, exist_ok=True)
        
        print(f"  Created: {video_name}/all_frames/")
        print(f"  Created: {video_name}/release_frames/")
        
        print(f"  Extracting frames...")
        success, frame_count, error_msg = extract_frames(video_path, all_frames_dir)
        
        if success:
            print(f"  ✓ Extracted {frame_count} frames")
            processed_count += 1
        else:
            print(f"  ✗ Failed to extract frames")
            print(f"    Error: {error_msg}")
            failed_count += 1
        
        print()
    
    print(f"{'='*70}")
    print(f"SUMMARY")
    print(f"{'='*70}")
    print(f"  Processed: {processed_count}")
    print(f"  Skipped:   {skipped_count}")
    print(f"  Failed:    {failed_count}")
    print(f"  Total:     {len(video_files)}")
    print(f"{'='*70}\n")


def main():
    """Main entry point for the script."""
    parser = ArgumentParser(description="Extract frames from baseball videos for pitching analysis")
    parser.add_argument("--videos-dir", type=str, default=None,
                        help="Path to directory containing video files (default: ~/Desktop/baseball_vids)")
    parser.add_argument("--force", action="store_true",
                        help="Force reprocessing of already-processed videos")
    
    args = parser.parse_args()
    
    if args.videos_dir:
        videos_dir = Path(args.videos_dir)
    else:
        try:
            desktop = get_desktop_path()
            videos_dir = desktop / "baseball_vids"
        except FileNotFoundError as e:
            print(f"✗ Error: {e}")
            sys.exit(1)
    
    print(f"\nVideos directory: {videos_dir}\n")
    
    if not check_ffmpeg_installed():
        sys.exit(1)
    
    process_videos(videos_dir, force=args.force)


if __name__ == "__main__":
    main()
EOF

# Make the script executable
chmod +x scripts/extract_video_frames.py

echo "✓ Script created successfully!"
```

## Usage

After creating the script, you can use it like this:

```bash
# Make sure you're in the ViTPose directory
cd /Users/jaylen/Developer/Research/Pirates_Arm_Angle/ViTPose

# Process all videos in ~/Desktop/baseball_vids
python scripts/extract_video_frames.py

# Force reprocess already-processed videos
python scripts/extract_video_frames.py --force

# Use a custom videos directory
python scripts/extract_video_frames.py --videos-dir /path/to/your/videos
```

## What the Script Does

1. **Finds all MP4 videos** in the `baseball_vids` folder on your Desktop
2. **For each video** (e.g., `12650F12-7313-4C8D-A62B-032F5D5C1276.mp4`):
   - Creates a subdirectory: `baseball_vids/12650F12-7313-4C8D-A62B-032F5D5C1276/`
   - Extracts all frames to: `baseball_vids/12650F12-7313-4C8D-A62B-032F5D5C1276/all_frames/`
   - Creates empty directory: `baseball_vids/12650F12-7313-4C8D-A62B-032F5D5C1276/release_frames/`
3. **Skips videos** that have already been processed (unless `--force` is used)
4. **Uses PNG format** with naming `frame_0001.png`, `frame_0002.png`, etc.
5. **Works portably** - detects the Desktop automatically so multiple people can use it

## Directory Structure After Running

```
~/Desktop/baseball_vids/
├── 12650F12-7313-4C8D-A62B-032F5D5C1276.mp4
├── 12650F12-7313-4C8D-A62B-032F5D5C1276/
│   ├── all_frames/
│   │   ├── frame_0001.png
│   │   ├── frame_0002.png
│   │   └── ...
│   └── release_frames/  (empty - for manual selection)
├── another-video.mp4
├── another-video/
│   ├── all_frames/
│   └── release_frames/
└── ...
```

## The Problem with Your Original FFmpeg Command

The error you saw:
```
Could not open file : /Users/jaylen/Desktop/0C7A7E1B-487D-4150-9510-04AF3DE9DFF5/0C7A7E1B-487D-4150-9510-04AF3DE9DFF5/frame_0001.png
```

**Problem:** The directory name is duplicated, and the nested directory doesn't exist. FFmpeg cannot create nested directories automatically.

**Solution:** This script creates all necessary directories before running ffmpeg, so this error won't happen!
