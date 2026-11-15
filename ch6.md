# 第6章 懂：音频理解

**一句话**：音频理解让机器「听懂」声音——`AudioSet` 提供大规模标签数据，`PANNs` / `Cnn14` 做 `sound event detection` 与打标，`CLAP` 用 `contrastive learning` 实现 `zero-shot audio classification` 与检索。

**为什么需要它**：你有一万段监控或环境录音，要快速筛出「有玻璃破碎声」的片段，或给一段声音自动生成文字描述做检索。现场是：输入音频，输出「这是什么声音、发生在哪、和哪段文字最相关」。综合报告口径下，AudioSet 的体量与类别覆盖，使它成为几乎所有音频理解模型的共同底座。

**核心结构**：`AudioSet` 是 Google 的大规模音频数据集，约 `1.9 million` 段带标签音频，覆盖 `527` 个声音类别（ontology classes）。`PANNs`（Pretrained Audio Neural Networks）是在 AudioSet 上预训练的识别模型，`Cnn14` 是常用骨干，`Cnn14_DecisionLevelMax` 支持 `sound event detection`（定位声音在时间轴上的位置）。`CLAP`（Contrastive Language-Audio Pretraining）由 LAION-AI 提出，用 `contrastive learning` 对齐音频与文本表示，从而做 `zero-shot audio classification`（零样本音频分类）与 `audio retrieval`（音频检索）。相关任务还有 `automated audio captioning`（自动音频字幕）、`audio tagging`（音频打标）；数据集有 `AudioCaps`、`YAMNet`（521 类）。

**关键设计**：音频理解和 ASR 不一样——ASR 把「话」变成「字」，音频理解把「环境声/音乐/事件」变成「语义标签或检索向量」。`PANNs` 走的是「在 527 类上预训练 → 微调到你的少数类」；`CLAP` 走的是「音频与文本对齐到同一空间 → 用文字当查询」。后者最省标注：你不用为每类准备样本，直接用 `zero-shot` 问「有没有狗叫」。

**拆解**：
### 527 类意味着什么
`AudioSet` 的 `527` 类是一个层级本体，从「人类声音」到「钟声」到「消防车」层层细分。模型在这么广的标签上预训练，学到的不是某一类，而是「声音之间的共性表征」，所以迁移到新任务只需少量样本。这是它成为底座的原因。
### 事件检测 vs 打标
`audio tagging` 只回答「这段里有没有猫叫」；`sound event detection` 还要回答「猫叫在第 3.2 到 3.8 秒」。`Cnn14_DecisionLevelMax` 输出的是逐帧决策，因此能定位。`PANNs` 论文给出的 `Cnn14` 即在 527 类上预训练，再按需改最后一层类别数。
### CLAP 怎么用文字当查询
`CLAP` 把音频和文本分别编码进一个共享空间，训练目标是「匹配的音频-文本对靠近、不匹配的远离」。推理时，把候选文字也编码进去，谁最近就归谁类——这就是 `zero-shot`：类别名称可以是训练时从没见过的词。
### 检索与字幕
`audio retrieval` 是「给一段声音，找最相关的文字/声音」；`automated audio captioning` 是「给声音，生成一句描述」。两者都建立在「音频有了语义表示」之上，是内容审核、声景检索、无障碍描述的基础能力。

**今天怎么做**：做声学事件检测，用 `PANNs` 的 `Cnn14` 微调到你的目标类；做零样本筛选或语义检索，用 `CLAP` 直接拿文字查询；做大规模打标先借 `AudioSet` 预训练权重。先定清是「打标、定位还是检索」，再选模型。

<div class="chapend" markdown="1">

**容易误解的地方**：以为音频理解就是 ASR，忽略 ASR 只处理语音而音频理解覆盖环境声与音乐；以为 `CLAP` 需要每类样本，其实它主打 `zero-shot`；把 `527` 类当成死标签，忽略它是可迁移的预训练底座。

**记住这一句**：音频理解把声音变成语义标签或检索向量，CLAP 让「用文字查声音」也能零样本实现。

</div>
