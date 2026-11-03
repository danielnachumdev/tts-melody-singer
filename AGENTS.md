# TTS Singer – Agent Context

Implementation guide for coding agents changing **tts_singer**. Product pitch and user install: [`README.md`](README.md).

---

## Orientation

| Question | Answer |
| -------- | ------ |
| What is this? | Offline **TTS melody singer** experiment (pyttsx3 + WAV pitch/speed stubs) |
| Package root | `src/` (import as `from src import MaleSinger, Melody, …`) |
| Entry demos | `python main.py` (sing); `python suprise.py` (speak) |
| Toolchain | `pip` + [`requirements.txt`](requirements.txt) (no `pyproject.toml` / uv yet) |
| Tests | None yet |
| Secrets | None required; do not add API keys or commit credentials |

```text
.
├── .github/                 # Optional / in-flight: CODEOWNERS, dependabot, thin CI
├── src/
│   ├── consts/              # WPM_VALUES enum
│   └── models/
│       ├── melody.py        # Melody.from_words / random_from_words
│       ├── melody_info.py   # MelodyInfo dataclass
│       ├── speaker/         # Speaker + MaleSpeaker / FemaleSpeaker
│       ├── singer/          # Singer + workers + MaleSinger / FemaleSinger
│       └── audio_processing/# AudioSample (load/save; pitch/speed stubs)
├── main.py                  # Sing demo
├── suprise.py               # Speak demo (filename spelling is historical)
├── main.bat                 # Windows: ./venv → suprise.py
├── requirements.txt
├── README.md
└── AGENTS.md
```

Default branch: **`main`**.

Shared CI/CD (when workflows are present): [`danielnachumdev/actions`](https://github.com/danielnachumdev/actions) (`@latest`, `standards-ref: latest`). How to adopt: [`HOW-TO-USE.md`](https://github.com/danielnachumdev/actions/blob/v1/HOW-TO-USE.md). Thin CI may use `allow-empty: true` until real tests exist.

---

## Develop

```bash
python -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt
python main.py
```

| Task | Command / note |
| ---- | -------------- |
| Sing demo | `python main.py` |
| Speak demo | `python suprise.py` |
| Lint (local) | `.pylintrc` present; no enforced CI lint gate yet |
| Types / tests | None configured |

**Runtime deps:** `pyttsx3`, `danielutils`, `numpy`, `scipy`, `matplotlib`.

Prefer editing `requirements.txt` and reinstalling into the venv. Dependabot (when enabled) covers `pip` + `github-actions` with `noBump`.

---

## Code style & domain notes

- Public surface is re-exported from `src/__init__.py` via `consts` + `models` star-imports (`MaleSinger`, `FemaleSinger`, `MaleSpeaker`, `FemaleSpeaker`, `Speaker`, `Melody`, `MelodyInfo`, `AudioSample`, `WPM_VALUES`, …).
- **`Melody.from_words`:** `text.split(" ")` length must equal `pitches` and `wpms` lengths or it raises `ValueError`.
- **`Singer.sing`:** clears/creates `./tmp`, submits `(index, MelodyInfo, speaker)` jobs to `danielutils.WorkerPool` (`SingerWorker`). Worker writes `./tmp/{i}.wav` via `speaker.save`, then `AudioSample.load(…).set_pitch(…).set_speed(…).save(./tmp/{i}p.wav)`.
- **`AudioSample.set_pitch` / `set_speed`:** currently **no-ops** (return `self` unchanged). Do not document as working DSP until implemented. `Singer.save` is likewise a stub (`pass`).
- **`Speaker`:** wraps pyttsx3; `say` / `save` are `@atomic` (danielutils). `FemaleSpeaker` selects `voices[1]` — assume ≥2 system voices or handle IndexError when changing voice logic.
- **`main.bat`** expects `./venv/Scripts/python.exe` (not `.venv`). Keep that path or update the bat when changing Windows layout.
- Do not rewrite onto uv/pyproject in drive-by PRs unless that is the task; match existing `src/` + `requirements.txt` layout. Prefer class-first / SRP modules (writing-python skill) when adding code.
- Demo strings in tree may be personal; **new** examples / tests / PR artifacts should use fabricated text only (no real PII).

---

## PRs, CI, and releases

1. Branch off `main`; open a PR into `main`.
2. Attach **exactly one** semver label: `noBump` | `bumpPatch` | `bumpMinor` | `bumpMajor`.
3. When CI exists: prefer required checks **`CI / CI`** and **`Semver / Semver`** from thin orchestrators calling `danielnachumdev/actions`.
4. Dependabot PRs stay on `noBump`.
5. Change shared CI/semver behavior in **`danielnachumdev/actions`**, not by re-vendoring here.

### Sensitive / ignored

Do not commit: `.venv/`, `venv/`, `__pycache__/`, `./tmp/` audio artifacts, API keys, or local secrets. `tmp/` is already gitignored.
