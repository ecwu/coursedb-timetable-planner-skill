# CourseDB Timetable Planner Skill

这是一个通过 CourseDB 只读 MCP 数据规划学期课程的 Agent Skill。它会读取 Handbook、专业选修、Free Elective 和 Offering，检查 session 时间冲突，并生成有效期为 24 小时的课表预览链接。预览工具不可用时，返回可导入 CourseDB Timetable 的 JSON 计划。

本 Skill 不会修改 CourseDB Planner，也不能代替学校确认先修要求、毕业资格或最终选课结果。

## 仓库结构

这个仓库是 Skill 仓库，真正的 Skill 位于 `plan-semester-courses/` 子目录，而不是仓库根目录：

```text
coursedb-timetable-planner-skill/
├── README.md
├── LICENSE
└── plan-semester-courses/
    ├── SKILL.md
    ├── agents/
    │   └── openai.yaml
    └── references/
        ├── course-code-expansion-map.md
        ├── ge-programme.md
        ├── history-and-exclusions.md
        ├── mcp-tools.md
        ├── planning-examples.md
        ├── planning-rules.md
        ├── program-code-map.md
        └── timetable-json.md
```

## 功能

- 根据专业、入学年份、Handbook 年级和学期读取培养方案要求。
- 查询专业选修和 `FE(...)` Free Elective 候选课程。
- 排除已修、在修或之前已规划的课程，避免重复推荐。
- 补查具体课程的当前学分、双语名称、先修与分类学期。
- 查询指定日历学期的 Offering、session、时间和地点，继续读取被截断的 session。
- 检查所选 session 之间的时间冲突。
- 处理 GE Level 1、Level 2 和 Level 3 分类规则。
- 处理 `CHI1103 -> CHI11038002` 等已知课程编号变体。
- 正确标记通常没有固定上课时间的 FYP 或项目课程。
- 输出可导入 CourseDB Timetable 的 JSON。

## 快速开始

需要准备：

1. Node.js 18 或更高版本，用于运行 [`skills`](https://github.com/vercel-labs/skills) CLI。
2. 支持 Agent Skills 和 MCP `2026-07-28` 协议的客户端。本文使用 OpenCode 2.0.23。
3. CourseDB Developer Access 和一个有效的 Developer API Key。
4. CourseDB 管理后台已开启 MCP 服务。

### 1. 安装 Skill

推荐使用 Vercel Labs 的 `skills` CLI。它可以直接识别本仓库中的 `plan-semester-courses/SKILL.md`，不需要把 `SKILL.md` 移到仓库根目录。

安装到当前项目的 OpenCode：

```bash
npx skills add ecwu/coursedb-timetable-planner-skill \
  --skill plan-semester-courses \
  --agent opencode \
  --yes
```

项目级安装默认写入当前项目的 `.agents/skills/`，适合与项目一起提交和共享。

安装为 OpenCode 全局 Skill：

```bash
npx skills add ecwu/coursedb-timetable-planner-skill \
  --skill plan-semester-courses \
  --agent opencode \
  --global \
  --yes
```

全局安装位置是 `~/.config/opencode/skills/`，可在所有项目中使用。

只查看仓库中可安装的 Skill，不执行安装：

```bash
npx skills add ecwu/coursedb-timetable-planner-skill --list
```

也可以使用完整 GitHub URL：

```bash
npx skills add https://github.com/ecwu/coursedb-timetable-planner-skill \
  --skill plan-semester-courses \
  --agent opencode
```

### 2. 配置 CourseDB MCP

`skills` CLI 只安装 Skill 文件，不会添加 MCP 地址或 API Key。请在项目根目录的 `opencode.json` 中配置 CourseDB MCP，或把相同配置合并到全局的 `~/.config/opencode/opencode.json`。

本文使用 OpenCode V2 配置，已用 OpenCode 2.0.23 验证。OpenCode 默认发送旧版 `initialize` 握手。CourseDB 只接受 `2026-07-28`，因此必须显式设置 `protocol: "2026-07-28"`。

服务器配置放在 `mcp.servers` 下。使用 `oauth: false` 关闭 OAuth 自动鉴权，使用 Developer API Key 鉴权。V2 用 `disabled: false` 启用连接，Skill 搜索目录使用 `skills` 字符串数组。

保留已有的其他配置，加入以下 CourseDB 服务器配置：

```json
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "servers": {
      "coursedb": {
        "type": "remote",
        "url": "https://mis.bnbu.moe/api/mcp",
        "headers": {
          "Authorization": "Bearer {env:COURSEDB_DEV_API_KEY}"
        },
        "protocol": "2026-07-28",
        "oauth": false,
        "disabled": false
      }
    }
  }
}
```

设置环境变量后再启动 OpenCode：

```bash
export COURSEDB_DEV_API_KEY="<YOUR_DEV_API_KEY>"
opencode
```

不要把真实 API Key 写入 `opencode.json`、提交到 GitHub，或放进 URL 查询参数。修改 Skill 或 OpenCode 配置后，需要重启 OpenCode 才会加载新配置。

### 3. 开始规划

在 OpenCode 中输入：

```text
我想规划 2026 Fall 的课程。
我的专业是 CST，2024 年入学，目标是 Handbook Year 3 Term 1。
我已经修完 COMP1001 和 MATH1001，请读取 Handbook 和 Offering，
按专业必修、专业选修、Free Elective、其他课程的顺序规划，并输出 Timetable JSON。
```

为了避免重复选课，首次规划时请提供以下信息：

- 专业名称或准确的 program code。
- 入学年份，即 Handbook cohort year。
- 目标 Handbook study year 和 term。
- Offering 对应的实际日历学期，例如 `2026 Fall`。
- 已修、在修和之前已规划的课程；也可以提供之前导出的 Timetable JSON。

## 不使用 `skills` CLI

如果希望保留仓库的本地 clone，可以让 OpenCode 直接扫描仓库根目录。OpenCode V2 的 `skills` 配置使用字符串数组：

```bash
git clone https://github.com/ecwu/coursedb-timetable-planner-skill.git \
  ~/.config/opencode/coursedb-timetable-planner-skill
```

然后在 `opencode.json` 中加入：

```json
{
  "$schema": "https://opencode.ai/config.json",
  "skills": ["~/.config/opencode/coursedb-timetable-planner-skill"]
}
```

OpenCode 会递归发现其中的 `plan-semester-courses/SKILL.md`。搜索路径指向包含 Skill 子目录的仓库根目录，以保留 `plan-semester-courses` 这个 Skill ID。

如果 clone 就在当前项目中，可以使用相对于 OpenCode 工作目录的路径，例如 `"./coursedb-timetable-planner-skill"`。MCP 配置仍需按上一节单独添加。

## 获取 CourseDB API Key

### 1. 申请 Developer Access

登录 CourseDB 后打开：

```text
https://mis.bnbu.moe/account/developer
```

在 **Developer Platform / 开发者平台** 的申请页填写使用场景并提交。申请理由至少需要 20 个字符，且提交申请要求至少贡献过一条课程评价。建议说明使用的客户端、需要的只读能力和用途。

示例：

```text
I am integrating CourseDB with OpenCode to help students plan a semester using read-only Handbook, elective, and Offering data.
```

### 2. 创建 API Key

管理员审核通过后：

1. 打开 API Key 管理页签。
2. 填写 Key 名称，例如 `opencode-timetable-planner`。
3. 可选填写过期天数，然后创建 Key。
4. 立即保存完整 API Key。

完整 Key 只会展示一次。CourseDB 之后只显示 Key 前缀，不会保存可恢复的明文 Key。

## 验证 MCP 连接

可以用服务发现请求确认 MCP 服务和 API Key 是否可用：

```bash
curl -i -X POST "https://mis.bnbu.moe/api/mcp" \
  -H "Authorization: Bearer ${COURSEDB_DEV_API_KEY}" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -H "MCP-Protocol-Version: 2026-07-28" \
  -H "Mcp-Method: server/discover" \
  --data '{"jsonrpc":"2.0","id":1,"method":"server/discover","params":{"_meta":{"io.modelcontextprotocol/protocolVersion":"2026-07-28","io.modelcontextprotocol/clientCapabilities":{},"io.modelcontextprotocol/clientInfo":{"name":"curl","version":"1.0"}}}}'
```

常见结果：

- `200`：MCP 服务发现成功。
- `401`：API Key 缺失、无效、过期或已撤销。
- `410`：CourseDB MCP 全局开关关闭。
- `503`：Developer API 暂停或关闭。
- 返回 HTML：通常是 URL 错误、路由未部署，或请求被转发到了网页路由。

不要直接在浏览器中打开 MCP URL。MCP 客户端使用带认证信息的 POST JSON-RPC 请求。

### OpenCode 协议错误

如果 OpenCode 返回 `-32022`，且 `requested` 为 `2025-11-25`、`supported` 为 `["2026-07-28"]`，客户端仍在使用默认旧协议。检查 `mcp.servers.coursedb.protocol` 是否为 `"2026-07-28"`。

如果请求仍使用 `initialize`，只修改请求里的版本字符串不能转换协议。新版使用 `server/discover` 和每次请求的协议元数据。OpenCode 会在设置 `protocol` 后生成对应的请求格式。

保存配置后，重启 OpenCode 及其后台服务。在启动 OpenCode 的同一终端设置 `COURSEDB_DEV_API_KEY`。不要把密钥发到聊天或错误报告中。

如果项目配置也定义了 `coursedb`，它会替换全局同名服务器配置。项目配置必须保留完整的 URL、`protocol`、`oauth` 和鉴权头。

在设置密钥的终端运行 `opencode mcp list`，确认 CourseDB 显示为 `connected`。如果安装命令名是 `opencode2`，使用 `opencode2 mcp list`。

## MCP 工具

当前 Skill 使用以下只读工具：

- `get_handbook_term_requirements`
- `list_major_elective_courses`
- `list_free_elective_courses`
- `get_course_offerings`
- `get_course_details`
- `list_course_offering_sessions`

工具只提供 CourseDB 中的事实数据。课程排序、历史课程排除、候选筛选和冲突检查由 Skill 完成。

## Skill 2.0 数据与导出

MCP 使用 `2026-07-28` 协议。请求携带协议版本、客户端能力和匹配的 HTTP 请求头。工具调用还需要 `Mcp-Name`。旧版初始化握手与会话不再支持。

选修目录使用最近一个有分类信息的 Offering 学期。目标学期 session 的分类单独判断；分类冲突或缺失时，需求保持待确认。具体变体使用自己的学分，不能借用基础课程数据。多位讲师按关联行顺序展示已解析名称或原始姓名。

批量 Offering 查询每门课最多返回 100 个 session。若 `sessionsTruncated` 为 true，从最后一条的 `{session,id}` 继续调用分页工具。分页失败、游标重复或数量变化时，保留已读数据并说明分析不完整。

Timetable 导出保留 `version:2` 格式，条目 ID 使用实际 session ID。导出遵守时间、文本和数量限制，不静默截断。项目课与未知时间在正文标记，不能宣称完全无冲突。

## 安全与限制

- MCP 工具是只读的，不会创建或修改 Planner。
- API Key 应使用合理的过期时间，并且只保存在环境变量或安全的密钥管理工具中。
- CourseDB MCP 当前不能自动读取任意学生的私人修课历史，因此用户需要主动提供历史课程。
- 课程是否满足先修要求不能仅凭文本自动确认。
- `NO_RECORD` 只表示 CourseDB 当前没有对应学期记录，不代表学校一定不开课。
- FYP 通常没有固定授课时间；缺少 `timeSlots` 时应标记为项目或导师安排，而不是普通时间冲突。

## 相关文档

- [`skills` CLI](https://github.com/vercel-labs/skills)
- [OpenCode Skills](https://opencode.ai/v2/docs/skills/)
- [OpenCode MCP Servers](https://opencode.ai/v2/docs/mcp-servers/)
- [OpenCode Configuration](https://opencode.ai/v2/docs/config/)
- [Agent Skills specification](https://agentskills.io/specification)
- [Skill instructions](plan-semester-courses/SKILL.md)
- [MCP tool reference](plan-semester-courses/references/mcp-tools.md)
- [Planning rules](plan-semester-courses/references/planning-rules.md)
- [Timetable JSON format](plan-semester-courses/references/timetable-json.md)

详细示例：[规划示例与异常处理](plan-semester-courses/references/planning-examples.md)。

### 临时课表预览

`create_timetable_preview` 接收 `{requestId, name, data}`。`data` 使用原有 Timetable JSON 格式。工具返回预览链接、到期时间和剩余额度。

持链接者无需登录即可查看。登录用户可保存个人副本，随后编辑。预览创建后保留 24 小时。每个 API key 所属用户在滚动 24 小时内最多创建 20 张，所有 key 共用额度。正式副本继续受每人 25 张的限制。

预览工具失败或不可用时，Skill 返回 JSON 供手动导入。完整 MCP 请求上限为 64 KiB，不会静默截断内容。临时内容使用 Redis，数据丢失时链接可能提前失效。

上线前确认 Redis 容量和淘汰策略适合临时业务存储。观察创建、额度拒绝和保存失败的服务日志，不记录正文或链接 token。
