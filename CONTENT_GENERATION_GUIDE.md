# Content Generation Guide

This guide provides step-by-step instructions for generating audio and video content using the AI content generation framework.

## Prerequisites

1. **Environment Setup**:
   ```bash
   # Option 1: Using uv (recommended)
   curl -LsSf https://astral.sh/uv/install.sh | sh
   uv sync
   
   # Option 2: Using pip in virtual environment
   python3 -m venv venv
   source venv/bin/activate
   pip install -e .
   ```

2. **API Keys**: Ensure `.env` file exists with:
   ```env
   GEMINI_API_KEY=your_google_api_key_here
   AIMLAPI_KEY=your_aimlapi_key_here
   ```

3. **Verify Installation**:
   ```bash
   uv run ai-content --help
   uv run ai-content list-providers
   uv run ai-content list-presets
   ```

## Part 1: Generate Instrumental Music

### Using Presets

```bash
# Jazz music (30 seconds)
uv run ai-content music --style jazz --provider lyria --duration 30

# Blues music
uv run ai-content music --style blues --provider lyria --duration 30

# Ethiopian Jazz
uv run ai-content music --style ethiopian-jazz --provider lyria --duration 30

# Electronic
uv run ai-content music --style electronic --provider lyria --duration 30
```

### Using Custom Prompts

```bash
uv run ai-content music \
  --prompt "Smooth jazz fusion with walking bass, mellow saxophone, and gentle piano" \
  --provider lyria \
  --bpm 95 \
  --duration 30
```

**Expected Output**:
- File location: `exports/music/jazz_lyria_YYYYMMDD_HHMMSS.wav`
- Format: WAV or MP3
- Generation time: Real-time streaming

## Part 2: Generate Music with Vocals

### Step 1: Prepare Lyrics File

Create or use existing `data/lyrics.txt` with structured format:

```
[Verse 1]
Walking through the city lights
Lost in thought, on a lonely night
The rhythm of the street, a gentle hum
Waiting for a new day to come

[Chorus]
Oh, the city, it never sleeps
Secrets it holds, and promises it keeps
In every shadow, a story untold
A symphony of souls, brave and bold

[Verse 2]
Neon glow, a vibrant hue
Reflecting dreams, both old and new
Footsteps echo, a silent beat
Life's sweet melody, bittersweet

[Chorus]
Oh, the city, it never sleeps
Secrets it holds, and promises it keeps
In every shadow, a story untold
A symphony of souls, brave and bold

[Outro]
City lights fade, as dawn appears
Whispering hopes, calming fears
Another day begins, a fresh start
In the city's beating heart
```

### Step 2: Generate Music with Vocals

```bash
uv run ai-content music \
  --prompt "Contemporary R&B with smooth vocals, emotional delivery" \
  --provider minimax \
  --lyrics data/lyrics.txt \
  --duration 30
```

**Expected Output**:
- File location: `exports/music/custom_minimax_YYYYMMDD_HHMMSS.mp3`
- Format: MP3
- Generation time: 5-15 minutes (async job)
- Job ID will be displayed for tracking

### Step 3: Track Job Status

For long-running MiniMax jobs:

```bash
# Check job status
uv run ai-content music-status <job_id>

# Sync and download completed jobs
uv run ai-content jobs-sync --download
```

## Part 3: Generate Video

### Using Presets

```bash
# Nature video (5 seconds)
uv run ai-content video --style nature --provider veo --duration 5

# Space video
uv run ai-content video --style space --provider veo --duration 5

# Urban video
uv run ai-content video --style urban --provider veo --duration 5
```

### Using Custom Prompts

```bash
uv run ai-content video \
  --prompt "A majestic lion walking through savanna grass at golden hour, cinematic, documentary style" \
  --provider veo \
  --aspect 16:9 \
  --duration 5
```

**Expected Output**:
- File location: `exports/video/nature_veo_YYYYMMDD_HHMMSS.mp4`
- Format: MP4
- Generation time: ~30 seconds
- Aspect ratio: 16:9, 9:16, or 1:1

### Image-to-Video (Advanced)

```bash
# First generate an image, then convert to video
uv run ai-content image \
  --prompt "A serene mountain landscape at sunset" \
  --provider imagen

# Then use the image for video generation
uv run ai-content video \
  --prompt "The mountain landscape comes to life" \
  --provider veo \
  --image path/to/generated/image.jpg
```

## Part 4: Combine Music and Video

### Using FFmpeg

```bash
# Merge audio and video
ffmpeg -i exports/video/nature_veo_*.mp4 \
       -i exports/music/jazz_lyria_*.wav \
       -c:v copy \
       -c:a aac \
       -shortest \
       exports/music_video_combined.mp4
```

### Using Full Pipeline (Python)

```python
from ai_content.pipelines.full import FullContentPipeline

pipeline = FullContentPipeline()
result = await pipeline.generate_music_video(
    music_style="jazz",
    video_style="nature",
    merge_audio_video=True
)
```

## Part 5: Upload to YouTube

### Step 1: Prepare Video

Ensure your video file is ready:
- Format: MP4
- Resolution: 1080p or higher recommended
- Duration: 5-30 seconds (or longer)

### Step 2: Upload to YouTube

1. Go to [YouTube Studio](https://studio.youtube.com/)
2. Click "Create" → "Upload videos"
3. Select your video file
4. Set title: `[TRP1] Your Name - Content Description`
5. Set visibility: Unlisted (or Public)
6. Add description:

```
[TRP1] AI Content Generation Challenge Submission

Generated using the ai-content framework.

Music:
- Provider: Lyria (Google)
- Style: Jazz
- Prompt: Smooth jazz fusion with walking bass
- Duration: 30 seconds

Video:
- Provider: Veo (Google)
- Style: Nature
- Prompt: Majestic lion in savanna at golden hour
- Duration: 5 seconds

Creative Decisions:
- Combined instrumental jazz with nature footage for a calming, cinematic experience
- Used golden hour lighting to match the warm tones of jazz music
- Synchronized video duration with music tempo

Generated on: [Date]
Framework: ai-content (trp1-ai-artist)
```

7. Add tags: `TRP1`, `AI Content`, `Music Generation`, `Video Generation`
8. Click "Publish"

### Step 3: Get YouTube Link

After upload, copy the video URL:
- Format: `https://www.youtube.com/watch?v=VIDEO_ID`
- Or: `https://youtu.be/VIDEO_ID`

## Troubleshooting

### Issue: "uv: command not found"
**Solution**: Install uv or use pip in virtual environment:
```bash
python3 -m venv venv
source venv/bin/activate
pip install -e .
```

### Issue: "API key not found"
**Solution**: Ensure `.env` file exists in project root with correct keys

### Issue: "Job still processing"
**Solution**: MiniMax jobs take 5-15 minutes. Use:
```bash
uv run ai-content music-status <job_id>
```

### Issue: "FFmpeg not found"
**Solution**: Install FFmpeg:
```bash
# Ubuntu/Debian
sudo apt install ffmpeg

# macOS
brew install ffmpeg
```

## Generation Checklist

- [ ] Environment set up (uv or venv)
- [ ] API keys configured in `.env`
- [ ] Installation verified (`ai-content --help`)
- [ ] Generated at least 1 instrumental music file
- [ ] Generated at least 1 music file with vocals (if AIMLAPI available)
- [ ] Generated at least 1 video file
- [ ] Combined music and video (optional)
- [ ] Uploaded to YouTube
- [ ] Added YouTube link to SUBMISSION.md

## File Locations

- **Music**: `exports/music/`
- **Video**: `exports/video/`
- **Images**: `exports/images/`
- **Combined**: `exports/` (root)

## Next Steps

After generating content:
1. Update `SUBMISSION.md` with actual generation log
2. Add YouTube links to submission
3. Document any issues encountered
4. Commit all changes to repository
