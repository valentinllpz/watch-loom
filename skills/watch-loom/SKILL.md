---
name: watch-loom
description: Use whenever a PR, a ticket, a chat message or a task has a loom.com/share link and you are reviewing, implementing or acting on it. Watches the Loom (frames + transcript, on this Mac) and returns what it shows, so the work is judged against what the author demonstrated. Usage: /watch-loom <loom url> [what to check it against]
allowed-tools: Bash, Read, mcp__claude-in-chrome__*
---

# Watch a Loom

Loom links are private, so a plain fetch gets nothing. This downloads the video with your browser's Loom sign-in, pulls frames and the transcript, and you read the frames yourself. Nothing leaves the Mac.

Run it for **every** `loom.com/share/<id>` link in what you are working on (PR body, ticket description or comments, a chat thread), before you judge or build anything.

## 1. Frames + transcript

```bash
B=${CLAUDE_PLUGIN_ROOT:-$(ls -d ~/.claude/plugins/cache/valentinllpz/watch-loom/* 2>/dev/null | tail -1)}
WATCH_LOOM_OUT="<your scratchpad dir>" "$B/bin/watch-loom" "<loom url>"
```

- Run the Bash call with the **sandbox disabled**: the sandbox blocks the Keychain, the browser's cookies can't be decrypted and Loom answers 403.
- The command sets itself up on first use (ffmpeg, yt-dlp, the watch plugin, its local-only config); that can take a
  minute the first time. It always uses the local engine, never Gemini: company videos must not be uploaded.
- **Exit 3 means the browser isn't signed in to Loom.** The command has opened the user's default browser on loom.com.
  Tell the user in one line to sign in there and say when it's done, then run the same command once more. Don't retry
  on your own, and don't try other ways to fetch the video.
- **Exit 2** is a setup problem; its message says what to install. Say so in one line and go to step 2 only.
- The report lists frame paths with `t=MM:SS`. **Read every frame** with the Read tool. A longer video (over ~5 min): add `--max-frames 40`, or focus with `--start/--end`.
- **Cue frames:** when the transcript has the author pointing at something ("look here", "now you can see", "notice",
  "as you can see"), run the command again with `--detail transcript --timestamps <mm:ss,...>` to get those exact
  moments. A toast on screen for two seconds is easy to miss between evenly spaced frames.
- The sign-in is read from the Mac's default browser (Chrome, Safari, Firefox, Brave, Edge, Chromium, Vivaldi, Opera;
  not Arc). Another browser or profile: `WATCH_LOOM_BROWSER=firefox` or `WATCH_LOOM_BROWSER="chrome:Profile 1"` in front
  of the command.

## 2. Transcript fallback

If the report's transcript is empty or missing (WhisperX often returns nothing on Looms), get Loom's own transcript from a logged-in Chrome tab: open the share URL with the claude-in-chrome tools, then run with `javascript_tool`:

```js
const id = location.pathname.split('/').pop();
const r = await fetch('/graphql', {method: 'POST', headers: {'content-type': 'application/json', 'x-loom-request-source': 'loom_web'},
  body: JSON.stringify({operationName: 'FetchVideoTranscript', variables: {videoId: id, password: null},
  query: 'query FetchVideoTranscript($videoId: ID!, $password: String) { fetchVideoTranscript(videoId: $videoId, password: $password) { ... on VideoTranscriptDetails { source_url } } }'})}).then(r => r.json());
const t = await fetch(r.data.fetchVideoTranscript.source_url).then(r => r.json());
t.phrases.map(p => Math.round(p.ts) + 's ' + p.value).join('\n')
```

Read it in slices (`.slice(0, 3000)`, then on) if it is cut off. Don't click play on the page: playback hangs the browser tool. Close the tab when done.

## 3. What to take from it

Write a short **What the Loom shows** note for the work you are doing:
- the flow demonstrated, step by step, with timestamps;
- what is on screen: UI states, copy, numbers, errors;
- what the author says it should do, and anything they call out as unfinished or a known issue.

Then use it:
- **Reviewing a PR:** start the step 1 command first, in the background (`run_in_background`), and read the diff while it downloads. Then check the diff does what the Loom shows, and look hardest at flows the Loom skips or that differ from the code. The Loom informs the review; never post about the Loom itself, and a missing Loom never affects the verdict.
- **Implementing a ticket:** treat what the Loom shows as part of the spec (expected behaviour, repro steps for a bug, the target UI) and carry the note into the plan.

Delete the work dir the report names when you are done.
