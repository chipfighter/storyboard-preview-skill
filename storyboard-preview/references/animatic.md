# 剪预览视频

下面的做法用 ffmpeg。没有 ffmpeg 时，先问用户能不能装；不能装就按顺序交付单格图片，再附一张"每格停留几秒、用什么转场、备注写什么"的表，并说明这不是视频。

所有中间片段用同一套参数：同一分辨率、同一帧率、同一像素格式（下面用 1280×720、24fps、yuv420p）。不然 xfade 和 concat 会报错。预览用 720p 就够，出得快。

## 1. 裁格

先用 `ffprobe` 看图的宽高，再按版式算每格的位置。每格先切出来，再往里收 3% 去掉白边，最后铺满成片比例。

```bash
# 单格：只去白边、铺满
ffmpeg -y -i A3_sheet.png -vf "crop=iw*0.94:ih*0.94:iw*0.03:ih*0.03,scale=1280:720:force_original_aspect_ratio=increase,crop=1280:720" A3_1.png
# 上下 2 格：上格 crop=iw:ih/2:0:0，下格 crop=iw:ih/2:0:ih/2（下面是下格）
ffmpeg -y -i A2_sheet.png -vf "crop=iw:ih/2:0:ih/2,crop=iw*0.94:ih*0.94:iw*0.03:ih*0.03,scale=1280:720:force_original_aspect_ratio=increase,crop=1280:720" A2_2.png
# 2×2：右上格（第 2 格）
ffmpeg -y -i A1_sheet.png -vf "crop=iw/2:ih/2:iw/2:0,crop=iw*0.94:ih*0.94:iw*0.03:ih*0.03,scale=1280:720:force_original_aspect_ratio=increase,crop=1280:720" A1_2.png
```

生图模型画的格子不一定严格等分。裁完每张都看一眼：切到相邻格或者留下了白边，就手动调 crop 的偏移量。

## 2. 每个镜头做成一段

同一张图里的几格按顺序排成一段，每格停留的时间按分镜表分配。格与格之间默认硬切；动作是连续的，就加 0.2 秒左右的短叠化，看起来更顺。

```bash
# 每格先做成固定时长的小段
ffmpeg -y -loop 1 -i A1_1.png -t 1 -vf "fps=24,format=yuv420p" -c:v libx264 A1_1.mp4
# 同一镜头的小段直接拼起来（list.txt 每行写一个：file 'A1_1.mp4'）
ffmpeg -y -f concat -safe 0 -i list.txt -c copy A1_raw.mp4
```

## 3. 在这一段上做运镜

运镜加在整段上，而不是每格单独加，这样几格之间的镜头运动是连续的。只有 1 格的镜头，就在这一格上做推、拉、摇、移，示意镜头怎么动。下面的 T 是这一段的秒数，N 是总帧数（T×24），用的时候换成具体数字。

推近（从 1 倍放大到 1.15 倍，以画面中心为准）：

```bash
ffmpeg -y -i A1_raw.mp4 -vf "scale=2560:1440,zoompan=z='1+0.15*in/N':d=1:x='iw/2-iw/zoom/2':y='ih/2-ih/zoom/2':s=1280x720:fps=24,format=yuv420p" A1.mp4
```

拉远：把 z 改成 `1.15-0.15*in/N`。先放大到 2560 再推，可以减少 zoompan 的抖动。

横移（画框从左往右移；要从右往左，把 `t/T` 改成 `(1-t/T)`）：

```bash
ffmpeg -y -i A1_raw.mp4 -vf "crop=iw/1.2:ih/1.2:x='(iw-ow)*t/T':y='(ih-oh)/2',scale=1280:720,format=yuv420p" A1.mp4
```

摇镜和跟镜都可以用横移来示意；升降就把 x 和 y 的写法对调。固定机位不用处理。快速旋转、穿门而过这类做不像的运镜，不用硬做，做个简单的推或叠化，再加备注（见第 5 节）。

## 4. 镜头之间接转场

| 设计的转场 | ffmpeg 做法 |
|---|---|
| 硬切 | concat 直接拼接 |
| 叠化 | xfade `transition=fade`（不要用 `dissolve`，那是随机像素颗粒式的溶解，看着像噪点） |
| 黑场 / 白闪 | xfade `fadeblack` / `fadewhite` |
| 甩镜 | xfade `smoothleft`（或 `smoothright` 等），时长 0.3 秒左右 |
| 擦除、遮挡转场 | xfade `wipeleft` 等 |
| 圆形开合 | xfade `circleopen` / `circleclose` |
| 推进式转场 | xfade `zoomin` |
| 匹配剪辑 | 硬切；前后两格的主体形状和位置要对上，对不上就调裁切 |

xfade 的 offset 是前一段开始转场的时间点，等于前面所有片段的时长之和减去之前所有转场的重叠时长，再减去本次转场的时长。每做一次转场，总时长会少一个转场时长，所以分配镜头时长时把重叠算进去，不然后面的镜头会和音乐对不上。

```bash
# A1（4 秒）叠化 1 秒接 A2（3 秒）：offset = 4 - 1 = 3
ffmpeg -y -i A1.mp4 -i A2.mp4 -filter_complex "[0][1]xfade=transition=fade:duration=1:offset=3,format=yuv420p" A1A2.mp4
```

## 5. 加标注和备注

画面上加两种文字，都用用户的语言：

- **标注**：每个镜头左上角小字标出镜号、景别、运镜，例如"B1 中景 固定""A1 走廊 缓慢推近"。不熟悉镜头语言的人可以对照分镜表看。每个镜头都加。
- **备注**：复杂运镜和转场（快速旋转、穿门而过、甩镜、匹配剪辑），以及 4 格放不下的变化，在画面下方加一行，说明实际要怎么拍，例如"镜头快速环绕 180°，从她背后转到正面"。没有这类内容的镜头不加。

文字在第 3 步做完运镜之后、接转场之前加到那个镜头的片段上。drawtext 要指定一个支持这种文字的字体文件：不指定时，有的系统会因为找不到默认字体而直接崩溃。把字体复制到工作目录，用相对路径；文字写进 UTF-8 文本文件，用 textfile 读进来，避免冒号、引号的转义问题。

```bash
# Windows 中文字体：C:/Windows/Fonts/msyh.ttc；其他系统找一个支持用户语言的字体文件
cp C:/Windows/Fonts/msyh.ttc font.ttc
printf 'A2 中景 缓慢推近' > A2_label.txt
printf '镜头快速环绕 180°，从她背后转到正面' > A2_note.txt
# 标注加备注；没有备注的镜头，删掉第二个 drawtext
ffmpeg -y -i A2.mp4 -vf "drawtext=fontfile=font.ttc:textfile=A2_label.txt:fontsize=24:fontcolor=white:box=1:boxcolor=black@0.5:boxborderw=8:x=24:y=24,drawtext=fontfile=font.ttc:textfile=A2_note.txt:fontsize=30:fontcolor=white:box=1:boxcolor=black@0.55:boxborderw=12:x=(w-text_w)/2:y=h-text_h-40,format=yuv420p" A2_noted.mp4
```

## 6. 配乐

```bash
# 从原曲 0:12 开始取音乐，长度跟着画面
ffmpeg -y -i preview_silent.mp4 -ss 12 -i song.mp3 -map 0:v -map 1:a -c:v copy -c:a aac -shortest 预览_场景A_v1.mp4
```

## 7. 交付前自查

- 用 `ffprobe` 看总时长，和分镜表、音乐段落对得上
- 在每个转场点前后，以及每个镜头中间各抽一帧（`ffmpeg -ss 秒数 -i 预览.mp4 -frames:v 1 check.png`），亲自看一眼：裁切对不对、运镜方向对不对、转场有没有用错、标注和备注看不看得清
- 按正常速度播放：不要为了对上音乐把某段压得很快，时长不够就少放一格

## 注意

- 在 Windows 上，滤镜参数里的文件路径用相对路径，`C:/...` 这类写法会被滤镜解析器拆错。字体文件也一样，先复制到工作目录。
- 中间文件放在项目文件夹的一个临时目录里，交付后删掉。
