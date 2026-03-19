---
name: testcase-generator-from-prd
description: Analyze PRD documents (Markdown format) and generate comprehensive test cases in XMind-compatible format. Use when you need to: (1) Generate test cases from a PRD document, (2) Create test mind maps for testing teams, (3) Extract functional, workflow, permission, and exception test cases from requirements, (4) Convert PRD acceptance criteria into structured test cases.
---

# Testcase Generator From Prd

## Overview

Generate comprehensive test cases from Product Requirement Documents (PRD) and output them in XMind-compatible Markdown format for easy import into XMind mind mapping software.

## Workflow Decision Tree

```
收到 PRD → 读取 PRD 文档 → 分析 PRD 结构 → 生成测试用例 → 输出 XMind 格式
             │                    │                │
             ↓                    ↓                ↓
         Markdown 格式      识别关键章节      六大测试类别
                              (功能/流程/      功能/流程/
                              字段/权限/       权限/界面/
                              交互/异常)       异常/验收)
```

## Quick Start

### 基本使用

```bash
# 1. 提供你的 PRD 文档路径
PRD_PATH="/Users/tablee/Downloads/PRD-订阅消息管理-2026-03-13.md"

# 2. 读取 PRD 内容
read $PRD_PATH

# 3. 分析 PRD 并生成测试用例
# Codex 会自动:
# - 识别功能列表 (3.1)
# - 提取业务流程 (2.1, 2.2)
# - 解析字段定义 (3.5)
# - 分析权限矩阵 (3.7)
# - 提取交互规则 (3.4)
# - 识别异常处理 (3.8)
# - 转换验收标准 (3.10)

# 4. 输出为 XMind 格式的 Markdown 文件
write output-testcases-xmind.md
```

### 使用示例

用户请求示例：
- "根据这个 PRD 生成测试用例：/path/to/prd.md"
- "从 PRD 生成测试用例并输出为 XMind 格式"
- "分析这个 PRD，帮我创建完整的测试用例文档"

## Testing Categories

### 1. 功能测试 (TC-F)

从 PRD 功能列表 (3.1) 提取，每个功能点生成：
- **正向用例**: 按预期操作，验证功能正常工作
- **负向用例**: 异常输入、缺失必填字段、错误数据
- **边界用例**: 最大/最小长度、空值、特殊字符

**从 PRD 映射**:
- 功能列表 → 为每个功能点生成测试用例
- 字段定义 → 为每个字段的校验规则生成验证用例

### 2. 业务流程测试 (TC-BP)

从 PRD 业务流程 (2.1, 2.2) 提取：
- **主流程**: 完整的业务操作流程
- **分支流程**: 不同条件下的处理分支
- **异常流程**: 错误处理和恢复流程

**从 PRD 映射**:
- 流程步骤表 → 为每个步骤生成场景测试
- 流程图 → 提取分支条件生成分支用例

### 3. 权限测试 (TC-PERM)

从 PRD 权限控制矩阵 (3.7) 提取：
- **角色权限验证**: 每个角色的功能权限
- **功能访问控制**: 权限限制的验证
- **数据可见性**: 不同角色看到的数据范围

**从 PRD 映射**:
- 权限矩阵 → 为每个角色×每个功能生成权限测试

### 4. 界面测试 (TC-UI)

从 PRD 页面布局 (3.3) 和交互规则 (3.4) 提取：
- **布局验证**: 页面元素位置和展示
- **响应式测试**: 不同屏幕尺寸下的显示
- **交互验证**: 按钮状态、弹窗交互、实时校验

**从 PRD 映射**:
- 页面布局图 → 验证布局正确性
- 交互规则表 → 验证交互行为

### 5. 异常测试 (TC-EX)

从 PRD 异常处理 (3.8) 提取：
- **网络异常**: 网络请求失败的处理
- **数据异常**: 空数据、重复数据、无效数据
- **系统异常**: 会话超时、服务器错误

**从 PRD 映射**:
- 异常处理表 → 为每个异常场景生成测试用例

### 6. 验收测试 (TC-AC)

直接从 PRD 验收标准 (3.10) 提取：
- 将每个验收标准 (AC01, AC02...) 转换为测试用例
- 验收测试通常覆盖功能和交互

## Test Case Format

每个测试用例包含以下字段：

```
TC-模块标识-序号: 测试用例标题
- 优先级: P0/P1/P2
- 前置条件: 登录系统，拥有相应权限等
- 测试步骤:
  1. 步骤1
  2. 步骤2
- 预期结果: 预期的行为或结果
- 状态: 待执行/通过/失败/阻塞
```

### 优先级定义

- **P0**: 核心功能，影响业务闭环，阻塞发布
- **P1**: 重要功能，影响用户体验，但可降级处理
- **P2**: 一般功能，锦上添花或边缘场景

### 测试用例 ID 规则

- **TC**: Test Case（测试用例）
- **模块标识**:
  - F: Function（功能测试）
  - BP: Business Process（业务流程测试）
  - PERM: Permission（权限测试）
  - UI: User Interface（界面测试）
  - EX: Exception（异常测试）
  - AC: Acceptance Criteria（验收测试）
- **序号**: 三位数字（001, 002, ...）

示例:
- `TC-F01-001`: 功能测试-列表展示-第1个用例
- `TC-BP-001`: 业务流程测试-主流程-第1个用例

## XMind Output Format

生成符合 XMind 导入格式的 Markdown 文件：

```markdown
# 根节点（测试用例）

## 模块1 - 功能测试

### 功能点1

#### TC-F01-001 - 测试用例标题
- 优先级: P0
- 前置条件: ...
- 测试步骤:
  1. ...
- 预期结果: ...
- 状态: 待执行
```

**导入 XMind 方法**:
1. 打开 XMind 软件
2. 点击"文件" → "导入" → "Markdown"
3. 选择生成的 .md 文件
4. XMind 自动转换为思维导图

## PRD Analysis Tips

### 关键章节识别

| PRD 章节 | 测试类型 | 提取方法 |
|---------|---------|---------|
| 功能列表 (3.1) | 功能测试 | 每个功能点生成正向/负向用例 |
| 业务流程 (2.1, 2.2) | 流程测试 | 为每个流程步骤生成场景 |
| 页面布局 (3.3) | 界面测试 | 验证布局、样式、响应式 |
| 字段定义 (3.5) | 字段验证 | 为每个校验规则生成测试 |
| 交互规则 (3.4) | 交互测试 | 验证交互规则和状态变化 |
| 数据字典 (3.6) | 数据验证 | 验证枚举值、默认值 |
| 权限矩阵 (3.7) | 权限测试 | 验证每个角色权限 |
| 异常处理 (3.8) | 异常测试 | 验证异常场景处理 |
| 验收标准 (3.10) | 验收测试 | 直接转换为测试用例 |

### 缺陷标注

如果 PRD 缺少以下信息，在测试用例备注中标注：
- 缺少异常场景的处理说明
- 缺少边界值的明确定义
- 缺少性能和并发相关要求
- 缺少安全相关要求

## Resources

### references/test-coverage-strategy.md

详细的测试覆盖率策略，包括：
- 六大测试类别的定义和覆盖维度
- 测试用例设计模板
- 从 PRD 提取测试用例的映射关系表
- 优先级定义

### references/xmind-format-template.md

XMind 输出格式的完整指南，包括：
- Markdown 格式规则
- 完整的示例（订阅管理模块测试用例）
- XMind 节点层级说明
- 测试用例 ID 规则
- 导入 XMind 的详细方法
