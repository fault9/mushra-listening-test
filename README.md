# MUSHRA Listening Test — Setup Guide

## File Structure

```
mushra/
├── index.html          ← The entire test (edit the JSON config inside)
├── service/
│   └── write.php       ← PHP backend: saves CSV + raw JSON
├── results/            ← Created automatically; holds CSV & JSON results
│   ├── uncanny_valley_results.csv
│   └── uncanny_valley_raw.jsonl
└── audio/              ← Put your WAV/MP3/OGG files here
    ├── ref.wav
    ├── c1.wav
    ├── c2.wav
    └── c3.wav
```

---

## 1 — Add Your Audio Files

Drop your audio files into the `audio/` folder (any format the browser supports: WAV, MP3, OGG, FLAC).

Update the paths in the JSON config inside `index.html`:

```json
"reference": { "label": "Reference", "src": "audio/my_ref.wav" },
"stimuli": [
  { "id": "S1", "label": "Sample A", "src": "audio/my_c1.wav", "isHiddenRef": false },
  { "id": "REF","label": "Hidden Reference", "src": "audio/my_ref.wav", "isHiddenRef": true }
]
```

---

## 2 — Add / Remove Test Pages

Open `index.html` and find the `<script id="cfg">` block.
The `pages` array drives everything. Three page types are supported:

### `generic` — text / intro page
```json
{ "type": "generic", "name": "My Page Title", "content": "HTML content here." }
```

### `mushra` — rating page
```json
{
  "type":      "mushra",
  "name":      "My Criterion",
  "id":        "my_criterion",          ← must be unique; used in CSV output
  "content":   "Rate how X each voice sounds.",
  "randomize": true,                    ← shuffles samples per participant
  "reference": { "label": "Reference", "src": "audio/ref.wav" },
  "stimuli": [
    { "id": "S1",  "label": "Sample A", "src": "audio/c1.wav",  "isHiddenRef": false },
    { "id": "S2",  "label": "Sample B", "src": "audio/c2.wav",  "isHiddenRef": false },
    { "id": "S3",  "label": "Sample C", "src": "audio/c3.wav",  "isHiddenRef": false },
    { "id": "REF", "label": "Hidden Reference", "src": "audio/ref.wav", "isHiddenRef": true }
  ]
}
```

> **Add more samples**: just add more objects to the `stimuli` array.  
> **Remove a sample**: delete its object from `stimuli`.  
> One `isHiddenRef: true` entry is required per page for MUSHRA validity.

### `finish` — thank-you / results page
```json
{ "type": "finish", "name": "Thank You", "content": "Done!", "showResults": true }
```

---

## 3 — Results / Saving

### CSV download (always available)
Participants can download their results as a CSV directly from the Finish page. No server needed.

### Server-side saving (PHP)
If you're hosting on a PHP server, set `"remoteService": "service/write.php"` in the config.  
The backend appends to `results/<testId>_results.csv` and `results/<testId>_raw.jsonl`.

**CSV columns:**
| Column | Description |
|--------|-------------|
| session_id | Unique per participant |
| timestamp | ISO 8601 |
| test_id | From config |
| page_id | MUSHRA page identifier |
| page_name | Human-readable name |
| position | A / B / C … (randomised display order) |
| stimulus_id | Stable internal ID (S1, S2, REF…) |
| is_hidden_ref | true / false |
| score | 0–100 |
| plays | How many times this sample was played |

---

## 4 — Hosting

### Any static host (no PHP — CSV download only)
Netlify Drop, GitHub Pages, Cloudflare Pages, Vercel — just drop the folder.

### With PHP server saving
- Apache / Nginx with PHP 7.4+
- Make the `results/` folder writable: `chmod 755 results/`
- Point your domain at the `mushra/` folder

### Local testing
```bash
# Python (no PHP saving)
python3 -m http.server 8080

# PHP built-in server (full saving)
php -S localhost:8080
```

---

## 5 — Customisation Tips

- **Test name / ID**: change `"testname"` and `"testId"` in the config.
- **Back button**: set `"showButtonPreviousPage": false` to hide it.
- **Colour theme**: edit the CSS variables at the top of `<style>`.
- **Result table on finish page**: set `"showResults": false` to hide scores from participants.
