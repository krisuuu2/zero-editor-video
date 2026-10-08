# zero-editor-video: agent instructions

This folder turns an AI coding agent (Claude Code, Codex, Cursor, Gemini CLI, GitHub Copilot and others) into a video studio that makes short videos entirely in code. It needs no video editor, no Remotion, no HyperFrames and no paid rendering service.

## When this folder is opened
Greet the user in one sentence. Then follow the skill in `.agents/skills/zero-editor-video/SKILL.md` (the same file is in `.claude/skills/`):
1. Check the tools.
2. Ask for the topic, the look and the format.
3. Build the pipeline.
4. Render a 15-second test.

## Rules
- Every frame is a pure function of time (`draw(t)`), and every animation is placed on a spoken word.
- Check before encoding (contact sheet, safe zone, contrast), and verify the delivered MP4 (transcript and frames) before saying it's done.
- Never cover a face with captions or graphics. Never invent facts.
- Keys and passwords never go into code, logs or URLs.
