# agent-worklog-data

这是你的 **AI 工作账本**：集中保存 Codex、ChatGPT 等 Agent 的工作记录，以及由这些记录生成的报告和看板。

配套项目 `agent-worklog-skill` 负责“怎么记录、怎么统计”；本项目负责“把记录和结果存在哪里”。**使用时需要配套工具，本仓库本身不会自动运行。**

## 这里保存什么

| 目录 | 通俗解释 |
|---|---|
| `raw/` | 原始工作记录及修订，是统计的事实依据 |
| `coverage/` | 各 Agent 声明“这个日期、这个范围已经上报完毕” |
| `policy/` | 时区、参与 Agent 和调度时间等共同规则 |
| `overrides/` | 人工确认的归组或纠错规则 |
| `reports/` | 日、周、月、季、年报告，保存 JSON 和 Markdown |
| `metrics/` | 项目、技术和 Agent 等统计结果 |
| `manifests/`、`analysis/` | 构建来源清单及分析结果 |
| `site/` | 供人阅读的 HTML 看板 |

**日常优先阅读 `site/current/index.html`，需要文字版时看 `reports/`。** `raw/` 保留历史修订，不要为改一条统计直接删除旧文件。

## 一天的工作流程

```text
完成一段工作
  → Agent 整理为 JSON
  → 配套工具脱敏、校验并写入 raw/
  → 确认声明范围内已上报，写入 coverage/
  → 指定聚合执行者生成日报和周期报告
  → 打开本地看板；需要时同步私有 Git 远端
```

多个 Agent 各自提供记录，报告由一个指定执行者统一生成。周报和月报分别从日报统计，季报从月报统计，年报从季报统计。

## 第一次使用

1. 将本仓库和 `agent-worklog-skill` 放到本地，按工具项目 README 安装 CLI。
2. 检查 [policy/worklog.yaml](policy/worklog.yaml)：当前时区是 `Asia/Shanghai`，Codex 为必需上报 Agent，其他 Agent 为可选。
3. 将本仓库的绝对路径作为 CLI 的 `--data` 参数。机器配置示意见 [worklog.local.example.yaml](worklog.local.example.yaml)，当前 CLI 不自动读取它。
4. 如需跨机器同步，另行配置私有远端及 Git 凭据；本地写入成功不代表已经推送。

## Codex 怎么接入

在 Codex 中告诉它工具仓库和本仓库的位置，例如：

> 请读取 `/path/to/agent-worklog-skill/SKILL.md`，把当前任务的实际工作记录到 `/path/to/agent-worklog-data`。agent_id 使用 codex，通过工具的 import 命令写入，并返回 persisted 和 sync 状态。不确定的时间不要估造，不自动推送。

Codex 需要有读取工具和写入本仓库的权限。希望在多个项目中复用时，可按工具仓库 README 将 Skill 安装到 Codex 技能目录；日志仍统一写入本仓库。

## ChatGPT 怎么接入

推荐先用“ChatGPT 整理，本机导入”的方式：

1. 将工具仓库的 `tests/fixtures/synthetic-chatgpt-bridge.json` 样例提供给 ChatGPT。
2. 请它按样例整理当前会话的实际工作，真实记录设 `synthetic=false`，未知时长填 unknown/null，保存为 JSON。
3. 由本机 Codex或你自己使用 `scripts/convert_chatgpt_bridge.py` 转换，再执行 `worklog import`，目标指向本仓库。

也可以直接对本机 Codex 说：

> 请检查并脱敏 `/path/to/chatgpt-work.json`，使用工具仓库的 ChatGPT 桥接脚本转换后导入本数据仓，保留 agent_id=chatgpt，返回落盘收据，不自动推送。

这个 JSON 是本项目约定的工作摘要格式，不是 ChatGPT 的完整历史导出文件。不要仅提供本机路径就假定 ChatGPT 能读写它；真实会话接入目前仍待验证。

## 生成和查看报告

以下命令在 `agent-worklog-skill` 根目录执行；替换路径、日期及当前时间：

```sh
.venv/bin/worklog --data /path/to/agent-worklog-data validate all
.venv/bin/worklog --data /path/to/agent-worklog-data close-day --agent codex --date 2026-09-28 --scope current-task
.venv/bin/worklog --data /path/to/agent-worklog-data run-due --as-of 2026-09-29T00:15:00+08:00
```

只有确认该日期、声明范围已全部上报时才执行 `close-day`。随后打开 `site/current/index.html`。如果不完整，先检查缺失声明或记录冲突；不要直接把报告状态改成 complete。

## 几个容易误解的地方

- **provisional**：当天尚未结束的初版；**incomplete**：上报声明或数据仍不齐，二者含义不同。
- **persisted=true**：已保存本地；**sync=pending**：尚未完成远端同步。
- **unknown 时长**：不知道花了多久，不是花了零分钟；多个 Agent 的时长之和也不是个人专注时间。
- **补录和纠错**：通过工具新增记录或修订，再更新关闭声明、重跑报告；不要手改 HTML。
- **数据保管**：建议私有仓库、本地阅读，不存密码或完整聊天；备份也要包括尚未推送的本地记录。

当前已有本地处理工具；真实平台接入和七天连续试运行尚待验证。详细操作以配套 `agent-worklog-skill` 项目的 README 与 `docs/operations.md` 为准。
