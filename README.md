# zcode-plugins · 自建 ZCode 插件市场

这是一个 [ZCode](https://zcode.z.ai) 插件市场仓库（marketplace repo）。把本仓库地址添加为插件市场，即可一键安装其中收录的插件。

## 收录插件

| 插件 | 版本 | 说明 |
|---|---|---|
| **Multi-Model Pipeline**（多模型流水线）`multi-model-pipeline` | 0.2.0 | Supervisor-Worker 多模型流水线：GPT 需求/验收 → DeepSeek 开发 ⇄ GLM 测试（≤5 轮）→ GPT 终审 |

### 插件亮点

- **各角色锁定最合适的模型**：需求/验收用 GPT，开发用 DeepSeek，测试用 GLM，模型写在角色定义里，装完即用；
- **文档驱动交接**：PRD、验收清单、story、验收报告全部落盘 `docs/pipeline/`，任何新会话可断点接管；
- **真实测试门禁**：story 通过与否只认主管亲手跑出的测试退出码，专员口头宣布不作数；
- **防跑飞**：开发⇄测试循环上限 5 轮；小改动自动跳过流水线直接做。

## 安装（给使用者）

1. 打开 ZCode **设置 → 插件 → 插件市场（Marketplaces）**；
2. 点 **添加插件市场（Add marketplace）**，填入本仓库地址：

   ```
   https://github.com/yu-xiao/zcode-plugins
   ```

   （也支持 Git URL / 本地目录；本地开发可直接选本 `plugins/` 目录）
3. 确认风险提示 → **安装 Multi-Model Pipeline** → 在"已安装"里启用；
4. **新开一个会话**（agent 定义在会话启动时快照），输入框敲 `/pipeline` 验证命令可用。

## 使用

```
/pipeline <需求描述>     # 从需求分析开始跑全流程
/pipeline 继续           # 读取 docs/pipeline/ 现状，从断点继续
```

小改动（错别字、单行小修）主管会不走流水线直接做掉。

## 模型前置条件

插件默认按以下模型派工（`agents/*.md` 的 `model:` 行）：

| 角色 | 默认模型 |
|---|---|
| analyst / acceptor | `new-provider-3/gpt-6.1-sol$high` |
| developer | `new-provider-2/deepseek-v4.1-flash$max` |
| tester | `new-provider/glm-5.3-flash$max` |

使用前需在 ZCode **设置 → 模型设置** 里配置同名供应商与模型（任何 OpenAI 兼容 / Anthropic 兼容网关均可）；也可以在 **设置 → 子智能体 → 插件子智能体** 里给每个角色覆盖模型与推理档位，无需改文件。

## 更新

插件作者改了源码后：**插件市场 → 本市场 → 更新插件市场**，必要时卸载重装，然后新开会话。

## License

[MIT](./LICENSE)
