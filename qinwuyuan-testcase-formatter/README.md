# 秦务员测试用例格式化工具 (Qinwuyuan Test Case Formatter)

## 简介

这个工具用于将各种格式的测试用例数据转换为秦务员（陕西政务APP）项目所需的标准化格式。

## 功能特性

- 支持多种输入格式（JSON、XMind格式Markdown、XMind思维导图）
- 自动识别项目和模块结构
- 智能分组测试用例
- 生成符合秦务员规范的Markdown文档
- 可与 `testcase-generator-from-prd` 技能无缝集成

## 使用场景

### 场景1：从PRD生成秦务员格式测试用例

```bash
# 步骤1：使用 testcase-generator-from-prd 从PRD生成XMind格式测试用例
# (假设已完成)

# 步骤2：使用 qinwuyuan-testcase-formatter 转换为秦务员格式
qinwuyuan-formatter --input testcases-xmind.md --output testcases-qinwuyuan.md
```

### 场景2：直接转换JSON数据

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
            { "action": "点击'长辈版'按钮", "validation": "页面有\"常规版\"按钮" }
          ]
        }
      ]
    }
  ]
}
```

### 场景3：整理手动编写的测试用例

将手写的测试用例整理成标准格式，便于向下游交付。

## 格式说明

### 输出格式结构

```
项目名 (Level 1 #)
├── 模块名 (Level 2 ##)
│   ├── 用例名 (Level 3 ###)
│   │   ├── 操作步骤1
│   │   ├── 验证点
│   │   ├── 操作步骤2
│   │   └── 验证点
│   └── 用例名2
└── 模块名2
```

### 关键规则

- 项目名：`# 项目名：XXX`
- 模块名：`## 模块名：XXX`
- 用例名：`### 用例名：XXX`
- 操作步骤：`- 操作步骤N：XXX`
- 验证点：`- 验证：XXX`
- 使用中文冒号 `：`

详细格式规范请参考 `references/qinwuyuan-format-spec.md`

## 完整工作流

```mermaid
graph LR
    A[PRD文档] --> B[testcase-generator-from-prd]
    B --> C[XMind格式测试用例]
    C --> D[qinwuyuan-testcase-formatter]
    D --> E[秦务员格式测试用例]
    E --> F[下游测试团队]
```

## 技能说明

请阅读 `SKILL.md` 了解完整的使用方法和转换逻辑。

## 示例

### 输入示例

```markdown
# 订阅消息管理模块测试用例

## 功能测试

### 订阅列表展示

#### TC-F01-001 - 列表页展示所有订阅记录
- 优先级: P0
- 前置条件: 安全管理员登录系统
- 测试步骤:
  1. 进入"监控告警中心 > 订阅消息管理 > 订阅管理"
  2. 查看页面展示的订阅列表
- 预期结果: 显示所有订阅配置记录
```

### 输出示例

```markdown
# 项目名：陕西秦务员

## 模块名：订阅消息管理

### 用例名：订阅列表展示

- 操作步骤1：进入"监控告警中心 > 订阅消息管理 > 订阅管理"

- 操作步骤2：查看页面展示的订阅列表

- 验证：显示所有订阅配置记录
```

## 注意事项

1. **一个文档一个项目**: 确保输出文档只包含一个项目
2. **步骤编号连续**: 操作步骤编号在同一用例内必须连续
3. **使用中文冒号**: 所有冒号必须使用中文全角冒号 `：`
4. **验证点明确**: 每个验证点都应该清晰可验证
5. **模块命名规范**: 模块名称应该反映功能区域

## 验证检查

交付前请使用以下检查清单：

- [ ] 项目名格式正确
- [ ] 模块名格式正确
- [ ] 用例名格式正确
- [ ] 操作步骤编号连续
- [ ] 使用中文冒号
- [ ] 验证点格式正确
- [ ] 页面元素引用使用引号

详细检查清单请参考 `references/qinwuyuan-format-spec.md` 的"格式检查清单"章节。

## 维护和更新

如有格式变更或新的需求，请更新以下文件：

1. `SKILL.md` - 更新转换逻辑和使用说明
2. `references/qinwuyuan-format-spec.md` - 更新格式规范
3. `README.md` - 更新使用说明

## 相关资源

- `SKILL.md` - 技能完整说明
- `references/qinwuyuan-format-spec.md` - 格式规范详细文档
- `references/` - 其他参考文档

## 联系方式

如有问题或建议，请联系相关负责人。
