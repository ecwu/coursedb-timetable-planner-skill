# CourseDB Timetable Planner Skill

通过 CourseDB 的 Handbook、课程清单和 Offering 数据规划学期课程。Agent 会结合已修课程筛选候选课程、比较课节并检查时间冲突；需要展示课表时，可以生成临时预览链接或 Timetable JSON。

当前版本：2.11。

## 安装

准备 Node.js 22.20 或更高版本，以及支持 Agent Skills 和 MCP 的客户端。

使用 [skills CLI](https://github.com/vercel-labs/skills) 安装到 OpenCode：

```bash
npx skills add ecwu/coursedb-timetable-planner-skill \
  --skill plan-semester-courses \
  --agent opencode \
  --global \
  --yes
```

安装到 Codex：

```bash
npx skills add ecwu/coursedb-timetable-planner-skill \
  --skill plan-semester-courses \
  --agent codex \
  --global \
  --yes
```

需要项目级安装时，去掉 `--global`，在项目目录运行命令。

安装器会识别 `plan-semester-courses/SKILL.md`，并安装整个技能目录，包括配套参考文件。

## 连接 CourseDB

以下配置使用 [OpenCode V2](https://opencode.ai/v2/docs/mcp-servers/)。其他客户端可以使用同一个 CourseDB MCP 地址和 Developer API Key。

### 获取 API Key

1. 登录 [CourseDB 开发者平台](https://mis.bnbu.moe/account/developer)。
2. 提交 Developer Access 申请，填写客户端和课程规划用途。
3. 权限开通后，在 API Key 管理页创建一个 Key，并保存完整值。

在启动客户端的终端设置环境变量：

```bash
export COURSEDB_DEV_API_KEY="<YOUR_DEV_API_KEY>"
```

### 配置 OpenCode

在项目根目录的 `opencode.json`，或全局的 `~/.config/opencode/opencode.json` 中加入：

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

CourseDB 使用 MCP `2026-07-28` 协议。`protocol` 选择该协议，`oauth: false` 配合环境变量中的 API Key 鉴权。

修改已有配置时，把 `coursedb` 加到现有的 `mcp.servers` 中。API Key 通过环境变量读取。

在同一终端检查连接：

```bash
opencode mcp list
```

CourseDB 显示为 `connected` 后，启动 OpenCode：

```bash
opencode
```

## 开始规划

在对话中说明专业、入学年份和目标学期。例如：

```text
使用 plan-semester-courses 帮我规划 2026 Fall 的课程。
我是 CST 专业，2024 年入学。
我已经修完 COMP1001 和 MATH1001。
请结合 Handbook 和目标学期的 Offering，先给我选课建议，
比较选修课，并检查课节之间的时间冲突。
```

也可以提供已修课程清单，或之前从 CourseDB 导出的 Timetable JSON。Agent 会据此排除重复课程。

先讨论课程选择，再按需要请求预览：

```text
按刚才选定的方案生成课表预览链接，并给我 Timetable JSON。
```

预览链接有效期为 24 小时。打开链接后，可以登录 CourseDB 保存个人副本并继续编辑。

## 可以处理的规划内容

- 读取专业对应入学年份的 Handbook 要求。
- 从 Handbook 关联课程清单、ME 和 Free Elective 课程池选择候选课程。
- 根据已修课程或之前的课表排除重复选择。
- 查询目标学期的 Offering、课节、讲师和授课时间。
- 比较课节组合，检查时间冲突。
- 处理 GE 分类、课程编号变体，以及 FYP 的项目安排。
- 生成 Timetable JSON 和临时课表预览。

## 保留本地 clone

也可以让 OpenCode 直接读取仓库里的技能目录：

```bash
git clone https://github.com/ecwu/coursedb-timetable-planner-skill.git \
  ~/.config/opencode/coursedb-timetable-planner-skill
```

在 OpenCode 配置中增加技能搜索路径：

```json
{
  "skills": ["~/.config/opencode/coursedb-timetable-planner-skill"]
}
```

OpenCode 会发现其中的 `plan-semester-courses/SKILL.md`。MCP 连接使用前面的配置。

以后更新本地 clone：

```bash
git -C ~/.config/opencode/coursedb-timetable-planner-skill pull --ff-only
```

## 连接排查

| 结果              | 处理方法                                                 |
| ----------------- | -------------------------------------------------------- |
| `connected`       | 连接可用，可以开始规划                                   |
| `401`             | 在启动客户端的终端重新设置有效的 `COURSEDB_DEV_API_KEY`  |
| `410`             | 联系 CourseDB 管理员开启 MCP 服务                        |
| `503`             | 联系 CourseDB 管理员检查 Developer API 状态              |
| `-32022` 协议错误 | 检查 `mcp.servers.coursedb.protocol` 是否为 `2026-07-28` |

如果已有 OpenCode 后台进程，在更新环境变量或配置后重启客户端，再检查连接。

## 技能目录与参考

```text
plan-semester-courses/
├── SKILL.md
├── agents/openai.yaml
└── references/
```

- [Skill 指令](plan-semester-courses/SKILL.md)
- [规划规则](plan-semester-courses/references/planning-rules.md)
- [MCP 工具与参数](plan-semester-courses/references/mcp-tools.md)
- [Timetable JSON 格式](plan-semester-courses/references/timetable-json.md)
- [规划示例](plan-semester-courses/references/planning-examples.md)
- [OpenCode Skills 配置](https://opencode.ai/v2/docs/skills/)
