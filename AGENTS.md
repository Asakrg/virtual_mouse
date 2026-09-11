# AGENTS.md

## Repository Overview

Hand-tracking virtual mouse implemented as a single Jupyter notebook: `virtual_mouse.ipynb`.
It uses the webcam (`cv2.VideoCapture(0)`) plus MediaPipe Hands to track the index finger,
maps it to screen coordinates, and moves/clicks the mouse via `autopy`.

## Running

The project is a notebook (Python 3 kernel). No packaging, CLI, or scripts exist.

```bash
jupyter notebook virtual_mouse.ipynb
```

- Requires a webcam and a graphical display (the notebook opens an OpenCV window and controls the real mouse).
- Press `q` in the "Hand Tracking" window to stop the loop.

## Dependencies

Imported in the notebook: `opencv-python` (cv2), `mediapipe`, `autopy`, `numpy`.

TODO: no `requirements.txt`/environment file exists yet; versions are unpinned (`autopy` is unmaintained and may need Python <= 3.9).

## Tests / Lint / CI

TODO: none present in the repo. No test framework, linter config, or CI workflow discovered.
