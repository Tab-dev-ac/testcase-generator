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

| testcase-generator-from-prd | qinwuyuan-testcase-formatter |
|----------------------------|------------------------------|
| Root node (# 测试用例) | Project name |
| Level 2 nodes (## 模块) | Module name |
| Level 4 nodes (#### TC-XXX) | Testcase name |
| - 测试步骤: 1. ... | - 操作步骤1：... |
| - 预期结果: ... | - 验证：... |
| - 优先级: P0/P1/P2 | (Optional, can be noted in comments) |

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

- [x] Project name format: `# 项目名：XXX`
- [x] Module name format: `## 模块名：XXX`
- [x] Testcase name format: `### 用例名：XXX`
- [x] Step format: `- 操作步骤N：XXX`
- [x] Validation format: `- 验证：XXX`
- [x] Chinese colon `：` used consistently
- [x] Empty line between modules (optional but recommended)
- [x] Empty line between testcases (optional but recommended)

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
