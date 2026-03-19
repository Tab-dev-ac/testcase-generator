# 使用指南

## 在OpenClaw中使用这两个技能

### 方式1：直接使用（推荐用于开发环境）

在OpenClaw聊天界面中，直接引用技能：

```bash
# 步骤1：从PRD生成XMind格式测试用例
请使用 testcase-generator-from-prd 技能，
读取 /Users/tablee/testcase-generator/PRD/PRD-订阅消息管理-2026-03-13.md
并生成测试用例，输出为 testcases-xmind.md

# 步骤2：转换为秦务员格式
请使用 qinwuyuan-testcase-formatter 技能，
读取 testcases-xmind.md
并转换为秦务员格式，输出为 testcases-qinwuyuan.md
```

### 方式2：安装到OpenClaw系统

如果需要在不同项目中反复使用，可以将技能安装到OpenClaw系统目录：

```bash
# 安装技能到OpenClaw
sudo cp -r /Users/tablee/testcase-generator/testcase-generator-from-prd \
           /opt/homebrew/lib/node_modules/openclaw/skills/

sudo cp -r /Users/tablee/testcase-generator/qinwuyuan-testcase-formatter \
           /opt/homebrew/lib/node_modules/openclaw/skills/

# 安装后可以直接使用技能名称，无需指定路径
```

## 完整使用示例

### 示例1：从PRD到秦务员格式的完整流程

```bash
# 在OpenClaw中执行以下操作

# 1. 读取PRD文档
read /Users/tablee/testcase-generator/PRD/PRD-订阅消息管理-2026-03-13.md

# 2. 生成XMind格式测试用例
# OpenClaw会自动识别使用 testcase-generator-from-prd 技能
# 分析PRD并输出 XMind 格式的测试用例

# 3. 读取生成的XMind格式测试用例
read testcases-xmind.md

# 4. 转换为秦务员格式
# OpenClaw会自动识别使用 qinwuyuan-testcase-formatter 技能
# 输出符合秦务员规范的Markdown文档
```

### 示例2：处理JSON格式的测试用例

```bash
# 如果你有JSON格式的测试用例数据

# 创建JSON文件
cat > testcases.json << 'EOF'
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
EOF

# 在OpenClaw中转换
read testcases.json
# 使用 qinwuyuan-testcase-formatter 技能转换为秦务员格式
```

## 常见问题

### Q1: 如何确保格式正确？

A: 使用 qinwuyuan-testcase-formatter 技能时，它会自动：
- 检查并使用中文冒号 `：`
- 确保步骤编号连续
- 验证层级结构正确

参考：`qinwuyuan-testcase-formatter/references/qinwuyuan-format-spec.md`

### Q2: 如何自定义项目名称？

A: 有两种方式：

1. 在JSON输入中指定：
```json
{
  "project": "你的项目名称",
  ...
}
```

2. 在请求时指定：
```bash
请将测试用例转换为秦务员格式，项目名称为"我的项目"
```

### Q3: 如何分组模块？

A: 技能支持智能分组：

1. **按功能区域**（推荐）：自动首页、自动搜索、自动办事页
2. **按测试类型**：功能测试、业务流程测试、权限测试
3. **自定义分组**：在输入数据中明确指定模块名称

参考：`qinwuyuan-testcase-formatter/references/conversion-examples.md`

### Q4: 如何处理验证点？

A: 验证点可以：

1. 跟在每个步骤后（推荐）：
```markdown
- 操作步骤1：点击按钮
- 验证：页面显示正确
```

2. 在多个步骤后汇总：
```markdown
- 操作步骤1：步骤1
- 操作步骤2：步骤2
- 验证：所有步骤执行成功
```

## 输出格式检查

生成秦务员格式测试用例后，使用以下检查清单验证：

- [ ] 项目名格式：`# 项目名：XXX`
- [ ] 模块名格式：`## 模块名：XXX`
- [ ] 用例名格式：`### 用例名：XXX`
- [ ] 操作步骤：`- 操作步骤N：XXX`
- [ ] 验证点：`- 验证：XXX`
- [ ] 使用中文冒号 `：`
- [ ] 步骤编号连续
- [ ] 页面元素使用引号

## 进阶使用

### 批量处理多个PRD

```bash
# 创建批量处理脚本
cat > batch-process.sh << 'EOF'
#!/bin/bash

PRD_DIR="/Users/tablee/testcase-generator/PRD"
OUTPUT_DIR="/Users/tablee/testcase-generator/output"

mkdir -p $OUTPUT_DIR

for prd in $PRD_DIR/*.md; do
  filename=$(basename "$prd")
  echo "处理: $filename"

  # 在OpenClaw中处理每个PRD
  # read "$prd"
  # 生成XMind格式
  # 转换为秦务员格式
done
EOF

chmod +x batch-process.sh
```

### 自定义转换规则

如果需要自定义转换规则，可以：

1. 修改 `qinwuyuan-testcase-formatter/SKILL.md`
2. 更新转换逻辑和映射关系
3. 添加新的转换示例到 `references/conversion-examples.md`

## 性能优化建议

1. **批量处理**：一次处理多个PRD，减少重复初始化开销
2. **缓存中间结果**：保存XMind格式结果，避免重复生成
3. **并行处理**：如果处理大量PRD，可以考虑并行处理

## 集成到CI/CD

可以将测试用例生成集成到CI/CD流程中：

```yaml
# .github/workflows/generate-testcases.yml
name: Generate Test Cases

on:
  push:
    paths:
      - 'PRD/**/*.md'

jobs:
  generate:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v2

      - name: Generate XMind format
        run: |
          # 调用OpenClaw生成XMind格式测试用例

      - name: Convert to Qinwuyuan format
        run: |
          # 调用OpenClaw转换为秦务员格式

      - name: Upload artifacts
        uses: actions/upload-artifact@v2
        with:
          name: testcases
          path: output/*.md
```

## 支持

如遇问题，请：

1. 查看技能文档：`SKILL.md`
2. 查看格式规范：`references/qinwuyuan-format-spec.md`
3. 查看转换示例：`references/conversion-examples.md`
4. 检查OpenClaw日志和错误信息
