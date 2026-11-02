# 本地改动登记（`mine` 分支）

本仓库是 [mattpocock/skills](https://github.com/mattpocock/skills) 的 fork。分支约定：

| 分支 | 角色 | 纪律 |
|---|---|---|
| `main` | 上游纯镜像 | **永不提交**，只接受 `git merge --ff-only upstream/main` |
| `mine` | 本地改动 | 一个意图一个 commit，定期 `git rebase upstream/main` |

```bash
# 同步上游
git switch main && git merge --ff-only upstream/main
git switch mine && git rebase upstream/main
```

rebase 后扫一遍本表：**上游自己实现了某条改动，就删掉那一行**。留着没人解释的分叉，比没有改动更糟。

消费侧（各项目的 `.agents/skills`、`.claude/skills`）按需从本分支取，不做自动同步。

## 登记表

| skill | 本地改动 | 为什么 | 上游状态 | 模型情况 | 提交 |
|---|---|---|---|---|---|
| [`implement`](./skills/engineering/implement/SKILL.md) | 收口段由「先 review 后 commit」两段合并为一段：commit（作 diff 固定锚点）→ `code-review` → 终态 commit | `code-review` 按固定点取 diff；未提交的工作树没有锚点，评审面会漂移 | 未解决。上游 `6d6a5b9` 只加了 ticket 拉取，收口顺序仍是 review 在前 | 来源=opencode-go/deepseek-v4.1-flash-1017@pi(reasoning=high)；已验证=无（迁入时只做逐字节核对，未用该模型实跑 `/implement` 收口流程）；失效=无 | `1fc1986` |

## 「模型情况」列的口径

同一句指令在不同模型上的遵守程度不同，所以这一列不写「已用」，写清四件事，缺哪项就明写缺失，不要留空：

1. **来源**：哪个 `provider/model` 提出的改动，跑在哪个 harness，reasoning 档位
2. **已验证**：实际跑过这条改动、并观察到预期行为的模型
3. **失效**：跑过但没照做的模型，附观察到的偏差（例如跳过前置 commit 直接进评审）
4. **未跑过**：直接写「已验证：无」，别让未验证的改动冒充已验证

格式：`来源=<provider/model>@<harness>(reasoning=<level>)；已验证=<...>；失效=<...>`

登记示例：

```markdown
| 来源=opencode-go/deepseek-v4.1-flash@pi(reasoning=high)；已验证=同左；失效=无 |
| 来源=未记录；已验证=无；失效=anthropic/claude-sonnet@claude-code（跳过前置 commit，直接进 code-review） |
```

> 该列只记录「谁验证过这条指令有效」，不构成「该模型更好」的结论。样本一条不算证据，写 deviating 行为时附上当次的可复核痕迹（会话号、日志路径、commit）。
