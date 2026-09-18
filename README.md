# pi-jingle

Play sounds on pi events.

> Fork of [Git-Monke/pi-jingle](https://github.com/Git-Monke/pi-jingle), based on commit `7e265ce`.
> Changes in this fork:
> - Added the `agent_settled` and `ui_prompt_start` events.
> - With no `sounds` key configured, the bundled `done.mp3` now plays on `agent_settled` and
>   `ui_prompt_start` instead of `agent_end`. `agent_end` also fires when pi is about to
>   auto-retry, auto-compact and retry, or continue with queued follow-up messages, so it reports
>   "your turn" too early; `agent_settled` fires only once pi will not continue by itself. Both
>   fire back to back at the end of a run, so enabling both would ring twice.
> - Volume is applied by a player that supports it (`afplay -v` on macOS, `paplay --volume` or
>   `ffplay -af` on Linux) instead of always preferring ffplay.

## Install

```bash
pi install git:github.com/yuhaoxin/pi-jingle
```

## Configuration

Add to `~/.pi/agent/settings.json`:

```json
{
  "sounds": {
    "agent_settled": "/path/to/done.mp3",
    "ui_prompt_start": "/path/to/question.mp3",
    "when_coding": "/path/to/music.mp3"
  }
}
```

Only the event names present in `sounds` play; an absent key stays silent. When `sounds` is
absent entirely, the bundled `done.mp3` is used for `agent_settled` and `ui_prompt_start`.

**Path formats:**
- `/absolute/path.mp3` - absolute path
- `~/sounds/file.mp3` - resolves to `$HOME/sounds/file.mp3`
- `./sounds/file.mp3` - resolves to `~/.pi/sounds/file.mp3`

**Volume:** Use an object for volume control (0.0 - 1.0):
```json
{
  "sounds": {
    "agent_settled": { "path": "/path/to/sound.mp3", "volume": 0.5 }
  }
}
```

Volume needs a player that supports it: `afplay` on macOS (built-in), or `paplay` (PulseAudio,
0.0-1.0 is mapped to the integer range 0-65536) / `ffplay` on Linux. `aplay` and the Windows
`SoundPlayer` ignore the volume value; the sound still plays through them at full volume.

## Supported Events

| Event | Description |
|-------|-------------|
| `agent_start` | Task begins |
| `agent_end` | A low-level agent run ends; pi may still auto-retry, auto-compact and retry, or continue with queued follow-up messages |
| `agent_settled` | pi will not continue by itself: the run is over and it is the user's turn |
| `ui_prompt_start` | An extension opened a blocking `ctx.ui` prompt (`select`, `confirm`, `input`, `editor`, `custom`) and waits for the user; nested or overlapping prompts are coalesced into one event |
| `session_start` | pi starts |
| `session_shutdown` | pi closes |
| `turn_start` | User message received |
| `turn_end` | Response sent |
| `tool_call` | Tool execution |
| `tool_result` | Tool result received |

**`when_coding`:** Plays a song from `agent_start` until `agent_end`. Looping requires `ffplay`
(with a configured volume on macOS, always on Linux); without it, macOS falls back to `afplay`,
which plays the file once.

**Not covered:** pi's own dialogs (startup project trust, `/login`, session and model pickers) do
not go through `ctx.ui`, so they emit no `ui_prompt_start` and cannot trigger a sound.

## Commands

- `/sounds list` - Show configured sounds
- `/sounds reload` - Reload config

## Requirements

Sound player for your platform:
- **macOS**: afplay (built-in) or ffplay
- **Linux**: paplay, aplay, or ffplay
- **Windows**: PowerShell (built-in)

The Linux player order is written against the documented `paplay`/`aplay`/`ffplay` flags but has
not been verified on a Linux machine yet.
