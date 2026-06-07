# Labs

这个目录用来把课程里的概念变成可观察的实验。

每个 lab 最好只回答一个问题，比如：

- Does reflection improve the output?
- Does tool choice improve factuality?
- How much autonomy is useful for a research workflow?
- What eval catches a bad intermediate action?

## Naming

Use a numbered folder:

```text
labs/
  001-research-agent/
    README.md
    prompts.md
    experiments.md
    results.md
```

## Lab Rule

不要只记录 final answer。要记录 workflow decisions、tool calls、失败案例和 evaluation criteria。

## Using External References

如果 lab 参考了 `references/upstream/` 里的外部代码，不要直接改 upstream。建议流程：

1. 在 `references/maps/` 找到对应索引。
2. 阅读 upstream 的 README 和相关代码。
3. 在 `labs/` 下新建自己的实验目录。
4. 复制最小必要思路或重写最小版本。
5. 在 lab note 里写清楚 source inspiration 和你自己的改动。

推荐的第一个实验：

```text
labs/001-research-agent-minimal/
```

参考来源：

```text
references/upstream/agentic_ai_andrew/Module1/agentic-ai-public/
```
