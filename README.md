# dayrec

A tiny CLI that records audio in the background all day to plain WAV files. No GUI, no daemon framework, one POSIX shell script on top of `ffmpeg`.

```sh
dayrec start     # start recording in the background
dayrec status    # running? current file, disk used
dayrec stop      # stop; WAV headers are finalized cleanly
dayrec devices   # list mics
dayrec prune     # delete day folders older than DAYREC_KEEP_DAYS (30)
```

Output: one folder per day, one WAV per hour, aligned to the clock:

```
~/Recordings/dayrec/2026-09-25/09-00-00.wav
~/Recordings/dayrec/2026-09-25/10-00-00.wav
```

## Install

Requires `ffmpeg` (Linux: PulseAudio/PipeWire or ALSA; macOS: avfoundation).

```sh
cp dayrec ~/.local/bin/
dayrec start
```

Start automatically:

- **Linux (systemd):** `cp contrib/dayrec.service ~/.config/systemd/user/ && systemctl --user enable --now dayrec`
- **Anywhere with cron:** `@reboot $HOME/.local/bin/dayrec start` in `crontab -e`

On macOS, grant microphone access to your terminal (or `ffmpeg`) the first time.

## Settings

Environment variables, all optional:

| Variable | Default | Notes |
|---|---|---|
| `DAYREC_DIR` | `~/Recordings/dayrec` | output folder |
| `DAYREC_RATE` | `16000` | Hz; 16 kHz is plenty for speech |
| `DAYREC_CHANNELS` | `1` | mono |
| `DAYREC_CHUNK` | `3600` | seconds per file |
| `DAYREC_DEVICE` | system default | see `dayrec devices` |
| `DAYREC_FORMAT` | auto | `pulse`, `alsa`, `avfoundation` |
| `DAYREC_KEEP_DAYS` | `30` | used by `prune` |

## Resources

It's one `ffmpeg` process writing raw PCM (no encoding), run at `nice 10`. CPU use is well under 1%. Disk: 16 kHz mono 16-bit is about 115 MB/hour, 2.7 GB/day. Hourly chunks keep every file far below the 4 GB WAV limit and mean a crash or power loss costs at most the current hour's header.

If the mic disappears (unplugged, busy), dayrec logs it to `$DAYREC_DIR/.dayrec.log` and retries every 10 seconds.
