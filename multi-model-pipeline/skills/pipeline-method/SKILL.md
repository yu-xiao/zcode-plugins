---
name: pipeline-method
description: 多模型流水线（multi-model-pipeline 插件）的方法论与约定：文档布局、story 状态机、测试门禁规则、失败处理、以及如何更换每个专员使用的模型。当用户询问流水线的用法/文档格式/状态含义/换模型方法，或 /pipeline 执行中需要查阅约定时使用。
---

# 多模型流水线方法论（multi-model-pipeline）

一个 Supervisor-Worker 多模型流水线：主管 = 主会话（执行 `/pipeline` 命令的那个会话），专员 = 本插件的四个 agent 类型。**状态全部落盘在仓库 `docs/pipeline/`，任何新会话读文件即可接管。**

## 角色与模型（换模型 = 改对应 agent 文件的 `model:` 一行）

| Agent 文件（插件内） | 类型名 | 职责 | 当前模型 |
|---|---|---|---|
| `agents/analyst.md` | `multi-model-pipeline:analyst` | 需求 → PRD + 验收清单 + story | `new-provider-3/gpt-6.1-sol$high` |
| `agents/developer.md` | `multi-model-pipeline:developer` | 按 story 实现代码 | `new-provider-2/deepseek-v4.1-flash$max` |
| `agents/tester.md` | `multi-model-pipeline:tester` | 按验收标准写测试、诊断失败 | `new-provider/glm-5.3-flash$max` |
| `agents/acceptor.md` | `multi-model-pipeline:acceptor` | 对照清单逐条验收 | `new-provider-3/gpt-6.1-sol$high` |

换模型规则：模型 id 必须写全 `providerId/modelId`；删掉 `model:` 行 = 该专员跟随当前会话模型；同一 provider 下可换其他模型（如 `gpt-6.1-terra`、`gpt-6-sol`）。改完插件源文件后需在插件管理里更新重装生效。

## 文档布局（跟仓库走，可版本控制、可团队共享）

```
docs/pipeline/
├── PRD.md                     需求说明（背景/目标/范围/方案要点）
├── acceptance-checklist.md    验收清单（AC-1..AC-n，每条可执行验证）
├── stories/story-XX.md        story 文件（frontmatter 带 status）
└── reports/acceptance-report.md  验收报告（逐条 PASS/FAIL/UNVERIFIED + 证据）
```

## story 状态机

```
draft → ready → in-progress → impl-done → tested → accepted
                     ↑            ↓          (主管按退出码判定)
                     └── test-fail ←┘   （⇄ 循环，上限 5 轮）
任何阶段 → blocked（需写明阻塞原因，等用户裁决）
```

状态由不同角色推进：analyst 写 draft/ready；developer 写 in-progress/impl-done/blocked；**主管**按真实测试退出码写 test-fail/tested；acceptor 的结论由主管落到 accepted。

## 铁律

1. **测试门禁只认退出码**：过没过由主管亲手运行测试命令判定，专员的口头结论一律不作数。
2. **tester 不许弱化断言**：测试错就修测试，实现错就报"疑似实现问题"，二者不许混。
3. **acceptor 只读**：验收只依据书面标准和实际代码/行为，FAIL 必须带证据。
4. **5 轮上限**：开发⇄测试连续 5 轮不过 → 停止并上报用户，不无限烧钱。
5. **文档即交接**：专员间不传话、不留口头依赖；中断恢复 = 读文件。

## 故障排查

- **`multi-model-pipeline:analyst` 类型不存在** → 插件未安装/未启用：设置 → 插件管理检查；或本会话启动早于插件安装，开新会话。
- **专员报模型错误/配额不足** → 对应 provider 的 key 或余额问题；临时方案：改 agent 文件 `model:` 行或删除该行跟随会话模型。
- **找不到测试命令** → 主管按 package.json / Makefile / CI 配置 / README 的顺序探测；都没有就派 tester 首轮建立并报告它。
- **想跳过流水线做小改动** → 直接告诉主会话做，不必 `/pipeline`；主管也会按尺寸自适应规则自行判断。
