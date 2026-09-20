# Skills

本目录是本机全局 Cursor skills（`~/.agents/skills`）。

需要认证的 CLI：飞书 `lark-cli`，钉钉 `dws`。登录走 `lark-shared` / `dingtalk-shared`。

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

## 飞书（open.feishu.cn / lark-cli）

从 `~/.agents.bak` 拷入。命令走 `lark-cli`。`lark-vc`、`lark-vc-agent`、`lark-minutes`、`lark-note` 实际交给 `lark-meeting`。

### 协作与通讯

| Skill | 做什么 |
| --- | --- |
| `lark-shared` | 登录、鉴权、身份、scope |
| `lark-im` | 发消息、群聊、卡片回调 |
| `lark-contact` | 姓名/邮箱 ↔ open_id |
| `lark-mail` | 邮件读写、搜索、回复 |
| `lark-event` | 实时事件订阅 |

### 文档与文件

| Skill | 做什么 |
| --- | --- |
| `lark-doc` | 云文档读写、图片附件 |
| `lark-drive` | 云盘上传下载、权限、导入 |
| `lark-wiki` | 知识空间与节点 |
| `lark-markdown` | 原生 Markdown 文件 |
| `lark-whiteboard` | 画板导出与编辑 |

### 表格与演示

| Skill | 做什么 |
| --- | --- |
| `lark-sheets` | 电子表格单元格、公式、图表 |
| `lark-base` | 多维表格记录、视图、权限 |
| `lark-slides` | 幻灯片创建与改页 |

### 日程、任务、会议

| Skill | 做什么 |
| --- | --- |
| `lark-calendar` | 日程、会议室、忙闲 |
| `lark-task` | 待办、清单、任务智能体 |
| `lark-meeting` | 视频会议、妙记、会中内容 |
| `lark-approval` | 审批待办/实例/发起 |
| `lark-attendance` | 查自己的打卡记录 |

### 应用、目标与扩展

| Skill | 做什么 |
| --- | --- |
| `lark-apps` | 妙搭应用开发、部署、监控 |
| `lark-okr` | OKR 周期、目标、关键结果 |
| `lark-openapi-explorer` | 查未封装的原生 OpenAPI |
| `lark-skill-maker` | 把飞书 API 封装成自定义 skill |
| `lark-workflow-meeting-summary` | 一段时间内的会议纪要报告 |
| `lark-workflow-standup-report` | 指定日期的日程 + 未完成任务 |

## 钉钉（DWS / `dws`）

从 `~/.agents.bak` 拷入。命令前缀 `dws <产品>`。不在 `.skill-lock.json` 里（bak 也没有登记）。

### 人与消息

| Skill | 做什么 |
| --- | --- |
| `dingtalk-shared` | 泛称钉钉时的入口、鉴权、消歧 |
| `dingtalk-aisearch` | 按姓名/工号/职责搜人、跨源定位 |
| `dingtalk-contact` | 已有 userId 或完整手机号查详情 |
| `dingtalk-chat` | 单聊/群聊、建群、机器人 |
| `dingtalk-mail` | 邮箱收发、搜索、附件 |

### 文档与文件

| Skill | 做什么 |
| --- | --- |
| `dingtalk-doc` | 在线文档读写、评论、导入导出 |
| `dingtalk-drive` | 钉盘/文档空间文件管理 |
| `dingtalk-wiki` | 知识库空间与节点 |
| `dingtalk-aitable` | AI 表格（多维表）增删改查 |

### 日程、待办、听记

| Skill | 做什么 |
| --- | --- |
| `dingtalk-calendar` | 约会议、订会议室、查闲忙 |
| `dingtalk-todo` | 待办创建、指派、完成 |
| `dingtalk-minutes` | AI 听记摘要、逐字稿、行动项 |
| `dingtalk-event` | IM/审批/VoIP/待办长连接监听 |
| `dingtalk-misc` | 审批、考勤、OKR、电子表格、白板等长尾产品 |
