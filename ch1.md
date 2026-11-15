# 第1章 听：语音识别 ASR

**一句话**：`ASR`（自动语音识别）把声波变成文字，核心是用一个在大尺度弱监督数据上训出来的 `sequence-to-sequence` 模型，把整段音频切成 `30-second sliding window` 逐窗做自回归预测。

**为什么需要它**：假设你有一场两小时的会议录音，要立刻出可搜索的文字稿和字幕。你不会想让人逐句听写；你希望把文件丢进去，几分钟内拿到带时间戳的文本。这就是 `ASR` 的现场：输入是音频，输出是文字加时间线。综合报告口径下，现代通用 ASR 在几十种语言上已接近可用阈值，但长音频、重叠说话、专业术语仍是常见的失分点。

**核心结构**：以 `Whisper` 为例，官方用法是 `model = whisper.load_model("turbo")`，再 `result = model.transcribe("audio.mp3")`。`transcribe` 内部把文件读进来，用 30 秒滑动窗口逐窗处理，每窗做自回归的 `sequence-to-sequence` 预测。模型尺寸分 `tiny` / `base` / `small` / `medium` / `large` / `large-v3` / `large-v3-turbo`（`turbo` 把解码器层数从 32 减到 4，约 8 倍速、支持 99 种语言）。云端还有 `Azure AI Speech`、`Google Cloud Speech-to-Text`、`AWS Transcribe`，以及 `Nemo ASR`、`WeNet` 等可私有化方案。

**关键设计**：30 秒窗口是精度与上下文的折中——窗口太短丢上下文，太长显存和延迟失控；逐窗自回归保证了对长音频的可处理性。代价是原版 Whisper 给的是「语句级」时间戳，不够精细。`faster-whisper` 用 `CTranslate2` 推理引擎重写，配合 `INT8` 量化，在同等精度下约 4 倍速、显存更低，并内置 `Silero VAD` 默认剔除超过 2 秒的静音。`WhisperX` 则在 Whisper 之上用 VAD 预处理减少幻觉、做 `word-level timestamps` 与 `diarization`（说话人分离）。流式场景用 `streaming ASR`，配合 `endpoint detection` 决定何时断句。

**拆解**：
### 一次转写内部发生了什么
音频先重采样到统一 `sample rate`（常见 `16kHz`），再被切成 30 秒一段送进编码器—解码器；解码器自回归地吐出 token，拼成文本。`faster-whisper` 的 `WhisperModel("large-v3", device="cuda")` 把权重转进 `CTranslate2`，按批调度多个 30 秒块，`model.transcribe` 返回的 `info.language` 给出检测到的语言。这正是「整段拆窗、逐窗预测」在真实库里的投影。
### 为什么时间戳有粗有细
原版 Whisper 的时间戳来自注意力对窗口的对齐，粒度停在「这一句大概在哪一秒」；要词级精度就得后处理。`WhisperX` 先做 VAD 把语音段切准，再对齐到 `word-level`，所以同一段音频，`WhisperX` 能给「这个词在第 12.3 秒」，原版只能给「这句话在第 12 到 15 秒」。综合报告口径下，词级时间戳是后期对齐产物，不是模型原生输出。
### 速度从哪里来
`faster-whisper` 的三处提速：一是 `CTranslate2` 是 Transformer 专用推理引擎；二是 `INT8` 量化把 large-v2 在 GPU 上显存从 4525MB 降到 2926MB；三是 `Silero VAD` 先剪掉静音段，只对语音段推理。三者叠加，同样的 `large-v3` 在 CPU 上也能跑得动。
### 边界在哪
重叠说话（谁在什么时候说）、强口音、专业缩写、远场嘈杂，都会拉低 `WER`；`diarization` 与 ASR 是两段独立流水线，接错就会「话对上了人没对上」。综合报告口径下，长音频转写先把 VAD、ASR、diarization 三段解耦，再分别调，比一上来换大模型更划算。

**今天怎么做**：先定场景——离线批量转写用 `faster-whisper` + `Silero VAD`，要词级字幕加 `WhisperX`，要私有化部署用 `WeNet` 或 `Nemo ASR`；先量一段真实音频的 `WER` 与延迟，再决定要不要上更大模型。

<div class="chapend" markdown="1">

**容易误解的地方**：以为模型越大越好，忽略 `INT8` 量化与大模型在精度上几乎无损却能省数倍显存；以为时间戳天然精确，拿原版 Whisper 的语句级时间去做字幕对齐；把 `diarization` 当成 ASR 的内置能力，结果说话人归错了人。

**记住这一句**：ASR 不是「一次听懂一句话」，是「把长音频切成 30 秒窗口、逐窗自回归地拼出文字」。

</div>
