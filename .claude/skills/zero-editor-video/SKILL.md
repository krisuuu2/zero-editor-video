---
name: zero-editor-video
description: Make short videos (reels, Shorts, TikToks, explainers) entirely in code, with no video editor and no video framework such as Remotion or HyperFrames. The agent builds a small pipeline (word timings from the voiceover, a Canvas2D scene drawn frame by frame in a browser, FFmpeg to encode) and renders a vertical MP4 timed to the user's words. Use when the user wants to make a video, reel, Short or animated explainer from a script and a voiceover, or asks how to make videos with an AI agent without an editor.
---

# zero-editor-video

You are the user's video studio. You make the whole video with code: a browser draws every frame, and FFmpeg joins the frames and the voice into an MP4. Use no video framework (no Remotion, no HyperFrames) and no paid rendering service.

## Before you start
- Check that **Node.js 18+**, **Python 3.10+** and **FFmpeg** are installed (`node -v`, `python --version`, `ffmpeg -version`). If one is missing, tell the user how to install it for their system and wait.
- The user needs `script.txt` (what they say) and `vo.mp3` (their voiceover) in the project folder. If they don't have a voiceover yet, they can record one on their phone.
- Ask, in one short message, for the topic, the look (colours, fonts, mood) and the format. Default: vertical 1080×1920, 30 fps.

## Build the pipeline (once per project)
1. **Word timings.** `timings.py` transcribes `vo.mp3` locally with `faster-whisper` (`word_timestamps=True`) and writes `vo.json` as a list of `{ "w", "t", "e" }`. Compare the words with `script.txt` and fix spellings to match the script. Report any word the voice actually got wrong.
2. **The scene.** `scene.js` is a Canvas2D scene where **every frame is a pure function of time**: `draw(t)` draws second `t` from scratch, with no state between frames, so rendering is deterministic and frames can render in any order.
   - `at(word, n)` returns the start of the n-th occurrence of a word in `vo.json`. Put every animation on a spoken word, never on a hard-coded second.
   - Cuts are a list of `[startWord, beatFunction]`. A new beat starts on its word with a short zoom punch-in and about 0.2 s of motion blur.
   - Captions follow the voice in two tiers: the small words in a serif italic, the key word big, in bold caps and an accent colour.
   - Never leave a still frame. The ground always moves a little (slow gradient drift, a slow push-in).
   - Show real things, not stock images. When the script names a website, take a mobile full-page screenshot of the real page (Puppeteer), scroll it to the phrase being spoken and sweep a highlighter over it.
3. **Render.** `render.mjs` uses `puppeteer`, which downloads its own Chrome. It loads `index.html` + `scene.js`, waits for fonts and images, calls `draw(i / 30)` for every frame and saves `frames/f00000.jpg`. Render with several pages in parallel.
4. **Check before you encode.** `render.mjs --sheet=1,5,10,…` writes a contact sheet of chosen seconds. Look at it. `--check` logs every text box and fails when text leaves the safe zone (keep text away from the top 220 px, the bottom 380 px and 80–130 px at the sides, where the app's buttons sit) or reads under 4.5:1 contrast. Sample the background **before** you draw the text. Fix every failure.
5. **Encode.** `ffmpeg -framerate 30 -i frames/f%05d.jpg -i vo.mp3 -c:v libx264 -pix_fmt yuv420p -crf 18 -c:a aac -shortest -movflags +faststart out.mp4`
6. **Verify the file you deliver.** Transcribe `out.mp4` again and compare it with the script. Pull frames at the start, in the middle and at the end, and look at every one that shows a face or key text. Only then say it's done.

## Rules
- The first frame already shows content. There is no silence at the start: trim to the first sound.
- Never cover a face with captions or graphics.
- Don't invent facts or numbers in the video. Use what the script and the user's sources say.
- Show the user a small preview (720p, `-crf 24`, about 8 MB) before the final render if they want to approve it on their phone.
- Start with a 15-second test from the first lines of the script, get a yes, then render the whole video.
