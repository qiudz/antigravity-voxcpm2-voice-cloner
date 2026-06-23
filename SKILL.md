---
name: voxcpm2-voice-cloner
description: 用于控制 and 调用 VoxCPM2 语音克隆器项目进行录音、声音克隆（TTS生成）等操作。当用户提到「用xxx的声音说」「克隆xxx的声音」「录制xxx的声音」或需要操作本地语音克隆器时加载。
---

# VoxCPM2 语音克隆器 Agent 技能 (Skill)

本技能指导 AI Agent 如何调用并操作本地的 `voxcpm2-voice-cloner` 项目，通过克隆指定角色的声音来生成任意语音（TTS）。

---

## 🔍 动态路径探测与定位逻辑

由于每个用户的本地安装路径不同，Agent 应当在首次执行本技能时，通过以下逻辑动态寻找 `voxcpm2-voice-cloner` 项目的实际根路径 `<cloner_root>`：

1. **同级与父级目录探测**：检查当前 Agent 所处工作目录、同级目录、或父级目录中是否存在 `voxcpm2-voice-cloner`。
2. **常用路径列表探测**：检查以下常见开发路径（自适应用户 Home 目录）：
   * `~/IdeaProjects/AI/voxcpm2-voice-cloner`
   * `~/IdeaProjects/voxcpm2-voice-cloner`
   * `~/Code/AI/voxcpm2-voice-cloner`
   * `~/Code/voxcpm2-voice-cloner`
   * `~/Projects/voxcpm2-voice-cloner`
3. **全局检索或主动询问**：如果未能从常见路径中自动探测到项目：
   * 在 Windows 下，可在用户目录下通过 PowerShell 执行检索：
     ```powershell
     Get-ChildItem -Path $HOME -Filter "webui_record.py" -Recurse -Depth 4 -ErrorAction SilentlyContinue
     ```
   * 如果检索耗时过长或未找到，Agent 可以直接询问用户：*“未能在默认路径中定位到您的语音克隆器项目，请问它的本地安装路径是什么？”*

---

## 🛠️ 运行环境与脚本路径

一旦确定项目根路径 `<cloner_root>`：

* **Python 解释器路径**：
  * **Windows**: 使用 `<cloner_root>\.venv\Scripts\python.exe`
  * **macOS/Linux**: 使用 `<cloner_root>/.venv/bin/python`
* **GPU 推理支持**：程序会自动侦测 GPU（NVIDIA CUDA / Intel Arc XPU / CPU 自动切换）。

---

## 🚀 核心操作指南

当用户发出相关指令时，Agent 应在定位到的 `<cloner_root>` 目录下执行以下操作：

### 1. 录制参考音频 (录音)
有两种录音方式，Agent 应根据情况引导用户或运行相应工具：

* **方法 A：网页录制界面（推荐）**
  由 Agent 在终端运行此命令启动 UI，并引导用户访问浏览器进行录制：
  ```bash
  # Windows
  .\.venv\Scripts\python.exe app.py
  # macOS/Linux
  ./.venv/bin/python app.py
  ```
  *说明：该 UI 允许用户输入名字、看稿、录音并保存，声音会自动存入 `voices/<声音名字>/` 目录。*

* **方法 B：命令行交互录音（备用）**
  如果用户不方便用网页，可以直接通过命令行启动录音，要求用户根据提示朗读：
  ```bash
  # Windows
  .\.venv\Scripts\python.exe record.py --voice <声音名字>
  # macOS/Linux
  ./.venv/bin/python record.py --voice <声音名字>
  ```

### 2. 生成克隆语音 (TTS)
当用户要求“用某人的声音说一句话”时，Agent 应在后台自动生成并提供音频：

* **从文本字符串生成**：
  ```bash
  # Windows
  .\.venv\Scripts\python.exe clone.py "<要生成的文本内容>" --voice <声音名字>
  # macOS/Linux
  ./.venv/bin/python clone.py "<要生成的文本内容>" --voice <声音名字>
  ```
* **从文本文件批量生成**：
  ```bash
  # Windows
  .\.venv\Scripts\python.exe clone.py --file <文本文件路径> --voice <声音名字>
  ```
* **输出路径**：默认生成在项目根目录下的 `output/cloned_voice.wav`。Agent 生成后应及时告知用户输出路径，并可提供音频预览。

### 3. 多角色对话生成
若要生成一段多角色情景对话音频，可以使用：
```bash
# Windows
.\.venv\Scripts\python.exe dialogue.py
# macOS/Linux
./.venv/bin/python dialogue.py
```

---

## ⚠️ 故障排查与维护

* **Intel Arc (XPU) 显卡补丁**：
  如果用户使用 Intel Arc 显卡，且在更新依赖后由于官方包覆盖导致报错，Agent 可以运行以下命令重新应用 XPU 补丁：
  ```bash
  .\patches\repatch_xpu.ps1
  ```
* **依赖缺失**：
  如果运行命令报错，说明可能需要更新依赖，可重新执行 `install.ps1` (或 `install.bat`) 脚本进行一键修复。
