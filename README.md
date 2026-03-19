# Testcase Generator 工程项目

## 概述

这是一个测试用例生成和格式化工具项目，包含两个核心技能：

1. **testcase-generator-from-prd**: 从PRD文档生成XMind格式的测试用例
2. **qinwuyuan-testcase-formatter**: 将测试用例转换为秦务员（陕西秦务员）格式

## 项目结构

```
testcase-generator/
├── testcase-generator-from-prd/      # 从PRD生成测试用例
│   ├── SKILL.md                      # 技能文档
│   └── references/
│       ├── test-coverage-strategy.md
│       └── xmind-format-template.md
├── qinwuyuan-testcase-formatter/     # 格式化为秦务员格式
│   ├── SKILL.md                      # 技能文档
│   ├── README.md                     # 使用说明
│   └── references/
│       ├── qinwuyuan-format-spec.md  # 格式规范
│       └── conversion-examples.md    # 转换示例
├── PRD/                              # PRD文档目录
│   └── PRD-订阅消息管理-2026-03-13.md
└── 订阅消息管理模块测试用例-xmind.md  # 示例输出
```

## 快速开始

### 完整工作流

```bash
# 步骤1：使用 testcase-generator-from-prd 从PRD生成XMind格式测试用例
# (这会在Codex/Claude Code中完成)

# 步骤2：使用 qinwuyuan-testcase-formatter 转换为秦务员格式
# (这会在Codex/Claude Code中完成)
```

### 示例：在OpenClaw中使用

```bash
# 1. 读取PRD文档
read /Users/tablee/testcase-generator/PRD/PRD-订阅消息管理-2026-13.md

# 2. 使用 testcase-generator-from-prd 技能生成XMind格式测试用例
# Codex会自动分析PRD并生成测试用例

# 3. 将XMind格式转换为秦务员格式
# 使用 qinwuyuan-testcase-formatter 技能
```

## 技能说明

### testcase-generator-from-prd

**功能**:
- 从PRD文档（Markdown格式）生成全面的测试用例
- 输出XMind兼容的Markdown格式
- 支持六大测试类别：功能、流程、权限、界面、异常、验收

**使用场景**:
- 从PRD生成测试用例
- 创建测试思维导图
- 提取功能、流程、权限等测试用例

**详细文档**: [testcase-generator-from-prd/SKILL.md](testcase-generator-from-prd/SKILL.md)

### qinwuyuan-testcase-formatter

**功能**:
- 将各种格式的测试用例转换为秦务员（陕西秦务员）格式
- 支持JSON和XMind格式输入
- 生成标准化的Markdown文档

**目标格式**:
```markdown
# 项目名：陕西秦务员

## 模块名：自动首页

### 用例名：常规版切换到长辈版测试

- 操作步骤1：点击'长辈版'按钮

- 验证：页面有"常规版"按钮
```

**使用场景**:
- 向下游测试团队交付测试用例
- 标准化测试用例格式
- 整理和转换现有测试用例

**详细文档**: [qinwuyuan-testcase-formatter/SKILL.md](qinwuyuan-testcase-formatter/SKILL.md)

## 工作流程

```
PRD文档
    ↓
testcase-generator-from-prd
    ↓
XMind格式测试用例
    ↓
qinwuyuan-testcase-formatter
    ↓
秦务员格式测试用例
    ↓
下游测试团队
```

## 集成到OpenClaw

### 安装技能

如果需要将这些技能安装到OpenClaw中：

```bash
# 方法1：直接复制到OpenClaw skills目录
cp -r /Users/tablee/testcase-generator/testcase-generator-from-prd \
      /opt/homebrew/lib/node_modules/openclaw/skills/

cp -r /Users/tablee/testcase-generator/qinwuyuan-testcase-formatter \
      /opt/homebrew/lib/node_modules/openclaw/skills/
```

### 使用技能

安装后，在OpenClaw中可以直接引用这些技能：

```bash
# 生成测试用例
使用 testcase-generator-from-prd 技能读取PRD并生成测试用例

# 格式转换
使用 qinwuyuan-testcase-formatter 技能将测试用例转换为秦务员格式
```

## 示例文件

### 输入示例

PRD文档示例：`PRD/PRD-订阅消息管理-2026-03-13.md`

### 输出示例

1. XMind格式：`订阅消息管理模块测试用例-xmind.md`
2. 秦务员格式：需要使用 qinwuyuan-testcase-formatter 技能生成

## 开发和维护

### 贡献指南

1. 修改技能文档：编辑对应的 `SKILL.md` 文件
2. 更新参考文档：修改 `references/` 目录下的文档
3. 添加示例：在 `references/conversion-examples.md` 中添加新示例

### 版本控制

本项目使用Git进行版本管理：

```bash
# 查看状态
git status

# 提交更改
git add .
git commit -m "描述更改"

# 推送到远程
git push origin main
```

## 相关资源

- [testcase-generator-from-prd 技能文档](testcase-generator-from-prd/SKILL.md)
- [qinwuyuan-testcase-formatter 技能文档](qinwuyuan-testcase-formatter/SKILL.md)
- [秦务员格式规范](qinwuyuan-testcase-formatter/references/qinwuyuan-format-spec.md)
- [转换示例](qinwuyuan-testcase-formatter/references/conversion-examples.md)

## 许可证

[根据需要添加许可证信息]

## 联系方式

[根据需要添加联系信息]
