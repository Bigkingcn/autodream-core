# AutoDream-Core — 通用记忆整理引擎

> **让 AI 代理拥有"睡眠"能力，醒来更聪明**
> 
> *Give your AI agent the power to "sleep" — wake up smarter*

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![SkillHub](https://img.shields.io/badge/SkillHub-autodream--core-blue)](https://clawhub.ai/skills/autodream-core)
[![GitHub stars](https://img.shields.io/github/stars/Bigkingcn/autodream-core?style=social)](https://github.com/Bigkingcn/autodream-core)

---

## 📖 目录

- [什么是 AutoDream](#-什么是-autodream)
- [为什么需要记忆整理](#-为什么需要记忆整理)
- [快速开始](#-快速开始)
- [详细配置](#-详细配置)
- [使用示例](#-使用示例)
- [工作原理](#-工作原理)
- [最佳实践](#-最佳实践)
- [常见问题](#-常见问题)
- [贡献指南](#-贡献指南)

---

## 🧠 什么是 AutoDream

**AutoDream-Core** 是一个通用的记忆整理引擎，采用适配器模式设计，支持跨平台记忆管理。

### 核心理念

像人类一样，AI 代理也需要"睡眠"来整理记忆。AutoDream 定期运行，扫描你的记忆文件，执行：

1. **去重** — 合并重复条目
2. **清理** — 删除过时信息
3. **标准化** — 转换相对日期为绝对日期
4. **索引** — 构建结构化知识索引

### 支持的适配器

| 适配器 | 平台 | 状态 |
|--------|------|------|
| `OpenClawAdapter` | OpenClaw | ✅ 稳定 |
| `ClaudeCodeAdapter` | Claude Code | 🚧 开发中 |
| `CustomAdapter` | 自定义平台 | 🔧 可扩展 |

---

## ⚠️ 为什么需要记忆整理

### 问题场景

你的 AI 代理是否遇到过这些问题？

```
❌ "你昨天说的那个文件在哪？" 
   → 但"昨天"是 3 天前，相对日期失效

❌ "按照之前的配置改一下"
   → 但之前有 3 个矛盾的配置方案

❌ "这个调试方法还有用吗？"
   → 引用的文件已被删除

❌ MEMORY.md 超过 500 行
   → 搜索变慢，模型注意力分散
```

### 根本原因

AI 代理的记忆文件会随时间积累：

- **重复条目** — 同一信息多次记录
- **相对日期** — "昨天""上周"失去意义
- **过时引用** — 指向已删除的文件/配置
- **矛盾信息** — 多次尝试的不同方案并存
- **文件膨胀** — MEMORY.md 越来越大，影响性能

### AutoDream 的解决方案

```
整理前 (42 条)                    整理后 (35 条)
├── 重复条目 ×5    ──────────→    ├── 去重合并
├── 相对日期 ×8    ──────────→    ├── 绝对日期
├── 过时引用 ×3    ──────────→    ├── 清理无效
├── 矛盾配置 ×2    ──────────→    ├── 保留最优
└── 有效信息 24    ──────────→    └── 保留全部
```

---

## 🚀 快速开始

### 前置要求

- OpenClaw v2026.4.0+
- Python 3.8+
- 已有 MEMORY.md 和 memory/ 目录

### 安装

```bash
# 方式 1: 使用 skillhub (推荐，CN 优化)
skillhub install autodream-core

# 方式 2: 使用 clawhub (备用)
clawhub install autodream-core

# 方式 3: 手动安装
git clone https://github.com/Bigkingcn/autodream-core.git
cp -r autodream-core ~/.openclaw/workspace-research/skills/
```

### 验证安装

```bash
# 检查技能是否可用
skillhub list | grep autodream
```

### 首次运行

```bash
# 手动触发一次整理（推荐先测试）
python3 ~/.openclaw/workspace-research/skills/autodream-core/scripts/run.py --force

# 查看整理报告
cat ~/.openclaw/workspace-research/memory/dream-report-*.md
```

### 设置自动运行

```bash
# 配置定时任务（24 小时 + 5 个会话后触发）
bash ~/.openclaw/workspace-research/skills/autodream-core/scripts/setup-cron.sh
```

---

## ⚙️ 详细配置

### 基础配置

编辑 `config/config.json`:

```json
{
  "workspace": "~/.openclaw/workspace-research",
  "memory_dir": "memory",
  "memory_file": "MEMORY.md",
  "archive_dir": "memory/archive",
  
  "trigger": {
    "interval_hours": 24,
    "min_sessions": 5,
    "auto_trigger": true
  },
  
  "consolidation": {
    "max_memory_lines": 200,
    "stale_days": 30,
    "merge_similar_threshold": 0.85,
    "keep_backup": true
  },
  
  "performance": {
    "session_throttle_ms": 100,
    "max_files_scan": 50,
    "enable_analytics": true
  }
}
```

### 配置说明

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `workspace` | string | 自动检测 | OpenClaw 工作目录 |
| `memory_dir` | string | `memory` | 记忆文件目录 |
| `memory_file` | string | `MEMORY.md` | 长期记忆文件名 |
| `interval_hours` | number | `24` | 自动触发间隔（小时） |
| `min_sessions` | number | `5` | 最少会话数触发 |
| `max_memory_lines` | number | `200` | MEMORY.md 最大行数 |
| `stale_days` | number | `30` | 过期条目阈值（天） |
| `merge_similar_threshold` | number | `0.85` | 相似度合并阈值 |
| `keep_backup` | boolean | `true` | 保留备份文件 |
| `session_throttle_ms` | number | `100` | Session 扫描节流 |
| `enable_analytics` | boolean | `true` | 启用分析日志 |

### 高级配置（Python API）

```python
from pathlib import Path
from autodream_core import AutoDreamEngine, OpenClawAdapter

# 初始化适配器
adapter = OpenClawAdapter(
    workspace=Path("~/.openclaw/workspace-research").expanduser(),
    memory_dir=Path("~/.openclaw/workspace-research/memory").expanduser(),
)

# 创建引擎
engine = AutoDreamEngine(
    adapter,
    max_memory_lines=200,        # MEMORY.md 最大行数
    stale_days=30,               # 过期条目阈值（天）
    merge_threshold=0.85,        # 相似度合并阈值
    session_throttle=0.1,        # Session 扫描节流（秒/文件）
    enable_analytics=True,       # 启用分析日志
    backup_enabled=True,         # 启用备份
)

# 运行整理
result = engine.run(force=True)
```

---

## 📚 使用示例

### 示例 1: 手动触发整理

```bash
# 强制运行一次（忽略触发条件）
python3 skills/autodream-core/scripts/run.py --force

# 查看详细报告
python3 skills/autodream-core/scripts/run.py --force --verbose
```

### 示例 2: 仅预览不执行

```bash
# 预览模式（只分析，不修改文件）
python3 skills/autodream-core/scripts/run.py --dry-run

# 输出示例：
# 发现重复条目：5 条
# 发现过期条目：3 条
# 预计整理后行数：178 (当前 245)
```

### 示例 3: Python API 调用

```python
from autodream_core import AutoDreamEngine, OpenClawAdapter

adapter = OpenClawAdapter(workspace="/path/to/workspace")
engine = AutoDreamEngine(adapter)

# 运行整理
result = engine.run()

# 查看结果
print(f"整理前：{result['orientation']['total_entries']} 条")
print(f"整理后：{result['consolidation']['final_count']} 条")
print(f"删除过期：{result['consolidation']['removed_stale']} 条")
print(f"合并重复：{result['consolidation']['merged']} 条")
```

### 示例 4: 自定义触发条件

```python
# 仅在 MEMORY.md 超过 300 行时整理
if engine.get_memory_line_count() > 300:
    result = engine.run()
```

### 示例 5: 获取整理报告

```python
result = engine.run()

# 保存报告
report_path = engine.save_report(result, output_dir="memory/reports")
print(f"报告已保存：{report_path}")
```

---

## 🔬 工作原理

### 四阶段流程

```
┌─────────────────────────────────────────────────────────────┐
│                    AutoDream Pipeline                       │
└─────────────────────────────────────────────────────────────┘

Phase 1: Orientation（定位）
├── 扫描 MEMORY.md
├── 扫描 memory/*.md 文件
├── 统计条目数量
└── 评估整理需求

Phase 2: Gather Signal（收集信号）
├── 扫描最近的 Session 记录
├── 提取新的决策/配置
├── 识别过时引用
└── 发现矛盾条目

Phase 3: Consolidation（整理）
├── 去重合并（相似度 > 85%）
├── 转换相对日期 → 绝对日期
├── 标记过期条目（> 30 天）
└── 解决矛盾（保留最优方案）

Phase 4: Prune & Index（修剪和索引）
├── 删除过期条目
├── 截断超长文件
├── 重建索引
└── 生成整理报告
```

### 智能检测规则

| 检测类型 | 规则 | 动作 |
|----------|------|------|
| **重复条目** | 文本相似度 > 85% | 合并 |
| **相对日期** | 包含"昨天""上周""3 天前" | 转换为绝对日期 |
| **过期条目** | 最后修改 > 30 天 + 无引用 | 标记删除 |
| **矛盾配置** | 同一主题多个不同值 | 保留最新 |
| **无效引用** | 引用的文件不存在 | 标记警告 |

---

## 📋 最佳实践

### ✅ 推荐做法

1. **首次运行前备份**
   ```bash
   cp MEMORY.md MEMORY.md.backup.$(date +%Y%m%d)
   ```

2. **先预览再执行**
   ```bash
   python3 scripts/run.py --dry-run  # 先预览
   python3 scripts/run.py --force    # 确认后再执行
   ```

3. **定期检查报告**
   ```bash
   cat memory/dream-report-*.md | tail -50
   ```

4. **保留归档文件**
   ```bash
   # 整理后的条目会移动到 memory/archive/
   # 建议保留 90 天
   ```

### ❌ 避免做法

1. **不要在整理过程中修改 MEMORY.md**
   - 等待整理完成再继续编辑

2. **不要频繁手动触发**
   - 建议至少间隔 12 小时

3. **不要删除备份文件**
   - 至少保留最近 3 次备份

4. **不要在生产环境直接运行**
   - 先在测试环境验证

---

## ❓ 常见问题

### Q1: 整理会删除我的重要记忆吗？

**A:** 不会。AutoDream 采用保守策略：

- 只删除明确过期的条目（> 30 天且无引用）
- 合并重复条目时保留最完整版本
- 所有操作都有备份可恢复

### Q2: 如何恢复被误删的条目？

**A:** 有三种方式：

```bash
# 方式 1: 从备份恢复
cp MEMORY.md.backup.20260403 MEMORY.md

# 方式 2: 从归档找回
cat memory/archive.md | grep "你要找的条目"

# 方式 3: 查看整理报告
cat memory/dream-report-*.md
```

### Q3: 整理运行需要多长时间？

**A:** 取决于记忆文件大小：

| MEMORY.md 大小 | 预计时间 |
|---------------|----------|
| < 100 行 | < 10 秒 |
| 100-300 行 | 10-30 秒 |
| 300-500 行 | 30-60 秒 |
| > 500 行 | 1-3 分钟 |

### Q4: 可以自定义触发条件吗？

**A:** 可以。编辑 `config/config.json`:

```json
{
  "trigger": {
    "interval_hours": 12,    // 改为 12 小时
    "min_sessions": 3        // 改为 3 个会话
  }
}
```

### Q5: 支持其他平台吗（如 Claude Code）？

**A:** AutoDream-Core 采用适配器设计，理论上支持任何平台。目前：

- ✅ OpenClaw: 稳定支持
- 🚧 Claude Code: 开发中
- 🔧 自定义平台: 参考 `adapters/base.py` 实现自己的适配器

### Q6: 如何关闭自动触发？

**A:** 修改配置：

```json
{
  "trigger": {
    "auto_trigger": false
  }
}
```

然后只手动运行：
```bash
python3 scripts/run.py --force
```

---

## 🤝 贡献指南

### 提交 Bug

请在 GitHub Issues 中提供：

1. 问题描述
2. 重现步骤
3. 预期行为
4. 实际行为
5. 环境信息（OpenClaw 版本、Python 版本等）

### 提交功能建议

欢迎提出新功能想法！请说明：

1. 使用场景
2. 期望功能
3. 替代方案（如有）

### 提交代码

```bash
# 1. Fork 仓库
gh repo fork Bigkingcn/autodream-core --clone

# 2. 创建分支
git checkout -b feat/your-feature-name

# 3. 开发并测试
python -m pytest tests/ -v

# 4. 提交
git commit -m "feat: add your feature description"

# 5. 推送并创建 PR
git push origin feat/your-feature-name
gh pr create --title "feat: your feature" --body "Description..."
```

### 开发新适配器

参考 `adapters/base.py` 实现接口：

```python
from autodream_core.adapters.base import BaseAdapter

class MyCustomAdapter(BaseAdapter):
    def read_memory(self) -> str:
        # 实现读取记忆
        pass
    
    def write_memory(self, content: str) -> None:
        # 实现写入记忆
        pass
    
    def list_sessions(self) -> list:
        # 实现会话列表
        pass
```

---

## 📄 许可证

MIT License — 详见 [LICENSE](LICENSE) 文件

---

## 🔗 相关链接

- **GitHub**: https://github.com/Bigkingcn/autodream-core
- **SkillHub**: https://clawhub.ai/skills/autodream-core
- **OpenClaw 文档**: https://docs.openclaw.ai
- **问题反馈**: https://github.com/Bigkingcn/autodream-core/issues

---

## 🙏 致谢

感谢 OpenClaw 社区提供的记忆系统基础架构。

---

**Built with ❤️ by 無生滅 (Bigkingcn)**

*Last updated: 2026-04-03*
