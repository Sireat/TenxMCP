# Codebase Architecture Analysis

## Package Structure

The `ai-content` package is organized into a modular, extensible architecture with clear separation of concerns.

### Main Modules (`src/ai_content/`)

```
src/ai_content/
├── core/           # Core abstractions and protocols
├── config/         # Configuration management (Pydantic)
├── providers/      # Provider implementations (Google, AIMLAPI, Kling)
├── presets/        # Style presets for music and video
├── pipelines/      # Orchestration workflows
├── integrations/   # External service integrations
├── utils/          # Utility functions
└── cli/            # Command-line interface
```

### Core Module (`core/`)

The core module defines the foundational abstractions:

- **`provider.py`**: Protocol definitions for `MusicProvider`, `VideoProvider`, and `ImageProvider`
  - Uses Python's `Protocol` for structural subtyping (duck typing with type safety)
  - Defines required properties: `name`, `supports_vocals`, `supports_realtime`, etc.
  - All providers must implement the `generate()` method returning `GenerationResult`

- **`registry.py`**: Decorator-based provider registration system
  - Providers register themselves using `@ProviderRegistry.register_music/video/image()` decorators
  - Lazy-loaded singleton instances
  - Enables plugin-style extensibility without modifying core code

- **`result.py`**: `GenerationResult` dataclass
  - Standardized return type for all generation operations
  - Contains: `success`, `file_path`, `provider`, `error`, `metadata`

- **`exceptions.py`**: Custom exception hierarchy
  - `ProviderError`, `AuthenticationError`, `GenerationError`, `TimeoutError`

- **`job_tracker.py`**: SQLite-based job tracking system
  - Prevents duplicate API calls via MD5 hash
  - Tracks job lifecycle: `queued` → `processing` → `completed` → `downloaded` (or `failed`)
  - Cost management and status synchronization

### Config Module (`config/`)

- **`settings.py`**: Pydantic-based configuration
  - `GoogleSettings`: GEMINI_API_KEY, model configurations
  - `AIMLAPISettings`: AIMLAPI_KEY, model endpoints
  - `KlingSettings`: KLINGAI_API_KEY, KLINGAI_SECRET_KEY
  - Loads from `.env` file and optional YAML config

- **`loader.py`**: YAML configuration loader
  - Flattens nested YAML structure for Pydantic compatibility

### Providers Module (`providers/`)

Providers are organized by service:

- **`google/`**: Google Gemini API providers
  - `lyria.py`: Real-time music streaming (WebSocket)
  - `veo.py`: Video generation with async polling
  - `imagen.py`: Image generation

- **`aimlapi/`**: AIMLAPI.com proxy service
  - `client.py`: HTTP client for AIMLAPI endpoints
  - `minimax.py`: MiniMax Music 2.0 provider (supports vocals, lyrics, reference audio)

- **`kling/`**: Direct KlingAI API
  - `direct.py`: JWT-authenticated video generation (highest quality, 5-14 min wait)

### Presets Module (`presets/`)

Pre-configured style templates:

- **`music.py`**: 11 music presets (jazz, blues, ethiopian-jazz, cinematic, etc.)
  - Each preset contains: `name`, `prompt`, `bpm`, `mood`, `tags`
  - Registry pattern: `MUSIC_PRESETS` dictionary

- **`video.py`**: 7 video presets (nature, urban, space, abstract, etc.)
  - Each preset contains: `name`, `prompt`, `aspect_ratio`, `duration`, `style_keywords`

### Pipelines Module (`pipelines/`)

Orchestration workflows for complex generation tasks:

- **`base.py`**: Base abstractions
  - `PipelineResult`: Aggregates multiple generation results
  - `PipelineConfig`: Configuration for pipeline execution

- **`music.py`**: Music generation pipeline
  - Handles provider selection, preset application, output management

- **`video.py`**: Video generation pipeline
  - Supports image-to-video workflows

- **`full.py`**: End-to-end music video pipeline
  - Orchestrates: music → image → video → merge
  - Supports parallel generation (music + image simultaneously)
  - Optional upload to YouTube or S3

### Integrations Module (`integrations/`)

External service integrations:

- **`archive.py`**: Internet Archive integration
  - Search for source material (audio, images)
  - Extract metadata and URLs

- **`youtube.py`**: YouTube upload functionality
  - OAuth2 authentication
  - Video upload with metadata

- **`media.py`**: Media processing utilities
  - FFmpeg integration for audio/video merging

### CLI Module (`cli/`)

Typer-based command-line interface:

- **`main.py`**: CLI entry point
  - Commands: `music`, `video`, `list-providers`, `list-presets`
  - Job management: `jobs`, `jobs-stats`, `jobs-sync`
  - Status checking: `music-status`, `video-status`

## Provider Organization

Providers follow a consistent pattern:

1. **Registration**: Decorator-based registration in `__init__.py`
   ```python
   @ProviderRegistry.register_music("lyria")
   class GoogleLyriaProvider:
       ...
   ```

2. **Initialization**: Lazy-loading of API clients
   - Settings loaded from `get_settings()`
   - API clients created on first use

3. **Generation**: Async `generate()` method
   - Returns `GenerationResult`
   - Handles errors gracefully (no exceptions for expected failures)

4. **Output Management**: Automatic file path generation
   - Uses `utils/file_handlers.py` for consistent naming
   - Saves to `exports/` directory by default

## Pipeline Orchestration

The pipeline system enables complex workflows:

1. **Parallel Generation**: Music and image can be generated simultaneously
2. **Dependency Management**: Video generation waits for keyframe image
3. **Error Handling**: Partial failures don't stop entire pipeline
4. **Result Aggregation**: All outputs collected in `PipelineResult`

## Extension Points

The architecture is designed for extensibility:

1. **Adding a Provider**: Create new provider class, register with decorator
2. **Adding a Preset**: Add to `MUSIC_PRESETS` or `VIDEO_PRESETS` dictionary
3. **Adding a Pipeline**: Extend `PipelineConfig` and create new pipeline class

## Key Design Patterns

1. **Registry Pattern**: Provider discovery via decorators
2. **Protocol Pattern**: Structural subtyping for type safety
3. **Singleton Pattern**: Provider instances are lazy-loaded singletons
4. **Strategy Pattern**: Different providers for same content type
5. **Pipeline Pattern**: Complex workflows as composable pipelines

## Data Flow

```
CLI Command
    ↓
Provider Registry (get provider instance)
    ↓
Provider.generate() (async)
    ↓
API Call (httpx/google-genai)
    ↓
Polling/Streaming (for async operations)
    ↓
GenerationResult
    ↓
File Save (exports/)
    ↓
Job Tracker (SQLite)
```

## Configuration Flow

```
.env file
    ↓
Pydantic Settings (loads environment variables)
    ↓
Optional YAML config (merged)
    ↓
Provider Settings (nested settings objects)
    ↓
Provider initialization
```
