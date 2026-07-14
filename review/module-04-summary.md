# Module 4 Summary

Status: in progress

## Working Thesis

Module 4 的核心不是“多做几个测试”，而是把 agentic workflow 的迭代过程工程化：先做一个安全、可观察的 prototype，读真实 outputs 和 traces，识别 recurring failure modes，再把最值得解决的问题转成 eval、error analysis 和优先级队列。

第一节给出的关键框架是 eval 的二维分类：

- Objective code eval vs LLM-as-a-judge
- With per-example ground truth vs without per-example ground truth

这能帮助我们把“系统好不好”拆成更具体、可实现、可迭代的评估任务。

## Lessons Covered

- [x] Evaluations (evals)
- [x] Error analysis and prioritizing next steps
- [x] More error analysis examples
- [x] Component-level evaluations
- [ ] Ungraded Lab: Adding a component-level eval to the research workflow
- [x] How to address problems you identify
- [x] Latency, cost optimization
- [x] Development process summary

## Ideas To Revisit

- Quick prototype 的价值在于暴露真实 failure mode。
- Eval 应该从错误观察中长出来，而不是先凭空设计。
- 小 eval set 可以先启动迭代，之后再随系统演进扩充。
- End-to-end eval 看最终输出，component-level eval 帮助定位是哪一步坏了。
- Objective eval 优先用于格式、长度、日期、schema、SQL 结果等明确标准。
- LLM-as-a-judge 适合语义覆盖、文章质量、图表清晰度等主观标准。
- Per-example ground truth 适合每个输入都有不同正确答案的任务。
- 没有 per-example ground truth 时，也可以用统一约束或统一 rubric 做 eval。
- Eval metric 要持续和 human/expert judgment 对齐。
- Error analysis 要聚焦失败样例，读 traces/spans，而不是只看 final answer。
- 同一个最终错误可能由 search terms、search results、source selection、reading/synthesis 或 final writing 等不同 component 造成。
- 判断 component 是否出错时，要考虑它收到的 input quality；不要让下游替上游背锅。
- Prioritization = 错误频率 + 可改进性。错得多但没思路的组件，不一定马上排第一。
- Invoice due date 错误要拆开看：是 PDF-to-text 把文本搞坏了，还是 LLM 在清楚文本中抽错了日期。
- Customer email 错误要拆开看：是 query generation 错、database 数据错，还是 final email drafting 差。
- Error tags 不必互斥；一个失败样例可以同时标多个 component。
- Error analysis 的自然下一步是 component-level eval：既然知道哪个 component 最常错，就要单独评估和优化它。
- Component-level eval 单独评估一个 component，适合在调 prompt、换工具、换模型、调参数时快速比较。
- Web search component 可以用 gold standard web resources 和信息检索指标来评估。
- Component-level eval 的局部提升需要最后用 end-to-end eval 验证，避免局部指标和整体质量脱节。
- 找到问题 component 后，修法要先按 component 类型分流：non-LLM component 通常调 hyperparameters 或替换组件；LLM component 通常改 prompt、换模型、拆步骤或 fine-tune。
- Prompt 修法适合模型大体会做、但 instruction、格式、边界或示例不够清楚的情况。
- Model selection 适合 instruction following、reasoning、extraction accuracy 或 long-context handling 明显拖后腿的情况；不同 workflow step 可以用不同模型。
- Split step 能把一个难 LLM call 拆成更容易评估的小组件，但会增加 latency、cost 和编排复杂度。
- Fine-tuning 应该建立在稳定任务分布、足够数据和 eval set 之上，不应该成为第一反应。
- PII redaction 例子说明：模型不只是“聪明程度”不同，遵守格式、识别敏感信息和不漏项的能力也会明显不同。
- Latency/cost 优化通常排在 output quality 之后：先让系统真的有用，再让它更快、更便宜。
- 优化前要做 per-step timing 和 per-step cost breakdown，找到最大 contributor。
- Latency 可以通过 parallelism、更快 provider、更小模型、减少上下文或减少工具调用来优化。
- Cost 可以通过 token reduction、cheaper model routing、cache、减少昂贵 API 调用或替换 provider 来优化。
- 每次为了 latency/cost 换组件，都要保留 quality eval，避免把系统优化到“不好用但便宜”。
- Agentic workflow 开发不是线性的，而是在 build 和 analyze 之间来回切换。
- 初期快速做 end-to-end prototype，手动读 outputs/traces；成熟后加入小 eval set、error analysis、component-level evals 和 production monitoring。
- Analysis 本身就是进展：它决定下一轮 build effort 应该放在哪里。
- 经验不足的团队常常 build 很多、analyze 很少，因此容易在不重要的 component 上浪费时间。
- 通用 monitoring/logging/cost 工具有帮助，但 custom evals 才能捕捉自己应用里的真实 failure modes。

## Self-Test Questions

- 为什么 agentic workflow 开发通常要先做 quick prototype？
- Eval 为什么应该从真实 outputs/traces 中发现的 failure mode 出发？
- End-to-end eval 和 component-level eval 的区别是什么？
- Objective eval 适合哪些标准？
- LLM-as-a-judge 适合哪些标准？
- 什么是 per-example ground truth？
- Invoice due date extraction 为什么属于 objective eval + per-example ground truth？
- Marketing caption length 为什么不需要 per-example ground truth？
- Research agent key points 为什么适合 LLM-as-a-judge？
- Eval set 一开始只有 10 到 20 个样例有什么价值和限制？
- 如果 eval 指标和人工判断不一致，应该如何处理？
- Trace 和 span 分别是什么？
- 为什么 error analysis 应该聚焦失败样例？
- 为什么不能直接把 final output 的错误归咎于 final writing prompt？
- 如果 search results 很差，但 source selection 也没选出好 sources，应该如何判断哪个 component 负责？
- 选择下一步优化目标时，为什么要同时考虑错误频率和修复可行性？
- Invoice processing 中，如何区分 PDF-to-text 错误和 LLM extraction 错误？
- Customer email workflow 中，query generation、database quality、email drafting 分别可能怎么失败？
- 为什么 error percentages 可能加起来超过 100%？
- Error analysis 如何自然引出 component-level eval？
- 为什么只用 end-to-end eval 调单个 component 会低效？
- Component-level eval 如何减少其他组件随机性带来的噪声？
- Web search component 的 gold standard resources 可以如何标注？
- 为什么 component-level eval 之后仍然要跑 end-to-end eval？
- 发现问题 component 后，为什么不能默认先改 prompt？
- Non-LLM component 有哪些常见可调参数？
- LLM component 的四类常见修法是什么？
- 什么情况下应该先 try a new model，而不是继续 prompt engineering？
- 为什么 split step 既能提升质量，也可能带来 latency/cost 问题？
- Fine-tuning 前为什么必须先有 baseline 和 eval set？
- 对数据分析 agent，schema retrieval、SQL generation、chart spec 和 final explanation 为什么可能适合不同模型？
- 为什么 cost/latency optimization 通常不应该早于 output quality？
- Per-step timing breakdown 可以如何帮助定位 latency 瓶颈？
- Per-step cost breakdown 应该包含哪些成本来源？
- Parallelism 适合优化哪些 workflow steps？为什么不是所有 step 都能并行？
- 换小模型或便宜模型后，为什么还要跑 component-level eval？
- Build 和 analyze 在 agentic workflow 开发中分别指什么？
- 为什么 quick and dirty end-to-end prototype 反而是有价值的？
- 小 eval set 通常什么时候引入？
- Error analysis 如何帮助决定下一轮 build effort？
- 为什么通用 monitoring tools 不能替代 custom evals？
