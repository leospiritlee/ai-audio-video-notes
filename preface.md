## 序

你手机里一段 30 秒的视频，可能已经在背后跑过了一整条 AI 流水线：先用 `ASR` 把人声转成文字，再翻成另一种语言，用 `TTS` 或 `voice cloning` 合成新配音，用 `lip-sync` 把口型对上，最后由 `ffmpeg` 封装成新文件。另一边，一段「输入一句话、出来一段带环境音的视频」已经是 `Sora 2` 的原生能力。综合报告口径下，这一整条链路可以拆成约十个环节，每个环节背后都是一类成熟的模型。

这本书讲的就是这条链路。它不堆某一篇论文的公式，也不教你调某一个 App；它按「音频轴六章 + 视频轴六章」把「用 AI 实现音频视频」拆成十二个可落地的能力点，讲清每个能力点用什么模型、怎么接、卡在哪。

写法上延续上一本的做法：十二章不按学术分类排，而是按你真正要交付的东西排——先解决「声音进得去、出得来」（听与说），再解决「音色能仿、音乐能写」（生成），然后才是「画面生得出、看得懂、变清晰」（视频）。每章开头先给一个现场，再讲机制——这一路的现场叙述是编排者设定的叙事线索，属「延伸，非语料原文」。

它不重复《主流AI技术精读》：那本讲 `Transformer` / `diffusion` / `contrastive learning` 的范式，这本讲「把这些能力接成可运行功能」。也不重复《机器学习系统精读》：那本讲训练与推理系统层，这本讲「调用与编排这些模型的产品链路」。语料来自公开的官方文档、模型卡与教程（A 级一手事实）加多源综合研究报告（B 级归纳），属于网络多源检索合读，不是任何一本书的精读。

## 两条轴与一条链路

音频与视频各是一条能力轴，中间由「音视频联合」一章接起来：本地化视频就是一条把两条轴串起来的流水线。

```text
  用 AI 实现音视频：两条轴 + 一条联合链路
  ----------------------------------------------------------------
  音频轴（输入声音 / 输出声音）
   听   ASR        Whisper / faster-whisper / WhisperX
   说   TTS        OpenAI TTS / ElevenLabs / CosyVoice
   仿   声音克隆     GPT-SoVITS / OpenVoice / RVC
   写   音乐生成     MusicGen / Suno / Udio
   修   增强与分离   Demucs / Spleeter / RNNoise
   懂   音频理解     AudioSet / PANNs / CLAP

  视频轴（输入画面/文字 / 输出画面）
   生   T2V         Sora 2 / Runway / Pika / Kling
   生   I2V         I2VGen-XL / SVD / Runway
   人   数字人       Wav2Lip / SadTalker / HeyGen
   懂   视频理解     GPT-4V / Gemini / SAM
   清   超分与修复   Real-ESRGAN / BasicVSR / Real-BasicVSR

  联合链路（一条可落地视频的骨架）
   ASR → 翻译 → TTS/克隆 → lip-sync → ffmpeg 封装
```

这十二个能力点不是都要用上；做字幕只需要 ASR，做配音要 ASR+TTS，做数字人要 TTS+lip-sync+数字人。每读一章，你都能回答「这一环节我该调哪个旋钮」。

还有一条暗线：保真 vs 生成。监控取证场景要 `VRT` / `BasicVSR` 这类回归模型；短视频要 `SeedVR` / `FlashVSR` 这类生成式增强。这条线从第 5 章（分离要不要保真）一直牵到第 11 章（超分要不要保真）。

## 十二章速查表

| 章 | 主题 | 你读完能回答 |
|---|---|---|
| 1 听：语音识别 | `ASR`、`Whisper`、`faster-whisper`、`WhisperX` | 为什么长音频要切 30 秒窗口、词级时间戳怎么来 |
| 2 说：语音合成 | `TTS`、`OpenAI TTS`、`ElevenLabs`、`CosyVoice` | tts-1 和 tts-1-hd 差在哪、怎么选音色 |
| 3 仿：声音克隆 | `GPT-SoVITS`、`OpenVoice`、`SoVITS`、`RVC` | 5 秒样本能不能复刻一个人说话 |
| 4 写：音乐生成 | `MusicGen`、`AudioCraft`、`Suno`、`EnCodec` | 文本怎么变成一段带鼓点的音乐 |
| 5 修：增强与分离 | `Demucs`、`Spleeter`、`UVR5`、`RNNoise` | 怎么把人声和伴奏拆开、怎么降噪 |
| 6 懂：音频理解 | `AudioSet`、`PANNs`、`CLAP`、`Cnn14` | 怎么让机器「听懂」一段环境声 |
| 7 生：文生视频 | `Sora 2`、`Runway`、`Pika`、`Kling`、`DiT` | 一句话怎么变成一段视频、物理一致性从哪来 |
| 8 生：图生视频 | `I2VGen-XL`、`SVD`、`Stable Video Diffusion` | 一张图怎么动起来、首尾帧怎么控 |
| 9 人：数字人 | `Wav2Lip`、`SyncNet`、`SadTalker`、`HeyGen` | 怎么让照片跟着声音说话、口型怎么对 |
| 10 懂：视频理解 | `GPT-4V`、`Gemini`、`SAM`、`Grounded SAM` | 怎么让模型看懂视频再自动剪 |
| 11 清：超分修复 | `Real-ESRGAN`、`BasicVSR`、`Real-BasicVSR`、`PSNR` | 低清怎么变高清、为什么逐帧超分会闪 |
| 12 联：音视频流水线 | `ffmpeg`、`AV sync`、`dubbing`、`localization` | 一条本地化视频怎么从脚本走到成品 |

## 遇到这些问题去哪章找

- 开会录音想转文字、加字幕 → 第 1 章
- 想让助手开口说话、做有声书 → 第 2 章
- 想用 5 秒样本复刻自己或别人的声音 → 第 3 章
- 想做一段 BGM、写首歌 → 第 4 章
- 翻唱要扒伴奏、通话要降噪 → 第 5 章
- 要做内容审核、声景检索 → 第 6 章
- 想从提示词生成短视频、广告片 → 第 7 章
- 有产品图想让它动起来、做首尾帧可控 → 第 8 章
- 想做数字人播报、虚拟主播 → 第 9 章
- 想自动看懂长视频、智能剪辑 → 第 10 章
- 老片翻新、监控低清变高清 → 第 11 章
- 想把整条配音+字幕+口型流水线跑通 → 第 12 章

## 二十个自测问题

1. 为什么 `Whisper` 内部要用 `30-second sliding window` 而不是一次处理整段？
2. `faster-whisper` 相比原版快近 4 倍，主要省在哪（推理引擎 / 量化 / VAD）？
3. `WhisperX` 的 `word-level timestamps` 解决了原版 Whisper 的什么短板？
4. `OpenAI TTS` 的 `tts-1` 与 `tts-1-hd` 在延迟和音质上怎么取舍？
5. `gpt-4o-mini-tts` 相比前两者多了什么控制能力？
6. `GPT-SoVITS` 的 `zero-shot TTS` 与 `few-shot` 微调分别要多少样本？
7. `SoVITS` 与 `RVC` 在声音克隆路线上的区别是什么？
8. `OpenVoice` 除了克隆还能控制什么（风格 / 情感 / 口音）？
9. `MusicGen` 的 Small / Medium / Large 三档主要差在哪？
10. `EnCodec` 在音乐生成链路里扮演什么角色（音频分词器）？
11. `Demucs` 的 2 轨与 4 轨分离分别产出什么？
12. `Spleeter` 的 `spleeter:5stems` 比 4stems 多了哪一类？
13. `AudioSet` 的体量（约 `1.9 million` 段、多少类）意味着什么？
14. `CLAP` 是怎么用 `contrastive learning` 做到 `zero-shot audio classification` 的？
15. `Sora 2` 作为 `Diffusion Transformer`，和图像扩散模型的根本差异在时空建模哪一点？
16. `I2VGen-XL` 的 `cascaded diffusion` 为什么要分两级？
17. `Wav2Lip` 为什么要用 `SyncNet` 做同步损失？
18. `SAM` 做 `video object segmentation` 时，怎么和文本引导结合（Grounded SAM）？
19. `BasicVSR` 的循环传播与光流对齐，解决了单帧超分的什么毛病（闪烁）？
20. 一条本地化视频链路里，`ASR` → 翻译 → `TTS` → `lip-sync` 各卡在哪个环节？

## 一个提醒

三件事先说清楚。第一，本书是网络多源检索合读，不是任何一本书的精读；A 级是各模型官方文档、模型卡与教程的一手事实，B 级是多源综合研究报告。凡涉及能力对比、选型判断、成本排序与流水线设计，一律是「综合报告口径」，是便于理解的归纳，不是普适定律；文中出现的延迟、分辨率、时长、参数量均为来源里的特定示例条件。

第二，编排者给出的练习、建议、归纳、判断与每章开头的现场叙述，一律标注「延伸，非语料原文」；它们是为了让你读得下去而加的骨架，不是来源里的原句。语料之外的机制只在明确标注的段落出现。

第三，本书明确不写：具体云服务的逐版本 API 参数、商用产品的横向实测榜单、模型权重的训练细节与复现、版本号与发布时间线。它只讲「用什么能力、怎么接、卡在哪」。