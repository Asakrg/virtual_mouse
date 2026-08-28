# AGENTS.md — virtual_mouse

## Project Overview

A Python Jupyter notebook that implements a virtual mouse controlled by hand gestures via webcam. It uses **OpenCV** for video capture, **MediaPipe** for hand-landmark detection, and **AutoPy** for moving/clicking the system mouse cursor.

## Repository Structure

```
virtual_mouse.ipynb   # Single-cell notebook containing all logic (plus one empty cell)
.idea/                # PyCharm project config (Python 3.8 SDK, Black formatter)
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

### Install dependencies

The notebook imports `cv2`, `mediapipe`, `autopy`, and `numpy` (no `requirements.txt` exists yet). Install them in your environment:

```bash
pip install opencv-python mediapipe autopy numpy
```

### Launch

```bash
jupyter notebook virtual_mouse.ipynb
# or: jupyter lab virtual_mouse.ipynb
```

Open `virtual_mouse.ipynb` in Jupyter and run the single code cell. The notebook will:

1. Open the default webcam (`cv2.VideoCapture(0)`).
2. Flip each frame horizontally (`cv2.flip(frame, 1)`) for natural mirror-like hand movement.
3. Detect hand landmarks in each frame.
4. Map the index-finger tip (landmark 8) to screen coordinates with smoothing.
5. Move the mouse cursor via `autopy.mouse.move`.
6. Click when all four tracked fingers (index, middle, ring, pinky) are raised.
7. Press **q** (with the OpenCV window focused) to quit.

A webcam and a graphical display are required; the notebook cannot run headless as-is.

### Exporting to a standalone script

```bash
jupyter nbconvert --to script virtual_mouse.ipynb
```

This produces `virtual_mouse.py` that can be run outside Jupyter:

```bash
python virtual_mouse.py
```

## Gesture Controls

| Gesture                                          | Action        |
|--------------------------------------------------|---------------|
| Move index finger                                | Move cursor   |
| All four fingers raised (index + middle + ring + pinky) | Left-click    |
| Press **q** (OpenCV window focused)              | Quit          |

Only the index finger (landmark 8) is used for cursor position. The `fingers()` helper checks tips vs. corresponding PIP joints (8→7, 12→11, 16→15, 20→19) in the y-axis to determine if a finger is raised. The **thumb** (landmark 4) is not tracked.

## Architecture Notes

- **`handLandmarks(colorImg)`** — Processes an RGB frame through MediaPipe, draws landmarks, and returns a list of `[index, x, y]` for each of the 21 landmarks.
- **`fingers(landmarks)`** — Returns a 4-element binary list indicating which fingers (index/middle/ring/pinky) are raised.
- **Copy-paste comment errors in `fingers()`** — All four checks carry the comment `# Check if middle finger is up`, but they actually test index (8→7), middle (12→11), ring (16→15), and pinky (20→19) respectively. Three of the four comments are wrong.
- **`handLandmarks()` uses the global `frame`** — Despite taking `colorImg` as its parameter, the function processes `colorImg` through MediaPipe (`mainHand.process(colorImg)`) but draws onto and reads dimensions from the module-global `frame` variable (`draw.draw_landmarks(frame, ...)` and `h, w, c = frame.shape`). It therefore depends on `frame` being defined in the outer scope.
- **Redundant landmark drawing** — `draw.draw_landmarks(frame, hand, ...)` is called inside the per-landmark `for` loop in `handLandmarks()` (≈21 times per hand per frame), and the main loop draws landmarks again with `mp_drawing.draw_landmarks(...)`. `draw` and `mp_drawing` are both aliases of `mp.solutions.drawing_utils`, so the same landmarks are drawn repeatedly each frame.
- **Duplicate MediaPipe initialization** — The code creates two `mp.solutions.hands.Hands()` instances: `mainHand` (used inside `handLandmarks()`) and `hands` (used in the main loop for drawing and cursor mapping). Additionally, `initHand` and `mp_hands` are both aliases of `mp.solutions.hands`. This is redundant; a single instance and alias could serve all purposes.
- **Dead code: `prev_finger_pos`** — Assigned after each frame but never read. The smoothing logic uses only `curr_finger_pos`.
- **Smoothing** — Cursor movement is dampened with a rolling average (`curr += (target - curr) / 7`).
- **Screen mapping** — Finger x-coordinate is mirrored (`w_screen - x`) to match the horizontally flipped frame, producing natural left-right movement.
- **`landmarkChek` variable name typo** — In `handLandmarks()`, the variable `landmarkChek` is a misspelling of `landmarkCheck`.
- **`finger` variable scope risk** — `finger` is assigned inside `if len(lmList) != 0:` but referenced inside the separate `if results.multi_hand_landmarks:` block. Because two independent `Hands()` instances process the same frame, a disagreement between them (one detects a hand, the other does not) would leave `finger` undefined, causing a `NameError`. This latent bug is a direct consequence of the duplicate MediaPipe initialization noted above.
- **No click debounce** — `autopy.mouse.click()` is called inside the main loop whenever all four fingers are raised, with no edge detection or state tracking. Holding the gesture produces a click on every frame (at the camera frame rate), rather than a single click per gesture. A transition-based trigger (e.g., detect the rising edge of the four-finger state) would be needed for one click per gesture.
- **Default `Hands()` parameters & multi-hand gap** — Both `Hands()` instances are constructed with no arguments, so MediaPipe defaults apply (`max_num_hands=2`, `min_detection_confidence=0.5`, `min_tracking_confidence=0.5`). The main loop iterates over every detected hand in `results.multi_hand_landmarks`, so with two hands the cursor-mapping logic runs for each and the **last** hand wins the cursor position. Meanwhile `fingers(lmList)` reads from `lmList`, which `handLandmarks()` builds by concatenating all hands' landmarks — indices 8/12/16/20 always refer to the **first** hand. This means the click gesture may be evaluated from one hand while the cursor follows another.

## Development Environment

The `.idea/` directory indicates the project was developed in **PyCharm** with:
- **Python 3.8** SDK
- **Black** formatter configured
- **`.venv`** virtual environment (excluded from the project)
- **Package-requirements inspection** — `.idea/inspectionProfiles/Project_Default.xml` enables `PyPackageRequirementsInspection` with an ignore list of ML/CUDA-related packages (`nvidia-nccl-cu11`, `triton`, `pkgutil-resolve-name`, `backports.zoneinfo`, `typing-extensions`), consistent with a development environment that also had PyTorch/CUDA-related packages installed.
- **Project profile disabled** — `.idea/inspectionProfiles/profiles_settings.xml` sets `USE_PROJECT_PROFILE` to `false`, so the IDE uses its built-in default profile rather than the project-level `Project_Default.xml`. The inspection configuration above is defined but may not be active.
- **`.iml` filename typo** — The PyCharm module file is named `virtuel_mouse.iml` (misspelled "virtuel" vs. the repo name "virtual_mouse").

## TODO

- [ ] Add `requirements.txt` or `pyproject.toml` with pinned versions of opencv-python, mediapipe, autopy, numpy.
- [ ] Add right-click and drag gestures.
- [ ] Add a standalone `.py` script alternative for non-notebook usage.
- [ ] Consider configurable smoothing factor and webcam device index.
- [ ] Consolidate duplicate MediaPipe `Hands()` instances into one.
- [ ] Remove or utilize the unused `prev_finger_pos` variable.
- [ ] Move `draw.draw_landmarks()` out of the per-landmark loop in `handLandmarks()` and deduplicate with the main-loop drawing call.
- [ ] Fix copy-paste comment errors in `fingers()` — three of four comments incorrectly say "Check if middle finger is up".
- [ ] Notebook `language_info` metadata lists Python 2.7.6 with the `ipython2` lexer while the kernelspec targets Python 3; metadata appears stale — verify and fix.
- [ ] Remove the empty second code cell in the notebook.
- [ ] Fix `landmarkChek` variable name typo in `handLandmarks()` (should be `landmarkCheck`).
- [ ] Guard the `finger` reference against `NameError` — move the `finger = fingers(lmList)` assignment and click check into the same conditional block, or consolidate to a single `Hands()` instance.
- [ ] Add click debounce/edge detection so the four-finger gesture triggers a single click per transition instead of repeating every frame.
- [ ] Handle multi-hand scenarios explicitly — either set `max_num_hands=1` on `Hands()` or ensure cursor mapping and gesture detection use the same hand.
