---
description: "从小红书链接提取帖子内容（图文/视频），生成 Peter Thiel 风格笔记保存到 Obsidian。支持视频语音转录、图片下载、笔记分类归档和深挖扩展。"
---

# xhs-to-obsidian

从小红书链接提取帖子，生成结构化笔记保存到 Obsidian vault。

## 触发条件

当用户分享 `xiaohongshu.com` 链接，或说"提取小红书"、"保存到 Obsidian"、"帮我存这篇"时触发。

---

## 常量和配置

```
VAULT_PATH:     /Users/yee/Downloads/ai文件/
XHS_DIR:        xhs/
IMG_DIR:        xhs/img/
COOKIES_FILE:   ~/cookies.json
WHISPER_MODEL:  mlx-community/whisper-large-v3-turbo
HF_ENDPOINT:    https://hf-mirror.com
```

---

## 执行流程

### 第一步：判断帖子类型

1. 从 URL 提取 `note_id`（URL 中 `/item/` 后的那串）
2. 用 Python + urllib 请求页面，解析 `window.__INITIAL_STATE__` JSON
3. 数据路径：`data['note']['noteDetailMap'][note_id]['note']`（单层，不是双重嵌套）
4. 判断类型：`note['type']` — `"video"` 或 `"normal"`

### 第二步：处理图文帖子

1. 从 `note['imageList']` 获取图片 URL
2. 下载图片到 `{VAULT_PATH}/{IMG_DIR}/img/`，文件名用 `{note_id}_{index}.jpg`
3. 提取正文：`note['desc']`
4. 提取互动数据：点赞数、收藏数、评论数
5. 生成 Peter Thiel 风格笔记

### 第三步：处理视频帖子

1. 视频下载 URL：`note['video']['media']['stream']['h264'][0]['masterUrl']`
2. 下载视频（必须带 header：`Referer: https://www.xiaohongshu.com/`）
3. 用 ffmpeg 提取音频：
   ```
   ffmpeg -i /tmp/xhs_video.mp4 -vn -acodec pcm_s16le -ar 16000 -ac 1 /tmp/xhs_audio.wav
   ```
4. 语音转录（必须设置 HF 镜像）：
   ```python
   import os
   os.environ['HF_ENDPOINT'] = 'https://hf-mirror.com'
   import mlx_whisper
   result = mlx_whisper.transcribe(
       '/tmp/xhs_audio.wav',
       path_or_hf_repo='mlx-community/whisper-large-v3-turbo',
       language='zh'
   )
   ```
5. 清理转录文本末尾的重复短语（whisper 伪影）
6. 生成 Peter Thiel 风格笔记（包含转录文本）

### 第四步：生成笔记文件

保存到：`{VAULT_PATH}/{XHS_DIR}/{YYYY-MM-DD} {title}.md`

```markdown
# {反直觉洞察标题}

> {2-3 句核心论点，Peter Thiel 风格，说出一个被忽视的真相}

## 与我的关联

{结合 USER.md 中的岚岚画像，写 1-2 句为什么这对她重要}

## 值得深挖吗

{Yes/No + 一句话原因}

---
<details>
<summary>原始内容</summary>

{正文 / 转录文本}

</details>

<details>
<summary>帖子属性</summary>

- 作者：{author}
- 链接：{url}
- 类型：{图片/视频}
- 日期：{date}
- 互动：{likes}赞 {collects}收藏 {comments}评论

</details>
```

### 第五步：分类归档（用户说"分类"、"归档"、"整理"时执行）

对 `xhs/` 目录下所有笔记逐篇判断：

- **值得深挖**（关联性高、有行动启发）→ 移动到 `{VAULT_PATH}/{XHS_DIR}/有用/`
- **不值得深挖** → 用 macOS 回收站删除：
  ```bash
  osascript -e 'move POSIX file "完整路径" to trash'
  ```

然后为 `有用/` 目录下的所有笔记生成深挖笔记：

保存为：`{VAULT_PATH}/{XHS_DIR}/有用/深挖笔记 YYYY-MM-DD.md`

包含：
- 每个有用帖子的 3 个深挖点（洞察、为什么重要、和岚岚的目标关联）
- 可落地行动项表格：行动、时间估算、优先级

---

## 实现注意

- **终端中文乱码**：所有中间数据写 `/tmp/` 文件，用 Read 工具读取，不要在终端直接 print 中文
- **Python HTTP 请求**：用 `urllib` 或 `requests`，如遇到 SSL 问题可设置 `ssl.CERT_NONE`
- **解析 `__INITIAL_STATE__`**：用正则 `r'window\.__INITIAL_STATE__\s*=\s*({.+?})\s*</script>'`
- **视频 URL**：在 `note['video']['media']['stream']['h264'][0]['masterUrl']`，不是 `consumer.originVideoKey`
- **HF 镜像**：国内必须设置 `HF_ENDPOINT=https://hf-mirror.com`，否则模型下载超时
- **Whisper 伪影**：转录文本末尾常有重复短语，用正则清理

---

## 依赖检查

执行前确认以下工具可用：

| 工具 | 用途 | 安装方式 |
|------|------|----------|
| `ffmpeg` | 视频转音频 | `brew install ffmpeg` |
| `mlx-whisper` | 语音转录（Apple Silicon） | `pip3 install mlx-whisper` |
| Python + urllib/requests | 抓取页面 | 系统自带 |

---

*基于岚岚的 Obsidian 工作流定制，2026/5/1*
