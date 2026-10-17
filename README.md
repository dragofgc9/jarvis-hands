# jarvis-hands

Hand tracking in the browser, the visual front end for Jarvis. Today it draws a neon cat's cradle: glowing strings between the fingertips of your two hands, tracked by MediaPipe Hands and drawn with p5.js. Next it becomes a gesture layer for browsing the web.

**Live:** https://dragofgc9.github.io/jarvis-hands/

## Use it

- **Phone:** open the link, tap **Start camera**, allow the camera. Prop the phone up (landscape works best) so both hands are free.
  - Add to Home Screen for a full-screen app icon: iPhone Safari → Share → *Add to Home Screen*; Android Chrome → ⋮ → *Add to Home screen*.
  - **Flip camera** switches to the back camera, so someone else can hold the phone.
- **Computer:** open the link, or run `python3 -m http.server 8000` in this folder and open `http://localhost:8000/`.

| Control | Key | What it does |
|---|---|---|
| View | `V` | Camera view ↔ Void view (video hidden, midnight blue, hand silhouettes, CRT scanlines) |
| Weave | `W` | Adds the crossed cat's-cradle strings |
| HUD | `H` | Hides the readout and hints |

Pull your hands apart to tighten the strings (thinner, hotter colors, a hum when taut). Touch matching fingertips to set off sparks.

## How it's built

One file, `index.html`, no build step. p5.js 1.9.4 and MediaPipe Hands 0.4.1675469240 load from CDNs.

- Detection and rendering are decoupled: one camera frame is in flight to MediaPipe at a time, while p5 renders at full frame rate and eases toward the latest landmarks.
- Strings are verlet ropes pinned at both fingertips, so they sag, swing and stretch.
- Phones (`pointer: coarse`) get the lite model, 640×480 frames and softer glow. The screen is kept awake while tracking.

## Hooking it into Jarvis

Every detection fires a browser event with the raw landmarks:

```js
window.addEventListener("jarvis:hands", (e) => {
  const { t, left, right } = e.detail;   // left/right: 21 landmarks {x, y, z} in 0..1 screen space, or null
});
```

That is the seam for the next steps:

1. **Gestures:** turn landmarks into named gestures (pinch, point, open palm, swipe) in a small module that listens to `jarvis:hands`.
2. **Web surfing:** map gestures to browser actions (scroll, back, click at the pointed spot). Inside a normal page that only works on pages you control, so the likely path is a browser extension or a Jarvis-driven browser that receives the gesture stream.
3. **Jarvis link:** send gestures to Jarvis over a local WebSocket so it can act on them (open a page, read it, draft a reply), keeping Jarvis's rules: it drafts, you send.
