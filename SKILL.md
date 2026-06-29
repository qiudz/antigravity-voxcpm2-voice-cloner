---
name: voxcpm2-voice-cloner
description: 用于控制和调用 VoxCPM2 语音克隆器项目，支持参考音频录制、声音克隆 TTS 生成、多角色对话生成，以及字幕文件生成/字幕烧录辅助。当用户提到「用xxx的声音说」「克隆xxx的声音」「录制xxx的声音」「生成字幕」「给语音加字幕」或需要操作本地语音克隆器时加载。
---

# VoxCPM2 语音克隆器 Agent 技能（Skill）

本技能指导 AI Agent 调用并操作本地 `voxcpm2-voice-cloner` 项目，通过克隆指定角色声音生成任意语音（TTS），并在需要时生成字幕文件或带字幕视频。

> 设计原则：
> - **路径动态探测**：不假设用户本地安装位置。
> - **结果可追溯**：所有生成文件均明确告知路径。
> - **不后台承诺**：Agent 必须在当前响应中完成可执行操作或明确说明限制。
> - **字幕优先独立文件**：默认优先生成 `.srt` / `.vtt` 字幕文件，只有用户明确要求“烧录字幕/生成带字幕视频”时才执行视频合成。
> - **字幕大小可控**：字幕字号、行宽、位置必须可配置，避免默认字幕过大遮挡画面。

---

## 1. 适用触发场景

当用户出现以下意图时，应加载本技能：

- “用 xxx 的声音说一句话”
- “克隆 xxx 的声音”
- “录制 xxx 的声音”
- “生成一段多角色对话”
- “给这段语音生成字幕”
- “生成 SRT / VTT 字幕”
- “把字幕加到视频里”
- “字幕太大 / 太小 / 遮挡画面 / 调整字幕大小”
- “操作本地 VoxCPM2 / voxcpm2-voice-cloner 项目”

---

## 2. 动态路径探测与定位逻辑

由于用户本地安装路径可能不同，Agent 首次执行本技能时，应动态寻找项目根路径 `<cloner_root>`。

### 2.1 同级与父级目录探测

检查当前 Agent 工作目录、同级目录、父级目录中是否存在：

```text
voxcpm2-voice-cloner
```

建议探测顺序：

```bash
pwd
ls
ls ..
find .. -maxdepth 3 -type d -name "voxcpm2-voice-cloner" 2>/dev/null
```

### 2.2 常用开发路径探测

自适应用户 Home 目录，依次检查：

```text
~/IdeaProjects/AI/voxcpm2-voice-cloner
~/IdeaProjects/voxcpm2-voice-cloner
~/Code/AI/voxcpm2-voice-cloner
~/Code/voxcpm2-voice-cloner
~/Projects/voxcpm2-voice-cloner
~/AI/voxcpm2-voice-cloner
~/Workspace/voxcpm2-voice-cloner
```

### 2.3 未定位到项目时的处理

如果未能从上述路径定位项目，Agent 应提示用户：

> 未能在默认路径中定位到您的语音克隆器项目，请告诉我它的本地安装路径。

如果用户没有提供路径，Agent 可在当前合适目录自动下载并初始化项目：

```bash
git clone https://github.com/qiudz/voxcpm2-voice-cloner.git
cd voxcpm2-voice-cloner
```

然后根据系统类型执行安装脚本：

```powershell
# Windows PowerShell
.\install.ps1
```

```bat
:: Windows CMD
install.bat
```

```bash
# macOS / Linux
bash install.sh
```

安装完成后，将该目录设为：

```text
<cloner_root>
```

---

## 3. 运行环境与解释器路径

确定 `<cloner_root>` 后，统一在项目根目录下执行命令。

### 3.1 Python 解释器路径

```text
Windows:     <cloner_root>\.venv\Scripts\python.exe
macOS/Linux: <cloner_root>/.venv/bin/python
```

### 3.2 设备推理支持

项目会根据本机环境自动选择：

- NVIDIA CUDA
- Intel Arc XPU
- CPU

Agent 不应硬编码设备类型，除非用户明确要求。

---

## 4. 核心能力

### 4.1 录制参考音频

### 方法 A：网页录制界面（推荐）

Agent 在 `<cloner_root>` 目录下启动：

```bash
# Windows
.\.venv\Scripts\python.exe app.py
```

```bash
# macOS / Linux
./.venv/bin/python app.py
```

说明：

- UI 支持输入声音名称、查看录音稿、录音并保存。
- 参考音频默认保存到：

```text
<cloner_root>/voices/<声音名字>/
```

### 方法 B：命令行交互录音（备用）

```bash
# Windows
.\.venv\Scripts\python.exe record.py --voice <声音名字>
```

```bash
# macOS / Linux
./.venv/bin/python record.py --voice <声音名字>
```

---

### 4.2 生成克隆语音（TTS）

### 从文本字符串生成

```bash
# Windows
.\.venv\Scripts\python.exe clone.py "<要生成的文本内容>" --voice <声音名字>
```

```bash
# macOS / Linux
./.venv/bin/python clone.py "<要生成的文本内容>" --voice <声音名字>
```

### 从文本文件批量生成

```bash
# Windows
.\.venv\Scripts\python.exe clone.py --file <文本文件路径> --voice <声音名字>
```

```bash
# macOS / Linux
./.venv/bin/python clone.py --file <文本文件路径> --voice <声音名字>
```

### 默认输出路径

```text
<cloner_root>/output/cloned_voice.wav
```

Agent 生成完成后必须告知用户：

- 生成是否成功
- 音频输出路径
- 如有字幕，同时告知字幕路径

---

### 4.3 多角色对话生成

```bash
# Windows
.\.venv\Scripts\python.exe dialogue.py
```

```bash
# macOS / Linux
./.venv/bin/python dialogue.py
```

---

## 5. 字幕生成逻辑（新增）

字幕能力分为三种模式：

| 模式 | 说明 | 默认行为 |
|---|---|---|
| `subtitle_file` | 仅生成字幕文件 | 推荐默认 |
| `subtitle_burn` | 将字幕烧录到视频中 | 仅用户明确要求时执行 |
| `subtitle_audio_preview` | 生成音频 + 字幕文件，供剪辑工具使用 | 推荐用于 TTS 场景 |

---

### 5.1 字幕文件格式

默认支持：

- `.srt`：通用字幕格式，适合剪映、Premiere、Final Cut、播放器等。
- `.vtt`：适合网页播放。
- `.ass`：适合精细控制字幕字号、描边、位置、样式。

默认优先级：

```text
用户指定格式 > srt > vtt > ass
```

如果用户未指定格式，则生成：

```text
<cloner_root>/output/cloned_voice.srt
```

---

### 5.2 字幕生成输入来源

字幕文本来源按优先级判断：

1. 用户显式提供的文本。
2. `clone.py` 本次生成 TTS 使用的文本。
3. 文本文件 `--file` 中的内容。
4. 多角色对话脚本中的台词。

不要从音频自动转写，除非项目中明确提供 ASR 能力或用户明确要求接入外部转写工具。

---

### 5.3 字幕切分规则

为避免字幕过长或过大，字幕应按以下规则切分：

### 中文推荐规则

```text
每行 12～18 个中文字符
每条字幕最多 2 行
每条字幕建议 1.5～4 秒
```

### 英文推荐规则

```text
每行 32～42 个英文字符
每条字幕最多 2 行
每条字幕建议 1.5～4 秒
```

### 混合文本规则

- 中文、英文、数字混排时，按视觉宽度估算。
- 中文字符按 2 个宽度单位估算。
- 英文字母、数字、半角符号按 1 个宽度单位估算。
- 单条字幕视觉宽度建议不超过 36～42 单位。

---

### 5.4 字幕时间轴生成逻辑

如果项目未提供精确音素/字级时间戳，则使用估算时间轴。

### 估算规则

```text
中文：约 4～6 字/秒
英文：约 12～16 字符/秒
最短单条字幕：1.2 秒
最长单条字幕：5.0 秒
字幕间隔：0.08～0.15 秒
```

### 总时长对齐

如果已存在生成音频文件，Agent 应尝试读取音频总时长，并将字幕总时长缩放到音频时长范围内。

可选方案：

```bash
ffprobe -v error -show_entries format=duration -of default=noprint_wrappers=1:nokey=1 output/cloned_voice.wav
```

如果无法读取音频时长，则使用文本节奏估算。

---

## 6. 字幕大小规范（重点）

为避免字幕过大遮挡画面，字幕字号必须根据目标分辨率动态设置。

### 6.1 默认字号建议

| 视频分辨率 | 推荐字号 | 允许范围 | 备注 |
|---|---:|---:|---|
| 720p | 24 | 20～28 | 移动端可略大 |
| 1080p | 32 | 28～38 | 默认推荐 |
| 1440p | 42 | 36～48 | 保持克制 |
| 4K | 58 | 48～68 | 不建议超过 72 |
| 竖屏 1080x1920 | 38 | 32～44 | 适合短视频 |

默认值：

```text
1080p 横屏：32
1080x1920 竖屏：38
```

### 6.2 字号计算公式

如果能获取视频高度 `H`，推荐使用：

```text
font_size = round(H * 0.030)
```

并限制范围：

```text
min_font_size = 22
max_font_size = 68
```

即：

```text
font_size = max(22, min(round(H * 0.030), 68))
```

特殊场景：

- 中文信息密度高：可使用 `H * 0.028`
- 短视频口播：可使用 `H * 0.032`
- 屏幕内容较多：建议使用 `H * 0.026`

---

### 6.3 字幕位置规范

默认位置：底部居中。

推荐边距：

```text
bottom_margin = round(H * 0.075)
```

规则：

- 不要贴底。
- 不要遮挡人物嘴部。
- 如果视频有底部 UI、表格、代码、Logo，应上移字幕。
- 竖屏短视频可以适当上移，避免被平台按钮遮挡。

---

### 6.4 字幕样式规范

推荐样式：

```text
字体颜色：白色
描边颜色：黑色
描边宽度：2～3
阴影：轻微
背景框：默认不启用
```

若画面复杂，可启用半透明背景框，但应避免面积过大。

---

## 7. 字幕文件生成命令建议

如果项目已有字幕脚本，应优先使用项目内脚本，例如：

```bash
# 假设项目存在 subtitle.py
./.venv/bin/python subtitle.py --text "<字幕文本>" --audio output/cloned_voice.wav --format srt --output output/cloned_voice.srt
```

如果项目暂未提供字幕脚本，Agent 可生成一个轻量级字幕文件生成器 `subtitle.py`。

---

### 7.1 推荐 subtitle.py 行为

脚本参数建议：

```bash
python subtitle.py \
  --text "<字幕文本>" \
  --audio output/cloned_voice.wav \
  --format srt \
  --output output/cloned_voice.srt \
  --max-line-width 36 \
  --max-lines 2
```

支持参数：

| 参数 | 说明 | 默认值 |
|---|---|---|
| `--text` | 字幕文本 | 空 |
| `--file` | 从文本文件读取字幕 | 空 |
| `--audio` | 用于对齐总时长的音频文件 | 空 |
| `--format` | `srt` / `vtt` / `ass` | `srt` |
| `--output` | 输出字幕路径 | `output/cloned_voice.srt` |
| `--max-line-width` | 单行最大视觉宽度 | `36` |
| `--max-lines` | 单条字幕最大行数 | `2` |
| `--min-duration` | 单条最短时长 | `1.2` |
| `--max-duration` | 单条最长时长 | `5.0` |

---

## 8. 字幕烧录到视频（默认操作）

默认将“把字幕加到视频中 / 烧录字幕 / 生成带字幕视频”，可使用 `ffmpeg`。

### 8.1 生成 ASS 字幕后烧录（推荐）

ASS 字幕更适合控制字号、描边和位置。

示例：

```bash
ffmpeg -i input.mp4 -vf "ass=output/cloned_voice.ass" -c:a copy output/video_with_subtitles.mp4
```

### 8.2 使用 SRT 烧录字幕

```bash
ffmpeg -i input.mp4 \
  -vf "subtitles=output/cloned_voice.srt:force_style='FontSize=32,Outline=2,Shadow=1,Alignment=2,MarginV=80'" \
  -c:a copy output/video_with_subtitles.mp4
```

注意：

- `FontSize` 应根据视频高度动态计算。
- `MarginV` 应根据视频高度动态计算。
- Windows 路径中反斜杠可能需要转义，必要时应使用绝对路径或改为 ASS 字幕。

---

## 9. 推荐的一体化生成流程

当用户说：

> 用 xxx 的声音说：你好，欢迎使用系统，并生成字幕。

Agent 应执行如下流程：

1. 定位 `<cloner_root>`。
2. 确认 `<声音名字>` 是否存在于 `voices/`。
3. 调用 `clone.py` 生成音频。
4. 使用本次 TTS 文本生成 `.srt` 字幕。
5. 如果用户提供视频并要求烧录字幕，再生成 `.ass` 或调用 `ffmpeg`。
6. 返回结果路径。

示例命令：

```bash
cd <cloner_root>
./.venv/bin/python clone.py "你好，欢迎使用系统。" --voice <声音名字>
./.venv/bin/python subtitle.py --text "你好，欢迎使用系统。" --audio output/cloned_voice.wav --format srt --output output/cloned_voice.srt
```

返回示例：

```text
已生成克隆语音：output/cloned_voice.wav
已生成字幕文件：output/cloned_voice.srt
```

---

## 10. 文件命名规范

为避免覆盖历史结果，建议输出文件名包含时间戳或任务标识。

推荐格式：

```text
output/<voice_name>_<yyyyMMdd_HHmmss>.wav
output/<voice_name>_<yyyyMMdd_HHmmss>.srt
output/<voice_name>_<yyyyMMdd_HHmmss>.vtt
output/<voice_name>_<yyyyMMdd_HHmmss>.ass
```

如果项目脚本默认输出固定文件：

```text
output/cloned_voice.wav
```

Agent 可在生成后复制为带时间戳版本。

---

## 11. 故障排查与维护

### 11.1 Intel Arc XPU 补丁

如果用户使用 Intel Arc 显卡，且更新依赖后因官方包覆盖导致报错，可重新应用 XPU 补丁：

```powershell
.\patches\repatch_xpu.ps1
```

### 11.2 依赖缺失

如果运行命令报错，可能需要重新安装依赖。

```powershell
.\install.ps1
```

```bat
install.bat
```

```bash
bash install.sh
```

### 11.3 字幕乱码

如字幕在视频软件中乱码：

- 优先使用 UTF-8 编码保存字幕。
- Windows 下建议保存为 UTF-8 with BOM，兼容部分旧软件。
- ASS 字幕中应指定中文字体，例如：`Microsoft YaHei`、`SimHei`、`Noto Sans CJK SC`。

### 11.4 字幕过大或遮挡

处理顺序：

1. 降低 `FontSize`。
2. 减少每条字幕行数。
3. 缩短单条字幕文本。
4. 增大 `MarginV` 使字幕上移。
5. 使用 ASS 字幕精细控制位置。

---

## 12. Agent 响应规范

Agent 执行本技能后，回复用户时应包含：

```text
操作结果：成功 / 失败
声音名称：<voice_name>
音频路径：<audio_path>
字幕路径：<subtitle_path>
视频路径：<video_path，如有>
说明：<必要说明>
```

如果失败，应包含：

```text
失败阶段：路径定位 / 环境安装 / 录音 / TTS / 字幕生成 / 字幕烧录
错误摘要：<简短错误>
建议处理：<下一步动作>
```

---

## 13. 安全与合规提醒

- 不应帮助克隆他人声音用于欺骗、冒充、诈骗或绕过身份验证。
- 对涉及真实人物声音的请求，应提醒用户确保已获得授权。
- 不应生成违法、欺诈、骚扰、仇恨或造成现实伤害的音频内容。
- 如用户明确要求冒充某人进行欺骗，应拒绝执行。

---


## 14. 推荐增强点

后续可继续增强：

- 支持 ASS 字幕样式模板。
- 从 TTS 生成日志读取更精确的句级时间戳。
- 根据音频实际静音段自动切分字幕。
- 支持多角色字幕颜色区分。
- 支持横屏/竖屏自动字号与位置策略。
- 支持输出剪映可导入字幕格式。
