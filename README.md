# storyboard-preview：AI 视频的副导演 skill

和 AI 聊完视频创意、准备生成视频之前，让 agent 先帮你做一遍"文字分镜表 → 黑白线稿分镜 → 配乐的线稿预览视频"。花钱生成视频之前，你就能看到镜头、运镜、转场和节奏，看着预览再改。

适用于 MV、短剧、广告、动画等，原创或有参考视频都可以。只要是能调用生图工具、能看图的多模态 agent 都能用（Claude Code、Codex 等）。

## 它会做什么

1. 你和 agent 聊得差不多、准备开始做时，它一次问清三件事：要不要先确认文字分镜表？要不要做线稿预览？中途在哪里停下来给你看？
2. 出文字分镜表：场景 → 机位 → 镜头，写清运镜、动作、左右手、转场、时长和音乐位置。你不要确认，它也照样写表，只是贴出来以后不停，直接往下做。
3. 出线稿：一个场景对应生图工具的一个会话，一个机位对应一组图，并行调用。每组是一张黑白多格大图，格数选能放下的最小规格（1 格、2 格、2×2、3×3）。机位一变就另起一组，避免生图模型在上一个机位上直接改出下一个。氛围和质感另外用 0～2 张彩色样张来对齐。
4. 剪预览：用 ffmpeg 把大图裁成单格，配上音乐，在线稿上做推、拉、摇、移，按设计的真实转场接起来，按正常速度播放。

生图工具用你在对话里指定的那个（GPT、Gemini、Midjourney、Codex 等），不会自己换工具。

## 安装

### 最简单：让你的 AI 帮你装

把下面这句话发给你的 Claude Code 或 Codex：

```
帮我安装这个 skill：https://github.com/chipfighter/storyboard-preview-skill
```

它会自己下载并放到正确的位置。装好以后重开一个对话，正常聊你的视频创意就行，聊完准备开始做时，它会自动用上这个 skill。

### 给 AI 的安装说明

如果你是收到上面这句话的 AI：把本仓库里的 `storyboard-preview/` 整个文件夹（包括里面的 `references/`）复制到用户的 skills 目录，然后告诉用户重开对话即可。

- Claude Code：`~/.claude/skills/storyboard-preview/`
- Codex：`~/.agents/skills/storyboard-preview/`
- 其他 agent：把 `SKILL.md` 当作说明书，`references/` 里的文件放在旁边，按需读取。

剪预览视频需要 [ffmpeg](https://ffmpeg.org/)。用户电脑上没有的话，先告诉用户，问一下要不要装；不装的话，按顺序交付单格图片，再附一张时长和转场表。

### 手动安装

在本页点 **Code → Download ZIP**，解压后把 `storyboard-preview/` 文件夹复制到上面对应的目录。

## 文件

- `SKILL.md`：流程、分岔和核心规则
- `references/shot-table.md`：分镜表格式
- `references/line-sheets.md`：分组、提示词模板、格数、出图后自查
- `references/animatic.md`：ffmpeg 裁格、运镜、转场、配乐

## 许可

MIT
