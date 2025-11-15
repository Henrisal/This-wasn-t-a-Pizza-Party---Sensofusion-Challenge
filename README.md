# This-wasn-t-a-Pizza-Party---Sensofusion-Challenge
Code for the Junction 2025 Sensofusion challenge

This repository contains the code used for the Junction 2025 Sensofusion challenge. The project processes event-camera recordings (Prophesee/Metavision `.dat` files), performs analysis and visualization, and contains helper scripts to run playback and experiments.

This README explains how the code works at a high level, how to operate it, where to place the data, and describes the main variables and file-path conventions used by the code.

Important references
- evio (used to read `.dat` files): https://github.com/ahtihelminen/evio (see its README for detailed `.dat` format and the provided scripts)
- Data (Google Drive): https://drive.google.com/drive/folders/18ORzE9_aHABYqOHzVdL0GANk_eIMaSuE
- Download: drone_moving->drone_moving.dat

Quick overview
- Input: event-camera recordings in Prophesee/Metavision `.dat` format (binary + ASCII header).
- Event format (as decoded by evio): arrays `x_coords`, `y_coords`, `timestamps`, `polarities`.
- Core dependency used for reading `.dat` files: evio (you can install it or use its `scripts/play_dat.py` to validate / play recordings).
- This repo's code reads events using evio, applies processing/filters/algorithms, and writes results/visualizations to an output directory.

Setup / installation
1. Install Python 3.8+ (3.10+ recommended).
2. Clone this repository:
   git clone https://github.com/Henrisal/This-wasn-t-a-Pizza-Party---Sensofusion-Challenge.git
3. Obtain the data:
   - Download the `.dat` files from the provided Google Drive folder and place them into a local `data/` directory inside the repository, or point the code to your local download path.
   - Example:
     - repo-root/
       - data/
         - recording1.dat
         - recording2.dat
4. Install dependencies
   - Option A: requirements.txt or pyproject.toml, use:
     python -m venv .venv
     source .venv/bin/activate
     pip install -r requirements.txt
   - Option B: Install evio directly
     pip install git+https://github.com/ahtihelminen/evio.git
   - Option C: Use UV as described in evio README (if you prefer uv environments):
     - See evio README for uv instructions: https://github.com/ahtihelminen/evio

How the code works (high level)
1. Read `.dat` files:
   - We rely on evio to open and decode the Metavision `.dat` format. evio exposes standardized event packets with arrays:
     - x_coords: uint16
     - y_coords: uint16
     - timestamps: int64 (microseconds)
     - polarities: int8 (0=OFF, 1=ON)
   - The decoder extracts coordinates and polarity from the packed 32-bit word and timestamps from the 32-bit time field (promoted to int64).
2. Preprocessing:
   - Optionally filter or crop events by time range, spatial ROI (x/y ranges), or polarity.
   - Convert events to frame-like slices by grouping events in time windows (sliding or non-overlapping).
3. Processing / Analysis:
   - Algorithms operate on event batches or derived frames (e.g., counting events per pixel, motion estimation, heuristics for the Sensofusion challenge).
   - Results can be exported as visualizations (images/animated GIFs/video) or numerical outputs (CSV/NumPy arrays).
4. Output:
   - Results are written into an `output/` directory structured by run name or timestamp.

Main file path conventions
- data/: recommended place for input `.dat` files (relative to repo root)
  - Example: data/recording1.dat
- output/: recommended place for the outputs (created at runtime)
  - Example: output/run-2025-11-15_00-00/
- scripts/: location for small helper scripts (e.g., wrappers around evio playback)
- src/ or app/: (if present) main project code and modules
- config/: (optional) configuration files (JSON/YAML) for experiments

Main variables and their logic
Note: variable names below are the common names used across the codebase and examples. If a script uses a different name, the logic is equivalent.

- input_path (string)
  - What: path to the `.dat` file (absolute or relative)
  - Example: input_path="data/recording1.dat"
  - Logic: passed to evio reader to open and iterate events

- output_dir (string)
  - What: directory where processed outputs are saved
  - Example: output_dir="output/run-1"
  - Logic: created if it does not exist; images, logs and CSV outputs are written here

- start_time_us, end_time_us (int; microseconds)
  - What: optional time cropping window (timestamps in microseconds)
  - Logic: only events where start_time_us <= timestamps < end_time_us are processed

- window_ms (int or float; milliseconds)
  - What: time window size to group events into frames
  - Example: window_ms = 50 (ms)
  - Logic:
    - Convert to microseconds: window_us = int(window_ms * 1000)
    - Events are grouped into slices: [t0, t0+window_us), [t0+window_us, t0+2*window_us), ...
    - Use window_ms to control temporal resolution of frame-like displays or batch processing

- speed (float)
  - What: real-time playback multiplier (used only for playback tools)
  - Example: speed = 1.0 → real-time, 2.0 → twice as fast
  - Logic: playback waits between frames according to window_ms / speed

- roi (tuple of ints) or x_min,x_max,y_min,y_max
  - What: spatial region of interest to limit processing to a sub-area of the sensor
  - Example: roi = (100, 1180, 50, 670)
  - Logic: filter events with x and y coordinates inside ROI before further processing


- x_coords, y_coords, timestamps, polarities (arrays)
  - What: decoded arrays returned by evio or the project's event iterators
  - Types:
    - x_coords: numpy.uint16
    - y_coords: numpy.uint16
    - timestamps: numpy.int64 (microseconds)
    - polarities: numpy.int8 (0 or 1)
  - Logic: processing steps work on these arrays (vectorized NumPy operations are preferred to optimize performance)


Enjoy!
