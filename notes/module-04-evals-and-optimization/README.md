# Module 4: Evals and Optimization

## Lesson Checklist

- [x] 01 - Evaluations (evals)
- [x] 02 - Error analysis and prioritizing next steps
- [x] 03 - More error analysis examples
- [x] 04 - Component-level evaluations
- [ ] 05 - Ungraded Lab: Adding a component-level eval to the research workflow
- [ ] 06 - How to address problems you identify
- [ ] 07 - Latency, cost optimization
- [ ] 08 - Development process summary
- [ ] 09 - Module 4 quiz

## Module-Level Themes

- Agentic workflow 的改进通常从 quick prototype 开始，而不是先花很久理论化设计。
- Evals 的第一步不是追求完整 benchmark，而是先看真实 outputs/traces，找 recurring failure mode。
- End-to-end eval 评估整个系统从 input 到 final output 的表现。
- Objective eval 适合规则清楚、能用代码判断的质量标准。
- LLM-as-a-judge 适合更主观、表达方式多样的质量标准，但需要 rubric 和人工抽查。
- Eval set 可以一开始很小，比如 10 到 20 个样例；随着系统迭代再补覆盖面。
- Per-example ground truth 适合每个输入都有不同正确答案的任务，比如 invoice due date 或 research article key points。
- 没有 per-example ground truth 的 eval 也有价值，比如统一的 word-count limit 或通用 chart rubric。
- 好的 eval 应该反映 human/expert judgment，而不是只让一个容易优化但无业务意义的指标变高。
- 后续 error analysis 的重点是：在多组件 agentic system 里，找到最值得优先改的部分。
- Trace 是一次运行的完整中间记录；span 是其中某个 component/step 的输出。
- Prioritization 不只看哪个 component 错得多，也看有没有现实可行、成本可控的改进方案。
- Error analysis 的错误标签不一定互斥：一个失败样例可以同时暴露 query generation、database quality 和 final drafting 问题。
- Invoice processing 可以拆成 PDF-to-text 和 LLM extraction；customer email workflow 可以拆成 query generation、database quality 和 email drafting。
- Component-level eval 在单个组件上建立更快、更清晰的 signal，用于调参、换工具或改 prompt。
- Component-level eval 不能替代 end-to-end eval；局部优化后仍要验证整体 workflow 是否变好。

## Notes

- [01 - Evaluations (evals)](01-evaluations-evals.md)
- [02 - Error analysis and prioritizing next steps](02-error-analysis-and-prioritizing-next-steps.md)
- [03 - More error analysis examples](03-more-error-analysis-examples.md)
- [04 - Component-level evaluations](04-component-level-evaluations.md)

## Cross-Cutting Notes

- [Agentic evals](../../concepts/agentic-evals.md): eval taxonomy, objective eval, LLM-as-a-judge, end-to-end/component-level evals.
- [Error analysis](../../concepts/error-analysis.md): traces, spans, component attribution, prioritization.
- [Component-level eval](../../concepts/component-level-eval.md): local metrics for specific workflow components.
- [Open questions](../../review/open-questions.md): eval set size, judge reliability, metric vs human judgment.
