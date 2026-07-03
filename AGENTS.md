# AGENTS.md — virtual_mouse

## Project Overview

A Python Jupyter notebook that implements a virtual mouse controlled by hand gestures via webcam. It uses **OpenCV** for video capture, **MediaPipe** for hand-landmark detection, and **AutoPy** for moving/clicking the system mouse cursor.

## Repository Structure

```
virtual_mouse.ipynb   # Single-cell notebook containing all logic
```

There are no additional modules, tests, or configuration files.

## Key Dependencies

| Package     | Purpose                              |
|-------------|--------------------------------------|
| `opencv-python` (`cv2`) | Webcam capture and image display |
| `mediapipe`  | Hand-landmark detection (21 points)  |
| `autopy`     | Cross-platform mouse control         |
| `numpy`      | Coordinate interpolation             |

> **Note:** These are notebook-level imports only; there is no `requirements.txt` or `setup.py`. A TODO is needed to add dependency pinning.

## Running

Open `virtual_mouse.ipynb` in Jupyter and run the single code cell. The notebook will:

1. Open the default webcam (`cv2.VideoCapture(0)`).
2. Detect hand landmarks in each frame.
3. Map the index-finger tip (landmark 8) to screen coordinates with smoothing.
4. Move the mouse cursor via `autopy.mouse.move`.
5. Click when all four tracked fingers (index, middle, ring, pinky) are raised.
6. Press **q** in the OpenCV window to quit.

A webcam and a graphical display are required; the notebook cannot run headless as-is.

## Gesture Controls

| Gesture                                          | Action        |
|--------------------------------------------------|---------------|
| Move index finger                                | Move cursor   |
| All four fingers raised (index + middle + ring + pinky) | Left-click    |

Only the index finger (landmark 8) is used for cursor position. The `fingers()` helper checks tips vs. corresponding PIP joints (8→7, 12→11, 16→15, 20→19) in the y-axis to determine if a finger is raised.

## Architecture Notes

- **`handLandmarks(colorImg)`** — Processes an RGB frame through MediaPipe, draws landmarks, and returns a list of `[index, x, y]` for each of the 21 landmarks.
- **`fingers(landmarks)`** — Returns a 4-element binary list indicating which fingers (index/middle/ring/pinky) are raised.
- **Smoothing** — Cursor movement is dampened with a rolling average (`curr += (target - curr) / 7`).
- **Screen mapping** — Finger x-coordinate is mirrored (`w_screen - x`) for natural left-right movement.

## TODO

- [ ] Add `requirements.txt` or `pyproject.toml` with pinned versions of opencv-python, mediapipe, autopy, numpy.
- [ ] Add right-click and drag gestures.
- [ ] Add a standalone `.py` script alternative for non-notebook usage.
- [ ] Consider configurable smoothing factor and webcam device index.
