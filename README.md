# AI Football Analysis System

An AI-assisted football match analysis pipeline that turns match footage into an annotated video and structured player and team analytics. The project combines object detection, tracking, visual recognition, pitch mapping, and action classification in a configurable Python workflow.

> **Project status:** Research and graduation-project software. The repository does not include model weights, input videos, or a dependency lockfile. You must supply the required assets before running the pipeline.

## Features

- Detects and tracks players in video with Ultralytics YOLO and BoT-SORT.
- Detects the ball and estimates short gaps when it is temporarily occluded.
- Classifies teams using jersey-color ranges in HSV space, with voting to stabilize assignments.
- Reads jersey numbers and aggregates predictions across frames.
- Maps player positions to a pitch radar using a perspective transform and camera-motion compensation.
- Estimates possession, player movement, distance, and speed.
- Classifies selected on-ball actions with an R3D-18 video model.
- Produces an annotated video, per-player heatmaps, and match statistics in JSON.
- Offers a local output mode and an optional backend mode for fetching match data and submitting analysis results.

## Processing overview

```text
Match video
    │
    ├── Player detection and tracking ── Jersey number recognition
    │                                 └── Team classification
    ├── Ball detection and tracking
    ├── Pitch mapping and camera compensation
    │       └── Possession, distance, speed, radar, and heatmaps
    └── Action recognition
            │
            └── Annotated video + match_stats.json + heatmaps
                         └── Optional API submission
```

## Repository layout

```text
.
├── pipeline.py                  # Interactive analysis entry point
├── src/
│   ├── config.py                 # Paths, thresholds, match ID, and API settings
│   ├── api_client.py             # Optional backend and heatmap integration
│   ├── detectors/                # Player, jersey-number, team, and action models
│   ├── trackers/                 # Ball, radar, stats, speed, heatmap, and voting logic
│   └── visualizer.py              # Video overlays and action banners
├── data/
│   └── players_database.json      # Local jersey-number name mapping (optional)
├── notebooks/                    # Experiments and model development
├── graduation_project_book.md    # Project/thesis technical guide
└── graduation_thesis_content.md  # Thesis material
```

## Requirements

- Python 3.10 or newer is recommended.
- OpenCV, NumPy, Matplotlib, SciPy, Requests, Ultralytics, PyTorch, and Torchvision.
- The model checkpoints and pitch image listed below.
- A compatible video file at the configured input path.

There is currently no `requirements.txt` or `pyproject.toml` in the repository. Create and activate a virtual environment, install PyTorch and Torchvision using the official instructions for your operating system and hardware, then install the remaining packages:

```bash
python -m venv .venv

# Linux/macOS
source .venv/bin/activate

# Windows PowerShell
.venv\Scripts\Activate.ps1

python -m pip install --upgrade pip
# Install torch and torchvision from https://pytorch.org/get-started/locally/
python -m pip install ultralytics opencv-python numpy matplotlib scipy requests
```

A CUDA-capable GPU is recommended for practical processing speed. Install the PyTorch build that matches your CUDA setup; CPU execution may be significantly slower.

## Required assets

Model files and media are excluded from version control. Put the assets in the paths expected by `src/config.py`:

| Asset | Expected path | Purpose |
| --- | --- | --- |
| Player detector | `yolo26x.pt` | Player detection and tracking |
| Ball detector | `weights/football ball detection/weights/best.pt` | Ball detection |
| Jersey-number recognizer | `weights/jersey_recognition.pt` | Jersey-number detection |
| Stadium segmenter | `weights/Studiam_seg.pt` | Radar and pitch segmentation when radar is enabled |
| Action recognizer | `weights/action/best_soccer_r3d_model.pt` | R3D-18 action classification |
| Input video | `data/input_data/7.mp4` | Default video selected by the configuration |
| Top-down pitch image | `data/pitch_topdown.png` | Heatmap background |

The code expects model files at these exact paths unless you change the corresponding constants in `src/config.py`. Model weights may have separate licenses and usage conditions; obtain them from the project maintainers or their original sources.

Create the input and output folders before the first run. The program expects the output folder to exist when it opens the video writer.

```bash
mkdir -p data/input_data data/output_data
```

The default configuration processes a 30-second segment beginning at 350 seconds when that timestamp is within the video. If the video is shorter, processing starts from the beginning. Adjust the clip-selection values in `pipeline.py` if you want a different segment.

## Run the analysis

From the repository root, after installing dependencies and placing the required files:

```bash
python pipeline.py
```

The program prompts you to choose local or backend mode and whether to enable the pitch radar. To control radar from the command line:

```bash
python pipeline.py --radar
python pipeline.py --no-radar
```

The script still prompts for local or backend mode. Radar mode requires the stadium segmentation checkpoint; select `--no-radar` if that checkpoint is unavailable. The action recognizer can continue without its checkpoint, but action classifications will be disabled.

### Local mode

Local mode skips API reads and uploads. The pipeline writes its annotated video, statistics, and generated heatmaps under `data/output_data/`.

### Backend mode

Backend mode attempts to fetch match and player data, then uploads heatmaps and submits the analysis. The API base URL, Supabase settings, and match ID are currently defined in `src/config.py`. Use services and credentials you are authorized to access. Avoid committing private credentials; move secrets to environment variables or a local, untracked configuration before sharing a deployment.

## Outputs

Typical outputs include:

```text
data/output_data/
├── <configured output video>.mp4
├── match_stats.json
└── heatmaps/
    └── <player>_heatmap.png
```

`match_stats.json` contains team statistics, player statistics, and local heatmap file paths. In backend mode, heatmaps and the statistics payload are also sent to the configured services when requests succeed.

## Configuration

Edit `src/config.py` to change model paths, input/output video paths, confidence thresholds, team-color ranges, radar settings, match ID, and API configuration. The clip start and duration are currently set in `pipeline.py`.

## Troubleshooting

- **Cannot open video:** Confirm that the input exists at `data/input_data/7.mp4`, or update `INPUT_VIDEO_PATH`.
- **Weights not found:** Check every path in `src/config.py`; model checkpoints are not included in this repository.
- **Missing pitch image:** Add `data/pitch_topdown.png`; heatmap generation reads it when the pipeline finishes.
- **Radar/model initialization fails:** Check the stadium segmentation weights, or run with `--no-radar`.
- **Backend requests fail:** Verify network access, API configuration, and the configured `MATCH_ID`. Local mode does not require the backend.
- **Slow processing:** Use a supported CUDA-enabled PyTorch installation and compatible GPU where available.

## Contributing

Contributions are welcome. For substantial changes, open an issue to discuss the proposed behavior. Please avoid committing videos, trained checkpoints, credentials, or generated outputs.

## License

No license file is currently present in this repository. Unless a license is added by the maintainers, reuse and redistribution rights are not specified.


