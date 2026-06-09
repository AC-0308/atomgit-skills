---
name: gitcode-label-stats
description: |
  Statistics of PR label distribution across repositories in a GitCode organization.
  Supports multiple labels, monthly breakdown, and ratio analysis.
  TRIGGER when: user asks to count PRs by label, measure label adoption rate,
  compare labeled vs total PRs, generate label statistics report, or phrases like
  "标签统计", "ai-assisted占比", "PR标签分布", "标签覆盖率", "label stats",
  "label adoption", "组织PR统计".
---

# gitcode-label-stats

统计 GitCode 组织下各仓库的 PR 标签分布，支持多标签、月度维度和占比分析。

## 适用范围

- 按组织维度统计所有仓库的 PR 标签分布
- 支持同时追踪多个标签（如 `ai-assisted`、`bug`、`enhancement`）
- 支持按月度维度统计标签 PR 数量及占比
- 输出 CSV 报表，便于进一步分析

## 前置条件

- GitCode CLI 已安装且已认证：`gitcode auth status`
- 拥有目标组织的读取权限
- 跨平台：Windows 使用 `gitcode`，Linux/macOS 可使用 `gc`

## 工作流程

### Step 1: 获取组织仓库列表

```bash
gitcode repo list -o <org> --limit 100 --json
```

- 若组织仓库超过 100 个，需分页获取（增大 `--limit` 或多次调用）
- 从 JSON 输出中提取每个仓库的 `name` 字段
- 注意：列表中可能包含已删除的幽灵仓库（列表有缓存），后续步骤会自然跳过

### Step 2: 逐仓库获取 PR 列表

对每个仓库，使用 `gitcode pr list` 获取所有 PR：

```bash
gitcode pr list -R <org>/<repo> --state all --limit 100 --json
```

- `--state all` 包含已合并和已关闭的 PR
- 若 PR 数量超过 100 个，需分页处理：多次调用并递增 `--page` 参数（如 `--page 2`）
- 若仓库不存在或已删除，CLI 会返回错误，直接跳过该仓库

### Step 3: 解析 PR 标签与时间

对每个 PR 的 JSON 数据，提取以下字段：

| 字段 | 用途 |
|------|------|
| `number` | PR 编号 |
| `created_at` | 创建时间（用于月度统计） |
| `labels[].name` | 标签名称列表 |

判断 PR 是否携带目标标签：

```text
遍历 pr.labels 数组，检查 label.name 是否在目标标签集合中
```

按 `created_at` 的年月（格式 `YYYY-MM`）归类到对应月份。

### Step 4: 汇总统计

对每个仓库计算：

| 统计项 | 计算方式 |
|--------|----------|
| 总 PR 数 | 该仓库所有 PR 计数 |
| 各目标标签 PR 数 | 携带对应标签的 PR 计数 |
| 标签占比 | 标签 PR 数 / 总 PR 数 × 100% |
| 月度总 PR 数 | 按年月筛选后的 PR 计数 |
| 月度标签 PR 数 | 按年月筛选后携带标签的 PR 计数 |
| 月度标签占比 | 月度标签 PR 数 / 月度总 PR 数 × 100% |

### Step 5: 输出报表

生成 CSV 文件，包含以下列（按需组合）：

```text
Repository, Total PRs, <label> PRs, <label> Ratio,
<YYYY-MM> Total PRs, <YYYY-MM> <label> PRs, <YYYY-MM> <label> Ratio,
..., Overall <label> Ratio
```

- 多标签时，每个标签对应一组列
- 多月份时，每个月份对应一组列
- 最后一行为 TOTAL 汇总行

CSV 写入需使用 UTF-8 编码：

```powershell
# Windows PowerShell
[System.IO.File]::WriteAllLines($csvPath, $csvLines, [System.Text.Encoding]::UTF8)

# Linux/macOS
$csvLines | Out-File -FilePath $csvPath -Encoding utf8
```

## 实现方法

本 skill 不提供固定脚本，而是描述方法。AI agent 应根据以下指导，结合当前环境动态生成执行逻辑：

### 方法 A: 使用 GitCode CLI（推荐）

直接使用 `gitcode pr list` 逐仓库获取 PR 数据，解析 JSON 输出。

**优点**：跨平台，无需额外依赖，认证由 CLI 管理。

**注意事项**：
- Windows 下 CLI 的 JSON 输出可能存在编码问题，建议将输出写入临时文件后读取
- 大量仓库时需注意 API 速率限制，可在仓库间添加适当延迟
- `--json` 输出可能因终端编码导致中文乱码，使用文件中转可规避

### 方法 B: 使用 GitCode REST API

当 CLI 输出格式不稳定时，可直接调用 GitCode API v5：

```text
GET https://api.gitcode.com/api/v5/repos/<org>/<repo>/pulls?state=all&per_page=100&page=<n>
Authorization: Bearer <token>
```

**获取 token**：

```bash
gitcode auth token
```

**分页**：每页最多 100 条，通过 `page` 参数翻页，当返回数组长度小于 100 时停止。

**注意**：
- 仓库名含中文或特殊字符时，需对仓库名部分做 URL 编码（`EscapeDataString`），但不要编码 `/`
- 已删除的仓库会返回 HTTP 400/404，需捕获异常并跳过
- API 返回的 `created_at` 格式为 ISO 8601（如 `2026-05-12T18:58:28+08:00`），截取前 7 位即可得到年月

### 方法 C: 混合方式

用 CLI 获取仓库列表和 token，用 API 获取 PR 数据。兼顾 CLI 的便捷性和 API 的稳定性。

## 常见问题

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 仓库列表中有仓库但 API 返回 404 | 仓库已删除，列表有缓存 | 跳过该仓库，记录为不可访问 |
| CLI JSON 输出解析失败 | 终端编码或输出折行 | 将输出写入文件后读取，或改用 API |
| 中文仓库名 API 报 400 | URL 编码问题 | 仅对仓库名部分做 `EscapeDataString`，不要编码 `org/` 前缀 |
| PR 数量超过 100 | 单页上限 | 使用分页参数 `page` 逐页获取 |
| Windows 中文写入 CSV 乱码 | PowerShell 默认编码非 UTF-8 | 使用 `[System.IO.File]::WriteAllLines` 并指定 UTF-8 编码 |
| 标签 PR 占比显示 0% | 月度内无标签 PR 或除零 | 分母为 0 时占比显示 `0%` 或 `N/A` |

## 输出示例

```text
Repository,Total PRs,ai-assisted PRs,ai-assisted Ratio,2026-04 Total,2026-04 ai-assisted,2026-04 Ratio,2026-05 Total,2026-05 ai-assisted,2026-05 Ratio
repo-a,340,26,7.6%,95,0,0%,99,6,6.1%
repo-b,369,12,3.3%,33,0,0%,29,2,6.9%
repo-c,22,11,50%,8,1,12.5%,4,4,100%
TOTAL,3879,121,3.1%,613,1,0.2%,582,55,9.5%
```

## 规则

- 使用 `gitcode` 命令确保跨平台兼容性（Windows 下 `gc` 是 `Get-Content` 别名）
- 所有含中文的远端写操作必须用 `--body-file` + UTF-8 编码写盘
- CSV 输出必须使用 UTF-8 编码
- 遇到不可访问的仓库时跳过并记录，不中断整体统计
- 分母为 0 时占比显示 `0%`，不执行除法
- 大量仓库时注意 API 速率限制，适当添加延迟
