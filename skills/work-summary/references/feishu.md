# 通道 D：飞书 IM 采集

写日报时按本节拉飞书聊天记录。依赖：`lark-cli` + 已安装的 `lark-im` / `lark-shared` skill。

## 前置（首次或失败时）

1. 确认 CLI：`lark-cli --version`
2. **配置应用**（二选一，须用户确认后再执行）：
   - 检测到 Hermes / OpenClaw / Lark Channel：`lark-cli config bind --source hermes --identity user-default`（读个人聊天必须用 **user-default**，不要默认 bot-only）
   - 无 Agent 凭证、要单独建应用：`lark-cli config init --new`（在 Agent 环境里若被拒，需用户明确同意后再加 `--force-init`）
   - 全程按 `lark-shared`：输出里的 URL 须配二维码再交给用户
3. **用户身份授权**（日报读聊天必须用 **user** / `--as user`）：

```bash
lark-cli auth login --domain im --no-wait --json
```

若缺搜索权限，再补：

```bash
lark-cli auth login --scope "search:message" --no-wait --json
```

拿到 `verification_url` 后：`lark-cli auth qrcode --output feishu-auth.png "<url>"`，链接与二维码一起展示；用户回复「已授权」后再执行 `lark-cli auth login --device-code <code>`（同一轮不要阻塞轮询）。

4. 检查：`lark-cli auth status --json --verify`；把 `identities.user.openId` 写入 `feishu-sources.json` 的 `self_open_id`（若为空）

授权失败 / 未配置：跳过本通道，在日报末尾用一句话说明「飞书未授权，仅用 Cursor/Git」，不要中断整篇日报。

## 读配置

路径（按顺序，命中即用）：

1. `E:\工作日记\feishu-sources.json`
2. 本 skill 下 `references/feishu-sources.example.json`（仅作模板，不要当真实源）

配置示例：

```json
{
  "enabled": true,
  "timezone": "+08:00",
  "self_open_id": "",
  "chats": [
    { "name": "项目群名关键词", "chat_id": "", "include_all_senders": false },
    { "name": "某同事私聊", "user_id": "", "include_all_senders": true }
  ],
  "keywords": []
}
```

字段说明：

| 字段 | 含义 |
|------|------|
| `enabled` | `false` 则整通道跳过 |
| `probe_before_fetch` | 默认 `true`：先探活再拉全文（见下节）；`false` 则直接 `--page-all` |
| `self_open_id` | 本人 `ou_xxx`；空则先 `lark-cli auth status` / contact 解析，再尽量用 `--sender` 只收本人发言 |
| `chats[].chat_id` | 群 `oc_xxx`；空则用 `name` 做 `+chat-search` |
| `chats[].user_id` | 私聊对方 `ou_xxx`；与 `chat_id` 二选一（探活/拉消息优先用 `chat_id`） |
| `include_all_senders` | `true`：整段对话作上下文；`false`（默认）：优先本人发言 + @我 |
| `keywords` | 可选；有则额外按关键词搜一轮（不要把「日报/总结」当关键词） |

用户口头指定群名 /「从飞书某某群写日报」时，以口头为准，可临时覆盖配置。

## 采集步骤（目标日 YYYY-MM-DD）

时区默认 `+08:00`。时间窗：

- `--start "YYYY-MM-DDT00:00:00+08:00"`
- `--end "YYYY-MM-DDT23:59:59+08:00"`

### 0. 探活（MUST，除非 `probe_before_fetch: false`）

**目的**：当天无消息的会话不拉全文，省 token / 时间。

对每个已配置会话，只取 **1 条**（不要 `--page-all`）：

```bash
lark-cli im +chat-messages-list --as user --chat-id <oc_xxx> --start "<start>" --end "<end>" --order desc --page-size 1 --format json --no-reactions
```

判定：

- `data.messages` 为空 / `total==0` / 无消息数组 → **跳过**该会话，不进入全文拉取
- 有至少 1 条 → 记入「活跃列表」

可选加速（配置里 `chat_id` 都已知时）：用一次搜索探多个会话是否有本人/@我活动：

```bash
lark-cli im +messages-search --as user --query "" --chat-id "<id1,id2,...>" --start "<start>" --end "<end>" --page-size 20 --format json --no-reactions
```

把返回里出现过的 `chat_id` 标为活跃；**搜索未命中不能证明该群完全无聊天**（可能只有别人说话且未 @我），故对「未出现在搜索结果里的配置会话」仍须用上面的 `page-size 1` 探活，不可直接跳过。

全部配置会话探活后都无消息：飞书通道记「当日无活跃会话」，**不要**再跑策略 B 全库搜索（除非用户明确要求补漏）。

### 策略 A：已配置具体会话（优先）

对每个 chat：

1. 无 `chat_id` 有 `name`：

```bash
lark-cli im +chat-search --query "<name>" --format json
```

取最匹配的 `chat_id`（同名歧义则列给用户选，选不过则跳过该项）。

2. **先探活（节 0）**；仅活跃会话再拉全文：

```bash
# 群 / 已有 chat_id 的私聊
lark-cli im +chat-messages-list --as user --chat-id <oc_xxx> --start "<start>" --end "<end>" --order asc --page-size 50 --page-all --format json --no-reactions

# 仅有 user_id、无 chat_id 时
lark-cli im +chat-messages-list --as user --user-id <ou_xxx> --start "<start>" --end "<end>" --order asc --page-size 50 --page-all --format json --no-reactions
```

禁止对未探活的会话直接 `--page-all`。

### 策略 B：跨会话活动回顾（仅 `chats` 为空，或用户明确要求补漏）

有非空 `chats` 且已完成探活时，**默认不跑**本策略。

不把「看看/总结/日报」塞进 `--query`。用空 query + 时间窗：

```bash
# 本人发出的消息（有 self_open_id 时）
lark-cli im +messages-search --as user --query "" --sender=<self_open_id> --start "<start>" --end "<end>" --exclude-sender-type bot --page-size 50 --page-all --format json --no-reactions

# @我的消息
lark-cli im +messages-search --as user --query "" --is-at-me --start "<start>" --end "<end>" --page-size 50 --page-all --format json --no-reactions
```

有 `keywords` 时对每个关键词再搜一轮（可加 `--chat-id` 收窄）。

详情不足时，用结果里的 `chat_id` / `thread_id` 再调 `+chat-messages-list` / `+threads-messages-list` 补上下文（按需，别无脑拉全库）。
## 抽取规则（写入日报前）

从 JSON 提取：发送者名、时间、文本内容、群名/`chat_name`。跳过：

- `deleted == true`
- bot / 系统消息（除非用户明确要）
- 纯表情、无信息量的「收到/好的/1」
- 个人隐私、薪资、求职、与工作无关闲聊

归类成素材（不要贴原始 JSON / message_id 进对外日报）：

- **已拍板 / 已对齐**：需求结论、口径确认
- **联调 / 排障**：问题现象与处理结论
- **待办 / 未完成**：别人 @我或我承诺未收口的
- **会议 / 同步**：约时间、纪要要点

只把能证明「本人参与」的写成个人产出；别人的活不要写成自己的。与 Cursor/Git 矛盾时，以可核验的 Git / 明确交付为准，飞书只作起因与协作语境。

## 与无产出日

飞书当天只有核对口径、追问配置、复盘已交付 → 仍可判无产出日；把真实聊过的点当「核对」对象，禁止编造新模块。

## 图片 / 附件（默认不读）

日报采集 **禁止** 加 `--download-resources`，也 **禁止** 再调 `+messages-resources-download`。

默认 API 只返回占位，例如 `![Image](img_xxx)` / 文件 key，**不会下载二进制，也不会 OCR / 看图识字**。写日报时：

- 有配文：用配文
- 纯图无文字：可写「群内有截图/附图」，不要臆造图中内容
- 仅当用户明确说「看图 / 识别截图 / 下载附件」时，才对**指定消息**下载并解读

## 安全

- 禁止把 appSecret / accessToken 写入日报或终端复述
- 日报正文不要贴飞书 open_id、chat_id、完整消息原文长贴；用业务说法概括
- 不主动把日报发回飞书，除非用户明确要求
