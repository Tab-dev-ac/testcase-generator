---
name: qinwuyuan-testcase-formatter
description: Convert test case data into Qinwuyuan (秦务员) format for downstream delivery. Use when you need to: (1) Transform test cases from various formats into Qinwuyuan's project-module-testcase structure, (2) Generate formatted test case documents for testing teams, (3) Standardize test cases following Qinwuyuan's specification.
---

# Qinwuyuan Test Case Formatter

## Overview

Convert structured test case data into Qinwuyuan (秦务员) format with the hierarchical structure: **Project → Module → Testcase**. Each testcase contains multiple operation steps with validations.

## Target Format

```markdown
# 项目名：[项目名称]

## 模块名：[模块名称]

### 用例名：[测试用例名称]

- 操作步骤1：[操作描述]

- 验证：[验证点]

- 操作步骤2：[操作描述]

- 验证：[验证点]
```

### Format Rules

1. **项目层级**: `# 项目名：XXX` - Top level, one project per document
2. **模块层级**: `## 模块名：XXX` - Second level, each module under the project
3. **用例层级**: `### 用例名：XXX` - Third level, each testcase under the module
4. **步骤格式**: `- 操作步骤N：[描述]` - Numbered steps with Chinese colon
5. **验证格式**: `- 验证：[验证点]` - Validation point (optional after each step or after multiple steps)

### Numbering

- 操作步骤使用数字 1, 2, 3... 自动递增
- 步骤编号在同一用例内唯一
- 验证点可以使用"- 验证："或"- 验证：[具体内容]"

## Quick Start

### Basic Usage

```bash
# 1. Provide your test case data (JSON, Markdown, or other formats)
TESTCASE_DATA="/path/to/testcases.json"

# 2. Read the source data
read $TESTCASE_DATA

# 3. Analyze the data structure and extract test cases
# The formatter will identify:
# - Project name
# - Module names and groupings
# - Test case names
# - Operation steps and validation points

# 4. Output as Qinwuyuan format
write output-qinwuyuan-testcases.md
```

### Usage Examples

User requests:
- "将这些测试用例转换为秦务员格式"
- "生成满足秦务员格式的测试用例文档"
- "把测试用例按项目-模块-用例格式整理"

## Input Format Support

### JSON Format

```json
{
  "project": "陕西秦务员",
  "modules": [
    {
      "name": "自动首页",
      "testcases": [
        {
          "name": "常规版切换到长辈版测试",
          "steps": [
            { "action": "点击'长辈版'按钮", "validation": "页面有\"常规版\"按钮" },
            { "action": "点击'常规版'按钮", "validation": "页面有\"长辈版\"按钮" }
          ]
        }
      ]
    }
  ]
}
```

### XMind Format (from testcase-generator-from-prd)

```markdown
# 测试用例

## 功能测试

### 列表展示

#### TC-F01-001 - 列表页展示所有记录
- 优先级: P0
- 测试步骤:
  1. 进入订阅管理页面
  2. 查看页面展示的列表
- 预期结果: 显示所有订阅配置记录
```

**转换后（去除优先级、测试步骤编号、预期结果标题）**：
```markdown
### 用例名：列表页展示所有记录

- 操作步骤1：进入订阅管理页面

- 操作步骤2：查看页面展示的列表

- 验证：显示所有订阅配置记录
```

**注意**：
- ❌ 去除：优先级 (P0/P1/P2)
- ❌ 去除：测试用例编号 (TC-F01-001)
- ❌ 去除："- 测试步骤:" 标题行
- ❌ 去除："- 预期结果:" 标题行（直接保留验证内容）
- ✅ 保留：操作步骤内容
- ✅ 保留：验证内容

### Direct Description

Plain text descriptions of test cases that can be parsed into the structured format.

## Conversion Logic

### Project Identification

- Check for `project` field in data
- If not found, ask user for project name or use default: "未命名项目"

### Module Grouping

Strategy 1: **By Functional Area** (推荐)
- Analyze testcase keywords to group by functional modules
- Common module names: "首页", "搜索", "办事页", "用户中心", "设置"

Strategy 2: **By Test Type**
- Group by: "功能测试", "业务流程测试", "权限测试", "界面测试", "异常测试"

Strategy 3: **Custom Grouping**
- User provides explicit module assignments
- Preserve module structure from source data

### Testcase Extraction

For each testcase, extract:
- **Name**: Testcase title/name
- **Steps**: Operation steps (有序)
- **Validations**: Validation points (可穿插或汇总)

**重要：只保留模板中存在的字段**

根据 `/Users/tablee/Downloads/秦务员用例格式模板.md`，最终输出只包含以下字段：
- ✅ 项目名 (`# 项目名：XXX`)
- ✅ 模块名 (`## 模块名：XXX`)
- ✅ 用例名 (`### 用例名：XXX`)
- ✅ 操作步骤 (`- 操作步骤N：XXX`)
- ✅ 验证点 (`- 验证：XXX`)

**以下字段必须去除**：
- ❌ 优先级 (P0/P1/P2)
- ❌ 前置条件 (转换为步骤或去除)
- ❌ 状态 (待执行/通过/失败)
- ❌ 测试用例编号 (TC-XXX)
- ❌ 其他元数据标签

### Step Numbering

```javascript
// Auto-increment step numbers within each testcase
let stepNumber = 1;
for (step of testcase.steps) {
  output += `- 操作步骤${stepNumber}：${step.action}\n`;
  if (step.validation) {
    output += `- 验证：${step.validation}\n`;
  }
  stepNumber++;
}
```

### 前置条件处理规则

**原则**：前置条件应尽可能转化为操作步骤，如果信息不足则去除。

**情况1：可以转化为操作步骤**
当前置条件信息完整、可执行时，将其转换为第一个操作步骤：
```
输入：
- 前置条件: 安全管理员登录系统

转换后：
- 操作步骤1：安全管理员登录系统
```

**情况2：作为前置步骤保留**
当前置条件描述了多个准备工作时：
```
输入：
- 前置条件: 已登录系统，拥有订阅管理权限

转换后：
- 操作步骤1：登录系统
- 操作步骤2：确认拥有订阅管理权限
```

**情况3：信息不足，直接去除**
当前置条件过于抽象或无法直接执行时：
```
输入：
- 前置条件: 系统正常运行
- 前置条件: 数据已准备好

转换后：直接去除，不生成任何步骤
```

**处理逻辑**：
1. 检查前置条件是否包含可执行的动作
2. 如果包含动作动词（登录、进入、点击等），转换为操作步骤
3. 如果只是状态描述（正常、准备就绪等），直接去除
4. 不生成单独的"- 前置条件："行

## Output Generation

### Template

```markdown
# 项目名：{{projectName}}

{{#each modules}}
## 模块名：{{name}}

{{#each testcases}}
### 用例名：{{name}}

{{#each steps}}
- 操作步骤{{@index}}：{{action}}

{{#if validation}}
- 验证：{{validation}}
{{/if}}
{{/each}}

{{/each}}
{{/each}}
```

### Example Output

```markdown
# 项目名：陕西秦务员

## 模块名：自动首页

### 用例名：常规版切换到长辈版测试

- 操作步骤1：点击'长辈版'按钮

- 验证：页面有"常规版"按钮

- 操作步骤2：点击'常规版'按钮

- 验证：'页面有"长辈版"按钮'

### 用例名：点击全部跳转办事页

- 操作步骤1：点击'办事'按钮

- 验证：页面有"立即添加"

## 模块名：自动搜索

### 用例名：首页搜索

- 操作步骤1：点击页面中部"陕西省"后的输入框

- 操作步骤2：搜索框中输入'宗教政策及'

- 验证：页面有'宗教政策及法律法规咨询'
```

## Integration with testcase-generator-from-prd

This formatter can work as a post-processing step for the `testcase-generator-from-prd` skill:

```
PRD → testcase-generator-from-prd → XMind Format → qinwuyuan-testcase-formatter → Qinwuyuan Format
```

### Workflow

```bash
# Step 1: Generate test cases from PRD using testcase-generator-from-prd
# (Produces XMind format)

# Step 2: Convert to Qinwuyuan format using qinwuyuan-testcase-formatter
# (Produces Qinwuyuan format for downstream delivery)
```

### Mapping Table

| testcase-generator-from-prd | qinwuyuan-testcase-formatter | 说明 |
|----------------------------|------------------------------|------|
| Root node (# 测试用例) | Project name | 项目名 |
| Level 2 nodes (## 模块) | Module name | 模块名 |
| Level 4 nodes (#### TC-XXX) | Testcase name | 用例名（去除编号） |
| - 测试步骤: 1. ... | - 操作步骤1：... | 步骤内容 |
| - 预期结果: ... | - 验证：... | 验证内容 |
| - 优先级: P0/P1/P2 | ❌ **去除** | 不在模板中 |
| - 前置条件: ... | 操作步骤1 或 ❌ **去除** | 转换为步骤或去除 |
| - 状态: 待执行 | ❌ **去除** | 不在模板中 |
| TC-XXX 编号 | ❌ **去除** | 不在模板中 |

## Common Use Cases

### Use Case 1: Convert XMind Test Cases to Qinwuyuan Format

```bash
# Input: testcases-xmind.md (from testcase-generator-from-prd)
# Output: testcases-qinwuyuan.md
read testcases-xmind.md
# Analyze and convert
write testcases-qinwuyuan.md
```

### Use Case 2: Convert JSON Test Data to Qinwuyuan Format

```bash
# Input: testcases.json
# Output: testcases-qinwuyuan.md
read testcases.json
# Parse JSON, extract structure, generate output
write testcases-qinwuyuan.md
```

### Use Case 3: Generate Directly from User Description

```bash
# User provides: "测试首页切换功能"
# Generate structured output with inferred module grouping
```

## Validation

### Format Checklist

**必须包含**：
- [x] 项目名格式: `# 项目名：XXX`
- [x] 模块名格式: `## 模块名：XXX`
- [x] 用例名格式: `### 用例名：XXX`
- [x] 操作步骤格式: `- 操作步骤N：XXX`
- [x] 验证点格式: `- 验证：XXX`
- [x] 使用中文冒号 `：`

**必须去除**：
- [x] ❌ 优先级 (P0/P1/P2)
- [x] ❌ 前置条件 (已转换为步骤或去除)
- [x] ❌ 状态 (待执行/通过/失败)
- [x] ❌ 测试用例编号 (TC-F01-001等)
- [x] ❌ 其他元数据标签

**格式规范**：
- [x] 步骤编号连续（同一用例内从1开始）
- [x] 验证点紧跟对应步骤或汇总在最后
- [x] 模块之间可选空行分隔
- [x] 用例之间可选空行分隔

### Common Mistakes to Avoid

- ❌ Using English colon `:` instead of Chinese `：`
- ❌ Missing space after colon
- ❌ Incorrect heading level (use #, ##, ### correctly)
- ❌ Skipping step numbers
- ❌ Inconsistent indentation (should be no indentation for list items)

## Resources

### references/qinwuyuan-format-spec.md

Complete specification of Qinwuyuan test case format, including:
- Detailed format rules with examples
- Common patterns and variations
- Validation rules
- Integration guidelines with downstream systems
