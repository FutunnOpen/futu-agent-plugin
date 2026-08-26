# Futu MCP Plugin

富途行情、选股、新闻、交易一站式 MCP 插件。兼容 **Claude Code**、**Cursor**、**Codex**、**Agent Plugins** 四种格式。

## 目录结构

```
futu-agent-plugin/
├── plugin.json                    # Agent Plugins 开放标准 manifest
├── mcp.json                       # Cursor 格式 MCP 配置
├── .mcp.json                      # Codex / Claude Code 格式 MCP 配置
├── .claude-plugin/
│   ├── plugin.json                # Claude Code 插件 manifest
│   └── marketplace.json           # Claude Code marketplace 注册
├── .cursor-plugin/
│   ├── plugin.json                # Cursor 插件 manifest
│   └── marketplace.json           # Cursor Team Marketplace 注册
├── .codex-plugin/
│   └── plugin.json                # Codex 插件 manifest
├── .agents/plugins/
│   └── marketplace.json           # Codex marketplace 注册
└── skills/
    ├── futu-mcp-quote/SKILL.md    # 行情：报价/K线/盘口/分时/逐笔/板块/自选
    ├── futu-mcp-screen/SKILL.md   # 筛选：股票/窝轮/期权/IPO
    ├── futu-mcp-news/SKILL.md     # 研究：新闻/财报/估值/股东/资金流/期权
    └── futu-mcp-account/SKILL.md  # 交易：真实/模拟 下单/改单/撤单/持仓/资金
```

## 兼容说明

| 平台 | 识别方式 | MCP 配置 | Marketplace |
|------|----------|----------|-------------|
| **Claude Code** | `.claude-plugin/plugin.json` | `.mcp.json` | `.claude-plugin/marketplace.json` |
| **Cursor** | `.cursor-plugin/plugin.json` | `mcp.json` | `.cursor-plugin/marketplace.json` |
| **Codex** | `.codex-plugin/plugin.json` | `.mcp.json` | `.agents/plugins/marketplace.json` |
| **Agent Plugins** | 根目录 `plugin.json` | `mcp.json` | — |

四种格式各自读取自己的 manifest 和 MCP 配置，互不冲突。Skills 目录 (`skills/`) 是共享的，格式完全一致。

## MCP Server

所有平台连接同一远程 MCP 服务：

```
https://mcp.futunn.com/mcp
```

协议：Streamable HTTP（Claude Code / Cursor）/ HTTP（Codex）

## 安装方式

### Claude Code

**本地测试：**
```bash
claude plugin install --plugin-dir /path/to/futu-agent-plugin
```

或在 `settings.json` 中手动添加：
```json
{
  "enabledPlugins": {
    "futu-mcp@local": true
  }
}
```

**Marketplace 发布：**

将此仓库提交至 Claude Code 官方 marketplace，或作为自定义 marketplace 添加：
```bash
/plugin marketplace add <org>/futu-agent-plugin
```

### Cursor

**本地测试：**
```bash
ln -s /path/to/futu-agent-plugin ~/.cursor/plugins/local/futu-mcp
```
重启 Cursor 或执行 Developer: Reload Window。

**Team Marketplace：**
在 Dashboard → Plugins 中添加此仓库 URL，Cursor 自动读取 `.cursor-plugin/marketplace.json`。

### Codex

**本地：**
将此仓库 clone 到本地，`.agents/plugins/marketplace.json` 已注册插件。

**远程安装：**
```bash
codex plugin marketplace add <org>/futu-agent-plugin
```

### Agent Plugins

任何支持 [Agent Plugins](https://agent-plugins.org) 标准的工具均可通过根目录 `plugin.json` 识别。

## 功能概览

### 行情数据 (futu-mcp-quote)
- 实时报价、市场快照
- 日/周/月/分钟 K 线
- 买卖盘深度（LV1/LV2）
- 分时走势、逐笔成交
- 市场状态、交易日历
- 自选股管理、板块成分

### 选股筛选 (futu-mcp-screen)
- 股票多因子筛选（估值/涨跌/财务/技术形态/持仓等 120+ 维度）
- 窝轮/牛熊证筛选
- 期权筛选器
- 港/美/A/新/马 IPO 列表

### 资讯研究 (futu-mcp-news)
- 新闻/公告/研报搜索
- 经济日历（热点 + 搜索）
- 财务报表、营收构成
- PE/PB/PS 估值分析
- 分析师评级、晨星报告
- 股东/机构/内部人持仓
- 资金流向、做空数据
- 期权链/波动率/行权概率

### 账户交易 (futu-mcp-account)
- 真实交易：下单/改单/撤单/风控确认
- 模拟交易：完整模拟交易链路
- 账户资金、持仓查询
- 多市场支持（港/美/A/新/日/加/期货/期权）

## 支持市场

| 代码 | 市场 |
|------|------|
| HK | 香港 |
| US | 美国 |
| SH / SZ | 沪深 A 股 |
| SG | 新加坡 |
| JP | 日本 |
| CA | 加拿大 |
| AU | 澳大利亚 |
| MY | 马来西亚 |
| KR | 韩国 |

## 作用流程

1. IDE/CLI 启动时扫描对应的 marketplace 或 plugin manifest
2. 发现 futu-mcp 插件，指向本地路径（或 git/npm）
3. 用户在插件目录里看到它，可以安装/启用
4. 安装后加载 MCP 配置，连接 `https://mcp.futunn.com/mcp`，加载 skills

## marketplace 配置说明

### Claude Code (.claude-plugin/marketplace.json)

- `plugins[].source: ".."` — 相对于 marketplace 文件所在目录解析，指向仓库根
- 插件通过 `/plugin install` 或 settings.json 启用

### Codex (.agents/plugins/marketplace.json)

- `source.path: "../../"` — 相对于 marketplace 文件所在目录解析，指向仓库根
- `installation: "INSTALLED_BY_DEFAULT"` — clone 后自动启用
- `authentication: "ON_INSTALL"` — 安装时触发 OAuth 鉴权

### Cursor (.cursor-plugin/marketplace.json)

- `metadata.pluginRoot: "."` — 插件根目录就是仓库根
- `plugins[].source: "."` — 相对于 pluginRoot 解析

## 远程发布

**Claude Code 官方 marketplace：**

提交至 [plugin directory submission](https://clau.de/plugin-directory-submission)，或在自有 marketplace repo 的 `marketplace.json` 中添加：
```json
{
  "name": "futu-mcp",
  "source": {
    "source": "url",
    "url": "GitHub · Change is constant. GitHub keeps you ahead.<org>/futu-agent-plugin.git"
  },
  "description": "富途行情、选股、新闻、交易一站式 MCP 插件",
  "category": "Finance"
}
```

**Codex git-subdir 格式：**
```json
{
  "name": "futu-mcp",
  "source": {
    "source": "git-subdir",
    "url": "GitHub · Change is constant. GitHub keeps you ahead.<org>/futu-agent-plugin.git",
    "path": "./",
    "ref": "main"
  },
  "policy": {
    "installation": "AVAILABLE",
    "authentication": "ON_INSTALL"
  },
  "category": "Finance"
}
```
