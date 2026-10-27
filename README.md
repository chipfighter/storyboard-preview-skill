# storyboard-preview：AI 视频的副导演 skill

把聊定的视频创意做成文字分镜、预览图和带声音运镜的预览视频，帮助用户在正式生成前判断机位、演出与节奏。适用于 MV、短剧、广告、动画等，原创或有参考素材都可以。

## 它会做什么

1. **主动参与创作，用户最终拍板**：主动提出运镜、构图、表演、节奏和声音上的建议，也可以补充或反对用户的初步方案，说明理由并给出替代方案。用户听取建议后明确决定，就按其决定落实，不擅自替换、弱化要求或反复劝改；未采纳的建议不进入最终方案。用户未指定或授权模型决定的部分，结合故事、上下文和偏好判断。
2. **整理分镜与演出**：记录场景、镜头、预览格、机位、动作、表情、前后景活动、声音与转场。模型补充的设计标明，用户强调的表现保留。
3. **制作轻量预览图**：通常一个镜头一张图，图内 1～4 格，重点帮助用户看懂画面变化。复杂运镜和转场配合文字说明，按镜头选择所需参考，避免无关素材混入。
4. **制作预览视频**：裁出单格，配上音乐、推拉摇移、转场与镜头标注，帮助用户判断节奏和衔接。
5. **交接正式制作**：将已确认或用户授权模型决定的分镜、演出、最新要求、素材用途和已采纳建议交给 [video-prompt](https://github.com/chipfighter/video-prompt-skill)。人物图、场景图、关键帧、灰模、预演和真人参考视频都可按需交接。

预览图通常用于沟通，不自动交给视频模型；其中某一格也可以根据用户需要或模型建议选作关键帧。选中后说明参考什么，不自动作为首帧或要求逐帧复刻。

生图工具使用用户指定的可用工具。整个预览流程不开子代理。

## 安装

把下面这句话发给你的 Claude Code 或 Codex：

```text
帮我安装这个 skill：https://github.com/chipfighter/storyboard-preview-skill
```

将仓库中的 `storyboard-preview/` 整个文件夹复制到相应目录，保留内部文件结构：

- Claude Code，包括桌面端本地 Code 会话：`~/.claude/skills/storyboard-preview/`
- Codex：`~/.agents/skills/storyboard-preview/`
- 其他 agent：读取 `SKILL.md`，支持文件按需读取。

更新后在新会话中使用最新版。Claude 的聊天或 Cowork 技能库使用应用内的独立安装方式，本地 Code 目录不等于账户技能库。

剪预览视频需要 [ffmpeg](https://ffmpeg.org/)。无法使用时可交付按顺序排列的单格图片，并附时长与转场说明。也可以通过 **Code → Download ZIP** 下载仓库，再复制技能文件夹。

## 文件

- `storyboard-preview/SKILL.md`：创作决策、预览流程与交接规则。
- `storyboard-preview/references/shot-table.md`：分镜与演出的组织方式。
- `storyboard-preview/references/preview-images.md`：预览图的参考选择、格数和检查。
- `storyboard-preview/references/animatic.md`：预览视频的制作方法。

## 许可

MIT
