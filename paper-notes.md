# Paper Notes

## To Read

- [ ] 论文标题 — 为什么想读 — 链接

## Reading Notes

### Post-training makes large language models less human-like

- Direction: LLM post-training / cognitive modeling / human behavior simulation
- Year: 2026
- Link: https://arxiv.org/abs/2605.07632
- Status: 已读

#### Problem

当 LLM 被用作人类被试的替代品时，什么样的模型更能复现人的真实行为？论文重点检验三个问题：后训练是否改变模型与人类行为的相似度；这种影响是否随模型家族、规模、代际和后训练目标变化；加入特定参与者的人口统计与问卷信息，能否改善个体级行为预测。

这里的 behavioral alignment 指“模型输出与人类实际反应的统计相似性”，不是安全领域中“符合人类偏好与价值”的 alignment。

#### Core idea

作者构建 **Psych-201**，把行为实验中单个参与者的一整场 session 转成自然语言 transcript，包含实验说明、刺激、逐 trial 历史、参与者反应及任务相关上下文。随后以人类真实回答在 LLM 输出分布下的似然衡量 human-likeness，并在尽可能同家族、同规模的条件下比较 base model 与经过 instruction、reasoning、vision 等目标后训练的版本。

#### Experimental datasets

- **Psych-201**：208,021 名参与者、25,906,599 条行为反应、数百个实验；规模约为 Psych-101 的 3.5 倍、典型 mega-study 的 13 倍。
- 数据覆盖强化学习、风险选择、经济博弈、推理、心理语言学、记忆、分类等主要领域；补充材料还列出道德判断、认知控制、心理物理、跨期选择等任务。
- 包含发展心理学和跨文化研究，参与者元数据包括年龄、性别、国籍、教育、临床诊断及问卷统计等（按原数据可用性提供）。
- 每个贡献数据集按参与者划分测试集：保留 10% 的参与者，最多 100 人；所有论文结果均在 held-out test set 上报告。
- Psych-201 的一部分任务来自旧的 Psych-101；作者另用在 Psych-101 上微调的 **Centaur** 检验行为导向后训练能否迁移到未见任务。

#### Evaluation metrics

- **主指标：平均 negative log-likelihood（NLL）**。只计算 `<<...>>` 标记的人类回答；回答含多个 token 时，先把该回答内各 token 的 log-likelihood 相加，再对回答取平均。NLL 越低，模型越贴近人类回答分布。
- **后训练失配：Cohen's d**。先在每个实验内计算 base model 与对应 post-trained model 的 NLL 效应量，再跨实验平均。图中的正值表示后训练使 human-likeness 下降。
- **补充指标：离散选择准确率**。在 Psych-201 的 discrete-choice 子集上，比较模型预测的人类回答是否正确；用于排除“后训练只是让输出更确定、但众数预测不变”这一解释。
- **个体元数据收益：Cohen's d**。比较同一模型加入/不加入 participant-specific metadata 时的 NLL；正值表示 persona-induction 改善预测。
- 论文没有把传统 benchmark accuracy 当主指标，因为目标不是找规范意义上的正确答案，而是预测人实际会怎么答，包括错误、偏差和随机性。

#### Baselines and comparisons

- **核心 baseline：对应的 base model**。覆盖 Qwen3、Llama3.X、Olmo3.X 三个模型家族，以及 instruction-tuned、reasoning、vision/vision-thinking 等后训练目标和多种参数规模。
- **跨代比较**：Qwen2、Qwen2.5、Qwen3、Qwen3.5 的 base / instruct 配对，用于区分“预训练模型本身是否进步”和“后训练造成的额外偏移”。
- **Persona baseline**：不提供参与者元数据的 prompt，对比 interview-style persona prompt（年龄、性别、国籍、教育、诊断、问卷等）。
- **Prompt-format robustness baseline**：不使用 prompt template（主结果）对比各模型默认 chat template；默认 template 反而普遍更差，Qwen3 差距最大。
- **正向对照 Centaur**：在 Psych-101 子集上做行为数据微调，再到 Psych-201 的未见任务上评估，用于说明“后训练必然降低人类相似度”并非定律，关键在优化目标。

#### Key findings

1. **后训练稳定降低 human-likeness**：instruction-tuned、reasoning、vision 模型相对对应 base model 的平均效应量分别为 `d = 0.11`、`0.14`、`0.07`；几乎所有直接配对都由 base model 获胜。
2. **最像人的模型并不是最新 assistant**：总体最佳是 Qwen2.5-72B base，平均 NLL 为 `1.557`。不过最新 Qwen 代际缺少更大尺寸的 base model，因此不能把这一排名简单解释为“Qwen2.5 一定优于所有更新架构”。
3. **base model 跨代继续改善，但 post-training gap 扩大**：Qwen instruct 相对 base 的平均失配从 Qwen2 的 `d = 0.02`、Qwen2.5 的 `0.04`，增至 Qwen3 的 `0.13`、Qwen3.5 的 `0.16`。
4. **失配是跨领域的，但心理语言学和推理最明显**：七个主分析领域的平均 `d` 约为经济博弈 0.06、分类 0.07、强化学习 0.08、风险选择 0.08、记忆 0.09、推理 0.12、心理语言学 0.18。
5. **Persona-induction 没有改善个体级预测**：元数据收益仅在 `d = -0.02` 到 `0.02` 之间；对发展心理学实验单独分析也没有明显帮助。它可能改变群体层面的输出分布，但不等于能预测某个具体人。
6. **不是单纯的“输出变得更确定”**：在离散选择子集上，post-trained model 的准确率也普遍低于对应 base model。
7. **问题在训练目标，而非 post-training 这一形式本身**：Centaur 在未参与微调的新任务上提升行为对齐，平均 `d = 0.28`、`SEM = 0.10`，说明面向人类行为数据的后训练可以泛化。

#### Interpretation

预训练学习的是人类产生的语言分布，其中自然混合了人的启发式、偏差、错误和不确定性；常见后训练则奖励有帮助、规范正确、可解释或推理更强的答案，因此可能把模型从“描述人会怎么做”推向“规定应该怎么做”。这是一种 behavioral alignment tax：assistant quality 和 behavioral fidelity 不是同一个优化目标，有时甚至相互冲突。

#### Limitations

- 数据规模很大，但来源是研究者公开征集与已有行为实验的集合，不等于对一般人群和所有现实场景的随机抽样；领域、国家、语言及实验范式仍可能存在选择偏差。
- 数据集只做了 lightweight review，主要排除明显格式和实现错误；不同原始研究之间的实验质量、采样方式和元数据完整度并不完全一致。
- 主结论来自三个开放模型家族，不能直接外推到所有闭源模型或所有后训练配方；论文也无法隔离 RLHF、SFT、数据配比等每个具体环节的因果贡献。
- base 与 post-trained 模型通常可以配对，但跨代、跨规模比较并不总是完全对称；例如新一代 Qwen 缺少某些大尺寸 base checkpoint。
- NLL 很适合评估完整行为分布，但也受 tokenizer、答案表述形式和 prompt 序列化影响；论文用 chat-template 和 accuracy 分析做了稳健性检查，但没有消除所有接口层面的影响。
- Persona 实验使用的是简单 interview-style in-context conditioning。结论应表述为“这种常见 persona prompting 未改善个体预测”，而不是“个体信息永远无用”。
- Centaur 的正向结果基于与 Psych-201 有任务谱系关联的 Psych-101 训练数据；虽然测试任务未见过，仍需在来源更独立的数据集上验证迁移。

#### My takeaway

如果目标是模拟消费者、患者、学生或政策对象，不能默认使用最新 instruct/reasoning 模型就是最佳选择。第一步应把 base model 作为强 baseline，并明确产品到底优化“正确、有帮助的回答”还是“忠实复现目标人群的行为分布”。人口统计标签也不能替代个体行为数据：要做到个体预测，更可能需要历史轨迹、状态建模或专门的行为微调，而不是只在 prompt 里写一个 persona。

#### Ideas triggered

- Research idea: 将同一 base model 的 SFT、preference optimization、reasoning RL 等阶段逐一 checkpoint 化，定位 behavioral fidelity 从哪个阶段开始下降，并研究多目标训练的 Pareto frontier。
- Startup idea: 面向市场研究、教育或医疗模拟提供 behavioral-fidelity evaluation layer；同时报告群体分布校准、个体预测、随机性和规范正确性，避免只用通用 benchmark 选模型。
- Mini experiment: 从 Psych-201 选取推理、心理语言学、强化学习各 2 个任务，比较同尺寸 base / instruct / reasoning 模型的 NLL、accuracy、calibration 和响应熵，再测试少量行为示例是否比人口统计 persona 更有效。
