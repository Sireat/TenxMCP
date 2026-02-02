# Provider Capabilities Comparison

## Music Providers

### Google Lyria (`lyria`)

**Provider Class**: `GoogleLyriaProvider`  
**Location**: `src/ai_content/providers/google/lyria.py`

**Capabilities:**
- ✅ **Real-time streaming** via WebSocket
- ✅ **Instrumental music** generation
- ❌ **No vocals** support
- ❌ **No reference audio** support
- ✅ **BPM control** (beats per minute)
- ✅ **Temperature control** (creativity level)
- ✅ **Weighted prompts** (using brackets)

**Best For:**
- Fast instrumental music generation
- Real-time streaming applications
- Background music for videos

**Generation Speed:** Real-time (streams as it generates)

**API Requirements:** `GEMINI_API_KEY`

**Example:**
```python
provider = ProviderRegistry.get_music("lyria")
result = await provider.generate(
    "smooth jazz fusion",
    bpm=95,
    duration_seconds=30
)
```

---

### MiniMax Music 2.0 (`minimax`)

**Provider Class**: `MiniMaxMusicProvider`  
**Location**: `src/ai_content/providers/aimlapi/minimax.py`  
**Via**: AIMLAPI.com proxy service

**Capabilities:**
- ✅ **Vocals support** with lyrics
- ✅ **Reference audio** for style transfer
- ✅ **Lyrics with structure tags** (`[Verse]`, `[Chorus]`, etc.)
- ✅ **Non-English vocals** support
- ❌ **No real-time** streaming (async polling)
- ❌ **No BPM control** (MiniMax determines tempo)

**Best For:**
- Songs with vocals
- Style transfer from reference audio
- High-quality music production

**Generation Speed:** 5-15 minutes (long-running job)

**API Requirements:** `AIMLAPI_KEY`

**Example:**
```python
provider = ProviderRegistry.get_music("minimax")
result = await provider.generate(
    "Lo-fi hip-hop ambient",
    lyrics="[Verse] Walking through the rain...",
    reference_audio_url="https://example.com/reference.mp3"
)
```

---

### Comparison Table: Music Providers

| Feature | Lyria | MiniMax |
|---------|-------|---------|
| **Vocals** | ❌ | ✅ |
| **Real-time** | ✅ | ❌ |
| **Reference Audio** | ❌ | ✅ |
| **Lyrics Support** | ❌ | ✅ |
| **BPM Control** | ✅ | ❌ |
| **Speed** | Real-time | 5-15 min |
| **Best For** | Instrumentals | Vocals + Style |

---

## Video Providers

### Google Veo (`veo`)

**Provider Class**: `GoogleVeoProvider`  
**Location**: `src/ai_content/providers/google/veo.py`

**Capabilities:**
- ✅ **Text-to-video** generation
- ✅ **Image-to-video** (animate first frame)
- ✅ **Multiple aspect ratios** (16:9, 9:16, 1:1, 21:9)
- ✅ **Fast generation** (~30 seconds typical)
- ✅ **Fast model option** (`veo-3.1-fast-generate-preview`)
- ❌ **Max duration**: 8 seconds

**Best For:**
- Quick iterations
- Image-to-video animation
- Short-form content

**Generation Speed:** ~30 seconds

**API Requirements:** `GEMINI_API_KEY`

**Example:**
```python
provider = ProviderRegistry.get_video("veo")
result = await provider.generate(
    "Dragon soaring over mountains",
    aspect_ratio="16:9",
    first_frame_url="https://example.com/frame.jpg"
)
```

---

### KlingAI Direct (`kling`)

**Provider Class**: `KlingDirectProvider`  
**Location**: `src/ai_content/providers/kling/direct.py`

**Capabilities:**
- ✅ **Highest quality** video generation
- ✅ **Image-to-video** support
- ✅ **v2.1-master model**
- ✅ **JWT authentication**
- ❌ **Slow generation** (5-14 minutes)
- ❌ **Max duration**: 10 seconds

**Best For:**
- Final renders
- High-quality output
- When time is not a constraint

**Generation Speed:** 5-14 minutes

**API Requirements:** `KLINGAI_API_KEY`, `KLINGAI_SECRET_KEY`

**Example:**
```python
provider = ProviderRegistry.get_video("kling")
result = await provider.generate(
    "Dragon soaring over mountains",
    aspect_ratio="16:9"
)
```

---

### Comparison Table: Video Providers

| Feature | Veo | Kling |
|---------|-----|-------|
| **Speed** | ~30s | 5-14 min |
| **Quality** | Good | Highest |
| **Image-to-Video** | ✅ | ✅ |
| **Max Duration** | 8s | 10s |
| **Best For** | Quick iterations | Final renders |

---

## Image Providers

### Google Imagen (`imagen`)

**Provider Class**: `GoogleImagenProvider`  
**Location**: `src/ai_content/providers/google/imagen.py`

**Capabilities:**
- ✅ **High-quality** photorealistic images
- ✅ **Multiple models**: `gemini-3-pro-image-preview` (4K, complex), `gemini-2.5-flash-image` (fast, 1024px)
- ✅ **Multiple aspect ratios**
- ✅ **Batch generation** (multiple images)

**Best For:**
- Keyframe generation for video
- High-quality still images
- Photorealistic content

**API Requirements:** `GEMINI_API_KEY`

---

## Provider Selection Guide

### For Music:

**Choose Lyria if:**
- You need instrumental music
- Speed is important (real-time)
- You want BPM control

**Choose MiniMax if:**
- You need vocals/lyrics
- You have reference audio for style transfer
- Quality is more important than speed

### For Video:

**Choose Veo if:**
- You need fast iterations (~30s)
- You're experimenting with prompts
- You need image-to-video quickly

**Choose Kling if:**
- You need the highest quality
- You can wait 5-14 minutes
- This is your final render

---

## Provider Registration

All providers are registered using decorators:

```python
@ProviderRegistry.register_music("lyria")
class GoogleLyriaProvider:
    name = "lyria"
    supports_vocals = False
    supports_realtime = True
    supports_reference_audio = False
    # ...
```

Providers are discovered automatically when imported in `providers/__init__.py`.

---

## Job Tracking

Long-running jobs (like MiniMax) are tracked in SQLite:

- **Duplicate Detection**: MD5 hash prevents redundant API calls
- **Status Lifecycle**: `queued` → `processing` → `completed` → `downloaded`
- **Cost Management**: Track API usage
- **Sync**: `jobs-sync` command checks API status for pending jobs

---

## Error Handling

All providers return `GenerationResult` with:
- `success: bool`
- `file_path: Path | None`
- `error: str | None`
- `provider: str`
- `metadata: dict`

No exceptions are raised for expected failures (API errors, timeouts). Check `result.success` and `result.error` instead.
