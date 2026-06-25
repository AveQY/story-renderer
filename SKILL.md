---
name: story-renderer
description: 将故事脚本转换为分镜图片或视频。支持自动分镜和手动分镜，用户可配置生成模型（API/本地/skill），支持参考图（img2img）、角色一致性、断点续传、成本估算、后置验证和可配置宽高比。
version: 1.0.0
argument-hint: <command> [options]
allowed-tools: Bash, Read, Write, Edit, Glob, Grep, Agent, Skill
---

# Story Renderer

将文本故事脚本自动转换为分镜图片或完整视频。

**输出风格：简洁直接，不要过度解释。**

---

## 功能概览

- **智能分镜**：自动分析脚本，切分为合理的镜头
- **手动分镜**：支持用户提供的分镜脚本（如 storyboard.md）
- **多模型支持**：用户配置自己的图片/视频生成模型
- **灵活输出**：生成图片或视频，支持迭代优化
- **项目管理**：完整的项目结构，便于后续扩展

---

## 命令列表

| 命令 | 说明 | 示例 |
|------|------|------|
| `init` | 初始化 story-renderer，配置根路径和生成模型 | `初始化 story-renderer` |
| `render` | 渲染脚本为图片或视频 | `渲染脚本 scripts/xxx.md` |
| `status` | 查看项目状态和生成进度 | `story-renderer 状态` |
| `config` | 查看或修改配置 | `查看配置` / `修改配置` |
| `list` | 列出所有项目 | `列出项目` |
| `regenerate` | 重新生成某一镜 | `重新生成 project-xxx scene_05` |
| `export` | 导出分镜为图片集/ZIP | `导出 3f546fcfe019 --format zip` |
| `delete` | 删除项目 | `删除项目 3f546fcfe019` |
| `batch` | 批量操作（重生成/改风格） | `批量重生成 3f546fcfe019 --scenes 1-5` |

**命令行参数（覆盖用户偏好）：**

```bash
渲染脚本 <path> [options]

Options:
  --type <images|video>        覆盖默认生成类型
  --no-confirm                 跳过所有确认
  --verbose                    显示详细日志
  --retry <n>                  自定义重试次数
  --no-progress                不显示进度条
  --auto-select                自动选择第一个版本
```

**示例：**
```
渲染脚本 scripts/my_story.md --type video --no-confirm --verbose
```

---

## 目录结构

```
<用户指定根路径>/
├── .story-renderer/
│   ├── config.json              # 全局配置
│   ├── templates/               # 提示词模板库
│   │   ├── character_preset.json
│   │   ├── scene_style.json
│   │   └── ...
│   └── plugins/                 # 扩展插件（后续）
│
└── projects/
    └── <project-id>/
        ├── _meta/
        │   ├── project.json     # 项目配置
        │   ├── storyboard.json  # 分镜数据
        │   └── generation.log   # 生成日志
        │
        ├── source/              # 源文件
        │   ├── script.md
        │   ├── storyboard.md    # 分镜文件（可选）
        │   └── reference/       # 参考图（用于风格参考或 img2img）
        │       ├── character/   # 角色参考图
        │       ├── scene/       # 场景参考图
        │       └── style/       # 风格参考图
        │
        ├── generated/           # AI 生成内容
        │   ├── images/
        │   │   └── scene_01/
        │   │       ├── v1.png
        │   │       ├── v2.png
        │   │       └── selected.png
        │   ├── audio/
        │   └── video/
        │
        └── output/              # 最终输出
            ├── final_video.mp4
            ├── subtitles.srt
            └── thumbnail.png
```

## 脚本格式

支持两种脚本格式，无需额外工具即可直接编写。

### 格式一：纯文本（自动分镜）

直接写故事文本，系统根据段落、标点、语义自动切分为镜头：

```markdown
全校觉醒日，所有人都在等自己的能力降临。

觉醒广场上，巨大的测试石散发着幽蓝的光芒。

主角站在人群最后，低头看着自己的手。

只有他，还没有觉醒任何能力。

突然，测试石爆发出前所未有的金色光芒。
```

### 格式二：手动分镜标记

使用 `## 镜头 N` 标题标记每个镜头，更精确控制：

```markdown
## 镜头 1
**时长**：5秒
**画面**：觉醒广场全景，全校学生围在巨大的测试石前，石头发散幽蓝光芒
**旁白**：全校觉醒日，所有人都在等自己的能力降临。
**镜头**：缓慢推近到主角

## 镜头 2
**时长**：8秒
**画面**：主角站在人群最后，低头看着自己的手，周围同学的彩色光芒映在他脸上
**旁白**：只有他，还没有觉醒任何能力。
**镜头**：侧面跟拍

## 镜头 3
**时长**：6秒
**画面**：突然，测试石爆发出前所未有的金色光芒，所有人转头看向主角
**旁白**：直到那一天，命运终于对他开口。
**镜头**：环绕主角旋转
```

**标记说明：**
- `**时长**`：镜头时长（秒），可选，默认 5 秒
- `**画面**`：画面描述（用于生成提示词）
- `**旁白**`：旁白文本（用于 TTS 配音）
- `**镜头**`：运镜描述（推近、环绕、平移等）
- `**参考图**`：指定参考图路径（用于 img2img 或风格参考）

### 格式三：Storyboard 文件

支持独立的 `.storyboard.md` 文件，包含完整分镜表：

```markdown
# 分镜脚本：xxx

| 镜号 | 时长 | 画面描述 | 旁白 | 运镜 | 参考图 |
|------|------|---------|------|------|--------|
| 1 | 5s | 觉醒广场全景... | 全校觉醒日... | 推近 | — |
| 2 | 8s | 主角站在人群后... | 只有他... | 侧拍 | reference/pose.jpg |
```

系统自动按优先级检测：手动分镜标记 > Storyboard 文件 > 纯文本自动分镜。

---

## 使用流程

### 1. 初始化

用户第一次使用时必须运行初始化：

```
初始化 story-renderer
```

会询问：
1. **根路径**：所有项目和配置保存的位置
2. **稿子源**：
   - [1] 本地文件（从用户电脑选择 markdown 脚本文件）
   - [2] 在线接口（从远程 API 获取脚本内容）
3. **生成模型类型**：
   - API 模型（OpenAI DALL-E / Midjourney / Stable Diffusion API）
   - 本地模型（ComfyUI / Automatic1111）
   - 使用现有 skill（如 aweqy-image-generator）
4. **在线接口配置**（如果稿子源选择在线接口）：
   - 接口地址（如 `https://example.com/api/script`）
   - API Key
   - 请求方法（GET/POST）
   - 脚本内容在响应中的路径（如 `data.content`）
   - 测试接口连通性
5. **API 配置**（如果生成模型选择 API）：
   - API Key
   - Base URL
   - 默认参数
6. **模型测试**（关键步骤）：
   - 使用用户提供的配置发起测试请求
   - 生成一张简单的测试图片（如 "a red apple on a white table"）
   - 验证 API 响应状态和返回内容
   - **只有测试通过的配置才会保存**

配置保存在 `<根路径>/.story-renderer/config.json`

### 2. 渲染脚本

```
渲染脚本 <脚本路径>
```

**交互流程：**

#### Step 1: 读取脚本
- **如果稿子源为在线接口**：
  - 读取 `config.json` 中的 `script_source` 配置
  - 构造 HTTP 请求（GET/POST + api_key + params）
  - 根据 `response_path` 从响应 JSON 中提取脚本内容
  - 将内容写入临时文件，继续后续流程
  - 详细模板参见 `references/script-source-template.md`
- 自动检测脚本类型：
  - 纯文本脚本（自动分镜）
  - 带分镜标记的脚本（如 `## 镜头 1`）
  - Storyboard 格式（如你的 `ai_future_artist.storyboard.md`）

#### Step 2: 分镜设计
- **自动模式**：AI 分析脚本，切分为 N 个镜头
- **手动模式**：读取脚本中的分镜标记或表格

**读取用户偏好：**
```python
preferences = config["user_preferences"]

# 如果脚本既无分镜标记，又无 storyboard 文件，根据用户偏好决定
if preferences["storyboard"]["prefer_auto_split"]:
    mode = "auto"
else:
    # 询问用户选择
    mode = ask_user()
```

输出 `storyboard.json`：
```json
{
  "project_id": "3f546fcfe019",
  "script_path": "scripts/2026-06-14_xxx.md",
  "total_scenes": 17,
  "scenes": [
    {
      "scene_id": "scene_01",
      "duration": 8,
      "narration": "全校觉醒日，所有人都在等自己的能力降临。",
      "visual_description": "觉醒广场，全校学生围在巨大的测试石前...",
      "prompt": "A huge futuristic school awakening plaza...",
      "camera": "慢慢推近主角"
    },
    ...
  ]
}
```

**偏好应用：**
- 如果 `max_scenes_limit` 设置为 20，但自动分镜生成了 25 个镜头 → 警告用户并询问是否继续
- 如果 `min_scene_duration` 设置为 5 秒，但某些镜头只有 3 秒 → 标记并提示

#### Step 3: 生成类型选择

**读取用户偏好：**
```python
preferences = config["user_preferences"]

if preferences["workflow"]["default_generation_type"] == "ask":
    # 询问用户
    generation_type = ask_user("选择生成类型：1. 仅图片 / 2. 图片+视频")
elif preferences["workflow"]["default_generation_type"] == "images":
    generation_type = "images"
    print("使用默认设置：仅生成图片")
elif preferences["workflow"]["default_generation_type"] == "video":
    generation_type = "video"
    print("使用默认设置：生成图片+视频")
```

**命令行覆盖：**
```
渲染脚本 script.md --type video
```
覆盖用户偏好，直接生成视频。

#### Step 4: 开始生成

**应用用户偏好：**

```python
preferences = config["user_preferences"]

# 生成前确认
if preferences["interaction"]["confirm_before_generation"]:
    print(f"即将生成 {total_scenes} 个镜头，预计耗时 {estimated_time} 分钟")
    confirm = ask_user("是否继续？[Y/n]")
    if not confirm:
        return

# 显示进度条
show_progress = preferences["interaction"]["show_progress_bar"]

# 批量生成
for scene in scenes:
    try:
        generate_image(scene, show_progress=show_progress)
    except APIError as e:
        if preferences["rendering"]["retry_on_failure"]:
            # 自动重试
            max_retries = preferences["rendering"]["max_retries"]
            for i in range(max_retries):
                try:
                    generate_image(scene, show_progress=show_progress)
                    break
                except APIError:
                    if i == max_retries - 1:
                        log_error(f"Scene {scene.id} failed after {max_retries} retries")
        else:
            # 不重试，询问用户
            action = ask_user(f"Scene {scene.id} 生成失败，是否重试？")
```

- **图片模式**：
  - 批量生成所有分镜图
  - 根据 `save_all_versions` 决定是否保存所有版本
  - 每镜保存到 `generated/images/scene_XX/v1.png`
  - 如果 `show_progress_bar` 为 true，显示进度条
  
- **视频模式**：
  - 先生成所有分镜图
  - 生成配音（TTS）
  - 自动剪辑（ffmpeg）
  - 输出到 `output/final_video.mp4`

#### Step 5: Review 和迭代
生成完成后，用户可以：
- 查看所有分镜图
- 重新生成不满意的镜头
- 调整提示词后重试

**应用用户偏好：**
```python
preferences = config["user_preferences"]

# 重新生成时
if preferences["workflow"]["auto_select_first_version"]:
    # 自动选择 v1 作为 selected.png
    select_version(scene_id, "v1")
else:
    # 询问用户选择哪个版本
    selected = ask_user(f"Scene {scene_id} 有 3 个版本，选择哪个？")
    select_version(scene_id, selected)

# 是否保存所有版本
if not preferences["rendering"]["save_all_versions"]:
    # 只保留 selected.png，删除 v1/v2/v3
    cleanup_unused_versions(scene_id)
```

---

## 配置文件格式

### config.json

```json
{
  "version": "2.0.0",
  "root_path": "C:/Users/xxx/story-renderer-projects",
  "script_source": {
    "mode": "local",
    "api_url": "",
    "api_key": "",
    "method": "GET",
    "params": {},
    "headers": {},
    "response_path": "data.content",
    "cache_enabled": true,
    "cache_ttl": 3600
  },
  "model_config": {
    "type": "api|local|skill",
    "provider": "openai|midjourney|sd|comfyui|aweqy",
    "api_key": "xxx",
    "base_url": "https://api.openai.com/v1",
    "default_params": {
      "model": "dall-e-3",
      "size": "1024x1792",
      "quality": "hd"
    },
    "test_passed": true,
    "test_skipped": false,
    "tested_at": "2026-06-21T17:30:06+08:00"
  },
  "storyboard_config": {
    "auto_split": true,
    "max_scenes": 30,
    "min_scene_duration": 3
  },
  "aspect_ratio": {
    "ratio": "9:16",
    "options": ["16:9", "9:16", "1:1"],
    "width": 1024,
    "height": 1792
  },
  "consistency": {
    "enable_fixed_seed": true,
    "base_seed": 42,
    "character_fingerprint": "main_character: teen_male, black_hair, school_uniform, golden_aura",
    "negative_prompt": "blurry, low quality, distorted, extra limbs, watermark, text, ugly",
    "style_prompt_prefix": "consistent art style, same character throughout, professional illustration"
  },
  "video_config": {
    "fps": 30,
    "resolution": "1080x1920",
    "tts_provider": "openai|elevenlabs|local"
  },
  "user_preferences": {
    "workflow": {
      "auto_confirm_storyboard": false,
      "default_generation_type": "ask",
      "skip_test_prompt": false,
      "auto_select_first_version": false
    },
    "storyboard": {
      "prefer_auto_split": true,
      "max_scenes_limit": 20,
      "min_scene_duration": 5
    },
    "rendering": {
      "retry_on_failure": true,
      "max_retries": 3,
      "save_all_versions": true,
      "prompt_language": "zh-CN"
    },
    "interaction": {
      "verbose_mode": false,
      "confirm_before_generation": true,
      "show_progress_bar": true
    }
  }
}
```

**字段说明：**
- `test_passed`: 模型是否通过测试（必须为 true 才能正常使用）
- `test_skipped`: 用户是否跳过了测试（true 时每次渲染会警告）
- `tested_at`: 测试通过的时间戳

**script_source（稿子源）：**
- `mode`: 稿子源模式（`"local"` / `"api"`）
- `api_url`: 在线接口地址（mode 为 api 时必填）
- `api_key`: 接口 API 密钥（mode 为 api 时必填）
- `method`: HTTP 方法，默认 `"GET"`
- `params`: URL 查询参数对象
- `headers`: 自定义请求头对象
- `response_path`: 响应 JSON 中提取脚本内容的路径，支持点号分隔，默认 `"data.content"`
- `cache_enabled`: 是否缓存接口结果，默认 `true`
- `cache_ttl`: 缓存有效期（秒），默认 `3600`

**aspect_ratio（宽高比）：**
- `ratio`: 宽高比（"16:9" / "9:16" / "1:1"）
- `options`: 可选宽高比列表
- `width` / `height`: 生成图片的像素尺寸

**consistency（一致性策略）：**
- `enable_fixed_seed`: 是否使用固定 seed 保证画面一致性
- `base_seed`: 基础 seed 值，每个镜头的 seed = base_seed + scene_index
- `character_fingerprint`: 角色特征标签，附加到每个镜头的提示词中
- `negative_prompt`: 负面提示词，所有镜头通用
- `style_prompt_prefix`: 风格前缀，附加到每个镜头的提示词中

**user_preferences 说明：**

**workflow（工作流偏好）：**
- `auto_confirm_storyboard`: 分镜设计完成后是否自动开始生成（false = 需要人工 review）
- `default_generation_type`: 默认生成类型（"images" / "video" / "ask" = 每次询问）
- `skip_test_prompt`: 是否跳过测试提示词生成环节
- `auto_select_first_version`: 重新生成时是否自动选择第一个版本作为 selected.png

**storyboard（分镜偏好）：**
- `prefer_auto_split`: 有歧义时优先自动分镜还是手动分镜
- `max_scenes_limit`: 个人偏好的最大镜头数（超过会警告）
- `min_scene_duration`: 个人偏好的最小镜头时长（秒）

**rendering（渲染偏好）：**
- `retry_on_failure`: API 失败时是否自动重试
- `max_retries`: 最大重试次数
- `save_all_versions`: 是否保存所有生成版本（v1/v2/v3），false 只保留 selected.png
- `prompt_language`: 提示词生成语言（"zh-CN" / "en-US"）

**interaction（交互偏好）：**
- `verbose_mode`: 是否显示详细日志（调试用）
- `confirm_before_generation`: 批量生成前是否最后确认一次
- `show_progress_bar`: 是否显示进度条

### project.json

```json
{
  "project_id": "3f546fcfe019",
  "name": "全校最废的能力",
  "created_at": "2026-06-21T17:00:00+08:00",
  "script_path": "source/script.md",
  "storyboard_mode": "manual|auto",
  "generation_type": "images|video",
  "status": "pending|generating|completed|failed",
  "progress": {
    "total_scenes": 17,
    "completed_scenes": 5,
    "failed_scenes": []
  }
}
```

---

## 实现细节

### Phase 1: Init（初始化）

1. 检查是否已初始化（`~/.story-renderer/config.json` 是否存在）
2. 如果未初始化，询问配置：
   - **根路径**：询问用户项目保存位置
   - **模型类型**：API / 本地 / Skill
   - **API 配置**（根据模型类型）
3. **模型测试与验证**（关键步骤）：
   
   #### 测试流程
   
   a. **构造测试请求**
   - 生成一个简单的测试提示词（如 "a small red apple on a white table, simple illustration"）
   - 使用用户提供的配置发起 API 调用
   
   b. **针对不同模型类型的测试**
   
   **OpenAI DALL-E:**
   ```bash
   curl -X POST https://api.openai.com/v1/images/generations \
     -H "Authorization: Bearer $API_KEY" \
     -H "Content-Type: application/json" \
     -d '{
       "model": "dall-e-3",
       "prompt": "a small red apple on a white table",
       "size": "1024x1024",
       "n": 1
     }'
   ```
   验证：
   - HTTP 状态码 200
   - 响应包含 `data[0].url` 或 `data[0].b64_json`
   - 尝试下载图片并验证文件大小 > 0
   
   **Midjourney API:**
   ```bash
   # 根据具体 API 提供商（如 GoAPI / Midjourney API）调整
   curl -X POST <BASE_URL>/imagine \
     -H "Authorization: Bearer $API_KEY" \
     -d '{"prompt": "test prompt"}'
   ```
   验证：
   - 任务创建成功
   - 返回 task_id
   
   **Stable Diffusion API:**
   ```bash
   curl -X POST https://api.stability.ai/v1/generation/stable-diffusion-xl-1024-v1-0/text-to-image \
     -H "Authorization: Bearer $API_KEY" \
     -d '{"text_prompts": [{"text": "test"}]}'
   ```
   
   **ComfyUI (本地):**
   ```bash
   curl http://localhost:8188/system_stats
   ```
   验证：
   - 服务可访问
   - 返回系统状态
   
   **aweqy-image-generator (Skill):**
   ```
   调用 /aweqy-image-generator "test prompt"
   ```
   验证：
   - Skill 可用
   - 返回图片路径
   
   c. **处理测试结果**
   
   **成功（HTTP 200 + 有效响应）：**
   ```
   ✅ 模型测试通过！
   测试图片已生成：<临时路径>
   配置已保存到 config.json
   ```
   
   **失败情况与提示：**
   
   | 错误类型 | 可能原因 | 用户提示 |
   |---------|---------|---------|
   | 401 Unauthorized | API Key 错误 | ❌ API Key 无效，请检查是否正确复制 |
   | 403 Forbidden | 权限不足 | ❌ API Key 权限不足，请确认账户已开通图片生成权限 |
   | 404 Not Found | Base URL 错误 | ❌ API 地址不存在，请检查 Base URL 是否正确 |
   | 429 Too Many Requests | 超出配额 | ❌ API 调用频率超限，请稍后重试或升级账户 |
   | 500 Server Error | 服务端问题 | ❌ API 服务异常，请稍后重试 |
   | Timeout | 网络/服务超时 | ❌ 请求超时，请检查网络连接或 Base URL |
   | Connection Refused | 本地服务未启动 | ❌ 无法连接到本地服务（如 ComfyUI），请确认服务已启动 |
   | Invalid Response | 响应格式错误 | ❌ API 返回格式异常，可能选错了模型类型 |
   
   **失败后的操作：**
   1. 显示详细错误信息（HTTP 状态码、错误消息）
   2. 询问用户：
      ```
      配置测试失败，是否重新配置？
      1. 重新输入配置
      2. 查看配置详情（排查问题）
      3. 跳过测试（不推荐）
      4. 取消初始化
      ```
   3. **只有测试通过或用户明确选择"跳过测试"才保存配置**
   4. 如果跳过测试，在 config.json 中标记 `"test_skipped": true`，后续每次使用时警告
   
   d. **测试日志记录**
   
   所有测试记录保存到 `<根路径>/.story-renderer/test.log`：
   ```
   [2026-06-21 17:30:00] Testing model: openai-dalle-3
   [2026-06-21 17:30:01] Request: POST https://api.openai.com/v1/images/generations
   [2026-06-21 17:30:05] Response: 200 OK
   [2026-06-21 17:30:05] Image URL: https://...
   [2026-06-21 17:30:06] ✅ Test passed
   ```

4. **用户偏好设置**（可选，可跳过使用默认值）
   
   模型测试通过后，询问用户个人偏好：
   
   ```
   📋 个人偏好设置（可跳过，后续可通过 `修改配置` 调整）
   
   工作流偏好：
   1. 每次渲染前是否需要确认分镜设计？
      [Y] 是（默认，可 review 后再生成） / [N] 否（自动开始）
      > 
   
   2. 默认生成类型？
      [1] 仅图片（默认）
      [2] 图片+视频
      [3] 每次询问
      > 
   
   3. API 失败时是否自动重试？
      [Y] 是（默认，最多 3 次） / [N] 否（手动处理）
      > 
   
   交互偏好：
   4. 是否显示详细日志（调试模式）？
      [Y] 是 / [N] 否（默认）
      > 
   
   5. 批量生成前是否最后确认一次？
      [Y] 是（默认） / [N] 否（直接开始）
      > 
   
   ✅ 偏好设置已保存！
   ```
   
   **偏好的使用逻辑：**
   
   - 如果用户设置了 `default_generation_type: "images"`，后续渲染时**不再询问**类型，直接生成图片
   - 如果用户设置了 `auto_confirm_storyboard: true`，分镜设计完成后**直接开始生成**，不等待 review
   - 如果用户设置了 `retry_on_failure: true`，API 失败时**自动重试 3 次**，不中断流程
   
   **命令行覆盖：**
   
   即使用户有默认偏好，也可以在命令中临时覆盖：
   ```
   渲染脚本 script.md --type video --no-confirm --verbose
   ```
   
   - `--type video`: 覆盖默认生成类型
   - `--no-confirm`: 跳过所有确认
   - `--verbose`: 显示详细日志
   
5. **依赖检查**
   
   检查系统依赖是否可用：

   | 依赖 | 检查方式 | 失败处理 |
   |------|---------|---------|
   | ffmpeg | `ffmpeg -version` | 提示安装，视频模式跳过并警告 |
   | Python | `python --version` | 初始化失败 |
   | ComfyUI（本地模式） | `curl http://localhost:8188/system_stats` | 提示启动服务 |
   
   ```
   ✅ ffmpeg 已安装 (version 6.1)
   ✅ Python 3.11.5
   ⚠️ ComfyUI 未检测到（本地模式需要手动启动）
   ```

6. **成本估算**
   
   根据场景数和模型类型估算总费用：

   | 模型 | 每张图片成本（估算） |
   |------|-------------------|
   | DALL-E 3 HD | $0.08 |
   | DALL-E 3 Standard | $0.04 |
   | Midjourney | $0.05 |
   | Stable Diffusion API | $0.02 |
   | 本地模型（ComfyUI） | $0.00 |
   
   ```
   💰 成本估算
   场景数：17
   模型：DALL-E 3 HD ($0.08/张)
   预计图片费用：$1.36
   预计视频费用（TTS + 渲染）：约 $0.50
   ════════════════════
   总计：约 $1.86
   
   💡 提示：视频模式下每张图片会生成 1-2 个版本用于选择
   ```

   如果 `save_all_versions: true`，成本乘以 1.5-2 倍并提示。

7. 创建目录结构
8. 写入 `config.json`（仅当测试通过）

### Phase 2: Render（渲染）

**输入：** 脚本路径

**流程：**

1. **读取配置**
   - 从 `config.json` 读取模型配置
   - 从 `config.json` 读取宽高比、一致性策略配置

2. **依赖检查**
   - 渲染前检查依赖：
     - 视频模式：检查 ffmpeg 是否可用
     - TTS 模式：检查 TTS provider 是否可连接
   - 如果依赖缺失，提示用户安装或降级为图片模式

3. **读取脚本**
   - 检测脚本格式（手动标记 > Storyboard > 纯文本）
   - 提取内容
   - 加载 `source/reference/` 目录下的参考图（角色、场景、风格）

4. **分镜设计**
   - 自动模式：AI 分析脚本，切分镜头
   - 手动模式：解析现有分镜标记或 Storyboard 文件
   - 生成 `storyboard.json`

5. **断点续传检查**
   - 检查 `generated/images/` 目录
   - 统计已完成的镜头数
   - 如果有部分镜头已生成：
     ```
     ⏸️  检测到已有 5/17 个镜头已完成
     [R] 重新生成全部 / [C] 继续未完成的 / [S] 只重新生成指定镜头
     > C
     ```
   - 继续模式：跳过已完成的镜头，从断点继续
   - 应用 `save_all_versions` 策略（保留所有版本或仅保留 selected）

6. **成本确认**（如果偏好设置开启）
   - 显示估算成本
   - 确认是否继续

7. **生成提示词**
   - 为每一镜生成 AI 图片提示词
   - 应用一致性策略、参考图、宽高比

8. **提示词预览与编辑**（`--no-confirm` 跳过）
   - 展示每个镜头的完整提示词供用户审核：
     ```
     📝 提示词预览（17 个镜头）

     ── 镜头 01 (seed: 42) ──
     正向：consistent art style, same character throughout, professional illustration, teen male black hair school uniform golden aura, 觉醒广场全景，全校学生围在巨大的测试石前，石头发散幽蓝光芒
     负向：blurry, low quality, distorted, extra limbs, watermark, text, ugly
     参考图：reference/character/protagonist_front.jpg
     尺寸：1024x1792

     ── 镜头 02 (seed: 43) ──
     正向：consistent art style, same character throughout, professional illustration, teen male black hair school uniform golden aura, 主角站在人群最后，低头看着自己的手
     负向：blurry, low quality, distorted, extra limbs, watermark, text, ugly
     参考图：—
     尺寸：1024x1792

     [E] 编辑提示词 / [A] 全部确认 / [S] 跳过特定镜头 / [Q] 退出
     > E
     输入镜头编号（如 02）：02
     输入新提示词：> 主角站在人群最后，低头看着自己的手，周围的彩色光芒映在他脸上
     ✅ 镜头 02 提示词已更新
     ```
   - 允许的操作：
     - `[E]` 编辑指定镜头的正向/负向提示词
     - `[A]` 确认全部提示词，开始生成
     - `[S]` 跳过某些镜头（标记为 skipped）
     - `[Q]` 退出渲染流程
   - 如果 `interaction.confirm_before_generation` 为 false，跳过此步直接开始生成

8. **批量生成**
   - 调用配置的模型 API
   - 保存到 `generated/images/scene_XX/v1.png`
   - 记录日志
   - 应用重试策略
   - 每个镜头生成后检查是否成功
   - **取消生成**：生成过程中支持中断
     ```
     [Ctrl+C] 或输入 "cancel"
     ⏸️  生成已中断（已完成 8/17 镜头）
     [C] 继续 / [R] 放弃进度重新生成 / [Q] 退出
     > C
     ```
   - 中断后记录进度到 `generation.log`，下次渲染自动断点续传

9. **后置验证**
   - 对生成的图片进行验证：
     ```
     🔍 后置验证...
     scene_01: ✅ 有效 (1024x1792, 1.2MB)
     scene_02: ⚠️ 尺寸异常 (800x600，期望 1024x1792)
     scene_03: ✅ 有效 (1024x1792, 1.5MB)
     ```
   - 验证项：
     - 文件大小 > 0（非空文件）
     - 图片尺寸与 `aspect_ratio` 配置匹配（容差 10%）
     - 图片格式正确（PNG/JPG）
   - 验证失败的处理：
     - 自动重新生成（最多 2 次）
     - 仍失败则标记为 failed，记录到 `failed_scenes`
     - 提示用户手动处理

10. **视频剪辑**（如果选择视频模式）
    - 生成配音（TTS）
    - 用 ffmpeg 剪辑
    - 添加字幕
    - 应用宽高比设置输出尺寸
    - 输出最终视频到 `output/final_video.mp4`

11. **完成报告**
    ```
    ✅ 渲染完成！
    总镜头：17
    成功：16
    失败：1 (scene_05 — 可手动重新生成)
    总耗时：约 8 分钟
    实际费用：约 $1.82
    输出：output/final_video.mp4
    ```

### Phase 3: Regenerate（重新生成）

**输入：** project_id + scene_id

**流程：**
1. 读取 `storyboard.json`
2. 找到对应镜头
3. 询问是否调整提示词
4. 重新生成
5. 保存为 `v2.png` / `v3.png`

---

## 模型适配器

支持多种生成模型，通过适配器模式接入：

### 1. API 模型

#### OpenAI DALL-E
```python
POST https://api.openai.com/v1/images/generations
{
  "model": "dall-e-3",
  "prompt": "...",
  "size": "1024x1792",
  "quality": "hd"
}
```

#### Stable Diffusion API
```python
POST https://api.stability.ai/v1/generation/...
```

### 2. 本地模型

#### ComfyUI
通过 ComfyUI API：
```bash
curl -X POST http://localhost:8188/prompt \
  -d '{"prompt": {...}}'
```

### 3. 参考图支持（img2img / 风格参考）

#### 角色参考图

放置在 `source/reference/character/` 目录下，在脚本中通过 `**参考图**` 标记引用：

```markdown
## 镜头 1
**画面**：主角站在广场中央
**参考图**：reference/character/protagonist_front.jpg
```

系统行为：
- 自动读取参考图，提取视觉特征
- 将角色描述附加到提示词中（如 "a teen male with black hair, school uniform..."）
- 如果模型支持 img2img，使用参考图作为初始图

#### 场景参考图

放置在 `source/reference/scene/` 目录下，用于保持场景风格一致。

#### 风格参考图

放置在 `source/reference/style/` 目录下，用于设定整体画风（如水墨风、赛博朋克风）。

#### img2img 参数映射

| 模型 | 参考图参数 |
|------|-----------|
| DALL-E 3 | 不支持 img2img，使用描述引用 |
| Stable Diffusion API | `init_image` (base64), `strength: 0.7` |
| ComfyUI | 通过 API workflow 传入 LoadImage 节点 |
| Midjourney | 不支持本地 img2img，使用 URL 引用 |
| aweqy | 通过 skill 参数传入参考图 |

### 4. 使用现有 Skill

如果用户选择 `aweqy-image-generator`：
```
调用 /aweqy-image-generator <prompt>
```

### 5. aweqy 直连（已验证）

`image.aweqy.top` 已验证可用，可直接作为生成后端。配置与调用细节见 `references/image-aweqy-api.md`。
关键点：生成慢（30s–120s+），需 120–300s timeout；可能先 524 再成功；额度制，用尽回 429。

---

## 扩展点

### 1. 模板库

用户可以在 `.story-renderer/templates/` 下保存和复用提示词模板。系统自动将模板合并到每个镜头的提示词中。

#### 模板文件格式

```json
// templates/character_preset.json
{
  "name": "主角预设",
  "description": "主角外貌和服装模板",
  "fields": {
    "character_fingerprint": "teen male, black hair, school uniform, golden aura",
    "style_prompt_prefix": "consistent anime style, detailed illustration"
  }
}
```

```json
// templates/scene_style.json
{
  "name": "赛博朋克场景",
  "description": "赛博朋克风格场景模板",
  "fields": {
    "style_prompt_prefix": "cyberpunk style, neon lights, rain, futuristic city",
    "negative_prompt": "daylight, sunny, nature, low quality"
  }
}
```

```json
// templates/shot_language.json
{
  "name": "运镜语言",
  "description": "镜头运镜模板映射",
  "mappings": {
    "推近": "camera slowly pushes in, close-up detail",
    "环绕": "camera orbits around subject, dynamic angle",
    "平移": "camera pans across the scene, wide establishing shot",
    "拉远": "camera pulls back to reveal full scene",
    "俯拍": "top-down angle, looking down from above",
    "仰拍": "low angle shot, looking up at subject"
  }
}
```

#### 模板使用

```
[模板管理]
1. 查看可用模板
2. 创建新模板
3. 应用模板到项目
4. 删除模板
```

应用模板时，模板字段与项目配置合并（项目配置优先）：

```
最终提示词 = style_prompt_prefix + scene_prompt + character_fingerprint + negative_prompt
```

#### 场景风格自动匹配

如果多个镜头使用不同风格，系统根据镜头描述自动匹配：
- "学校" → 校园场景模板
- "战斗" → 动作场景模板
- "对话" → 人物特写模板

### 2. 插件系统
后续支持自定义插件：
- 后处理脚本（图片滤镜、视频特效）
- 自定义模型接入
- 自定义分镜逻辑

### 3. 批量操作
- 批量重新生成所有镜头
- 批量调整风格
- 导出所有图片为 ZIP

---

## 命令示例

### 初始化
```
初始化 story-renderer
```

### 渲染脚本
```
渲染脚本 ~/Desktop/scripts/ai_future_artist.md
```

### 查看状态
```
story-renderer 状态
```

### 列出所有项目
```
列出项目
```

扫描 `<根路径>/projects/` 目录，显示每个项目的基本信息：

```
📁 项目列表（共 3 个）

1. 全校最废的能力 (3f546fcfe019)
   状态：✅ 已完成 | 场景：17/17 | 类型：图片+视频
   创建：2026-06-21 | 更新：2026-06-22

2. 赛博朋克 2077 番外篇 (a1b2c3d4e5f6)
   状态：⏳ 进行中 | 场景：8/12 | 类型：图片
   创建：2026-06-20 | 更新：2026-06-23

3. 水墨江湖 (f7e8d9c0b1a2)
   状态：❌ 失败 | 场景：5/10 | 类型：图片
   创建：2026-06-19 | 更新：2026-06-19
```

每个项目卡片点击可进入详情视图。

### 查看项目状态
```
story-renderer 状态
```
或指定项目：
```
story-renderer 状态 3f546fcfe019
```

显示项目详细信息：

```
📊 项目状态：全校最废的能力

基本信息：
  项目 ID：3f546fcfe019
  脚本：source/script.md
  分镜模式：手动 | 生成类型：图片+视频
  宽高比：9:16

进度：
  ✅ 已完成：17/17 镜头
  ❌ 失败：0
  ⏳ 进行中：0
  总进度：100%

质量报告：
  验证通过：16 | 尺寸异常：1 (scene_05)

输出文件：
  视频：output/final_video.mp4 (45.2MB)
  字幕：output/subtitles.srt

历史记录：
  [06-22 14:30] 完成 scene_17
  [06-22 14:28] 完成 scene_16
  [06-22 14:25] 完成 scene_15
```

如果项目未完成，显示剩余工作量和预计时间。

### 重新生成某一镜
```
重新生成 3f546fcfe019 scene_05
```

### 修改配置

```
修改配置
```

会进入交互式配置修改界面：

```
当前配置：
1. 根路径：C:\Users\xxx\story-projects
2. 生成模型：OpenAI DALL-E 3
3. 宽高比：9:16（1024x1792）
4. 风格一致性：固定 seed + 角色指纹
5. 用户偏好：
   - 默认生成类型：仅图片
   - 自动重试：是（最多 3 次）
   - 显示进度条：是
   - 详细日志：否

选择要修改的项：
[1-5] 修改具体配置 / [P] 修改偏好 / [Q] 退出
> P

📋 用户偏好设置

当前偏好：
  工作流：
    - 分镜设计自动确认：否
    - 默认生成类型：images
    - 自动重试：是
  
  交互：
    - 详细日志：否
    - 生成前确认：是
    - 显示进度条：是

选择要修改的偏好组：
[1] 工作流偏好
[2] 分镜偏好
[3] 渲染偏好
[4] 交互偏好
[5] 宽高比设置
[6] 一致性策略
[7] 恢复默认设置
[Q] 返回
> 1

修改工作流偏好：
1. 默认生成类型（当前：images）
   [1] 仅图片 / [2] 图片+视频 / [3] 每次询问
   > 3

2. API 失败时自动重试（当前：是）
   [Y] 是 / [N] 否
   > Y

✅ 偏好已更新！
```

---

**宽高比设置：**

```
当前宽高比：9:16 (1024x1792)

可用选项：
[1] 16:9 — 横屏视频 / YouTube (1024x576)
[2] 9:16 — 竖屏短视频 / 抖音 (1024x1792) [当前]
[3] 1:1 — 方形封面 / Instagram (1024x1024)

选择宽高比 [1-3] 或 [Q] 返回：
> 1

✅ 宽高比已更新为 16:9 (1024x576)
注意：已有项目的图片尺寸不变，新生成的项目使用新尺寸
```

**一致性策略：**

```
当前一致性策略：
  固定 seed：是（base_seed = 42）
  角色指纹：teen male, black hair, school uniform, golden_aura
  负面提示词：blurry, low quality, distorted, extra limbs, watermark, text, ugly
  风格前缀：consistent art style, same character throughout, professional illustration

1. 切换固定 seed [Y/N]
2. 修改角色指纹
3. 修改负面提示词
4. 修改风格前缀
5. 重置为默认值
[Q] 返回
> 2

当前角色指纹：
  teen male, black hair, school uniform, golden_aura

输入新角色指纹（留空保持不变）：
> teen male, silver hair, black trench coat, lightning aura

✅ 角色指纹已更新
```

---

### 导出项目

```
导出 3f546fcfe019
导出 3f546fcfe019 --format zip
导出 3f546fcfe019 --format images
导出 3f546fcfe019 --format storyboard
```

**格式选项：**

| 格式 | 说明 | 输出 |
|------|------|------|
| `zip` | 所有分镜图打包 | `<项目名>_export.zip` |
| `images` | 分镜图片集（含 selected 版本） | 文件夹 |
| `storyboard` | 分镜表为 JSON / CSV | `<项目名>_storyboard.json` |
| `video` | 已渲染的视频文件 | `final_video.mp4` |

```
📦 导出项目：全校最废的能力

选择导出格式：
[1] ZIP 打包（所有分镜图 + 元数据）
[2] 图片集（仅图片）
[3] 分镜表（JSON / CSV）
[4] 视频文件（final_video.mp4）

> 1

正在打包...
✅ 导出完成：C:\Users\xxx\story-projects\exports\全校最废的能力_export.zip (23.4MB)
```

导出时自动排除临时文件（v1/v2/v3 未选中的版本）和敏感信息（API Key）。

### 删除项目

```
删除项目 3f546fcfe019
```

```
🗑️  确认删除项目：全校最废的能力

项目信息：
  场景数：17 | 生成类型：图片+视频
  创建时间：2026-06-21

此操作不可恢复，确认删除？[Y/n]
> Y

✅ 项目已删除
```

删除前检查：
- 项目状态为 completed 或 failed（不删除进行中的项目）
- 确认用户意图（防止误删）

### 批量操作

```
批量重生成 3f546fcfe019
批量重生成 3f546fcfe019 --scenes 1-5,8
批量改风格 3f546fcfe019 --style "cyberpunk, neon lights"
批量导出 3f546fcfe019
```

**批量重新生成：**

```
🔧 批量重新生成
项目：全校最废的能力

选择范围：
[1] 全部镜头（17 个）
[2] 指定镜头（如 1-5,8）
[3] 失败的镜头（scene_05）
[4] 评分最低的镜头（基于后置验证）

> 3

将重新生成以下镜头：scene_05
预计费用：$0.08
确认？[Y/n]
> Y

⏳ 重新生成 scene_05...
✅ scene_05 完成
```

**批量改风格：**

```
🎨 批量改风格
项目：全校最废的能力

当前风格前缀：
  consistent art style, same character throughout, professional illustration

输入新风格前缀：
> watercolor painting style, soft brush strokes, pastel colors, dreamy atmosphere

将更新以下镜头：
  scene_01 ~ scene_17（17 个镜头）

确认？[Y/n]
> Y

⏳ 重新生成 17 个镜头...
```

批量操作中支持 `--dry-run` 预览变更，不实际执行。

---

## Refusals（拒绝场景）

- 用户未初始化就尝试渲染 → 提示先运行 `init`
- 脚本路径不存在 → 报错并询问正确路径
- **模型测试失败且用户未选择"跳过"** → 拒绝保存配置，要求重新配置或排查问题
- **配置标记为 `test_skipped: true`** → 每次渲染前警告："⚠️ 当前模型配置未经测试，可能无法正常工作"
- 模型 API 调用失败 → 记录日志，询问是否重试或跳过该镜头
- 视频生成缺少配音配置 → 提示配置 TTS provider
- **用户添加多个模型配置时**，每个都必须单独测试通过才能加入配置列表

---

## Integration（集成）

可以与其他 skill 配合使用：
- **cheat-on-content**：先打分预测脚本，再用 story-renderer 生成视频
- **aweqy-image-generator**：作为生成模型之一
- **browser-automation**：自动上传生成的视频到平台

---

## Notes

- 所有 API 调用都需要处理超时和重试
- 生成大量图片时，建议显示进度条
- 视频生成可能需要较长时间，建议使用后台任务
- 用户的 API Key 应加密存储（后续改进）

---

## 版本升级

### v1.0.0 → v2.0.0

当检测到 config.json 中 `version` 为 "1.0.0" 时，执行迁移：

1. **新增字段**：
   - `config.script_source`：默认 `{"mode": "local", "api_url": "", "api_key": "", "method": "GET", "params": {}, "headers": {}, "response_path": "data.content", "cache_enabled": true, "cache_ttl": 3600}`
   - `config.aspect_ratio`：默认 `{"ratio": "9:16", "options": ["16:9", "9:16", "1:1"], "width": 1024, "height": 1792}`
   - `config.consistency`：默认 `{"enable_fixed_seed": true, "base_seed": 42, "character_fingerprint": "", "negative_prompt": "blurry, low quality, distorted, extra limbs, watermark, text, ugly", "style_prompt_prefix": "consistent art style"}`
   - `user_preferences.storyboard.prefer_auto_split`：默认 `true`
   - `user_preferences.rendering.prompt_language`：默认 `"zh-CN"`

2. **目录扩展**：创建 `source/reference/character/`、`source/reference/scene/`、`source/reference/style/`

3. **更新 `project.json` 结构**：增加 `aspect_ratio` 和 `consistency` 字段

4. **迁移日志**：记录到 `.story-renderer/migration.log`

```
[2026-06-23] Migrating config v1.0.0 → v2.0.0
[2026-06-23] Added aspect_ratio config
[2026-06-23] Added consistency config
[2026-06-23] Created reference directories
[2026-06-23] Migration complete ✅
```

## 接口一致性

`story-renderer` 的在线稿子源接口（`script_source` 配置 + `references/script-source-template.md`）是 Hermes 生态中**在线接口的规范模板**。其他 skill（如 `comic-script-generator`）需要实现在线接口时，必须对齐本 skill 的接口设计原则：

- 单一职责：一个请求 = 一次完整任务
- 幂等性：相同参数返回相同结果
- 版本化：通过 `version` 字段兼容格式
- 状态追踪：通过 `job_id` 查询长时间任务
- 错误标准化：统一 `error` 结构（code/message/details）

1. **角色一致性增强**：ControlNet / IP-Adapter 保持角色外貌一致
2. **视频特效**：转场、滤镜、字幕动画
3. **多语言配音**：支持多种 TTS 引擎
4. **协作功能**：多人同时编辑分镜
5. **Web UI**：提供可视化界面预览和编辑
6. **API Key 加密存储**：使用系统密钥链存储敏感信息
7. **批量导出**：导出分镜为 PDF / PPT / 视频剪辑工程文件
8. **智能参考图匹配**：自动从已有镜头中提取最佳参考图
9. **提示词自动优化**：根据历史生成结果优化提示词模板
10. **版本对比**：并排对比同一镜头的不同版本
