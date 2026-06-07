# References

这个目录放外部参考资料，不是你的主笔记。

## Layout

- `upstream/`: 原样保存外部仓库，尽量保持 read-only 心态。
- `maps/`: 我们自己维护的索引，把外部资料映射到你的课程笔记、概念卡和 labs。

## Rule

不要直接在 `upstream/` 里改代码来做自己的实验。需要动手时，把相关思路或最小代码复制/重写到 `labs/`，并在 lab note 里说明参考来源。

## Current References

- `upstream/agentic_ai_andrew/`: GitHub repo `nhatnam2609/agentic_ai_andrew` as a git submodule.
- `maps/agentic_ai_andrew-map.md`: 这个 upstream repo 的阅读和使用索引。

## New Machine Setup

如果 clone 后发现 `upstream/agentic_ai_andrew/` 是空目录或没有文件，运行：

```bash
git submodule update --init --recursive
```

更新这个外部仓库到远端最新版本：

```bash
git submodule update --remote references/upstream/agentic_ai_andrew
```

更新后记得检查 map 是否还准确：

```bash
git status --short
git submodule status
```
