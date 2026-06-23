# Story Renderer

[![GitHub stars](https://img.shields.io/github/stars/AveQY/story-renderer?style=social)](https://github.com/AveQY/story-renderer)

Automatically convert text story scripts into storyboard images or complete videos.

## Features

- **Smart Storyboarding**: Automatically analyze scripts and split into logical scenes
- **Manual Storyboarding**: Support for user-provided storyboard scripts
- **Multi-Model Support**: API models (OpenAI / Midjourney / SD), local models (ComfyUI), or other skills
- **Flexible Output**: Generate images or videos with iterative refinement
- **Resume Capability**: Skip completed scenes after interruption
- **Style Consistency**: Reference images + fixed seed strategy for character/scene consistency
- **Cost Estimation**: Estimate total cost during initialization

## Quick Start

### 1. Initialize

```
初始化 story-renderer
```

Guides you through configuring the root path, generation model, and API settings. System automatically tests model connectivity.

### 2. Render Script

```
渲染脚本 scripts/xxx.md
```

Supports both auto-split and manual storyboard modes. Configurable output type (images/video).

### 3. Common Commands

| Command | Description |
|---------|-------------|
| `初始化 story-renderer` | Initialize config and test model |
| `渲染脚本 <path>` | Render script to images or video |
| `渲染脚本 <path> --type video` | Render as video (overrides default) |
| `重新生成 <project_id> <scene_id>` | Regenerate a specific scene |
| `修改配置` | Modify config and preferences |
| `story-renderer 状态` | Check project generation progress |
| `列出项目` | List all projects |

## Configuration

Config file at `<root_path>/.story-renderer/config.json`:

- Model config (API Key, Base URL, default params)
- Storyboard preferences (max scenes, min scene duration)
- Rendering preferences (retry count, version retention)
- Interaction preferences (progress bar, confirmations)

## Directory Structure

```
<root_path>/
├── .story-renderer/
│   ├── config.json
│   └── templates/          # Prompt templates
└── projects/
    └── <project-id>/
        ├── _meta/
        │   ├── project.json
        │   ├── storyboard.json
        │   └── generation.log
        ├── source/
        │   ├── script.md
        │   └── reference/      # Reference images
        ├── generated/
        │   ├── images/
        │   ├── audio/
        │   └── video/
        └── output/
            ├── final_video.mp4
            └── subtitles.srt
```

## Script Format

### Plain Text (Auto-split)

Write story text directly. System will automatically split into scenes.

### Manual Storyboard Markers

Use `## 镜头 N` markers in your script:

```markdown
## 镜头 1
**时长**：5秒
**画面**：觉醒广场，全校学生围在测试石前
**旁白**：全校觉醒日，所有人都在等自己的能力降临。

## 镜头 2
**时长**：8秒
**画面**：主角站在人群最后，低头看着自己的手
**旁白**：只有他，还没有觉醒任何能力。
```

## Aspect Ratio

Configurable aspect ratios:

| Ratio | Use Case |
|-------|----------|
| 16:9  | Landscape video, YouTube |
| 9:16  | Portrait short video, TikTok |
| 1:1   | Square cover, Instagram |

Set in config: `"aspect_ratio": "9:16"`

## Dependencies

- Python 3.8+
- ffmpeg (required for video generation)
- Optional: ComfyUI / Automatic1111 (local models)

## Integration

- **aweqy-image-generator**: Use as a generation model
- **browser-automation**: Auto-upload generated videos
- **cheat-on-content**: Score and predict scripts before rendering

## License

MIT
