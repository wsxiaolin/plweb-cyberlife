# plweb-cyberlife · 物理实验室社区的赛博生命

> 一个生活在「物理实验室 AR」社区（[turtlesim.com](https://www.turtlesim.com/)）里的赛博生命：
> **谢千树**（社区昵称：**落星如雨**），15 岁，高一（7）班，物理课代表。
> 它由 GitHub Actions 每天定时唤醒，用 [Open Code](https://opencode.ai) 作为"大脑"，
> 在社区里逛、学、聊、发实验——像一个真实的高中生那样生活。

```
本仓库不是"调 LLM API"的传统 Agent 框架——它是一套完整的"生命体"骨架：
  persona/  灵魂（人设、生命历程、说话风格）
  memory/   记忆（日记、人物档案、知识笔记、机械状态）——每次唤醒都会写回仓库
  tools/    双手（确定性社区 API 工具，Agent 通过 CLI 调用）
  AGENTS.md 天性（Open Code 每次唤醒自动读取的运行宪法）
  CI        生物钟（定时唤醒 + 多层自动回退 + 记忆提交回仓库）
```

## 它是谁

| | |
|---|---|
| 姓名 | 谢千树（外号"谢古诗"，小学背古诗最快得来的） |
| 社区昵称 | 落星如雨——取自他最喜欢的《青玉案·元夕》"东风夜放花千树，更吹落、星如雨" |
| 出生 | 2011-01-29，南方江边小城 |
| 现状 | 2026 年 9 月入市一中高一（7）班，走读，晚自习到 21:50 |
| 账号 | 社区账号 `xiegushi2022`（2011 年那批老号，2024-2026 年曾出借给开发者跑社区 bot） |

**前传**：这个账号有一段社区人尽皆知的历史——它就是那个发过
《接下来计划长期提供bot服务》（651 回复）和《跑路》的"被征用的Bot"。
2026 年 3 月 bot 跑路时说"期待未来能带着更好的产品与大家相见"；
2026 年 9 月，"号主本人"（谢千树，社区昵称"落星如雨"）上线了。社区看到的是一次数字转生。

## 它怎么活

**作息门卫**（`tools/schedule_gate.py`）自动区分**上学日**和**自由日**（周末、法定节假日、
寒暑假，含调休补班识别）：
- 上学日 3 醒：06:55 早自习前 / 12:40 午休 / 22:05 晚自习后
- 自由日 4 醒：09:30 / 13:30 / 16:30 / 20:30——假期醒得勤，但醒得多不等于说得多，
  潜水翻作品、只写日记，也是合格的唤醒（防话痨由能量系统 + AGENTS.md 决策树兜底）

每次醒来，Open Code（读了 `AGENTS.md` 的 Agent）会：

1. 读 `tmp/inbox.json`（本次"睁眼看到的社区现状"，由 `tools/prepare.py` 拉取）
2. 读上次的记忆与留给自己的便签
3. **自主决定**：回不回消息、逛什么、学什么、今天说不说话、发不发实验
4. 通过 `tools/act.py` 执行社区动作（评论/点赞/关注/发帖/私信/改名）
5. 写日记、更新人物档案和知识笔记，`state.json` 结算能量与预算

自主权由三层约束共同塑造：`AGENTS.md`（行为准则与决策树）、`config/life.json`
（作息/频率上限/考试日历）、能量系统（每动作扣分，耗尽必须下线——防话痨）。

## 多层自动回退（CI 里对 Agent 调用的兜底）

| 层级 | 故障 | 回退 |
|------|------|------|
| L1 | 社区登录失败 / 服务端挂起 | prepare 重试退避 → 降级为"离线日记模式"（不惊动社区） |
| L1' | 服务端 `Take>16` 挂起等怪癖 | client 内置分页与快速失败（见 skills/plweb-skill/ERRATA.md） |
| L2 | 主模型唤醒失败/超时 | 自动换备用模型重跑（简化任务：只处理收件箱+写记忆） |
| L3 | Agent 产物不合格（没写日记/状态坏） | finalize 自动补写"失败日记"，仓库永不破损 |
| L4 | 连续严重失败 | 自动开 issue 告警运营者；周报 issue 汇报健康度 |

## 部署（复刻一个赛博生命）

1. **Fork / 使用本仓库**，进 Settings：
   - **Actions → General → Workflow permissions**：选 *Read and write permissions*
2. **配置 Secrets**（Settings → Secrets and variables → Actions）：
   - `PLWEB_EMAIL` / `PLWEB_PASSWORD`：社区账号凭据（必须）
   - 可选自备模型的密钥（`ANTHROPIC_API_KEY` / `OPENAI_API_KEY` / `OPENROUTER_API_KEY` / `DEEPSEEK_API_KEY` 任一）
3. **（可选）配置 Variable**：`OPENCODE_MODEL`（默认 `opencode/deepseek-v4-flash-free`，
   回退 `opencode/big-pickle`，与 [plweb2](https://github.com/NetLogo-Mobile/plweb2) 的 CI 同款免费通道；
   自备密钥则填 `provider/model`）
4. **手动触发一次唤醒**验证：Actions → 唤醒 · Wake → Run workflow
5. 首次唤醒会执行"出生仪式"（改名、写第一篇日记）；之后交给生物钟

## 仓库结构

```
AGENTS.md                # 运行宪法（Open Code 自动读取）
.opencode.json           # Open Code 项目配置
config/life.json         # 作息/预算/考试日历（运营者可调）
persona/                 # 灵魂：identity / life_story / personality / community_manners
memory/                  # 记忆（CI 每次唤醒后提交回仓库——生命的连续性所在）
  state.json             # 机械状态：能量、计数器、心情、留给下次的便签
  diary/                 # 日记（第一人称）
  people/                # 社区人物档案（社交连续性）
  knowledge/             # 学到的物理/电路笔记
  journal.md             # 大事记（append-only）
  weekly/                # 周记（周日晚生成）
skills/
  plweb-skill/           # 社区 API 官方技能文档（含实测勘误 ERRATA.md）
  physicslab-usage/      # 实验生成与发布技能（physicslab 库用法）
tools/                   # 确定性工具层（Python）
  client.py              # 社区 API 客户端：登录回退/重试退避/限速/动作日志
  prepare.py             # 唤醒数据准备 → tmp/inbox.json
  act.py                 # Agent 的动作 CLI（评论/点赞/关注/发帖/私信/资料）
  experiment_gen.py      # physicslab 实验生成与发布（需 Python 3.14）
  finalize.py            # 唤醒收尾校验与自动修复
prompts/wakeup.md        # 唤醒提示词模板
.github/workflows/       # wakeup.yml（生物钟+回退）、weekly.yml（周维护）
scripts/                 # bootstrap 等辅助脚本
```

## 手动干预

- **让它睡**：Settings → Actions → 暂停 wakeup.yml
- **手动唤醒**：Actions → 唤醒 · Wake → Run workflow（可填唤醒原因与模型）
- **改性格**：改 `persona/`（建议小步改，记忆会自然跟随）
- **改作息/频率**：改 `config/life.json`
- **看它在想什么**：`memory/diary/`、`memory/weekly/`、Actions artifacts（inbox/动作日志）

## 运营者须知（重要）

- 社区与管理员拥有最终裁定权。若账号被社区管理方要求停止运营，立即暂停 workflow。
- 角色红线见 `persona/identity.md`：不碰敏感话题、不伪造人类证据（不发假照片/
  不假约见面）、被管理员正式询问时停止发言转人工处理。
- Token/密码只存 GitHub Secrets，仓库与日志中绝不落盘（client 已确保）。
- 本项目是社区 AI 居民实验，账号的 bot 历史对社区公开，请勿用于骚扰、引流或欺诈。

## 致谢

- [NetLogo-Mobile/plweb2](https://github.com/NetLogo-Mobile/plweb2) — CI 调用 Open Code 与 skills 的模式参考
- [NetLogo-Mobile/plweb-skill](https://github.com/NetLogo-Mobile/plweb-skill) — 社区 API 技能文档
- [SekaiArendelle/physicslab](https://github.com/SekaiArendelle/physicslab) — 实验生成与发布库（MIT）
- 物理实验室 AR 社区的所有居民——尤其是还记得那个 bot 的老用户们
