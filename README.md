# xhs-to-obsidian

小红书帖子一键提取到 Obsidian —— 支持图文和视频转录，生成 Peter Thiel 风格洞察笔记。

## 功能

- 🔗 **链接提取** — 粘贴小红书分享链接，自动解析帖子内容
- 🖼️ **图片下载** — 图文帖子自动下载图片到 Obsidian 附件目录
- 🎥 **视频转录** — 下载视频 → ffmpeg 提取音频 → mlx-whisper 转录中文
- 📝 **Peter Thiel 风格笔记** — 反直觉洞察 + 与你的关联 + 深挖判断
- 🗂️ **自动归档** — 有用笔记移入 `xhs/有用/`，无用笔记移入回收站
- 💡 **深挖扩展** — 为有用笔记生成深挖点和可落地行动项

## 安装

将 `skills/xhs-to-obsidian/` 目录放入你的 WorkBuddy skills 目录：

```bash
cp -r skills ~/.workbuddy/skills/xhs-to-obsidian/
```

## 前置依赖

| 依赖 | 用途 | 安装 |
|------|------|------|
| ffmpeg | 视频转音频 | `brew install ffmpeg` |
| mlx-whisper | 语音转录（Apple Silicon 优化） | `pip3 install mlx-whisper` |
| Obsidian | 笔记存储 | 自行安装 |

> ⚠️ 国内用户需配置 HF 镜像：`export HF_ENDPOINT=https://hf-mirror.com`

## 使用方法

在 WorkBuddy 中粘贴小红书分享链接：

```
66 【标题】... https://www.xiaohongshu.com/discovery/item/xxxxxxxxxxxx
```

skill 会自动：
1. 解析帖子类型（图文 / 视频）
2. 下载媒体内容
3. 转录视频语音（如适用）
4. 生成 Peter Thiel 风格笔记保存到 Obsidian
5. 等待你指示是否深挖或归档

## 笔记格式

```markdown
# 反直觉洞察标题

> 2-3 句核心论点，Peter Thiel 风格

## 与我的关联
{为什么对你重要}

## 值得深挖吗
{Yes/No + 原因}

---
<details>原始内容</details>
<details>帖子属性</details>
```

## 目录结构

```
skills/xhs-to-obsidian/
└── SKILL.md          ← skill 主文件
```

## 技术实现

- 解析 `window.__INITIAL_STATE__` JSON 获取帖子数据（一次 HTTP 请求）
- 视频 URL 路径：`note['video']['media']['stream']['h264'][0]['masterUrl']`
- 数据路径：`data['note']['noteDetailMap'][note_id]['note']`（单层，非双重嵌套）
- 终端中文编码问题：所有中间数据写 `/tmp/` 文件，用 Read 工具读取

## License

MIT
