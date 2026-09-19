---
name: git-commit-message
description: >-
  Generate concise Conventional Commit messages from git diffs: type(scope)
  title plus a factual body. Use when writing or generating git commit messages,
  git commit copy, 提交文案, 提交说明, or when the user asks to commit and needs a message.
---

# Git 提交文案

根据当前改动生成提交说明。只写事实，不写评价。

未明确要求执行 `git commit` 时，只输出文案，不提交。

## 工作流

1. 并行读取：`git status`、`git diff`（含 staged/unstaged）、必要时 `git diff --cached`、`git log -8 --oneline`。
2. 按路径判断 **type** 和 **scope**，归纳真实改动。
3. 按下方格式输出完整文案。用户要求提交时，用该文案作为 commit message。

## 格式

```
<type>(<scope>): <标题>

<正文>
```

- 第一行：`type(scope):` 后接一句标题，无句号、无空话。
- 空一行后写正文：具体改了什么、为何改（仅当原因不是显而易见时）。
- 标题与正文用中文；`type`、`scope` 用英文小写。
- 不要把标题再重复一遍当正文。

## type

只选一个，以改动意图为准：

| type | 何时用 |
|------|--------|
| `feat` | 新行为、新接口、新配置项 |
| `fix` | 修缺陷、错误处理、错误逻辑 |
| `refactor` | 不改对外行为的结构调整 |
| `perf` | 有可核对手段的性能改动（耗时、查询次数、内存），禁止空喊 |
| `docs` | 仅文档 |
| `test` | 仅测试 |
| `chore` | 构建、依赖、脚本、非功能杂项 |
| `style` | 仅格式，无逻辑变化 |
| `revert` | 回滚 |

混合改动：以主要意图为准。修 bug 顺带小清理 → `fix`，不要写成 `refactor`。

## scope

从路径取**一个**模块短名，与用户一眼能对应的目录一致：

| 路径线索 | scope |
|----------|--------|
| `yudao-gateway` | `gateway` |
| `yudao-module-<name>` | `<name>`（如 `system`、`bpm`、`wms`、`pay`、`infra`） |
| mall 子模块 | `product` / `trade` / `promotion` / `statistics`，不要笼统 `mall`（除非跨子模块） |
| `yudao-framework/...-mybatis` 等 starter | starter 名：`mybatis`、`security`、`web`、`redis`、`mq` |
| `yudao-framework/yudao-common` 或跨多个 starter | `framework` |
| `yudao-dependencies` | `deps` |
| `script/docker`、compose | `docker` |
| `sql/` | `sql` |

多模块且无法归到一个 scope：用改动最大的模块；同等量级再用逗号，如 `feat(gateway,system)`。不要用 `misc`、`all`、`update`。

## 标题

- 10～30 个汉字为宜，说清**对象 + 动作**。
- 动词具体：`补齐 CORS 预检`、`纠正 Token 校验跳过条件`，不要 `优化网关`、`调整代码`。
- 禁止：提升性能、优化体验、完善功能、代码优化、增强稳定性、改进可读性、小幅调整，以及英文空话（improve performance、cleanup 等）。
- `perf` 标题必须带可感知点：如 `减少 N+1 查询`、`避免重复反序列化`。

## 正文

用短句或 `-` 列表，每条对应一处真实改动：

- 写行为变化、接口/配置/SQL/过滤器等具体点。
- 写清关键类、配置项、接口路径（有助于对照 diff）。
- 修 bug：写原错误表现或错误条件，不要只写「修复问题」。
- 不写「顺便」「整体」「进一步」等填充词。
- 无关文件（格式化、误改）不要写进正文；必要时提醒用户不要提交。

## 正反例

**好：**

```
feat(gateway): 按请求头做灰度实例选择

- GrayLoadBalancer 按 version 头过滤服务实例，无匹配时回退普通轮询
- 接入 GrayReactiveLoadBalancerClientFilter，替换默认负载均衡过滤器
```

```
fix(system): 纠正社交账号 social 拼写

- 枚举、表字段注释与接口路径由 socail 改为 social
```

**坏：**

```
feat(gateway): 优化网关性能并完善灰度功能
本次改动提升了系统稳定性和代码质量。
```

```
fix: 修复问题
```
