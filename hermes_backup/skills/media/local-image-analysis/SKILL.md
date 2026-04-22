---
name: local-image-analysis
description: Analyze local image files when direct vision_analyze fails
---

# Local Image Analysis (Fallback)

Use when `vision_analyze` fails or isn't available for local image files at paths like `/opt/data/cache/images/...`.

## Problem
- `vision_analyze(image_url="/path/to/image.jpg")` may fail with dependency errors (e.g., `anyio` version conflicts)
- `browser_navigate(url="file:///path/to/image.jpg")` renders an empty page — browsers don't reliably display raw image files via file:// protocol

## Solution: Base64 Data URI + Browser Vision

### Steps
1. **Base64 encode the image and create an HTML wrapper:**
```python
import base64

with open("/path/to/image.jpg", "rb") as f:
    img_data = base64.b64encode(f.read()).decode()

html = f"""<!DOCTYPE html>
<html><head><style>body{{margin:0;padding:20px;background:#fff;}}img{{max-width:100%;}}</style></head>
<body><img src="data:image/jpeg;base64,{img_data}" /></body></html>"""

with open("/tmp/image_viewer.html", "w") as f:
    f.write(html)
```

2. **Open in browser:**
```
browser_navigate(url="file:///tmp/image_viewer.html")
```

3. **Analyze with browser_vision:**
```
browser_vision(question="请仔细阅读并完整提取图片中的所有文字内容")
```

### Notes
- For PNG images, change `data:image/jpeg` to `data:image/png`
- Large images (>2MB) produce ~2.8MB HTML files; browser may need a moment to load
- This approach works because the browser renders the base64 image inline, making it visible to browser_vision's screenshot capability

### Alternative: Inline SVG directly (no base64 needed)
For SVG files, embed the SVG content directly in HTML — avoids large base64 overhead:
```python
with open("/path/to/design.svg", "r") as f:
    svg_content = f.read()
html = f"""<!DOCTYPE html>
<html><head><style>body{{margin:0;padding:20px;background:#fff;}}</style></head>
<body>{svg_content}</body></html>"""
```

### Approaches that DON'T work
- `file:///path/to/image.jpg` — browser shows empty page (can't render raw images)
- `<object data="/path/to/file.svg">` — doesn't reliably render in headless browser
- `<img src="/absolute/path.jpg">` — browser blocks local file paths
- `python3 -m http.server` in background — process gets killed/timed out by the system
- `rsvg-convert` / `cairosvg` — usually not installed; `pip` may be unavailable in sandboxed env

### When NOT to use
- If `vision_analyze` works fine, use it directly — simpler and faster
- For remote/URL images, just pass the URL to `vision_analyze`
