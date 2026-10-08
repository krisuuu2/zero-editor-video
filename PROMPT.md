# The prompt: Claude makes the whole video (no Remotion, no HyperFrames)

Paste everything below the line into Claude Code (or Codex, Cursor, Gemini CLI) in an empty folder.
You need **Node.js 18+**, **Python 3.10+** and **FFmpeg**. All of them are free.
Put your voiceover in the folder as `vo.mp3` and your script as `script.txt`.

---

You are my video studio. Build a small, reusable pipeline in this folder that turns `script.txt` + `vo.mp3` into a vertical 1080×1920, 30 fps MP4 with motion graphics timed to my words. Use no video framework (no Remotion, no HyperFrames) and no paid service. Use only a browser canvas, Node, Python and FFmpeg.

**1. Word timings.** Write `timings.py`, which transcribes `vo.mp3` locally with `faster-whisper` (word timestamps on) and writes `vo.json`: a list of `{ "w": word, "t": start, "e": end }`. Compare the words with `script.txt`, list any differences and fix the spellings to match the script.

**2. The scene.** Write `scene.js`, a Canvas2D scene where **every frame is a pure function of time**: `draw(t)` draws the frame for second `t` from scratch, with no state carried between frames. That makes rendering deterministic and lets frames render in any order.
- Helpers: `at(word, n)` returns the time of the n-th occurrence of a word in `vo.json`, so every animation is placed on a spoken word, never on a hard-coded second.
- Cuts: a list of beats `[startWord, drawFunction]`. A new beat starts on its word, with a short zoom punch-in and 0.2 s of motion blur.
- Captions: two-tier captions on every word group: small words in a serif italic, the key word big in bold caps and an accent colour.
- Never a still frame: the ground always moves slightly (slow gradient drift, a slow push-in).
- Real things, not stock: when I mention a website, take a mobile full-page screenshot of the real page with Puppeteer and scroll it to the phrase I'm saying, with a highlighter sweep.

**3. Render.** Write `render.mjs` with `puppeteer` (it downloads its own Chrome). It loads `index.html` + `scene.js`, waits for fonts and images, and for each frame calls `draw(i / 30)`, then saves `frames/f00000.jpg`. Use several pages in parallel.

**4. Check before encoding.** Make `render.mjs --sheet=1,5,10,...` write a contact sheet of chosen seconds so you can look at it. Also add `--check`, which logs every text box and fails if text leaves the 9:16 safe zone (away from the top 220 px, the bottom 380 px and the sides, where the app's buttons sit) or reads under 4.5:1 contrast. Sample the background before drawing the text. Fix every failure before rendering.

**5. Encode.** Join the frames and the voiceover with FFmpeg: `ffmpeg -framerate 30 -i frames/f%05d.jpg -i vo.mp3 -c:v libx264 -pix_fmt yuv420p -crf 18 -c:a aac -shortest -movflags +faststart out.mp4`.

**6. Verify the file you deliver.** Transcribe `out.mp4` again and check it says the script. Pull frames at the start, middle and end with FFmpeg, look at every one with a face or key text (captions must never cover a face), and only then tell me it's done.

Start by asking me for the topic and style of the first video (colours, fonts, mood). Then build the pipeline and render a 15-second test from the first lines of my script.
