---
name: "generate"
description: "Generate images and short animated clips via Kie AI. Use for /generate, generate an image, thumbnail, animate a concept, motion graphics or short video clips."
---

# /generate

Makes images and short videos through the Kie AI API on the cheapest capable model, never spends on video without an explicit go, and files every output flat in one folder.

## Before the first run on a computer
1. This needs a shell with internet access to api.kie.ai. A cloud sandbox with a network allowlist cannot do it. If a call is refused, say so plainly, do not retry with other commands, and ask the user to run /generate from the Claude desktop app linked to their computer or from Claude Code.
2. API key: read KIE_API_KEY from the environment or from the file ~/.config/generate/.env (one line: KIE_API_KEY=...). If it is missing, tell the user to create that file themselves. Never ask for the key in chat, never print it, never put it in code or in a generated file.
3. Output folder: ~/generations (create it if missing). If the user's CLAUDE.md names another folder, use that one. Keep it flat, no subfolders.
4. Helper script: if ~/.config/generate/kie.py does not exist, write it from the Helper script section below. Check the balance with `python3 ~/.config/generate/kie.py --credits` (the number is Kie credits; 1 credit is roughly $0.005 according to Kie's pricing page, verify there).
5. Gallery: if gallery.html is missing in the output folder, offer to build it (see Gallery). Never open it automatically.

## Models
| Task | Model id | Notes |
|---|---|---|
| Draft image (default) | nano-banana-2-lite | Cheapest, everyday images, supports reference image URLs |
| Quality image | nano-banana-2 | Finals only. Verify the id on https://docs.kie.ai/market/google/nanobanana2 before first use |
| Readable text in image | GPT Image 2 | Look up the id on https://docs.kie.ai/market/gpt/gpt-image-2-text-to-image before first use |
| Video (default) | kling-3.0/video | Animated concepts, 3 to 15 seconds |

Every Kie job is async: createTask returns a taskId, then the status endpoint is polled. The helper script does all of that. Extra fields go in via --input as JSON.

### nano-banana-2-lite (image)
Fields: prompt (required); image_urls (optional, up to 10 public URLs); aspect_ratio, one of 1:1, 1:4, 1:8, 2:3, 3:2, 3:4, 4:1, 4:3, 4:5, 5:4, 8:1, 9:16, 16:9, 21:9, auto (default auto).

Example:
`python3 ~/.config/generate/kie.py --model nano-banana-2-lite --name project_description --prompt 'the prompt' --input '{"aspect_ratio":"16:9"}'`

### kling-3.0/video (video)
Fields: prompt; image_urls (optional first frame, optional second URL as last frame; aspect_ratio is then optional); duration as a string from '3' to '15'; aspect_ratio 16:9, 9:16 or 1:1; mode std (720p), pro or 4K; sound true or false (sound costs more in std and pro); multi_shots false.
Defaults for concept animations: mode std, sound false, duration '5', 16:9.
Multi shot: multi_shots true plus multi_prompt, an array of {prompt, duration 1 to 12}, max 500 characters per shot, billed on the sum of durations, first frame image only.

Example:
`python3 ~/.config/generate/kie.py --model kling-3.0/video --name project_description --prompt 'the prompt' --input '{"mode":"std","sound":false,"duration":"5","aspect_ratio":"16:9"}'`

### Workflow for an animated concept
1. Draft a still on nano-banana-2-lite.
2. When the user likes it, animate it: pass the still's Kie URL (printed as urls by the helper, valid 14 days) in image_urls to Kling. This keeps the style consistent and is more controllable than text only.
3. Prompt for video: one clear motion plus at most one camera move, a clean flat or vector motion graphics look, plain background. Do not ask a video model for readable text; add text afterwards.

## Rules
- Images: draft on the cheap model and just run. Report credits used after each run.
- Video: before any run, state model, mode, duration, sound and resolution plus the expected cost, then wait for an explicit go. A quote is not approval, and one approval covers one run. Find the price on https://kie.ai/pricing or the model page; if it cannot be confirmed, say so and show the credit balance instead of guessing. Check --credits before and after every paid video run and report creditsConsumed with the dollar equivalent.
- Rerun on a quality model only when the user picks a favourite draft.
- Run multiple generations one at a time.
- Never describe a real logo or face in words. If the user wants one, they must supply the image as a public URL (Kie has a File Upload API at https://docs.kie.ai/file-upload-api/quickstart, not wired into the helper). If it is missing, stop and ask.
- Kie deletes generated files after 14 days. The helper downloads immediately; confirm the file exists before reporting done.
- Suggest once that the user sets hourly, daily or total usage caps on the key at https://kie.ai/api-key.
- Say which model and settings ran. Never hide a model swap.

## Output
Every file goes flat into the output folder as {project}_{description}_{unix timestamp}.{ext} (pass --name project_description). Next to each file the helper writes a JSON note with the same basename: model, prompt, params, taskId, sourceUrl, creditsConsumed, created. It also rewrites gallery-files.js in the folder (media files, newest first).

## Gallery
Build one self contained gallery.html in the output folder. It loads gallery-files.js with a script tag (window.FILES is an array of file names, newest first) and needs no server and no external libraries, so it works by opening the file. Spec:
- Masonry wall, 4 columns wide and fewer as the window narrows, items distributed over the columns round robin in JS, tiles keep their own shape (no cropping or squashing), rounded cards, even gaps.
- Videos start playing muted on hover and pause when the mouse leaves; images just sit there.
- Click a tile to open it large in the centre of the screen; click outside to close.
- No search, no filters, no tabs, no side panels. Finished, polished look, not a rough draft.
The user refreshes the page to see new files. With the user's OK, add a line to their CLAUDE.md saying that /generate saves into this folder and gallery.html shows it.

## Helper script (~/.config/generate/kie.py)
```python
#!/usr/bin/env python3
import argparse, json, os, sys, time, urllib.request, urllib.error
from datetime import datetime, timezone
from pathlib import Path

API = 'https://api.kie.ai'
UA = {'User-Agent': 'Mozilla/5.0'}
MEDIA = ('.jpg', '.jpeg', '.png', '.webp', '.gif', '.mp4', '.webm', '.mov')

def key():
    k = os.environ.get('KIE_API_KEY')
    f = Path.home() / '.config' / 'generate' / '.env'
    if not k and f.exists():
        for line in f.read_text().splitlines():
            if line.startswith('KIE_API_KEY='):
                k = line.split('=', 1)[1].strip().strip('"').strip("'")
    if not k:
        sys.exit('No KIE_API_KEY. Put KIE_API_KEY=... in ~/.config/generate/.env')
    return k

def call(method, path, body=None):
    headers = dict(UA, Authorization='Bearer ' + key())
    data = None
    if body is not None:
        headers['Content-Type'] = 'application/json'
        data = json.dumps(body).encode()
    req = urllib.request.Request(API + path, data=data, headers=headers, method=method)
    try:
        with urllib.request.urlopen(req, timeout=60) as r:
            out = json.loads(r.read())
    except urllib.error.HTTPError as e:
        sys.exit('HTTP %s: %s' % (e.code, e.read()[:300]))
    if out.get('code') != 200:
        sys.exit('Kie error %s: %s' % (out.get('code'), out.get('msg')))
    return out['data']

def index(folder):
    files = sorted((p for p in folder.iterdir() if p.suffix.lower() in MEDIA),
                   key=lambda p: p.stat().st_mtime, reverse=True)
    (folder / 'gallery-files.js').write_text('window.FILES = ' + json.dumps([p.name for p in files]) + ';')

def main():
    a = argparse.ArgumentParser()
    a.add_argument('--credits', action='store_true')
    a.add_argument('--index', action='store_true')
    a.add_argument('--model')
    a.add_argument('--prompt')
    a.add_argument('--name', default='gen_output')
    a.add_argument('--input', default='{}', help='extra input fields as JSON')
    a.add_argument('--out', default=str(Path.home() / 'generations'))
    a.add_argument('--timeout', type=int, default=1500)
    args = a.parse_args()
    out = Path(args.out).expanduser()
    out.mkdir(parents=True, exist_ok=True)
    if args.credits:
        print(call('GET', '/api/v1/chat/credit'))
        return
    if args.index:
        index(out)
        return
    extra = json.loads(args.input)
    inp = dict(extra, prompt=args.prompt)
    task = call('POST', '/api/v1/jobs/createTask', {'model': args.model, 'input': inp})['taskId']
    print('task', task, flush=True)
    start, wait = time.time(), 5
    while True:
        time.sleep(wait)
        wait = min(wait + 2, 15)
        d = call('GET', '/api/v1/jobs/recordInfo?taskId=' + task)
        if d['state'] == 'success':
            break
        if d['state'] == 'fail':
            sys.exit('Failed: %s %s' % (d.get('failCode'), d.get('failMsg')))
        if time.time() - start > args.timeout:
            sys.exit('Timed out; task %s may still finish, check kie.ai/logs' % task)
    urls = json.loads(d['resultJson'])['resultUrls']
    stamp = int(time.time())
    saved = []
    for i, u in enumerate(urls):
        ext = Path(u.split('?')[0]).suffix or '.bin'
        base = '%s_%s%s' % (args.name, stamp, '' if len(urls) == 1 else '_%d' % (i + 1))
        dest = out / (base + ext)
        req = urllib.request.Request(u, headers=UA)
        with urllib.request.urlopen(req, timeout=300) as r, open(dest, 'wb') as f:
            while True:
                chunk = r.read(1 << 20)
                if not chunk:
                    break
                f.write(chunk)
        meta = {'model': args.model, 'prompt': args.prompt, 'params': extra, 'taskId': task,
                'sourceUrl': u, 'creditsConsumed': d.get('creditsConsumed'),
                'created': datetime.now(timezone.utc).isoformat(timespec='seconds')}
        dest.with_suffix('.json').write_text(json.dumps(meta, indent=2))
        saved.append(str(dest))
    index(out)
    print(json.dumps({'saved': saved, 'creditsConsumed': d.get('creditsConsumed'), 'urls': urls}))

main()
```