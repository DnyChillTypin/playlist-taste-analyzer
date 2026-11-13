# Playlist Taste Analyzer: Project Plan

> **Goal:** paste a playlist link and get an in-depth explanation of *what you actually like*.
> We look for patterns that cross genres ("restrained passages building toward a big release" across metal, folk and electronic), not "you like pop and rap".
>
> **Second goal:** learn real ML / data-science skills along the way, and end with a project worth putting on a CV.

---

## 1. The core idea (read this first)

Genre is mostly **timbre**: which instruments are used and how the track is produced.
Taste is often **behaviour over time**: how energy moves, how dense the arrangement is, how much repeats, and where the peak lands.

So the pipeline has five steps:

1. Measure *time-based structure* for every song (build-ups, sparseness, repetition and so on).
2. Compare each measurement with a **baseline** of "typical music", so "high" really means high.
3. **Cluster** songs using only the structural features, deliberately leaving out the genre-like timbre features.
4. Explain each cluster with **evidence**: numbers, example songs and timestamps.
5. Show it all in a small web app.

Choices already made:
- Input is a Spotify or YouTube playlist link; audio is downloaded temporarily, analyzed, then deleted.
- The interface is a web app built with Streamlit.
- The text comes from templates, with no LLM. The depth has to come from the measurements.

---

## 2. Tech stack

| Tool | What it does | Why this one |
|---|---|---|
| Python 3.12 + venv | language + isolated packages | standard for ML/DS |
| Jupyter notebooks | exploring and experimenting | how data scientists actually work |
| `yt-dlp` + `ffmpeg` | fetch playlists and audio | free, works without API keys |
| `spotipy` | read Spotify playlist track lists | official API wrapper (needs a free key) |
| `librosa`, `numpy`, `scipy` | audio → numbers | the standard audio-analysis library |
| `pandas` | tables of features | the core DS tool |
| `scikit-learn` | scaling, clustering, classifiers, metrics | the core ML tool |
| Last.fm API | genre tags per song | free; needed to show *which genres* a pattern spans |
| `plotly` | charts | interactive and works in Streamlit |
| Streamlit | web app in pure Python | no HTML or JavaScript needed |
| SQLite | cache of features | no song is ever analyzed twice |
| `pytest` | tests | shows engineering discipline on a CV |

---

## 3. How we'll work (learning-first)

Every module follows the same loop:

1. **Lesson.** I explain the concept: intuition first, then a little maths, then a tiny code example.
2. **Check yourself.** 2–3 quick questions or "predict what happens" prompts. Answer them before moving on.
3. **Build together.** I write the code, and you read it, run it and ask about any line. You don't have to type it by hand, but you should be able to *explain* it.
4. **Experiment.** You change one thing (a window size, a threshold, a song) and see what changes. This is where the learning sticks.
5. **Log it.** Write 3–5 sentences in `LEARNING_LOG.md`: what you learned, what surprised you, what you'd do differently. These notes become your interview answers.
6. **Commit.** Save the module to git with a clear message, so your GitHub history shows steady progress.

---

## 4. Curriculum + build phases

Each module = **🎓 what you'll learn** → **🔨 what we build** → **✅ how we know it works**.

### Module 0: Tooling and project setup
- 🎓 virtual environments, pip, project structure, git basics (commit, branch, push), Jupyter vs `.py` scripts
- 🔨 venv, installed packages, ffmpeg, `.gitignore`, README skeleton, GitHub repo, `.env` for API keys (never committed!)
- ✅ `python -c "import librosa"` works and the first commit is on GitHub

### Module 1: What digital audio actually is
- 🎓 sample rate, waveform, amplitude, mono/stereo, loudness (RMS, decibels), framing and windowing, smoothing a noisy curve
- 🔨 `audio/fetch.py` (download one song → numpy array) and a notebook that plots its waveform and energy curve
- ✅ you can point at the plot and say "the chorus starts here"

### Module 2: Frequency and the spectrogram
- 🎓 Fourier transform (intuition: a song is a recipe of frequencies), STFT, spectrogram, mel scale, why we use log scales
- 🔨 notebook comparing spectrograms of 3 very different songs
- ✅ you can tell a sparse acoustic song from a dense metal song *just by looking* at the spectrogram

### Module 3: Feature engineering (the heart of the project)
- 🎓 turning signals into meaningful numbers; summary statistics (mean, percentiles, slope); designing features *you can explain*; testing features on synthetic signals
- 🔨 `features/` modules:

| Family | Features | Measured with | Feeds patterns like |
|---|---|---|---|
| Dynamics / arc | build score, peak position (% of song), dynamic range, number of drops | smoothed RMS loudness curve | "restrained → big release" |
| Density / texture | sparseness, fullness, harmonic vs percussive balance | onset density, spectral flatness, HPSS | "sparse and spacious" |
| Vocals | vocal prominence | vocal-band energy of the harmonic part (Demucs later) | "exposed vocals" |
| Repetition / groove | repetition score, steadiness, slow-change score | self-similarity matrix, beat stability | "repetitive grooves, accumulating change" |
| Rhythm | tempo, syncopation, pulse clarity | beat tracker + onsets | "off-kilter rhythms" |
| Harmony / mood | major/minor lean, harmonic complexity, stasis | chroma, key templates | "drone-like", "melancholy" |
| Structure | section count, intro length, surprise | novelty curve segmentation | "patient intros" |
| Timbre (genre-ish) | brightness, warmth, distortion | spectral centroid / rolloff | **excluded from clustering**, used only for contrast |

- ✅ `pytest` passes on synthetic audio: a crescendo sine gets a high build score and a looped pattern gets a high repetition score

### Module 4: Self-similarity and musical structure
- 🎓 similarity matrices, cosine distance, novelty curves, segmentation (how a computer "sees" verse and chorus)
- 🔨 `features/structure.py` + `repetition` features; a notebook visualizing a song's self-similarity matrix
- ✅ the matrix visibly shows the chorus repeating

### Module 5: Data pipelines and caching
- 🎓 tidy data, pandas DataFrames, APIs and JSON, parallel processing, caching, why reproducibility matters
- 🔨 `ingest/` (Spotify / YouTube / pasted list → track list), `resolve.py` (find the right YouTube audio and check its duration), `tags.py` (Last.fm genres), `store.py` (SQLite)
- ✅ a 20-song playlist turns into a features table; running it again is instant

### Module 6: Exploratory data analysis (EDA) and baselines
- 🎓 distributions, histograms, correlation, outliers, standardization (z-scores) vs percentiles, **why you need a reference population**, sampling bias
- 🔨 run the pipeline on ~300 "typical" songs (charts plus mixed-genre playlists) → `baseline.json`; an EDA notebook
- ✅ you can answer "is a build score of 0.7 high?" with a percentile

### Module 7: Unsupervised learning (finding the hidden threads)
- 🎓 distance in many dimensions, the curse of dimensionality, PCA and UMAP for visualization, k-means vs Gaussian mixtures vs hierarchical clustering, choosing k (silhouette score)
- 🔨 `analysis/clusters.py`: cluster on structural features only, then name each cluster from its most distinctive features
- ✅ a 2D map of your playlist where clusters mix genres

### Module 8: Evaluation (the most important DS skill)
- 🎓 how to evaluate when there are no labels: hand-labelled ground truth, Adjusted Rand Index, train/test splits, cross-validation, supervised classifiers (logistic regression, random forest)
- 🔨 two experiments:
  1. **Ground-truth playlist:** 5 slow-builders + 5 sparse-vocal songs + 5 groove songs, each from mixed genres. Does clustering recover the groups? Target: ARI ≥ 0.6.
  2. **Does structure ignore genre?** Train a classifier to predict genre from *timbre* features and then from *structural* features. If structure predicts genre poorly, that **proves** the threads cut across genres. This is the headline result for your CV.
- ✅ a results table with real numbers in the README

### Module 9: Statistics for insights
- 🎓 bimodality (finding "contradictions"), nearest neighbours ("structural twins"), outlier detection, the danger of small samples
- 🔨 `analysis/insights.py` (contradictions, avoidances, peak moments, twins, bridges, outliers, taste drift)
- ✅ every insight cites numbers and songs you can check by listening

### Module 10: Communicating results
- 🎓 explanation design (claim + number + comparison + examples + exception), data visualization principles
- 🔨 `analysis/templates.py` + `app.py` (Streamlit): URL box → progress bar → report with radar chart, energy curves and audio clips that start at the peak moment
- ✅ a friend reads their report and says "huh, that's true"

### Module 11: Shipping
- 🎓 READMEs that sell a project, deployment, demos
- 🔨 polished README, live demo, write-up (see section 6)
- ✅ the project link goes on the CV

### Later (optional stretch goals)
- Demucs vocal separation for better vocal measures
- pretrained music embeddings (CLAP / MERT) compared with your hand-made features; a great "deep learning vs feature engineering" experiment
- an optional LLM narration layer that reuses the same evidence objects

---

## 5. The report the app will produce

1. **Hidden threads:** cross-genre clusters, as in your reference table, each with a name, the genres involved and example songs.
2. **Taste fingerprint:** about 8 dimensions (patience, density, repetition, vocal exposure, rhythmic complexity, darkness, unpredictability, intensity) shown as percentiles against typical music.
3. **Your contradictions:** where your taste splits in two ("very sparse *or* very dense, nothing in between").
4. **What you avoid:** kinds of music you never touch.
5. **The moments you're chasing:** the exact peak moment in each song, with a timestamp and a playable clip.
6. **Structural twins:** song pairs from different genres that are built the same way.
7. **Bridge songs and outliers:** gateway songs between threads, and songs that fit nowhere.
8. **Taste drift:** how older additions differ from recent ones (when dates are available).

Example of the target writing quality:
> *"You're drawn to patience. In 14 of your 38 songs the loudest moment comes after 70% of the song (typical music: around 45%). It happens across metal (Song A, peak at 5:12), folk (Song B, 3:40) and electronic (Song C, 6:05). The exception is your hip-hop picks, which hit early and stay level."*

---

## 6. Making it CV-worthy

**GitHub (main home)**
- Clean README: a one-line pitch, a screenshot or GIF, "How it works" diagram, **Results** section with the Module 8 numbers, and "What I learned".
- Steady commit history (one module = several commits), tests, and an `requirements.txt` so others can run it.
- Pin the repo on your GitHub profile.

**Beyond GitHub**
- **Live demo** on Hugging Face Spaces or Streamlit Community Cloud (both free). YouTube blocks downloads from cloud servers, so the hosted demo shows a few *pre-analyzed* playlists, while the full version runs locally. Recruiters click links, not code.
- **Technical write-up** (personal blog, Medium, or dev.to): "Can a computer tell *why* you like a song?" Showing the experiments and charts proves you can communicate, which data scientists are hired for.
- **Demo video** (60–90 s, Loom or YouTube) embedded in the README.
- **Public dataset** on Kaggle or Hugging Face Datasets: the numeric features of the ~300 baseline songs (no audio). It adds a "published a dataset" line.
- **LinkedIn post** with the demo video once it's live.
- **CV bullet** (example, fill in real numbers at the end):
  > *Built an end-to-end audio ML pipeline (librosa, scikit-learn, Streamlit) that extracts 30+ interpretable structural features from full songs and clusters playlists into cross-genre taste patterns; validated with hand-labelled ground truth (ARI 0.7) and showed structural features predict genre 40% worse than timbre features, confirming genre-independent patterns.*

---

## 7. Project layout (target)

```
A:\Music\
  README.md  PLAN.md  LEARNING_LOG.md  requirements.txt  .gitignore  .env (not committed)
  notebooks/              # 01_waveform, 02_spectrogram, 06_eda, 07_clustering, 08_evaluation ...
  taste/
    app.py
    ingest/   spotify.py youtube.py resolve.py tags.py
    audio/    fetch.py
    features/ dynamics.py texture.py rhythm.py harmony.py structure.py vocals.py
    analysis/ baseline.py fingerprint.py clusters.py insights.py templates.py
    store.py
  tests/
```

---

## 8. Scalability (later, after Module 11)

Version 1 runs on one laptop for one user. These upgrades would let it serve many users and big playlists. Each is also a good learning topic and CV talking point.

| Problem at scale | Upgrade | What you'd learn |
|---|---|---|
| Analysis is slow (10–15 s per song) | Background job queue (e.g. Celery or RQ + Redis), so the web app stays responsive while workers process songs | async processing, workers, queues |
| One machine isn't enough | Several workers in Docker containers, scaled up or down | Docker, containerization, horizontal scaling |
| The same songs get analyzed by different users | **Shared feature cache** keyed by track ID: a popular song is analyzed once, ever | caching strategy, cache hit rate |
| SQLite can't handle many writers | Move to PostgreSQL; store curves in Parquet files or object storage (S3-style) | databases, data storage formats |
| Comparing a song with thousands of others is slow | Vector index for nearest-neighbour search (e.g. FAISS or pgvector) | approximate nearest neighbours, embeddings |
| Code changes break old results | Version the feature code; re-compute only when the version changes | reproducibility, data versioning |
| Hard to know what's slow or broken | Logging, timing per pipeline step, simple dashboards | monitoring / observability |
| YouTube blocks cloud servers | Let users upload audio or connect their own library; respect rate limits | real-world data constraints |

Design choices *now* that make this easier later (keep to them from Module 1):
- Each pipeline step is a separate function with clear inputs and outputs (fetch → features → store → analyze), so steps can later run on different machines.
- Features are stored per song, never per playlist, so they can be reused across users.
- Every stored feature row records the feature-code version.

---

## 9. Known risks
- **Wrong YouTube match** (live versions, covers): checked by duration and by preferring official channels.
- **Spotify API rules keep changing:** YouTube links and pasted track lists always work as a fallback.
- **Speed:** about 10–15 s per song on the first run; cached results are instant.
- **Downloading audio is a legal gray area:** personal use only, audio is deleted right after analysis, and only numbers are stored.
