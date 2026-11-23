# RepXSet (Rep Counter)

A self-contained rep counting app with a voiced AI coach. Plan a workout, pick exercises and equipment, set sets/reps/weight, and the coach counts you through it with form tips and rest-break cheers. Saved workouts and an Analytics tab track progress over time.

## What's here

| Path | What it is |
| --- | --- |
| `rep-counter.html` | The app (single file, current version) |
| `voices/` | Pre-recorded coach clips: `smoky` (Shimmer), `steady` (Onyx), `bright` (Alloy) |
| `coach-lines.json` | Script and delivery notes the voices were recorded from |
| `form-tips-v2.*` | Form tips written with visual imagery |
| `voice-samples/` | Original voice audition clips |
| `reviews/` | Design, product and UX reviews |
| `rep-counter.v1-backup.html`, `rep-counter.v2-backup.html` | Older versions |

## Get it on your Mac

```sh
git clone https://github.com/klosetk/repxset-app.git
cd repxset-app
```

## Run locally

The app loads its voice files with `fetch`, so serve the folder rather than double-clicking the file:

```sh
python3 -m http.server 8000
```

Then open <http://localhost:8000/rep-counter.html>.

Running locally, saved workouts and analytics live in your browser's localStorage. In the published Claude artifact they sync to your private account storage instead.

## Notes

- No API keys live in this repo. The coach voices were generated once with OpenAI TTS and are stored as audio in `voices/`; the app makes no API calls.
- Keep `voices/` next to `rep-counter.html`; the app looks for `voices/<id>.json` relative to the page.
