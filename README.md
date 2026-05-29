# AI Gaming Montage and Highlight Generator

An enterprise-grade, automated video editing pipeline designed for content creators, gaming networks, and esports production agencies. This software eliminates hours of tedious manual clipping by utilizing multi-modal artificial intelligence and signal processing to scan long, raw gameplay footage, extract high-action moments, and synchronize cuts seamlessly to the beat grid of any background music track.

---

## Technical Architecture and How It Works

The system operates as an advanced multi-phase data pipeline inside a Google Colab environment powered by an NVIDIA Tesla T4 GPU (16GB VRAM). Unlike standard video editing scripts that cause massive system memory leaks, this architecture is split into decoupled analytical and assembly modules to guarantee absolute processing stability on long-form video files.

### 1. Signal Isolation and Extraction

The pipeline isolates the native audio track from the uploaded gameplay video, down-sampling it to a standardized 16kHz mono WAV file. This uniform structure reduces computational overhead and ensures optimal performance during structural analysis.

### 2. Multi-Modal Action Mapping

The engine analyzes the video using two independent tracking methodologies:

* **Dynamic Audio Peak Detection:** The pipeline tracks the short-time energy of the audio stream. Instead of using a hardcoded volume limit, it calculates a relative threshold based on the top 15 percent loudest moments of the specific video. This allows the AI to accurately flag gunshots (such as sniper cracks in Call of Duty or Free Fire) and explosive action events, even if the recording mixer profile is quiet.
* **GPU-Accelerated Spatial Mapping:** To track visual motion spikes without crashing the standard system RAM, the frame matrices are converted to grayscale and pushed directly onto the T4 GPU VRAM as PyTorch tensors. Mathematical differencing and absolute structural deviation tracking happen completely on the hardware accelerator.

### 3. Audio Rhythm Tracking

The pipeline loads the custom background music track via the Librosa audio analysis engine. It calculates the global tempo (BPM) and maps every precise timestamp where a musical beat drop occurs, creating a target synchronization grid.

### 4. Zero-RAM Structural Assembly

Standard python video editing libraries suffer from significant memory degradation when opening dozens of individual sub-clips simultaneously. This project circumvents that limitation entirely via a custom Zero-RAM text assembly matrix.

The software maps the exact cut points mathematically and writes the trim instructions to a lightweight text file. It then runs a native, sub-processed FFmpeg command to slice and concatenate the video directly on the storage disk. This methodology keeps the system RAM flat, preventing Google Colab from terminating the session.

---

## Core Features

* **Dual Analytical Engine:** Optimized for both commentary tracks (using semantic speech tracking) and raw, non-commentary competitive footage (using audio energy tracking).
* **True Beat Synchronization:** Hard-trims clips exactly on the musical beat nodes without unnatural time-stretching or audio distortion.
* **Dynamic Volume Ducking:** Automatically applies an attenuation matrix to lower the background music track volume by 75 percent during high-intensity native gameplay moments, preserving structural clarity.
* **Overlapping Guard Rails:** Algorithmic filtering prevents the script from creating overlapping duplicate clips when multiple action events occur in rapid succession.

---

## Step-by-Step Getting Started Guide

To run this project successfully, follow these exact instructions to configure your environment and execute the pipeline.

### Prerequisites

* A Google account to access Google Colab.
* A raw gameplay video file (Recommended length: 15 to 30 minutes for a full-length montage).
* A background music file (Ensure the song length is equal to or greater than your target montage length).

### Step 1: Initialize the Hardware Environment

1. Open your Google Colab notebook.
2. Navigate to the top menu and select **Runtime** -> **Change runtime type**.
3. Under **Hardware accelerator**, select **T4 GPU**.
4. Click **Save**.

### Step 2: Upload Your Media Assets

1. Look at the left sidebar of Google Colab and click the **Folder** icon to open the file directory.
2. Drag and drop your raw gameplay video into the empty space. Rename this file to exactly: `gameplay.mp4`
3. Drag and drop your custom music track into the same area. Rename this file to exactly: `music.mp3`
4. Wait for the circular upload progress indicators at the bottom left to fill completely. Do not close the tab while uploading.

### Step 3: Run the Dependency Architecture (Phase 1)

Execute the first cell block to install the required open-source libraries into your virtual machine environment.

```python
!pip install -q open-source-libraries librosa moviepy opencv-python-headless numpy scipy tqdm torch faster-whisper

```

### Step 4: Configure Variables (Phase 2)

Verify that your configuration targets the matching file names. Set your maximum desired output duration in seconds (Example: `180` for a 3-minute video).

```python
VIDEO_INPUT_PATH = "/content/gameplay.mp4"
BGM_INPUT_PATH = "/content/music.mp3"
OUTPUT_DIR = "/content/output"
FINAL_VIDEO_NAME = "ai_montage_final.mp4"
TARGET_MONTAGE_DURATION = 180 

```

### Step 5: Execute Sequential Processing

Run every subsequent phase cell sequentially from Phase 3 down to Phase 15. Do not skip blocks. The pipeline will print live updates, showing you the exact number of identified audio spikes and tracking the rendering status with a visual progress bar.

### Step 6: Export the Completed Montage

Because downloading large media files directly through the browser file panel can cause a browser connection timeout error ("Failed to fetch"), run the dedicated download script in the final cell block to transfer the compiled video directly to your system or Google Drive account.

---

## Configuration Parameter Definitions

| Parameter Name | Data Type | Default Value | Functional Description |
| --- | --- | --- | --- |
| `TARGET_MONTAGE_DURATION` | Integer | 180 | The maximum runtime limit of the completed output file in seconds. |
| `CLIP_PADDING_SECONDS` | Float | 2.0 | The time padding added before and after an identified peak to provide visual context. |
| `dynamic_threshold` | Percentile | 85 | Located in Phase 6. Adjusting this down (e.g., to 75) makes the audio peak selection more sensitive to quiet sounds. |
| `valid_beats` | Timeline offset | + 2.0 | Located in Phase 11. Dictates that a video clip must play for at least 2 seconds before cutting to the next beat drop. |

---

## Troubleshooting and Error Resolution

### Error: OSError: Accessing time t=X past clip duration

* **Cause:** This occurs when a script tries to read video frames or audio frequencies slightly beyond the physical boundary of a clipped sub-file due to floating-point rounding anomalies.
* **Resolution:** Ensure you are using the updated Phase 11 Zero-RAM engine block, which applies a strict hard-coded `0.1` second safety margin reduction to the clip sub-bounds, completely preventing frame-reading overflow.

### Error: BrokenPipeError / Filter not found: 'drawtext'

* **Cause:** Google Colab servers utilize a headless Linux command-line compiler for FFmpeg that does not include the system-level `libfreetype` text rendering font engine. Attempting to draw metadata watermarks or text via video filters will cause the video engine pipe to shatter.
* **Resolution:** Ensure Phase 13 returns an empty array `[]`. This bypasses font filter calls entirely and allows native software encoding to proceed without reliance on local server font binaries.

### Error: System RAM Crushes / Session Disconnects

* **Cause:** Processing long high-definition video clips via standard CPU-bound matrix transformations completely overflows the standard 12GB virtual system memory allocated to free Colab tiers.
* **Resolution:** Confirm that you are executing the PyTorch GPU-accelerated configuration for Phase 7. This transfers the entire array footprint onto the 16GB VRAM of the T4 card, protecting system memory from experiencing allocation errors. Use the decoupled text-list assembly system inside Phase 11 to keep background file operations contained.
