---
name: project-documentation
description: Keep a software repository's documentation and architecture maps synchronized with its source code. Use when the user asks to update project docs, generate contribution or runbook documents, create code maps, audit documentation freshness, or refresh architecture information. Run either mode or both when requested.
---

# 项目文档与代码地图

这个 Skill 有两个可独立使用的模式：

- **文档同步**：从代码库的单一事实来源更新项目文档。
- **代码地图**：扫描项目结构，生成简洁的架构与代码地图。

如果用户同时要求两项工作，先完成代码扫描和代码地图，再用扫描结果更新文档，并在最后分别报告变更。

## 文档同步模式

### 单一事实来源

优先从这些文件或源码结构提取信息：

| 来源 | 生成内容 |
|--------|-----------|
| `package.json`、`Makefile`、`Cargo.toml`、`pyproject.toml` | 可用命令参考 |
| `.env.example`、`.env.template`、`.env.sample` | 环境变量文档 |
| `openapi.yaml` 或路由文件 | API 端点参考 |
| 源代码导出 | 公共 API 文档 |
| `Dockerfile`、`docker-compose.yml` | 基础设施设置文档 |

### 工作内容

1. 读取项目中的包配置和环境变量模板，提取脚本、命令、变量、用途、格式、默认值和必需性。
2. 识别 API 路由、公共导出和基础设施设置，按照现有文档结构补充参考内容。
3. 生成或更新 `docs/CONTRIBUTING.md`，包括开发环境、安装、脚本、测试、代码风格和 PR 清单。
4. 生成或更新 `docs/RUNBOOK.md`，包括部署、健康检查、监控、常见问题、回滚、告警和升级路径。
5. 查找超过 90 天未修改的文档，并与近期代码变更交叉检查；只标记可疑过时文档，交由人工复核。
6. 最后输出已更新、已标记和已跳过的文件摘要。

### 文档约束

- 始终以代码和配置为事实来源，不凭空补写项目行为。
- 只修改生成区域，保留手工编写内容。
- 在自动生成区域周围使用 `<!-- AUTO-GENERATED -->` 标记。
- 除非用户明确要求，不主动创建新的文档文件；优先更新已有文件。

## 代码地图模式

### 扫描范围

1. 判断项目类型：单体仓库、单应用、库或微服务。
2. 查找 `src/`、`lib/`、`app/`、`packages/` 等源码目录。
3. 定位入口点，例如 `main.ts`、`index.ts`、`app.py`、`main.go`。
4. 识别路由、中间件、服务、仓库、页面、组件、状态管理、数据表、迁移和外部依赖。

### 输出位置与文件

优先写入 `docs/CODEMAPS/`；如果项目已有约定，则使用 `.reports/codemaps/`。按项目实际情况创建或更新：

| 文件 | 内容 |
|------|----------|
| `architecture.md` | 高层系统图、服务边界、数据流 |
| `backend.md` | API 路由、中间件链、服务到仓库的映射 |
| `frontend.md` | 页面树、组件层级、状态管理流 |
| `data.md` | 数据库表、关系、迁移历史 |
| `dependencies.md` | 外部服务、第三方集成、共享库 |

不适用的文件可以跳过。每个代码地图保持简洁，优先记录文件路径、函数签名、调用关系和高层数据流，而不是复制完整代码。

每个代码地图开头添加新鲜度头部，例如：

```markdown
<!-- Generated: 2026-02-11 | Files scanned: 142 | Token estimate: ~800 -->
```

### 差异与报告

1. 如果已有代码地图，比较本次扫描与上次扫描的变更。
2. 变更超过 30% 时，先展示差异并请求用户批准，再覆盖原地图。
3. 变更不超过 30% 时，可以原地更新。
4. 将摘要写入 `.reports/codemap-diff.txt`，包括新增、删除、修改文件，新依赖、架构变化和超过 90 天未更新的文档警告。

主要功能添加或重构后，建议重新运行代码地图模式。
