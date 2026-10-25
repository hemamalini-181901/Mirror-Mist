# Breath on Glass

A bathroom mirror that has fogged up, except the mirror is your webcam.

- **Breathe** at your laptop and fog blooms where your breath lands.
- **Pinch** your thumb and index finger together and draw on the fog. Your face shows through wherever you draw.
- **Make a fist** and swipe to wipe a wide patch clear, like using your sleeve.

It's one file: `index.html`. There's no build step, no npm and no framework.

## Run it

Browsers only allow camera and microphone access on `https` or `localhost`. Double-clicking the file (`file://`) won't work. From this folder, run:

```bash
python3 -m http.server 8000
```

Then open <http://localhost:8000> and click the button. You need an internet connection the first time. The hand and face tracking code and models download from a CDN.

## Controls

| Input | What it does |
|---|---|
| Breathe at the mic | Fogs the glass near your mouth |
| Hold **Space** | Fogs without breathing (good for silent demos) |
| Pinch (thumb + index) | Draws on the fog |
| Fist, then swipe | Wipes a wide area |
| **C** | Clears your drawing (the fog returns to its starting state) |
| **S** or the shutter button | Saves a PNG |
| **D** | Shows live readouts (mic level, pinch ratio, fps) |
| **Esc** | Back to the start screen (also turns the camera and mic off) |
| `brush` slider (bottom left) | Sets the erase radius, 7 to 38 px |

## Privacy

Everything happens in your browser tab. No video frames, audio or data are sent anywhere. The only network traffic is downloading the MediaPipe code and model files.

## How it works, in simple terms

The screen is built from three layers, redrawn every frame:

1. **The camera**, flipped left to right so it acts like a real mirror.
2. **The fog map.** This is a hidden canvas that remembers only *how foggy each spot is*. You never see it directly. Breathing adds soft white blobs to it. Drawing and wiping *subtract* from it.
3. **The frost.** This is the camera picture again, but blurred, brightened and tinted white. The fog map decides where it shows. Where there's fog the frost is visible. Where you've wiped it away you see the sharp camera underneath.

Erasing never paints anything. It only removes fog, so the clear spots are your real face coming through.

### Sensing your breath

The microphone is listened to continuously and its loudness is measured. The app keeps a running guess at the room's background noise (the "floor"), which sinks quickly and rises very slowly. Breath counts when the loudness goes a little above that floor. This works in a quiet room or a noisy one, with no fixed number to tune. The browser's normal noise suppression is switched off, because it is designed to delete the sound of breathing.

### Sensing your hand

MediaPipe Hands reports 21 points on your hand each frame.

- **Pinch** is the thumb-to-fingertip distance divided by the size of your hand. That way it works the same near or far from the camera. Two thresholds (0.22 to start, 0.38 to stop) stop it flickering on and off.
- **Fist** means all four fingertips are curled closer to the wrist than their middle joints.
- Drawing needs your index finger straight when the pen goes down. This stops a fist, where the thumb rests on the index finger, from also counting as a pinch.
- The line is smoothed twice, so it looks like pen strokes rather than shaky tracking.
- Wiping lays down one soft clear stamp for every bit of distance your hand moves, not per frame. A fast hand and a slow hand therefore clear the same amount.

### Finding your mouth

MediaPipe Face Detection runs every 5th frame to find your mouth. The fog centre follows it gradually, so it trails your head instead of jumping.

## Technologies used

- **HTML, CSS and plain JavaScript.** One file, with strict mode on so mistakes show up as errors.
- **Canvas 2D.** `destination-out` erases fog, `destination-in` masks the frost, and `ctx.filter` blurs.
- **MediaPipe Hands** (`@mediapipe/hands@0.4.1675469240`) for hand landmarks.
- **MediaPipe Face Detection** (`@mediapipe/face_detection@0.4.1646425229`) for finding your mouth.
- **`getUserMedia`** for the camera and microphone.
- **Web Audio API** (`AnalyserNode`) for measuring breath loudness.
- **`requestAnimationFrame`** for the one loop that feeds MediaPipe and draws.

The MediaPipe versions are pinned on purpose. Unpinned versions pull a broken build.

## Troubleshooting

- **"Camera access was refused":** click the camera icon in the address bar, allow it, and press the button again.
- **Camera works but breathing does nothing:** press **D** and watch `mic` against `floor` while you blow. Hold **Space** as a fallback.
- **The screen fogs by itself:** it's probably a noisy room or fan. Wait a few seconds for the floor to settle.
- **Safari:** it works, but it doesn't support `ctx.filter`, so the blur uses a cheaper fallback and looks a bit softer.
