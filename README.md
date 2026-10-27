# watch-loom

A Claude Code plugin that watches a Loom before the agent judges or builds anything: it downloads the video with your
browser's Loom sign-in, pulls timestamped frames and the transcript on your Mac, and makes the agent read every frame. Nothing is
uploaded anywhere.

Loom links are private, so a plain fetch gets nothing. Without this, an agent reviewing a PR "with a Loom" reads the diff
and guesses what the video showed.

## Install (macOS)

```sh
claude plugin marketplace add valentinllpz/watch-loom && claude plugin install watch-loom@valentinllpz
```

That's it. The first Loom you watch installs the rest by itself: `ffmpeg` and `yt-dlp` through Homebrew, and the
[watch plugin](https://github.com/bradautomates/claude-video) (MIT) this one relies on for the download and the frames,
configured for the local engine only. `claude plugin update watch-loom@valentinllpz` picks up changes.

Sign-in happens on first use too: when Loom refuses the download, your default browser opens on loom.com; sign in, say so, and
the agent runs it again. Cookies are read from the Mac's default browser (Chrome, Safari, Firefox, Brave, Edge, Chromium, Vivaldi, Opera; Arc is not supported by yt-dlp). Another browser or profile: `WATCH_LOOM_BROWSER=firefox` or `WATCH_LOOM_BROWSER="chrome:Profile 1"`. Safari needs Full Disk Access for the terminal.

Optional: local speech-to-text for videos without captions (about 1.5 GB):
`python3 ~/.claude/plugins/cache/claude-video/watch/*/skills/watch/scripts/setup.py --engine local --backend whisperx`.
Loom's own captions cover most videos without it.

## Use

`/watch-loom https://www.loom.com/share/<id>`, or just work on a PR or ticket that contains a Loom link; the skill
triggers on its own. Run the watch command with the sandbox off: it reads the browser's cookies from the Keychain.

From a terminal: `~/.claude/plugins/cache/valentinllpz/watch-loom/*/bin/watch-loom <url>` prints the report with the frame paths;
`--setup` only installs what's missing, `--login` only opens the sign-in window.

## What the agent does with it

1. Frames (up to 24 at 1024px, more for long videos) and the transcript. When the author says "look here" or "as you
   can see", the exact seconds are re-extracted so a two-second toast isn't lost between evenly spaced frames.
2. Transcript fallback: when the local transcript is empty, it fetches Loom's own transcript from a logged-in Chrome tab.
3. A "What the Loom shows" note: the flow step by step with timestamps, what's on screen, what the author says it should
   do and calls out as unfinished.

For PR reviews, `review/loom-analysis.md` turns that note into an analysis: the author's claims with timestamps, a
coverage row per acceptance criterion (demoed, implemented), where it was recorded and whether it's stale, anomalies on
screen, contradictions between what's said, shown and coded, and design drift when the ticket has a reference. Paste
that section into your review skill. Its rules keep it from becoming noise: nothing from it is posted as it is, a problem
becomes a candidate that still needs proof, two of them at most per PR.

## Privacy

The download and the frames stay on your Mac. The local engine is forced (`--engine local`); the watch plugin's Gemini
option, which uploads the video to Google, is never used. Delete the work directory the report names when done.

## Files

- `skills/watch-loom/SKILL.md`: the skill.
- `review/loom-analysis.md`: the analysis section for a review skill.
- `bin/watch-loom`: the command the skill runs, usable from a terminal too.
