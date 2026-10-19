# Youtube-Downloader

Drop your links in a JSON file, run one command, and watch them download.

![Python](https://img.shields.io/badge/Python-3.9%2B-3776AB?logo=python&logoColor=white)
![yt-dlp](https://img.shields.io/badge/powered%20by-yt--dlp-red)
![rich](https://img.shields.io/badge/CLI-rich-8A2BE2)

## Table of Contents

- [Features](#features)
- [Project Structure](#project-structure)
- [Quick Start](#quick-start)
- [Link File Format](#link-file-format)
- [Usage](#usage)
- [Options](#options)
- [How It Works](#how-it-works)
- [Troubleshooting](#troubleshooting)
- [Legal](#legal)

## Features

- **JSON-driven queue:** put links in `Links/ToDownload.json`, nothing else to configure
- **Auto-tracking:** finished links are copied to `Links/Completed.json` right away
- **Smart skipping:** links already in `Completed.json` are never downloaded twice
- **Video or audio:** MP4 video (360p to 4K) or audio-only (MP3 / M4A / Opus)
- **Playlist friendly:** playlist links download into their own subfolder
- **Clean CLI:** banner, queue table, live progress bars with speed and ETA, summary report
- **Safe to interrupt:** press `Ctrl+C` any time and the next run continues where you left off
- **Retries and resume:** automatic retries and partial-file resume
- **Optional pruning:** `--prune` removes finished links from the queue

## Project Structure

```
your-project/
├── Core/
│   └── Downloader.py        # the script
├── Downloads/               # downloaded files land here
└── Links/
    ├── ToDownload.json      # input: links to download
    └── Completed.json       # output: links that finished
```

## Quick Start

### 1. Install dependencies

```bash
pip install yt-dlp rich
```

### 2. Install FFmpeg

FFmpeg merges video and audio streams and converts audio to MP3.

```bash
# macOS
brew install ffmpeg

# Debian / Ubuntu
sudo apt install ffmpeg

# Windows
winget install ffmpeg
```

### 3. Add your links

Edit `Links/ToDownload.json`:

```json
[
  "https://www.youtube.com/watch?v=VIDEO_ID_1",
  "https://www.youtube.com/watch?v=VIDEO_ID_2"
]
```

### 4. Run it

```bash
python Core/Downloader.py
```

Run it from the project root (the folder that contains `Core/`, `Links/` and `Downloads/`).

## Link File Format

`ToDownload.json` is flexible. All of these work:

**A plain list** (recommended)

```json
["https://example.com/a", "https://example.com/b"]
```

**A list of objects** (using a `url` or `link` key)

```json
[
  { "url": "https://example.com/a", "title": "Optional label" },
  { "link": "https://example.com/b" }
]
```

**A wrapped list**

```json
{ "links": ["https://example.com/a", "https://example.com/b"] }
```

Duplicate links are ignored automatically.

### What `Completed.json` looks like

If the file is empty or already holds objects, each finished link is saved with details:

```json
[
  {
    "url": "https://www.youtube.com/watch?v=VIDEO_ID_1",
    "title": "Some Video Title",
    "mode": "video",
    "quality": "1080p",
    "file": "/path/to/Downloads/Some Video Title [VIDEO_ID_1].mp4",
    "downloaded_at": "2026-10-19T14:32:10",
    "seconds": 42.7
  }
]
```

If the file already contains plain strings, the script keeps that style and only appends the link.

## Usage

### Interactive mode

```bash
python Core/Downloader.py
```

You will be asked whether you want video or audio, and which quality.

### One-liners

```bash
# 1080p video, no questions asked
python Core/Downloader.py -m video -q 1080p -y

# 320k MP3 audio
python Core/Downloader.py -m audio -q 320k -a mp3

# Preview the queue without downloading
python Core/Downloader.py --dry-run

# Download, then remove finished links from ToDownload.json
python Core/Downloader.py -m video -q 720p --prune
```

## Options

| Flag | Description | Values / Default |
|---|---|---|
| `-m`, `--mode` | What to download | `video` or `audio` |
| `-q`, `--quality` | Quality target | Video: `best` `2160p` `1440p` `1080p` `720p` `480p` `360p`. Audio: `320k` `256k` `192k` `128k` |
| `-a`, `--audio-format` | Audio container | `mp3` (default), `m4a`, `opus` |
| `--prune` | Remove finished links from `ToDownload.json` | off |
| `--dry-run` | Show the queue and exit | off |
| `-y`, `--yes` | Skip the confirmation prompt | off |

## How It Works

```
 ToDownload.json
        |
        v
 +------------------+
 | 1. Load + dedupe |   skip anything already in Completed.json
 +--------+---------+
          v
 +------------------+
 | 2. Pick settings |   video/audio + quality (flags or prompts)
 +--------+---------+
          v
 +------------------+
 | 3. yt-dlp        |   one link at a time, live progress bars
 +--------+---------+
          v
   +------+-------+
   v              v
 success        failure
 copied to      stays in
 Completed.json ToDownload.json
```

- Each success is written to disk immediately, so a crash or `Ctrl+C` never loses progress.
- Files are written atomically, so the JSON files can't be left half-written.
- Failed links stay in the queue so you can simply run the script again.

## Troubleshooting

<details>
<summary><b>"Missing dependency" on startup</b></summary>

Run `pip install yt-dlp rich`.
</details>

<details>
<summary><b>FFmpeg warning, or downloads finish without merging / MP3 conversion fails</b></summary>

Install FFmpeg (see [Quick Start](#quick-start)) and make sure `ffmpeg -version` works in your terminal.
</details>

<details>
<summary><b>"ToDownload.json is not valid JSON"</b></summary>

Check for trailing commas or missing quotes. Pasting the file into a JSON validator will point to the exact line.
</details>

<details>
<summary><b>Nothing to do, but I have links</b></summary>

They are probably already listed in `Completed.json`. Remove an entry from there to download it again.
</details>

<details>
<summary><b>Downloads suddenly start failing</b></summary>

Sites change often. Update the downloader engine: `pip install -U yt-dlp`.
</details>
