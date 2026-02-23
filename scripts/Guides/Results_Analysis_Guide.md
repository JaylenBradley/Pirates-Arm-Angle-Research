# Results Analysis Guide

This guide covers the data analysis scripts that process calculated pitcher angles and generate statistics and visualizations.

## Prerequisites

1. Completed all pipeline stages (pose extraction → labeling → angle calculation)
2. `pitcher_calculations/` directories populated with angle data
3. Ground truth CSV available

## Scripts Overview

### 1. generate_results_csv.py

Collects all calculated angle data and creates a comprehensive per-frame CSV file.

**Output CSV Columns:**
- `video_id`: Video identifier
- `frame_name`: Frame filename
- `pitcher_angle_shoulder_wrist`: Calculated angle from shoulder to wrist (or N/A)
- `pitcher_angle_elbow_wrist`: Calculated angle from elbow to wrist (or N/A)
- `ground_truth_angle`: Ground truth angle from CSV

**Usage:**
```bash
# Generate results CSV (default output: baseball_vids/data_analysis/results.csv)
python scripts/generate_results_csv.py

# Specify custom output path
python scripts/generate_results_csv.py --output /path/to/output.csv

# Force regeneration
python scripts/generate_results_csv.py --force
```

**Arguments:**
- `--videos-dir PATH`: Path to baseball_vids directory (default: ~/Desktop/baseball_vids)
- `--output PATH`: Output CSV path (default: baseball_vids/data_analysis/results.csv)
- `--force`: Force regeneration even if output exists

---

### 2. generate_summary_statistics.py

Computes summary statistics from the results CSV and optionally generates error distribution plots.

**Statistics Calculated:**
- **MAE (Closest Prediction)**: Mean absolute error using individual frame predictions
- **MAE (Average Prediction)**: Mean absolute error using per-video average predictions
- **Standard Deviations:**
  - Ground Truth angles
  - Closest Prediction angles
  - Average Prediction angles
  - Absolute Errors (Closest)
  - Absolute Errors (Average)
- **Percentage Metrics:**
  - % Predictions Above Ground Truth
  - % Predictions Below Ground Truth
  - % Within 3 Degrees of Ground Truth
  - % Within 8 Degrees of Ground Truth

**Usage:**
```bash
# Generate summary statistics only
python scripts/generate_summary_statistics.py

# Generate statistics with plots
python scripts/generate_summary_statistics.py --plot

# Customize plot format and bins
python scripts/generate_summary_statistics.py --plot --plot-format pdf --bins 30

# Use custom bin width
python scripts/generate_summary_statistics.py --plot --bin-width 0.5
```

**Arguments:**
- `--input PATH`: Input results CSV (default: baseball_vids/data_analysis/results.csv)
- `--output PATH`: Output summary CSV (default: baseball_vids/data_analysis/summary_statistics.csv)
- `--plot`: Generate error distribution plots
- `--plot-format FORMAT`: Plot file format - options: png, pdf, svg, jpg (default: png)
- `--bins NUMBER`: Number of bins for histograms (default: 20)
- `--bin-width FLOAT`: Bin width for histograms (overrides --bins if specified)
- `--force`: Force regeneration even if output exists

**Output Files:**
- `summary_statistics.csv`: CSV with overall statistics (one row per angle type)
- `plots/error_distribution_shoulder.{format}`: Shoulder-wrist error histogram (if --plot)
- `plots/error_distribution_elbow.{format}`: Elbow-wrist error histogram (if --plot)

**Plot Features:**
- Histogram of absolute errors
- Red dashed line showing MAE
- Title with frame/video counts
- Clean, readable styling

---

### 3. run_full_pipeline.py

Master script that runs the complete pipeline in sequence.

**Pipeline Stages:**
1. **Pose Extraction** (`process_release_frames.py`)
2. **Pitcher Labeling** (`label_pitchers.py`)
3. **Angle Calculation** (`calculate_pitcher_angles.py`)
4. **CSV Generation** (`generate_results_csv.py`)

**Usage:**
```bash
# Run full pipeline with shoulder-to-wrist (default)
python scripts/run_full_pipeline.py

# Run with elbow-to-wrist angles
python scripts/run_full_pipeline.py --start-joint elbow

# Skip already-completed stages
python scripts/run_full_pipeline.py --skip-poses --skip-labeling

# Force reprocess everything
python scripts/run_full_pipeline.py --force

# Specify custom paths
python scripts/run_full_pipeline.py --videos-dir /path/to/videos --csv /path/to/ground_truth.csv
```

**Arguments:**
- `--videos-dir PATH`: Path to baseball_vids directory
- `--csv PATH`: Path to ground truth CSV
- `--start-joint JOINT`: shoulder or elbow (default: shoulder)
- `--device DEVICE`: cpu or cuda:0 (default: cpu)
- `--force`: Force reprocessing of all stages
- `--skip-poses`: Skip pose extraction stage
- `--skip-labeling`: Skip pitcher labeling stage
- `--skip-angles`: Skip angle calculation stage
- `--skip-csv`: Skip CSV generation stage

**Features:**
- Interactive confirmation before starting
- Progress tracking for each stage
- Error handling with pipeline halt on failure
- Summary at completion

---

## Complete Workflow Example

### Option 1: Run Everything with Master Script

```bash
# Run entire pipeline
python scripts/run_full_pipeline.py

# Then generate statistics and plots
python scripts/generate_summary_statistics.py --plot --plot-format png
```

### Option 2: Run Stages Individually

```bash
# Stage 1: Extract poses from release frames
python scripts/process_release_frames.py

# Stage 2: Label pitchers interactively
python scripts/label_pitchers.py

# Stage 3: Calculate angles (shoulder-to-wrist)
python scripts/calculate_pitcher_angles.py --start-joint shoulder

# Stage 4: Generate per-frame results CSV
python scripts/generate_results_csv.py

# Stage 5: Generate summary statistics with plots
python scripts/generate_summary_statistics.py --plot
```

### Option 3: Calculate Both Angle Types

```bash
# Calculate shoulder-to-wrist angles
python scripts/calculate_pitcher_angles.py --start-joint shoulder

# Calculate elbow-to-wrist angles (reuses existing labels)
python scripts/calculate_pitcher_angles.py --start-joint elbow

# Generate combined results CSV (includes both angle types)
python scripts/generate_results_csv.py --force

# Generate statistics
python scripts/generate_summary_statistics.py --plot
```

---

## Output Directory Structure

After running all scripts:

```
~/Desktop/baseball_vids/
├── VIDEO_ID/
│   ├── all_frames/
│   ├── release_frames/
│   ├── poses/
│   ├── pitcher_labels/
│   └── pitcher_calculations/
│       └── frame_XXXX_angle/
│           ├── data.json
│           └── frame_XXXX_angle.jpg
├── data_analysis/
│   ├── results.csv                    # Per-frame results
│   ├── summary_statistics.csv         # Overall statistics
│   └── plots/
│       ├── error_distribution_shoulder.png
│       └── error_distribution_elbow.png
└── arm_angles_high_speed.csv         # Ground truth
```

---

## Tips and Best Practices

### Calculating Both Angle Types

If you want both shoulder-to-wrist and elbow-to-wrist calculations:

1. Run the pipeline once with `--start-joint shoulder`
2. Run only the angle calculation again with `--start-joint elbow --force`
3. Run `generate_results_csv.py --force` to merge both angle types

The results CSV will have both angle columns populated for frames where both were calculated.

### Handling Failed Frames

Frames where no pitcher was detected are automatically excluded from:
- Results CSV (they won't have entries in `pitcher_calculations/`)
- Summary statistics calculations
- Plot generation

### Plot Customization

For publication-quality plots:
```bash
python scripts/generate_summary_statistics.py \
    --plot \
    --plot-format pdf \
    --bin-width 0.5
```

For presentations:
```bash
python scripts/generate_summary_statistics.py \
    --plot \
    --plot-format png \
    --bins 15
```

### Incremental Analysis

You can regenerate analysis outputs without rerunning the entire pipeline:

```bash
# Recalculate with different joint
python scripts/calculate_pitcher_angles.py --start-joint elbow --force

# Regenerate CSV
python scripts/generate_results_csv.py --force

# Regenerate statistics
python scripts/generate_summary_statistics.py --force --plot
```

---

## Troubleshooting

### "No calculated frame data found"

**Cause**: No `pitcher_calculations/` directories with data.json files  
**Solution**: Run `calculate_pitcher_angles.py` first

### "Input CSV not found"

**Cause**: Results CSV hasn't been generated yet  
**Solution**: Run `generate_results_csv.py` first

### "No valid angle data found"

**Cause**: All frames have N/A for angles (likely all failed)  
**Solution**: Check `calculate_pitcher_angles.py` output for errors

### Matplotlib Import Error

**Cause**: Matplotlib not installed  
**Solution**: 
```bash
pip install matplotlib
# Or
conda install matplotlib
```

### Plots Not Generated

**Cause**: Forgot `--plot` flag  
**Solution**: Add `--plot` flag to command

---

## Understanding the Statistics

### MAE (Mean Absolute Error)
- **Closest Prediction**: Average error across all individual frames
- **Average Prediction**: Average error using per-video average angles
  - Useful for understanding if averaging multiple frames helps

### Standard Deviations
- **Ground Truth**: Variability in actual pitcher angles
- **Predictions**: Variability in model predictions
- **Absolute Errors**: Consistency of errors (lower = more consistent)

### Percentage Metrics
- **Above/Below GT**: Bias detection (model tends to over/under-predict)
- **Within X Degrees**: Practical accuracy measure
  - 3 degrees: High precision threshold
  - 8 degrees: Acceptable precision threshold

---

## Next Steps

After generating results and statistics:

1. **Review the summary statistics CSV** for overall performance metrics
2. **Examine the plots** for error distribution patterns
3. **Analyze the per-frame results CSV** to identify problematic videos/frames
4. **Compare shoulder vs. elbow measurements** if both were calculated
5. **Iterate on the pipeline** to improve results if needed
