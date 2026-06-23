---
name: story-renderer
description: 将故事脚本转换为分镜图片或视频。支持自动分镜和手动分镜两种模式，用户可配置生成模型（API 或本地），输出高质量的视觉化内容。
argument-hint: <command> [options]
allowed-tools: Bash, Read, Write, Edit, Glob, Grep, Agent, Skill
---

# Story Renderer

将文本故事脚本自动转换为分镜图片或完整视频。

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
        │   └── reference/       # 参考图
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

---

## 使用流程

### 1. 初始化

用户第一次使用时必须运行初始化：

```
初始化 story-renderer
```

会询问：
1. **根路径**：所有项目和配置保存的位置
2. **生成模型类型**：
   - API 模型（OpenAI DALL-E / Midjourney / Stable Diffusion API）
   - 本地模型（ComfyUI / Automatic1111）
   - 使用现有 skill（如 aweqy-image-generator）
3. **API 配置**（如果选择 API）：
   - API Key
   - Base URL
   - 默认参数
4. **模型测试**（关键步骤）：
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
  "version": "1.0.0",
  "root_path": "C:/Users/xxx/story-renderer-projects",
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
   
5. 创建目录结构
6. 写入 `config.json`（仅当测试通过）

### Phase 2: Render（渲染）

**输入：** 脚本路径

**流程：**

1. **读取配置**
   - 从 `config.json` 读取模型配置

2. **读取脚本**
   - 检测脚本格式
   - 提取内容

3. **分镜设计**
   - 自动模式：AI 分析脚本，切分镜头
   - 手动模式：解析现有分镜标记
   - 生成 `storyboard.json`

4. **询问生成类型**
   - 仅图片 / 图片+视频

5. **生成提示词**
   - 为每一镜生成 AI 图片提示词
   - 支持风格一致性（角色、场景、色调）

6. **批量生成**
   - 调用配置的模型 API
   - 保存到 `generated/images/scene_XX/v1.png`
   - 记录日志

7. **视频剪辑**（如果选择视频模式）
   - 生成配音（TTS）
   - 用 ffmpeg 剪辑
   - 添加字幕
   - 输出最终视频

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

### 3. 使用现有 Skill

如果用户选择 `aweqy-image-generator`：
```
调用 /aweqy-image-generator <prompt>
```

---

## 扩展点

### 1. 模板库
用户可以保存常用的提示词模板：
- 角色预设（主角外貌、服装）
- 场景风格（赛博朋克、水墨画、动漫风）
- 镜头语言（特写、全景、运镜）

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
3. 用户偏好：
   - 默认生成类型：仅图片
   - 自动重试：是（最多 3 次）
   - 显示进度条：是
   - 详细日志：否

选择要修改的项：
[1-3] 修改具体配置 / [P] 修改偏好 / [Q] 退出
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
[5] 恢复默认设置
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

## Future Enhancements

1. **角色一致性**：使用 ControlNet / IP-Adapter 保持角色外貌一致
2. **视频特效**：转场、滤镜、字幕动画
3. **多语言配音**：支持多种 TTS 引擎
4. **协作功能**：多人同时编辑分镜
5. **Web UI**：提供可视化界面预览和编辑
