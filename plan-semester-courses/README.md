# CourseDB Timetable Planner Skill

这是一个用于规划学生学期课程的 Agent Skill。它通过 CourseDB 的只读 MCP 工具读取 Handbook、专业选修、Free Elective 和 Offering，结合学生历史课程或之前的 Timetable 计划排除重复课程，再按“专业必修 → 专业选修 → Free Elective → 其他课程”的顺序生成一个暂定课程建议。只有用户明确要求预览或导出时，才生成有效期为 24 小时的课表预览链接。预览工具不可用时，输出可被 CourseDB Timetable 导入的 JSON。

仓库：[ecwu/coursedb-timetable-planner-skill](https://github.com/ecwu/coursedb-timetable-planner-skill)

本 Skill 不会修改 CourseDB 中的 Planner，也不会代替学校确认先修要求、毕业资格或最终选课结果。

## 功能

- 根据专业代码、入学年份、年级和目标学期读取 Handbook 要求。
- 从 Handbook 绑定的课程清单读取专业候选课程。
- 按 `FE(ALL)` 或 `FE(SAI)` 等精确课程类型查询 Free Elective 课程池。
- 根据已修课程、已选课程和之前的 Timetable 计划排除重复课程。
- 补查具体课程的当前学分、先修、互斥和分类学期。
- 查询课程在指定学年学期的 Offering 和 session，并继续读取被截断的 session。
- 检查已选 session 的时间冲突。
- 支持 GE 课程的 Level 1、Level 2、Level 3 规则。
- 处理 Handbook 固定课程的已知课程编号变体，例如 `CHI1103 → CHI11038002`。
- 识别 FYP（Final Year Project）通常没有固定授课时间的情况。

## 目录结构

```text
.
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

## 使用前提

需要准备：

1. 支持 Agent Skills 和 MCP `2026-07-28` 的客户端。本文使用 OpenCode 2.0.23。
2. CourseDB 的 `/api/mcp` 地址。
3. 一个有效的 CourseDB Developer API Key。
4. CourseDB 管理后台已开启 MCP 全局开关。

MCP 的查课工具提供只读操作，目前包括：

- `list_course_lists`
- `list_course_list_courses`
- `get_handbook_term_requirements`
- `list_free_elective_courses`
- `get_course_offerings`
- `get_course_details`
- `list_course_offering_sessions`

MCP 当前不会自动读取任意学生的已修课程或个人 Planner。规划包含选修课时，可以提供已修课程清单、之前的学期计划，或从 CourseDB 导出的 Timetable JSON，供 Skill 排除重复课程。未提供历史时，Skill 仍可给出暂定选修课建议，同时说明重复课程排除尚未核实。只有用户明确要求预览或导出时，才创建完整预览。

## 安装 Skill

### 方式一：克隆 GitHub 仓库

```bash
git clone https://github.com/ecwu/coursedb-timetable-planner-skill.git \
  ~/.config/opencode/coursedb-timetable-planner-skill
```

独立仓库中的 Skill 位于 `plan-semester-courses/` 子目录。让 OpenCode 扫描该子目录或包含它的仓库目录。

### 方式二：本地项目目录

如果 Skill 位于当前项目的 `skills/plan-semester-courses`，可以让 OpenCode 指向：

```text
./skills
```

具体配置见下一节。

## OpenCode 配置

OpenCode 对远程 HTTP MCP 使用 `"type": "remote"`。远程 MCP 的 HTTP 传输由 OpenCode 自动处理。

本文使用 OpenCode V2 配置，已用 OpenCode 2.0.23 验证。OpenCode 默认发送旧版 `initialize` 握手。CourseDB 只接受 `2026-07-28`，因此必须显式设置 `protocol: "2026-07-28"`。

服务器配置放在 `mcp.servers` 下。使用 `oauth: false` 关闭 OAuth 自动鉴权，使用 Developer API Key 鉴权。V2 用 `disabled: false` 启用连接，Skill 搜索目录使用 `skills` 字符串数组。

在项目根目录创建 `opencode.json`，或者修改 OpenCode 的全局配置：

```json
{
  "$schema": "https://opencode.ai/config.json",
  "skills": ["~/.config/opencode/coursedb-timetable-planner-skill"],
  "mcp": {
    "servers": {
      "coursedb": {
        "type": "remote",
        "url": "https://<COURSEDB_HOST>/api/mcp",
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

如果 Skill 是当前项目中的目录，使用：

```json
{
  "$schema": "https://opencode.ai/config.json",
  "skills": ["./skills"],
  "mcp": {
    "servers": {
      "coursedb": {
        "type": "remote",
        "url": "https://<COURSEDB_HOST>/api/mcp",
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

OpenCode 的配置支持 `{env:VARIABLE_NAME}` 环境变量引用。不要把真实 API Key 写进 `opencode.json`，也不要提交到 GitHub。

启动 OpenCode 前设置环境变量：

```bash
export COURSEDB_DEV_API_KEY="<YOUR_DEV_API_KEY>"
opencode
```

本地 CourseDB 开发环境示例：

```json
{
  "$schema": "https://opencode.ai/config.json",
  "skills": ["./skills"],
  "mcp": {
    "servers": {
      "coursedb": {
        "type": "remote",
        "url": "http://localhost:3000/api/mcp",
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

本地环境还需要先启动 CourseDB：

```bash
pnpm dev
```

不要直接在浏览器中打开 MCP URL。MCP 客户端需要使用 POST JSON-RPC 请求，并在每次请求中携带 API Key。

## 在 CourseDB 注册 Developer

### 1. 登录 CourseDB

先登录 CourseDB，然后打开：

```text
https://<COURSEDB_HOST>/account/developer
```

页面名称是 **Developer Platform / 开发者平台**。

### 2. 提交开发者权限申请

在“申请”页签填写使用场景并提交。申请理由至少需要 20 个字符，建议说明：

- 准备用什么客户端调用，例如 OpenCode。
- 计划使用哪些只读能力，例如 Handbook、专业选修和 Offering 查询。
- 使用范围，例如帮助学生规划某个学期的课程。
- 不会执行写入、选课或修改 Planner 的操作。

示例：

```text
I am integrating CourseDB with OpenCode to help students plan a semester using read-only Handbook, major-elective, and Offering data. The integration will not modify planners, enroll students, or save personal timetables automatically. It can create temporary timetable preview snapshots.
```

提交后状态会变成 **Pending review / 待审核**。审核期间不能重复提交申请。

### 3. 等待审核通过

管理员审核通过后，账户会获得 Developer Access。状态会显示为 **Developer enabled / 已开通开发者权限**。

如果申请被拒绝或撤销，可以根据审核备注修改使用场景后重新提交。

### 4. 创建 API Key

审核通过后进入“API Key 管理”页签：

1. 填写 Key 名称，例如 `opencode-timetable-planner`。
2. 可选填写过期天数，范围是 1 到 3650 天。
3. 点击创建。
4. 立即复制完整 API Key。

完整 Key 只会展示一次。CourseDB 只保存 Key 的哈希值，之后页面只能看到 Key 前缀。

把 Key 设置为环境变量：

```bash
export COURSEDB_DEV_API_KEY="<COPIED_API_KEY>"
```

不要把 Key 放在：

- GitHub 仓库。
- `opencode.json`。
- URL 查询参数，例如 `?apiKey=...`。
- 截图、Issue 或聊天记录。

### 5. 确认系统开关

要让 MCP 正常工作，需要同时满足：

- CourseDB 管理后台的 MCP 全局开关已开启。
- Developer API mode 不是 `DISABLED`。
- API Key 没有过期或被撤销。
- API Key 所属账户仍然拥有 Developer Access。

如果 MCP 全局开关关闭，接口会返回 `410 Gone`。如果 Developer API 暂停，接口可能返回 `503`。API Key 缺失或无效时会返回 `401`。

## 验证连接

可以先用 `curl` 验证 MCP 服务是否返回 JSON：

```bash
curl -i -X POST "https://<COURSEDB_HOST>/api/mcp" \
  -H "Authorization: Bearer ${COURSEDB_DEV_API_KEY}" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -H "MCP-Protocol-Version: 2026-07-28" \
  -H "Mcp-Method: server/discover" \
  --data '{"jsonrpc":"2.0","id":1,"method":"server/discover","params":{"_meta":{"io.modelcontextprotocol/protocolVersion":"2026-07-28","io.modelcontextprotocol/clientCapabilities":{},"io.modelcontextprotocol/clientInfo":{"name":"curl","version":"1.0"}}}}'
```

预期结果：

- `200`：MCP 服务发现成功。
- `401`：API Key 缺失、无效、过期或已撤销。
- `410`：MCP 全局开关关闭。
- `503`：Developer API 暂停或关闭。
- HTML 页面：通常表示 URL 错误、服务未启动、路由未部署，或请求被代理到网页路由。

### OpenCode 协议错误

如果 OpenCode 返回 `-32022`，且 `requested` 为 `2025-11-25`、`supported` 为 `["2026-07-28"]`，客户端仍在使用默认旧协议。检查 `mcp.servers.coursedb.protocol` 是否为 `"2026-07-28"`。

如果请求仍使用 `initialize`，只修改请求里的版本字符串不能转换协议。新版使用 `server/discover` 和每次请求的协议元数据。OpenCode 会在设置 `protocol` 后生成对应的请求格式。

保存配置后，重启 OpenCode 及其后台服务。在启动 OpenCode 的同一终端设置 `COURSEDB_DEV_API_KEY`。不要把密钥发到聊天或错误报告中。

如果项目配置也定义了 `coursedb`，它会替换全局同名服务器配置。项目配置必须保留完整的 URL、`protocol`、`oauth` 和鉴权头。

在设置密钥的终端运行 `opencode mcp list`，确认 CourseDB 显示为 `connected`。如果安装命令名是 `opencode2`，使用 `opencode2 mcp list`。

## 使用示例

配置完成后，可以在 OpenCode 中直接提出类似请求：

```text
我想规划 2026 Fall 的课程。
我的专业是 CST，2024 年入学，现在大三。
请先读取 Handbook，再按专业必修、专业选修、Free Elective、其他课程的顺序规划。
```

Skill 会首先确认：

- 专业和专业代码。
- 入学年份。
- 当前年级。
- 目标日历学期。

之后通过 MCP 查询 Handbook 和 Offering，并把历史排除、事实、推断、时间冲突和未确认事项分开说明。若学生没有提供历史课程或之前的计划，涉及 ME/FE 的选择会标记为暂定。用户明确要求预览或导出时，这些暂定选择才纳入 JSON 或课表预览。历史缺失、先修未核实或需求归类待确认不会阻止这次已请求的展示。用户要求忽略先修时，Skill 会按该假设生成暂定方案，但不会声称满足先修要求。

## 安全与限制

- MCP 工具是只读的，不会创建或修改 Planner。
- API Key 应使用最小权限和合理过期时间。
- 课程是否满足先修要求不能仅凭文本自动确认。
- `NO_RECORD` 只表示 CourseDB 当前没有对应学期记录，不代表学校一定不开课。
- FYP（Final Year Project）通常没有固定授课时间；没有 `timeSlots` 时应标记为项目/导师安排，而不是普通时间冲突。

## 相关文档

- [OpenCode MCP Servers](https://opencode.ai/v2/docs/mcp-servers/)
- [OpenCode configuration](https://opencode.ai/v2/docs/config/)
- [Agent Skills specification](https://agentskills.io/specification)
- [CourseDB MCP Skill](SKILL.md)
- [MCP tool reference](references/mcp-tools.md)
- [Timetable import JSON](references/timetable-json.md)
- [GE programme reference](references/ge-programme.md)
- [Course-code expansion map](references/course-code-expansion-map.md)

## 新版协议与数据

MCP 使用最新稳定协议 `2026-07-28`，客户端必须支持该协议。
旧版 `initialize` 握手、协议会话和 GET 事件流不再支持。
每次 POST 必须携带协议版本、客户端能力和匹配的 `MCP-Protocol-Version`、`Mcp-Method` 请求头。
工具调用还需要匹配的 `Mcp-Name`。
客户端必须支持配置中的 `protocol` 字段。服务器支持新协议不会自动改变客户端的默认请求格式。

API 统一使用 `/api/dev/*`，不再提供 `/api/dev/v2/*` 兼容入口。
成功响应使用 `data` 和 `meta`，错误使用结构化 `error`。
MCP 与 API 的开课、教师别名和课程分类只读新版 Offering 数据。
选修分类来自最近一个有分类信息学期的全部 session，不再读取课程版本的旧 `type` 字段。
批量 Offering 查询每门课最多返回 100 个 session。`sessionsTruncated` 为 true 时，使用 `list_course_offering_sessions` 从最后一条的 `{session,id}` 继续读取。分页失败、游标重复或数量变化时，保留已读结果并说明分析不完整。

Skill 发布版本为 `2.8`。站内 Agent 按需加载 Skill：只有排课任务才通过 `load_skill` 读取本 Skill，再用 `read_skill_reference` 读取工具说明和排课规则。秋季学期代码为 `1`。课程、时间和地点必须有查询结果依据。

开课结果过大时，Agent 按错误中的恢复参数读取课节分页。本轮额度耗尽时停止读取，并报告遗漏。课程详情不提供课节，不能用于判断目标学期开课情况。站内预览校验课节和时间的查询依据，并返回实际包含的课程与尚未覆盖的固定必修课。

课程详情的最新分类学期与目标 session 分类分别处理；冲突或目标分类缺失时，选修需求保持待确认。具体变体使用自己的学分和双语名称，不能借用基础课程的数据。讲师按关联行顺序读取已解析名称或原始姓名。

Timetable JSON 继续使用导入格式 `version:2`，条目 `id` 直接复制实际 session ID。导出前检查时间、文本与数量限制，不静默截断。未知时间和项目课在正文说明。详见 [字段与分页参考](references/mcp-tools.md)、[导出规则](references/timetable-json.md) 和 [规划示例](references/planning-examples.md)。

协议参考：[MCP 2026-07-28](https://modelcontextprotocol.io/specification/2026-07-28)。

### 临时课表预览

仅在当前用户明确要求预览或导出时调用。只要求规划课程或先看候选课程，不构成预览请求。

`create_timetable_preview` 接收 `{requestId, name, data}`。`data` 使用原有 Timetable JSON 格式。工具返回预览链接、到期时间和剩余额度。

持链接者无需登录即可查看。登录用户可保存个人副本，随后编辑。预览创建后保留 24 小时。每个 API key 所属用户在滚动 24 小时内最多创建 20 张，所有 key 共用额度。正式副本继续受每人 25 张的限制。

预览工具失败或不可用时，Skill 返回 JSON 供手动导入。完整 MCP 请求上限为 64 KiB，不会静默截断内容。临时内容使用 Redis，数据丢失时链接可能提前失效。

上线前确认 Redis 容量和淘汰策略适合临时业务存储。观察创建、额度拒绝和保存失败的服务日志，不记录正文或链接 token。

### 基于课程清单查课

现有 `plan-semester-courses` Skill 先读取 Handbook 返回的 `courseLists`，再使用其 ID 查询课程清单成员、课程详情和目标学期 Offering。

`get_handbook_term_requirements` 返回该 Handbook 直接绑定的首批清单，每页最多 50 个。超过一页时，使用返回的 `handbook.handbookId` 调用 `list_course_lists` 继续读取。Agent 无需再次按专业和入学年份搜索。

`list_course_lists` 支持 Handbook ID、清单双语名称、专业代码、入学年份和具体课程代码筛选。`list_course_list_courses` 支持清单成员分页、课程代码与双语名称搜索，以及跨所有年份的季节筛选。两个工具每页最多返回 50 条。

清单成员关系、Handbook 要求和 Offering 分类是独立事实。季节筛选及其 session 数量覆盖所有年份。查询某年实际开课时，仍需调用 `get_course_offerings`。没有绑定清单时，Skill 会说明缺失，不自动改用其他入学年份。
