# learnin-download

Download video courses from 复旦大学终身教育学院 (learnin.com.cn / media.o-learn.cn).

## Prerequisites

- [N_m3u8DL-RE](https://github.com/nilaoda/N_m3u8DL-RE) (CLI tool)
- `ffmpeg` (for merging ts segments into mp4)

## Usage

### 1. Get the m3u8 URL

Open browser DevTools → Network → filter `m3u8`, find the `.../api/video/play/m3u8/{vid}.m3u8` request and copy the full URL.

Or extract `vid` from the page and construct:

```
https://media.o-learn.cn/api/video/play/m3u8/{vid}.m3u8
```

### 2. Download

```bash
N_m3u8DL-RE "https://media.o-learn.cn/api/video/play/m3u8/{vid}.m3u8" \
  -H "Referer: https://www.learnin.com.cn/" \
  --save-dir /root/downloads \
  --save-name "course-name" \
  --auto-select \
  --del-after-done
```

**Key points:**

- Only `Referer` header is required — no cookie/token needed
- `--auto-select` picks the best quality
- `--del-after-done` removes temp files after merging
- Output is an mp4 file

### 3. Manual AES key (if auto-decrypt fails)

```bash
# Download the key
curl -s -H "Referer: https://www.learnin.com.cn/" \
  "https://media.o-learn.cn/api/video/play/key/{vid}?ts=xxx&sign=xxx" \
  -o /tmp/video.key

# Pass to N_m3u8DL-RE
N_m3u8DL-RE "m3u8_url" \
  -H "Referer: https://www.learnin.com.cn/" \
  --key "KID:$(xxd -p /tmp/video.key | tr -d '\n')" \
  --save-dir /root/downloads \
  --save-name "course-name" \
  --auto-select \
  --del-after-done
```

## Notes

- `media.o-learn.cn` only checks `Referer: https://www.learnin.com.cn/`
- Videos are AES-128 encrypted; N_m3u8DL-RE handles decryption automatically
- m3u8 URLs with `ts`/`sign` params expire — always use a fresh URL
- Final output is mp4 (merged via ffmpeg)
