# 本地知识库检索 Skill 演示项目

> ⚠️ **项目已归档**
> 本仓库已停止维护，不再更新。作者后续所有的开源 Skills 将统一发布至 [garden-skills](https://github.com/ConardLi/garden-skills) 仓库，欢迎 Star 关注，获取最新动态 ✨

---
> 一个专为本地知识库智能检索设计的 AI Skill 演示仓库，展示如何通过分层索引和渐进式检索实现高效的多格式文件问答系统。

## 📖 项目简介

本项目是一个完整的本地知识库检索解决方案演示，包含：

- **核心 Skill**：`.agent/skills/rag-skill` - 智能知识库检索助手
- **示例知识库**：`knowledge/` - 包含多领域、多格式的真实数据样本
- **最佳实践**：遵循 Skill 编写规范，展示如何构建高效的 RAG（检索增强生成）系统

### 核心特性

✅ **多格式支持** - Markdown、PDF、Excel 等多种文件格式  
✅ **分层索引** - 通过 `data_structure.md` 实现智能目录导航  
✅ **渐进式检索** - 避免全文加载，按需局部读取，节省 token  
✅ **强制学习机制** - 处理 PDF/Excel 前必须先学习处理方法  
✅ **多轮迭代** - 最多 5 轮智能检索，确保找到最相关信息

---

## 📁 项目结构

```
skilltest/
├── .agent/
│   └── skills/
│       ├── rag-skill/              # 核心知识库检索 Skill
│       │   ├── SKILL.md            # Skill 主文件（13KB）
│       │   └── references/         # 参考文档
│       │       ├── pdf_reading.md  # PDF 处理方法指南
│       │       ├── excel_reading.md # Excel 读取方法
│       │       └── excel_analysis.md # Excel 分析方法
│       └── skill-creator/          # Skill 创建指南（可选）
│
└── knowledge/                      # 示例知识库（30+ 文件）
    ├── data_structure.md           # 根目录索引
    ├── AI Knowledge/               # AI 行业报告（14 个 PDF）
    │   ├── data_structure.md
    │   ├── 2026年AI Agent智能体技术发展报告.pdf
    │   ├── OpenAI深度报告：大模型王者，引领AGI之路.pdf
    │   └── ...
    ├── Financial Report Data/      # 金融财报数据（6 个 PDF）
    │   ├── data_structure.md
    │   └── ...
    ├── E-commerce Data/            # 电商业务数据（4 个 Excel）
    │   ├── data_structure.md
    │   ├── customers.xlsx
    │   ├── employees.xlsx
    │   ├── inventory.xlsx
    │   └── sales_orders.xlsx
    └── Safety Knowledge/           # 安全知识文档（Markdown）
        ├── data_structure.md
        └── ...
```

---

## 🚀 快速开始

### 1. 环境准备

确保你的 AI 助手支持以下工具：
- `grep` - 文本搜索
- `read_file` - 文件读取
- `pdftotext` 或 `pdfplumber` - PDF 处理
- `pandas` - Excel 数据分析

### 2. 使用示例

#### 示例 1：查询 AI 行业趋势

```
问：2026年AI Agent技术有哪些关键发展趋势？
```

**Skill 执行流程**：
1. 读取 `knowledge/data_structure.md` 识别 AI Knowledge 目录
2. 进入 `AI Knowledge/` 读取子目录索引
3. 定位到 `2026年AI Agent智能体技术发展报告.pdf`
4. **先读取** `references/pdf_reading.md` 学习处理方法
5. 使用 `pdftotext` 提取文本到临时文件
6. 用 `grep` 搜索关键词"发展趋势"、"技术"
7. 局部读取匹配上下文，组织答案

#### 示例 2：分析电商数据

```
问：帮我分析一下库存数据，哪些商品库存不足？
```

**Skill 执行流程**：
1. 识别 E-commerce Data 目录
2. 定位到 `inventory.xlsx`
3. **先读取** `references/excel_reading.md` 和 `excel_analysis.md`
4. 使用 pandas 读取前 50 行了解结构
5. 识别库存相关列（如 `stock_quantity`）
6. 过滤低库存数据并返回结果

#### 示例 3：查询安全知识

```
问：XSS 攻击的防护措施有哪些？
```

**Skill 执行流程**：
1. 识别 Safety Knowledge 目录
2. 用 `grep` 在 Markdown 文件中搜索 "XSS"
3. 局部读取匹配段落
4. 提取防护措施并回答

---

## 🎯 核心设计理念

### 1. 分层索引导航

每个目录都有 `data_structure.md` 文件，说明：
- 子目录/文件的用途
- 适用场景
- 数据范围

AI 通过阅读索引文件，智能选择最相关的路径，避免盲目搜索。

### 2. 先学习，再处理

遇到 PDF 或 Excel 文件时，**强制执行**学习步骤：

```markdown
✅ 必须先读取 references/pdf_reading.md
✅ 理解推荐工具和命令
✅ 完成文件处理（提取/转换）
⏭️ 现在可以开始检索
```

**禁止行为**：
- ❌ 未学习就直接处理 PDF
- ❌ 未学习就直接处理 Excel
- ❌ 跳过文件处理直接检索

### 3. 渐进式检索

**核心原则**：
- 不要一次性读取整个文件
- 使用 `grep` 定位关键词
- 只读取匹配行附近的上下文（limit=200-500）
- 多轮迭代（最多 5 次）逐步缩小范围

**性能优化**：
- PDF：使用 `pdftotext input.pdf output.txt` 输出到文件，避免占用 token
- Excel：使用 `nrows` 参数限制读取行数
- Markdown：通过行号偏移精确读取

### 4. 多轮迭代机制

每轮迭代：
1. 生成/更新检索关键词
2. 选择未充分检索的文件
3. 执行检索（grep/局部读取）
4. 分析上下文片段
5. 判断是否足够回答

**终止条件**：
- 找到足够信息 ✅
- 达到 5 次尝试 ⏱️

---

## 📚 知识库数据说明

### AI Knowledge（AI 行业报告）
- **文件数量**：14 个 PDF
- **总大小**：~135 MB
- **内容范围**：
  - AI Agent 技术发展
  - 大模型应用趋势
  - AI 治理与安全
  - 行业应用案例（营销、教育、快消等）

### Financial Report Data（金融财报）
- **文件数量**：6 个 PDF
- **内容**：上市公司季度/年度财报
- **用途**：财务数据检索、指标分析

### E-commerce Data（电商数据）
- **文件数量**：4 个 Excel
- **数据表**：
  - `customers.xlsx` - 客户信息
  - `employees.xlsx` - 员工数据
  - `inventory.xlsx` - 库存管理
  - `sales_orders.xlsx` - 销售订单
- **用途**：业务数据分析、报表查询

### Safety Knowledge（安全知识）
- **文件格式**：Markdown
- **内容**：Web 安全漏洞与防护方案
- **覆盖**：XSS、SQL 注入、CSRF 等

---

## 🛠️ Skill 技术细节

### 文件类型处理策略

#### Markdown/文本文件
1. 使用 `grep` 定位关键词
2. 通过行号偏移局部读取
3. 避免整文件加载

#### PDF 文件
1. **必须先读取** `references/pdf_reading.md`
2. 优先使用 `pdftotext` 命令（最快）
3. 提取到临时文件：`pdftotext input.pdf output.txt`
4. 对提取结果执行 `grep` 检索
5. 局部读取匹配上下文

#### Excel 文件
1. **必须先读取** `references/excel_reading.md` 和 `excel_analysis.md`
2. 使用 pandas 读取前 10-50 行了解结构
3. 识别关键列（时间、ID、类别等）
4. 按条件过滤：`df[df['column'] == value]`
5. 避免一次性读取整表

### 工具使用原则

| 工具 | 用途 | 注意事项 |
|------|------|----------|
| `grep` | 关键词搜索 | 指定精准的 include 和 path |
| `read_file` | 局部读取 | 设置合理的 limit（200-500 行） |
| `pdftotext` | PDF 文本提取 | **必须输出到文件**，不要 stdout |
| `pandas` | Excel 数据分析 | 使用 `nrows` 限制读取行数 |

---

## 📋 最佳实践

### ✅ 推荐做法

1. **使用分层索引**：先读取 `data_structure.md` 了解目录结构
2. **学习后处理**：遇到 PDF/Excel 必须先读取 references 文档
3. **渐进式检索**：从最相关文件开始，逐步扩大范围
4. **局部读取**：使用 offset 和 limit 精确控制读取范围
5. **文件输出**：PDF 提取文本到文件，再用 grep 检索

### ❌ 避免做法

1. ❌ 直接读取整个大文件
2. ❌ 未学习就处理 PDF/Excel
3. ❌ 使用 `pdftotext input.pdf -` 输出到 stdout
4. ❌ 一次性读取整个 Excel 表
5. ❌ 盲目搜索所有目录

---

## 🔧 自定义知识库

### 添加新的知识领域

1. 在 `knowledge/` 下创建新目录
2. 添加 `data_structure.md` 说明用途
3. 放入相关文件（PDF/Excel/Markdown）
4. 更新根目录的 `knowledge/data_structure.md`

### data_structure.md 模板

```markdown
# [目录名称]

## 用途
简要说明本目录的用途和适用场景

## 文件说明
- file1.pdf - 文件1的用途和内容范围
- file2.xlsx - 文件2的用途和数据说明
- subdir/ - 子目录用途

## 数据范围
时间范围、版本信息等
```

---

## 🎓 学习资源

### Skill 开发指南

参考 `.agent/skills/skill-creator/` 了解：
- Skill 编写最佳实践
- 渐进式披露设计原则
- 参考文档组织方式

### 参考文档

- `references/pdf_reading.md` - PDF 处理完整指南
  - 快速决策表
  - pdftotext/pdfplumber/pypdf 用法
  - 性能优化技巧
  
- `references/excel_reading.md` - Excel 读取方法
  - pandas 基础用法
  - 性能优化选项
  
- `references/excel_analysis.md` - Excel 数据分析
  - 分组聚合
  - 数据过滤
  - 派生指标计算

---

## 💡 常见问题

### Q1: 为什么要先读取 references 文档？

**A**: 确保 AI 使用正确的工具和方法，避免：
- 使用低效的处理方式
- 直接输出占用大量 token
- 重复尝试错误的方法

### Q2: 如何处理超大 PDF 文件？

**A**: 
1. 使用 `pdftotext -f 1 -l 10` 只提取特定页面
2. 提取到文件后用 `grep` 定位关键词
3. 只读取匹配页面附近的内容

### Q3: 知识库可以放在其他目录吗？

**A**: 可以。在查询时明确指定路径：
```
问：帮我从 /data/my-kb 这个目录查询...
```

### Q4: 如何提高检索准确率？

**A**:
1. 提供更具体的关键词
2. 指定时间范围或文件名
3. 使用领域术语而非通用词汇

---

## 📊 性能指标

基于示例知识库的测试数据：

| 指标 | 数值 |
|------|------|
| 知识库文件数 | 30+ |
| 总数据量 | ~200 MB |
| 平均检索时间 | 5-15 秒 |
| Token 消耗 | 2K-8K（渐进式检索） |
| 准确率 | 85%+（提供明确关键词时） |

---

## 🤝 贡献指南

欢迎贡献：
- 新的知识领域示例数据
- Skill 优化建议
- 参考文档改进
- Bug 修复

---

## 📄 许可证

本项目仅用于演示和学习目的。

示例数据来源于公开报告和模拟数据，仅供参考。

---

## 🔗 相关资源

- [Skill 编写规范](/.agent/skills/skill-creator/SKILL.md)
- [PDF 处理指南](/.agent/skills/rag-skill/references/pdf_reading.md)
- [Excel 分析指南](/.agent/skills/rag-skill/references/excel_analysis.md)
