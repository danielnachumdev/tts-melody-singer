# tts_singer

Sing text as a simple melody via offline TTS — Python experiments with [pyttsx3](https://pyttsx3.readthedocs.io/), pitch/speed stubs, and a small worker pool.

Build a `Melody` (words + pitch Hz + WPM), then have a `MaleSinger` / `FemaleSinger` synthesize each phrase to WAV under `./tmp` and (eventually) reshape pitch/speed. Plain speech demos use `MaleSpeaker` / `FemaleSpeaker`.

## Quick start

Needs a system TTS backend that pyttsx3 can drive (e.g. SAPI5 on Windows, espeak/NSSpeech on Linux/macOS).

```bash
python -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt
python main.py              # MaleSinger + Melody.random_from_words demo
# or: python suprise.py     # MaleSpeaker.say demo (filename spelling is historical)
```

Minimal API sketch:

```python
from src import MaleSinger, Melody

melody = Melody.random_from_words("hello world from the melody singer")
MaleSinger().sing(melody)
```

Or pin pitches / rates explicitly:

```python
from src import Melody

melody = Melody.from_words(
    "hello world",
    pitches=[220, 330],
    wpms=[200, 200],
)
```

## Layout

| Path | Role |
|------|------|
| `main.py` | Demo: random melody → `MaleSinger.sing` |
| `suprise.py` | Demo: `MaleSpeaker.say` (typo in filename kept) |
| `main.bat` | Windows helper that runs `suprise.py` via `./venv` |
| `src/` | Library: speakers, singers, melody, audio stubs |
| `requirements.txt` | Runtime deps |

Agent / contributor notes: [`AGENTS.md`](./AGENTS.md).
