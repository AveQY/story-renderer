# Story Renderer

[![GitHub stars](https://img.shields.io/github/stars/AveQY/story-renderer?style=social)](https://github.com/AveQY/story-renderer)

将文本故事脚本自动转换为分镜图片或完整视频。

## 功能

- **智能分镜**：自动分析脚本，切分为合理的镜头
- **手动分镜**：支持用户提供的分镜脚本
- **多模型支持**：API 模型（OpenAI / Midjourney / SD）、本地模型（ComfyUI）、或其他 skill
- **灵活输出**：生成图片或视频，支持迭代优化
- **断点续传**：生成中断后自动跳过已完成镜头继续
- **风格一致**：参考图 + 固定 seed 策略，保持角色和场景一致
- **成本估算**：初始化时预估总费用

## 快速开始

### 1. 初始化

```
初始化 story-renderer
```

会引导你配置根路径、生成模型和 API 设置。系统会自动测试模型连接。

### 2. 渲染脚本

```
渲染脚本 scripts/xxx.md
```

支持自动分镜和手动分镜两种模式。可配置输出类型（图片/视频）。

### 3. 常用命令

| 命令                             | 说明            |
| ------------------------------ | ------------- |
| `初始化 story-renderer`           | 初始化配置和模型测试    |
| `渲染脚本 <路径>`                    | 渲染脚本为图片或视频    |
| `渲染脚本 <路径> --type video`       | 渲染为视频（覆盖默认类型） |
| `重新生成 <project_id> <scene_id>` | 重新生成指定镜头      |
| `修改配置`                         | 修改配置和偏好       |
| `story-renderer 状态`            | 查看项目生成进度      |
| `列出项目`                         | 列出所有项目        |

## 配置

配置文件位于 `<根路径>/.story-renderer/config.json`，包含：

- 模型配置（API Key、Base URL、默认参数）
- 分镜偏好（最大镜头数、最小镜头时长）
- 渲染偏好（重试次数、版本保留策略）
- 交互偏好（进度条、确认提示）

## 目录结构

```
<根路径>/
├── .story-renderer/
│   ├── config.json
│   └── templates/          # 提示词模板
└── projects/
    └── <project-id>/
        ├── _meta/
        │   ├── project.json
        │   ├── storyboard.json
        │   └── generation.log
        ├── source/
        │   ├── script.md
        │   └── reference/      # 参考图
        ├── generated/
        │   ├── images/
        │   ├── audio/
        │   └── video/
        └── output/
            ├── final_video.mp4
            └── subtitles.srt
```

## 脚本格式

### 纯文本（自动分镜）

直接写故事文本，系统会自动切分镜头。

### 手动分镜标记

在脚本中使用 `## 镜头 N` 标记：

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

## 宽高比

支持可配置的宽高比：

| 比例   | 适用场景           |
| ---- | -------------- |
| 16:9 | 横屏视频、YouTube   |
| 9:16 | 竖屏短视频、抖音       |
| 1:1  | 方形封面、Instagram |

在配置中设置：`"aspect_ratio": "9:16"`

## 依赖

- Python 3.8+
- ffmpeg（视频生成必需）
- 可选：ComfyUI / Automatic1111（本地模型）

## 集成

- **aweqy-image-generator**：作为生成模型之一
- **browser-automation**：自动上传生成的视频
- **cheat-on-content**：先打分预测脚本，再生成视频

## License

MIT
