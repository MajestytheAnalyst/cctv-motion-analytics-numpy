# Group 3: Estate CCTV Motion Analytics (ShieldGuard Security)

NumPy pipeline for the PAU MSc Data Science group project (Python Programming for Data Science).
It turns a raw CCTV frame stream from an estate gate in Lekki into a log of real movement events.

## What it does

The gate camera takes one frame every 2 minutes from 18:00 to 06:00. The pipeline:

1. Loads the frames and converts them to float (uint8 subtraction would wrap around).
2. Computes the frame-to-frame change and averages it over a 4 x 4 grid of zones.
3. Finds and removes the three data problems (see below).
4. Groups consecutive active frame differences into events, with true clock times and zones.
5. Reports the alert log and how many events happened after midnight.

Only NumPy and Matplotlib are used (no pandas, SciPy or scikit-learn).

## Repository contents

| File | Purpose |
|---|---|
| `Group_3_Project.ipynb` | Full pipeline, step by step, with charts |
| `README.md` | This file |

The data file (`group03_cctv.npz`) is **not** included. Load it from a path on your own machine.

## Requirements

* Python 3.8 or later
* numpy
* matplotlib

```
pip install numpy matplotlib
```

## How to run

1. Clone the repository and open `Group_3_Project.ipynb` in Jupyter or Google Colab.
2. In the first code cell, set `DATA_PATH` to the location of your `.npz` file:
   ```python
   DATA_PATH = "path/to/group03_cctv.npz"
   ```
3. Run all cells from top to bottom.

### Run on a new data file

Everything is wrapped in one function:

```python
results = run_pipeline("path/to/new_file.npz")
```

It returns a dictionary with the real events, the removed camera shifts, the flash frames, the clock jumps, the zone map and event counts.

## Shapes at each stage

(T = number of frames, H x W = frame size, zr x zc = zone grid)

| Stage | Shape (this dataset) |
|---|---|
| Raw frames | (T, H, W, 3) = (360, 72, 96, 3) |
| Frame differences (averaged over colour) | (T-1, H, W) = (359, 72, 96) |
| Reshaped into zones | (T-1, zr, H/zr, zc, W/zc) = (359, 4, 18, 4, 24) |
| After transpose | (T-1, zr, zc, H/zr, W/zc) = (359, 4, 4, 18, 24) |
| Zone change | (T-1, zr, zc) = (359, 4, 4) |

Every reshape is checked with `assert`, and each function states its input and output shapes in its docstring.

## Data problems found and fixed

| Problem | Where | Fix |
|---|---|---|
| Lighting flashes | Frames 10, 93, 195 | Detected as frames far brighter than the average of their neighbours (median and MAD threshold); the affected frame differences are ignored |
| Camera shifted | Frame 243 (02:06) | A real person leaves and the picture goes back to normal; a knocked camera stays shifted. Events whose before/after pictures still differ are removed |
| Recorder clock jump | Between frames 174 and 175 (22 min gap instead of 2) | Ignored the recorder timestamps; true time = 18:00 + 2 minutes x frame number |

Effect on results: 11 events with no fixes, 8 after masking flashes, 7 after also removing the camera shift.

## Results

* 7 real events (18:52, 20:30, 22:50, 23:04, 00:10, 03:26, 03:40).
* 3 events after midnight (00:10, 03:26, 03:40).

## Method notes and limitations

* Thresholds use the median and MAD, so they adapt to each file. The constants (6, 3 and the floor of 1) were tuned on this file.
* The camera-shift test compares the frame before an event with the frame after it. Something that stays in view after the event (for example a parked car) would look like a camera shift.
* Small loops over events are used. All image processing is vectorised.
* An optional benchmark in the notebook compares a Python-loop version of the frame differencing with the NumPy version.

## Team

| Member | Part owned |
|---|---|
| [MAJESTY JOHN EBONG] | Data and structure |
| [Name] | Data problems |
| [Name] | Analysis |
| [Name] | Business answers |
| [Name] | Integration and report |

Each member committed their own part.
