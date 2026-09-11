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

### Unveiling the Capabilities of Large Language Models in Simulating Student Behavioral Dynamics and Supporting Peer Feedback to Augment Task Performance

- Direction: LLM student simulation / learning analytics / virtual peers / metacognition
- Authors: Songlin Xu, Xinyu Zhang
- Venue: CHI 2026
- Link: https://doi.org/10.1145/3772318.3790346
- Code: https://github.com/songlinxu/EduTwin
- Status: 已读

#### Problem

已有 LLM 学生模拟研究大多只预测最终答案是否正确，较少覆盖学生学习过程中的连续变化，例如历史表现、实时理解、先验知识、参与度、眼动和认知状态，也缺乏对“什么信息让 LLM 模拟成功或失败”的解释。论文进一步追问：从这些能力探测实验中得到的规律，能否用于设计更好的学生模拟器，并让模拟学生作为 virtual peer 改善真实学生的任务表现？

#### Core idea

论文包含三个连续模块：

1. 使用多个真实教育数据集设计六组 simulation experiments，通过输入信息消融，探测 LLM 在何种条件下能复现学生行为动态。
2. 根据前述实验归纳的规律，提出双层 **Meta-Cognitive Refinement（MCR）**，模拟“学生如何监控自己的认知过程”，并让 LLM 再检查自己的模拟推理。
3. 在 188 人用户实验中，将 MCR 生成的答题速度作为 virtual peer progress feedback，检验其能否产生类似真实同伴的促进作用。

#### Related work positioning

- 相比 Classroom Simulacra、EduAgent、PeerGPT 等工作，本文不只模拟问答正确性或对话，而是研究 slide-level understanding、gaze、cognitive states、item-level performance 和 response-time trajectory。
- 相比把 LLM 当 tutor 直接讲解，本文把它当作可控制的 peer simulator，通过进度线索产生 social facilitation、social comparison 和 accountability。
- 方法上不把单一预测分数当作全部结果，而是通过输入消融和 embedding-space 分析解释不同信息的作用。

#### Simulation experiments, datasets, and findings

##### Experiment 1: Demographics → final grade

- 数据：145 名学生，28 类人口统计及学习习惯因素；模拟重复 4 次，加上真实数据共 725 个样本。
- 任务：GPT-3.5-Turbo 根据学校、家庭、父母教育、课外活动、记笔记和听课习惯等信息，预测 0–7 的最终成绩等级。
- 指标：比较各人口因素变化时，模拟成绩趋势与真实成绩趋势的 Pearson correlation。
- 结果：记笔记 `r=0.94`、交通方式 `r=0.80`、居住条件 `r=0.65`、课堂听讲 `r=0.63`；部分因素如是否工作并不一致。
- 解释：LLM 的预训练知识包含人口因素与学业表现的常识关系；embedding 降维也显示语义相关因素形成聚类。但这种能力可能反映模型内置刻板印象，而非对具体学生的真实理解。

##### Experiment 2: Assessment history → final exam score

- 数据：Open University Learning Analytics Dataset，4,524 名学生，每人至少 5 次历史 assessment；包括 8 种模拟配置与真实数据，共 40,716 个样本。
- 消融：只用 demographics、只用过去成绩、两者结合，以及只输入过去 1–5 次 assessment。
- 指标：模拟与真实期末成绩的 Pearson correlation，并比较分布和不同地区、IMD band 下的趋势。
- 结果：只用学习历史 `r=0.69`，历史＋demographics `r=0.68`，只用 demographics 仅 `r=0.03`。输入更多历史 assessment 会持续提高表现；LLM 同时利用历史平均水平和上升/下降趋势。
- 结论：与宽泛的人口标签相比，个人近期行为历史是更直接、更可靠的模拟依据。

##### Experiment 3: Course experience → understanding trajectory

- 数据：27 名学生观看两段课程视频，分别含 10 和 6 个 slide；数据包括 eye tracking、pre-test 和 post-test。
- 任务：使用 pupil size 作为认知负荷及理解水平的代理，模拟每个 slide 后的理解变化。
- 消融：一次输入全部材料、只输入当前 slide、当前 slide＋过去理解历史。
- 结果：加入理解历史达到 `r=0.45`，而全部材料和只看当前 slide 分别为 `r=0.02`、`-0.01`。随着可用历史增加，最后一个 slide 的相关性在两门课程中分别达到 `0.89` 和 `0.86`。
- 结论：课程内容本身不足以区分学生；过去理解轨迹使模拟更准确，也让输出分布更接近真实学生的连续性和多样性。

##### Experiment 4: Contextual knowledge → item-level post-test outcome

- 数据：沿用 Experiment 3 的 27 人课程数据。
- 消融：整体预测所有题、输入全部材料后逐题预测、只为每道题输入相关材料和对应理解水平。
- 指标：模拟与真实逐题正确性的 Pearson correlation。
- 结果：整体预测 `r=-0.001`，全部材料逐题预测 `r=-0.293`，精确匹配题目与上下文后提高至 `r=0.367`；此时模拟结果与相关理解水平的关系达到 `r=0.812`。
- 结论：更多 context 不必然更好；相关上下文的精确选择比把全部课程材料塞进 prompt 更重要。

##### Experiment 5: Prior knowledge and engagement

- 数据：仍沿用 27 人课程数据；pre-test 表示 prior knowledge，gaze inter-subject correlation 表示 engagement。
- 结果：最佳 contextual model 从 `r=0.367` 提升到加入 pre-test 后的 `0.539`，再加入 engagement 后为 `0.545`。
- 结论：先验知识带来主要增益，engagement 只产生很小的额外提升；两者都能让不同学生的预测模式更加多样，并更符合真实行为关系。

##### Experiment 6: LLM embeddings augment machine learning

- 数据：EduAgent 的 311 名学生，包含 gaze、workload、curiosity、valid focus、course following 和 post-test skill。
- 任务：使用 Decision Tree、Linear Regression、Random Forest、Transformer、RNN 预测 gaze AOI、认知状态和答题正确性。
- Baseline：文本的简单 numeric encoding；对比 BERT embedding 与 OpenAI `text-embedding-3-small`。
- 指标：先计算 gaze、认知状态、skill 之间的 behavioral correlation，再计算模拟相关结构与真人相关结构之间的 Pearson correlation。
- 结果：numeric encoding 为 `0.658–0.819`，BERT 为 `0.751–0.857`，OpenAI embedding 为 `0.798–0.902`，后者整体最好。
- 结论：LLM 不一定要直接输出模拟结果；作为语义编码器，也能增强传统机器学习模型对复杂行为关系的建模。

#### Evaluation methodology

- Experiments 1–6 的直接模拟主要使用 GPT-3.5-Turbo，temperature 固定为 0，以增强确定性和可复现性。
- 主要指标是 Pearson correlation，因为作者认为人类行为具有随机性，复现随刺激变化的相对轨迹比逐样本精确预测更符合这些探索实验的目标。
- 主要 baseline 是输入信息的 ablation，而不是其他 prompting 或模型方法；因此这些实验回答的是“哪些信息重要”，而不是“GPT-3.5 是否优于其他模型”。
- MCR 模拟阶段改用 GPT-4o mini，并使用 MAE、trial-level correlation 和 Linear Mixed-Effects Model 进行方法比较。
- 用户实验对 calibration 到正式实验的 accuracy 和 response time 计算相对变化，再用包含 participant random intercept 和 trial variance component 的 Linear Mixed-Effects Model 做组间检验。

#### Meta-Cognitive Refinement

MCR 根据一名真实参与者 5 个 calibration trials 的 response time，模拟其后续 50 个正式 trials。它包含两层：

- **Student-level metacognition**：结合历史表现、人口信息、题目难度和上下文，推理该学生如何监控进度、理解困难并调整策略。
- **Model-level metacognition**：LLM 回看第一轮推理，检查输入、任务难度和预测结果是否一致，再修正不合理的 response-time trajectory。

对比 baseline 包括 Standard prompt、Chain-of-Thought、Program-of-Thought 和 Self-Refine。个体×trial 层面的 MAE 分别为：

- MCR：`3.225`
- Standard：`3.363`
- PoT：`3.378`
- CoT：`3.412`
- Self-Refine：`3.803`

所有 baseline 都显著差于 MCR。在按 trial 聚合的平均 response-time trajectory 上，MCR 达到 `MAE=0.840`、`r=0.375, p=.007`。一个重要观察是：为数学解题设计的 CoT、PoT 和 Self-Refine 并不会自动改善“模拟学生如何解题”，task solving 与 behavior simulation 是不同目标。

#### Virtual peer user study

- 样本：初始招募 191 人，排除 3 人后 `N=188`；Control 46、Random Peer 45、Real Peer 49、Virtual Peer 48，共 10,340 个 trials。
- 设计：四组 between-subjects experiment；每人先完成 5 个 calibration trials，再完成相同顺序的 50 个正式算术判断题。
- Control 只有自己的进度条；Real Peer 使用 Control 组真实平均速度；Virtual Peer 使用 MCR 模拟速度；Random Peer 从匹配 Control 组均值、标准差和范围的随机分布取速度。
- 指标：accuracy 和 response time；任务要求优先保证准确率，再尽快回答。

主要结果：

- 三种 peer treatment 相对 Control 都没有显著改变 accuracy。
- Virtual Peer 的 accuracy 高于 Real Peer：`β=0.160, p=.045`，但属于边缘性的单个组间差异。
- Virtual Peer 比 Control 更快：`β=0.182, p<.001`。
- Virtual Peer 比 Random Peer 更快：`β=0.082, p=.043`。
- Virtual Peer 与 Real Peer 的速度差异不显著：`β=0.053, p=.186`。
- Real Peer 比 Control 更快：`β=0.129, p=.001`；Random Peer 也比 Control 更快：`β=0.100, p=.015`。

最稳妥的解释是：单纯时间压力就能加速表现，但依据合理任务难度与群体趋势生成的 virtual peer 比随机压力更有效；其效果与 real peer 相当，而不是被证明显著优于 real peer。

#### Key findings

1. LLM 能复现某些群体层面的教育规律，但 demographics simulation 很可能混合了预训练常识与刻板印象。
2. **个人行为历史通常比人口统计标签更有预测价值**，更多历史还能帮助模型识别水平和趋势。
3. **局部、任务相关的 contextual information 比未经筛选的长 context 更有效**；把全部材料输入可能反而降低表现。
4. Prior knowledge 带来明显增益，engagement 的边际增益较小。
5. LLM embedding 能帮助传统模型复现 gaze、认知状态和学习结果之间的相关结构。
6. MCR 优于通用 CoT、PoT 和 Self-Refine，说明学生模拟需要专门针对行为动态设计推理过程。
7. Virtual peer feedback 可以加快真实参与者的解题速度且不明显损害准确率，效果与真实同伴相当。

#### Limitations

- 六组能力探测的直接模拟主要只使用 GPT-3.5-Turbo，MCR 只使用 GPT-4o mini，缺少跨模型和开源模型复现。
- Experiments 3–5 只有 27 名学生，大量相关系数可能对少量个体和异常值敏感；论文的多重比较控制也不突出。
- Experiment 5 中 engagement 只使相关性从 `0.539` 增至 `0.545`，实际增益很小。
- Pearson correlation 衡量趋势一致性，但不能证明预测在数值上准确，也可能掩盖系统性的尺度偏差。
- Temperature 设为 0 有利于复现，却没有检验模拟是否能再现人类行为的随机性和个体内变异。
- 对 demographics 的模拟可能复制社会刻板印象；若用这种模拟器优化教学策略，可能把模型偏见固化到教育系统中。
- 用户实验只验证简单算术任务和进度条式 peer pressure，尚不能外推到写作、编程、概念学习或长期教育结果。
- Virtual Peer 并未在 response time 上显著优于 Real Peer；论文支持的是“效果相当”，而非“AI 同伴优于真人”。
- 参与者被告知进度代表其他参与者，但 Virtual/Random 条件的数据并非直接来自真人，实际部署时需要考虑透明度与实验欺瞒问题。

#### My takeaway

这篇论文最有价值的不是笼统地证明“LLM 可以模拟学生”，而是给出了一个具有工程意义的信息优先级：

> 个人行为历史 > 与当前任务精确匹配的上下文 > 先验知识 > 参与度 > 泛化人口统计信息。

学生模拟器不应主要靠 persona 标签，而应围绕 longitudinal behavior、local task context 和个体状态轨迹构建。另一个重要提醒是：一个模型越擅长把题做对，并不意味着它越能模拟真实学生的解题时间、错误和策略变化。

#### Ideas triggered

- Research idea: 在同一数据划分上比较 GPT、开源 instruct/base model、时序模型与认知模型，分别评估数值误差、轨迹相关、分布校准和个体内随机性。
- Research idea: 将 MCR 拆成 student-level 与 model-level 两层做严格消融，确认增益来自元认知结构，而不是更长 prompt 或额外推理 token。
- Startup idea: 为在线学习平台构建基于历史行为和当前题目上下文的 virtual cohort，提供可调强度的同伴节奏、鼓励和挑战，但避免用敏感人口属性直接推断能力。
- Mini experiment: 选一个公开知识追踪数据集，以 past attempts、demographics、retrieved local context 三类输入逐步消融，同时比较 Pearson `r`、MAE、calibration 和跨学生泛化。
