# Playlist Taste Analyzer

> Paste a playlist, find out **why** you like what you like: the musical patterns that run across genres, not just a list of genres.

🚧 **Work in progress.** Built step by step as a learning project; see [PLAN.md](PLAN.md) for the roadmap.

## The idea

Most "music taste" tools stop at *"you like pop and rap"*. This project looks at **how songs behave over time** and finds patterns that cut across genres, for example:

| Pattern                                              | Could appear in                 |
| ---------------------------------------------------- | ------------------------------- |
| Restrained passages building toward a big release    | metal, folk, electronic         |
| Exposed vocals against sparse, spacious arrangements | R&B, acoustic pop, ambient      |
| Repetitive grooves with small, accumulating changes  | funk, techno, experimental rock |

## How it works (planned)

1. **Fetch:** read a Spotify/YouTube playlist and temporarily download each song's audio.
2. **Measure:** extract interpretable features from the full song (energy arc, density, repetition, rhythm, harmony, structure).
3. **Compare:** turn each measurement into a percentile against a baseline of typical music.
4. **Cluster:** group songs on *structural* features only, so the groups cut across genres.
5. **Explain:** produce a report where every claim cites numbers, example songs and timestamps.

## Tech stack

Python · librosa · NumPy · scikit-learn · pandas · Streamlit · yt-dlp

## Setup

```powershell
py -3.12 -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
winget install ffmpeg
```

## Results

*Coming in Module 8. Only numbers from experiments actually run will appear here.*
