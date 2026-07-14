# CourseDB Timetable Planner Skill

这是一个用于规划学生学期课程的 Agent Skill。它通过 CourseDB 的只读 MCP 工具读取 Handbook、专业选修课程和 Offering，再按“专业必修 → 专业选修 → 其他课程”的顺序生成一个临时课表建议。

仓库：[ecwu/coursedb-timetable-planner-skill](https://github.com/ecwu/coursedb-timetable-planner-skill)

本 Skill 不会修改 CourseDB 中的 Planner，也不会代替学校确认先修要求、毕业资格或最终选课结果。

## 功能

- 根据专业代码、入学年份、年级和目标学期读取 Handbook 要求。
- 查询专业选修课程池。
- 查询课程在指定学年学期的 Offering 和 session。
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
    ├── mcp-tools.md
    ├── planning-rules.md
    └── program-code-map.md
```

## 使用前提

需要准备：

1. 一个可以连接远程 MCP 的 Agent 客户端，例如 OpenCode。
2. CourseDB 的 `/api/mcp` 地址。
3. 一个有效的 CourseDB Developer API Key。
4. CourseDB 管理后台已开启 MCP 全局开关。

MCP 只提供读取操作，目前的工具包括：

- `get_handbook_term_requirements`
- `list_major_elective_courses`
- `get_course_offerings`

## 安装 Skill

### 方式一：克隆 GitHub 仓库

```bash
git clone https://github.com/ecwu/coursedb-timetable-planner-skill.git \
  ~/.config/opencode/coursedb-timetable-planner-skill
```

这个仓库的根目录应直接包含 `SKILL.md`。如果你把仓库作为更大的 Skill 集合使用，则让 OpenCode 指向包含各个 Skill 子目录的父目录。

### 方式二：本地项目目录

如果 Skill 位于当前项目的 `skills/plan-semester-courses`，可以让 OpenCode 指向：

```text
./skills
```

具体配置见下一节。

## OpenCode 配置

OpenCode 对远程 HTTP MCP 使用 `"type": "remote"`。不要写成 `stdio`、`sse` 或 `streamableHttp`。远程 MCP 的 HTTP 传输由 OpenCode 自动处理。

在项目根目录创建 `opencode.json`，或者修改 OpenCode 的全局配置：

```json
{
  "$schema": "https://opencode.ai/config.json",
  "skills": ["~/.config/opencode/coursedb-timetable-planner-skill"],
  "mcp": {
    "coursedb": {
      "type": "remote",
      "url": "https://mis.bnbu.moe/api/mcp",
      "enabled": true,
      "headers": {
        "Authorization": "Bearer {env:COURSEDB_DEV_API_KEY}"
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
    "coursedb": {
      "type": "remote",
      "url": "https://mis.bnbu.moe/api/mcp",
      "enabled": true,
      "headers": {
        "Authorization": "Bearer {env:COURSEDB_DEV_API_KEY}"
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


本地环境还需要先启动 CourseDB：

```bash
pnpm dev
```

不要直接在浏览器中打开 MCP URL。MCP 客户端需要使用 POST JSON-RPC 请求，并在每次请求中携带 API Key。

## 在 CourseDB 注册 Developer
`
### 1. 登录 CourseDB

先登录 CourseDB，然后打开：

```text
https://mis.bnbu.moe/account/developer
```

页面名称是 **Developer Platform / 开发者平台**。

### 2. 提交开发者权限申请

在“申请”页签填写使用场景并提交。申请理由至少需要 20 个字符，建议说明：

- 准备用什么客户端调用，例如 OpenCode。
- 计划使用哪些只读能力，例如 Handbook、专业选修和 Offering 查询。
- 使用范围，例如帮助学生规划某个学期的课程。
- 不会执行写入、选课或修改 Planner 的操作。

> 提交申请要求至少贡献过一条课程评价。

示例：

```text
I am integrating CourseDB with OpenCode to help students plan a semester using read-only Handbook, major-elective, and Offering data.
```

提交后状态会变成 **Pending review / 待审核**。审核期间不能重复提交申请。

### 3. 等待审核通过

管理员审核通过后，账户会获得 Developer Access。状态会显示为 **Developer enabled / 已开通开发者权限**。

如果申请被拒绝或撤销，可以根据审核备注修改使用场景后重新提交。

### 4. 创建 API Key

审核通过后进入“API Key 管理”页签：

1. 填写 Key 名称，例如 `opencode-timetable-planner`。
2. 可选填写过期天数。
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
- API Key 没有过期或被撤销。
- API Key 所属账户仍然拥有 Developer Access。

如果 MCP 全局开关关闭，接口会返回 `410 Gone`。如果 Developer API 暂停，接口可能返回 `503`。API Key 缺失或无效时会返回 `401`。

## 验证连接

可以先用 `curl` 验证 MCP 服务是否返回 JSON：

```bash
curl -i -X POST "https://mis.bnbu.moe/api/mcp" \
  -H "Authorization: Bearer ${COURSEDB_DEV_API_KEY}" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  --data '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-06-18","capabilities":{},"clientInfo":{"name":"curl","version":"1.0"}}}'
```

预期结果：

- `200`：MCP 初始化成功。
- `401`：API Key 缺失、无效、过期或已撤销。
- `410`：MCP 全局开关关闭。
- `503`：Developer API 暂停或关闭。
- HTML 页面：通常表示 URL 错误、服务未启动、路由未部署，或请求被代理到网页路由。

## 使用示例

配置完成后，可以在 OpenCode 中直接提出类似请求：

```text
我想规划 2026 Fall 的课程。
我的专业是 CST，2024 年入学，现在大三。
请先读取 Handbook，再按专业必修、专业选修、其他课程的顺序规划。
```

Skill 会首先确认：

- 专业和专业代码。
- 入学年份。
- 当前年级。
- 目标日历学期。

之后通过 MCP 查询 Handbook 和 Offering，并把事实、推断、时间冲突和未确认事项分开说明。

## 安全与限制

- MCP 工具是只读的，不会创建或修改 Planner。
- API Key 应使用最小权限和合理过期时间。
- 课程是否满足先修要求不能仅凭文本自动确认。
- `NO_RECORD` 只表示 CourseDB 当前没有对应学期记录，不代表学校一定不开课。
- FYP（Final Year Project）通常没有固定授课时间；没有 `timeSlots` 时应标记为项目/导师安排，而不是普通时间冲突。

## 相关文档

- [OpenCode MCP Servers](https://opencode.ai/docs/mcp-servers/)
- [OpenCode configuration](https://opencode.ai/docs/config/)
- [Agent Skills specification](https://agentskills.io/specification)
- [CourseDB MCP Skill](SKILL.md)
- [MCP tool reference](references/mcp-tools.md)
- [GE programme reference](references/ge-programme.md)
- [Course-code expansion map](references/course-code-expansion-map.md)
