# AGENTS.md

Guide for AI agents working in this repository.

## Project Overview

Virtual Mouse is a hand-tracking-based mouse controller. It uses a
webcam to detect hand landmarks and maps the index finger position to
screen coordinates, moving the system mouse accordingly. A click is
triggered when all four non-thumb fingers are raised.

The entire implementation lives in a single Jupyter notebook:
`virtual_mouse.ipynb`.

## Tech Stack

- **Language:** Python (3.8, per `.idea/misc.xml`)
- **Runtime:** Jupyter Notebook
- **Key dependencies** (inferred from notebook imports; no
  `requirements.txt` exists yet):
  - `opencv-python` (imported as `cv2`) — video capture and frame processing
  - `mediapipe` — hand landmark detection (`mp.solutions.hands`)
  - `autopy` — cross-platform mouse control (`autopy.mouse.move`, `.click`)
  - `numpy` — coordinate interpolation and array math

## How to Run

1. Install dependencies:
   ```bash
   pip install opencv-python mediapipe autopy numpy jupyter
   ```
2. Launch Jupyter and open the notebook:
   ```bash
   jupyter notebook virtual_mouse.ipynb
   ```
3. Run the single code cell. A webcam window titled **Hand Tracking**
   opens. Move your index finger to control the mouse; raise all four
   fingers to click. Press **q** to quit.

> A webcam is required. `autopy` needs a graphical desktop session.

## Code Structure

The notebook contains one code cell with:

- `handLandmarks(colorImg)` — processes an RGB frame with MediaPipe and
  returns a list of `[index, centerX, centerY]` landmarks; draws
  landmarks on the frame.
- `fingers(landmarks)` — returns a 4-element list indicating whether the
  index, middle, ring, and pinky fingers are raised (tip above the PIP
  joint in the y-axis).
- Main loop — reads frames, flips horizontally, converts to RGB, detects
  landmarks, smooths the index-finger position, moves the mouse via
  `autopy`, and clicks when all four fingers are up.

## Formatting

- **Black** is configured as the project formatter (see
  `.idea/misc.xml`, SDK name `Python 3.8 (virtuel_mouse)`).

## Testing

- No automated tests or test framework are currently present in the
  repository.

## Repository Notes

- `.idea/` contains PyCharm project configuration (not application code).
- `.venv/` is excluded in the PyCharm module config.
- Only one commit exists (`da3af42 version 1.0`).

## TODO

- [ ] Add `requirements.txt` to pin dependency versions.
- [ ] Consider converting the notebook to a `.py` script for easier
      CLI usage and testing.
- [ ] Add a README with setup and usage instructions.
