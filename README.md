# Skills

本目录是本机全局 Cursor skills（`~/.agents/skills`）。

## 来自 GitHub

### [archify](https://github.com/tt-a1i/archify)

把架构、流程、时序、数据流和状态机画成可交互的独立 HTML 图，支持明暗主题和导出；也可以把 Mermaid 转成这种图。

### [i-have-adhd](https://github.com/ayghri/i-have-adhd)

把回复改成适合 ADHD 阅读的写法：先给下一步、多步编号、每轮重述进度。用 `/i-have-adhd` 开启，说 “stop adhd mode” 关闭。

## 自创

### git-commit-message

根据当前 git diff 生成 Conventional Commit 文案。写提交说明、要「提交文案」或「提交说明」时会用到。没有明确要求执行 `git commit` 时，只输出文案，不提交。

格式：

```
<type>(<scope>): <标题>

<正文>
```

- 标题用中文，10～30 字，写清对象和动作；`type`、`scope` 用英文小写。
- `type` 按改动意图选一个：`feat`、`fix`、`refactor`、`perf`、`docs`、`test`、`chore`、`style`、`revert`。
- `scope` 从路径取一个模块短名，例如 `gateway`、`system`、`bpm`、`docker`。
- 正文只写真实改动（类、配置、接口、原错误表现），不写「优化」「完善」这类评价。
