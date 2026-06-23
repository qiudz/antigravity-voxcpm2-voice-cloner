---
name: voxcpm2-voice-cloner
description: 用于控制 and 调用 VoxCPM2 语音克隆器项目进行录音、声音克隆（TTS生成）等操作。当用户提到「用xxx的声音说」「克隆xxx的声音」「录制xxx的声音」或需要操作本地语音克隆器时加载。
---

# VoxCPM2 语音克隆器 Agent 技能 (Skill)

本技能指导 AI Agent 如何调用并操作本地的 [voxcpm2-voice-cloner](file:///C:/Users/qiudx/IdeaProjects/AI/voxcpm2-voice-cloner) 项目，通过克隆指定角色的声音来生成任意语音（TTS）。

---

## 🛠️ 项目定位与运行环境

* **项目根目录**：`C:\Users\qiudx\IdeaProjects\AI\voxcpm2-voice-cloner`
* **Python 解释器**：使用项目目录下的虚拟环境 `.venv\Scripts\python.exe`（Windows 环境）。
* **GPU 推理支持**：程序会自动侦测 GPU（NVIDIA CUDA / Intel Arc XPU / CPU 自动切换）。

---

## 🚀 核心操作指南

当用户发出相关指令时，Agent 应通过在项目根目录下执行以下命令来完成：

### 1. 录制参考音频 (录音)
有两种录音方式，Agent 应根据情况引导用户或运行相应工具：

* **方法 A：网页录制界面（推荐）**
  由 Agent 在终端运行此命令启动 UI，并引导用户访问浏览器进行录制：
  ```powershell
  .\.venv\Scripts\python.exe app.py
  ```
  *说明：该 UI 允许用户输入名字、看稿、录音并保存，声音会自动存入 `voices/<声音名字>/` 目录。*

* **方法 B：命令行交互录音（备用）**
  如果用户不方便用网页，可以直接通过命令行启动录音，要求用户根据提示朗读：
  ```powershell
  .\.venv\Scripts\python.exe record.py --voice <声音名字>
  ```

### 2. 生成克隆语音 (TTS)
当用户要求“用某人的声音说一句话”时，Agent 应在后台自动生成并提供音频：

* **从文本字符串生成**：
  ```powershell
  .\.venv\Scripts\python.exe clone.py "<要生成的文本内容>" --voice <声音名字>
  ```
* **从文本文件批量生成**：
  ```powershell
  .\.venv\Scripts\python.exe clone.py --file <文本文件路径> --voice <声音名字>
  ```
* **输出路径**：默认生成在项目根目录下的 `output/cloned_voice.wav`。Agent 生成后应及时告知用户输出路径，并可提供音频预览。

### 3. 多角色对话生成
若要生成一段多角色情景对话音频，可以使用：
```powershell
.\.venv\Scripts\python.exe dialogue.py
```

---

## ⚠️ 故障排查与维护

* **Intel Arc (XPU) 显卡补丁**：
  如果用户使用 Intel Arc 显卡，且在更新依赖后由于官方包覆盖导致报错，Agent 可以运行以下命令重新应用 XPU 补丁：
  ```powershell
  .\patches\repatch_xpu.ps1
  ```
* **依赖缺失**：
  如果运行命令报错，说明可能需要更新依赖，可重新执行 `.\install.ps1` 脚本进行一键修复。
