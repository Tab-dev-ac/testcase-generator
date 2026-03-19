# 测试用例提取算法

## 概述

本文档详细说明了如何从XMind格式的测试用例中完整提取所有测试用例。

## 完整提取流程

### 第一步：解析文档结构

```python
def parse_xmind_document(content):
    """
    解析XMind格式的Markdown文档
    """
    document = {
        "title": None,  # 根节点标题
        "test_types": {}  # 测试类型及其测试用例
    }

    # 提取根节点
    document["title"] = extract_root_node(content)

    # 提取所有测试类型
    for test_type in ALL_TEST_TYPES:
        if test_type in content:
            testcases = extract_testcases_under_type(content, test_type)
            document["test_types"][test_type] = testcases

    return document
```

### 第二步：提取每个测试类型下的所有测试用例

```python
ALL_TEST_TYPES = [
    "功能测试",
    "业务流程测试",
    "权限测试",
    "界面测试",
    "异常测试",
    "验收测试"
]

def extract_testcases_under_type(content, test_type):
    """
    提取指定测试类型下的所有测试用例
    """
    testcases = []

    # 找到测试类型标题 (## 功能测试)
    type_section = find_section(content, f"## {test_type}")

    if not type_section:
        return testcases

    # 提取该类型下的所有功能点 (### F01 - 订阅列表展示)
    function_points = extract_sections(type_section, level=3)

    for function_point in function_points:
        # 提取功能点下的所有测试用例 (#### TC-F01-001)
        tc_sections = extract_sections(function_point, level=4)

        for tc_section in tc_sections:
            testcase = parse_testcase(tc_section)
            testcases.append(testcase)

    return testcases
```

### 第三步：解析单个测试用例

```python
def parse_testcase(tc_section):
    """
    解析单个测试用例的详细信息
    """
    testcase = {
        "id": None,  # TC-F01-001
        "name": None,  # 列表页展示所有订阅记录
        "priority": None,  # P0
        "precondition": None,  # 安全管理员登录系统
        "steps": [],  # 操作步骤列表
        "expected_result": None,  # 预期结果
        "status": None  # 待执行
    }

    lines = tc_section.split('\n')

    for line in lines:
        line = line.strip()

        # 提取用例编号和名称
        if line.startswith('#### TC-'):
            testcase["id"] = extract_tc_id(line)
            testcase["name"] = extract_tc_name(line)

        # 提取优先级
        elif line.startswith('- 优先级:'):
            testcase["priority"] = extract_priority(line)

        # 提取前置条件
        elif line.startswith('- 前置条件:'):
            testcase["precondition"] = extract_precondition(line)

        # 提取测试步骤
        elif line.startswith('- 测试步骤:'):
            steps_text = extract_multiline_text(lines, line)
            testcase["steps"] = parse_steps(steps_text)

        # 提取预期结果
        elif line.startswith('- 预期结果:'):
            testcase["expected_result"] = extract_expected_result(line)

        # 提取状态
        elif line.startswith('- 状态:'):
            testcase["status"] = extract_status(line)

    return testcase
```

### 第四步：解析操作步骤

```python
def parse_steps(steps_text):
    """
    解析测试步骤文本
    """
    steps = []

    # 步骤格式:
    #   1. 进入订阅管理页面
    #   2. 查看页面

    lines = steps_text.split('\n')

    for line in lines:
        line = line.strip()

        # 跳过标题行
        if line.startswith('- 测试步骤:'):
            continue

        # 解析步骤 (数字开头的行)
        if re.match(r'^\d+\.\s+', line):
            step_number = int(re.match(r'^(\d+)\.', line).group(1))
            step_description = re.sub(r'^\d+\.\s+', '', line)

            steps.append({
                "number": step_number,
                "description": step_description
            })

    return steps
```

### 第五步：模块分组

```python
def group_by_module(testcases):
    """
    将测试用例按模块分组
    """
    modules = {}

    for testcase in testcases:
        # 从用例名称或ID推断模块
        module = infer_module(testcase)

        if module not in modules:
            modules[module] = []

        modules[module].append(testcase)

    return modules

def infer_module(testcase):
    """
    根据测试用例推断所属模块
    """
    # 方式1: 从功能点标题提取 (### F01 - 订阅列表展示)
    if testcase["function_point"]:
        return extract_module_from_function_point(testcase["function_point"])

    # 方式2: 从用例名称推断
    keywords = {
        "订阅管理": ["订阅", "新增", "编辑", "删除"],
        "列表操作": ["分页", "搜索", "筛选", "排序"],
        "权限管理": ["权限", "安全管理员", "应用安全负责人"],
        "用户选择": ["订阅人", "用户"],
        "界面测试": ["布局", "样式", "响应式"],
        "异常处理": ["网络", "异常", "错误", "超时"],
        "业务流程": ["流程", "连续", "完整"]
    }

    for module, keyword_list in keywords.items():
        for keyword in keyword_list:
            if keyword in testcase["name"]:
                return module

    # 默认: 使用测试类型作为模块
    return testcase["test_type"]
```

### 第六步：生成输出

```python
def generate_qinwuyuan_format(modules, project_name="陕西秦务员"):
    """
    生成秦务员格式的输出
    """
    output = f"# 项目名：{project_name}\n\n"

    for module_name, testcases in modules.items():
        output += f"## 模块名：{module_name}\n\n"

        for testcase in testcases:
            output += generate_testcase(testcase)

    return output

def generate_testcase(testcase):
    """
    生成单个测试用例的秦务员格式
    """
    output = f"### 用例名：{testcase['name']}\n\n"

    # 处理前置条件
    if testcase["precondition"]:
        if is_executable(testcase["precondition"]):
            output += f"- 操作步骤1：{testcase['precondition']}\n\n"

    # 添加操作步骤
    step_offset = 1 if testcase["precondition"] else 0

    for i, step in enumerate(testcase["steps"], 1):
        step_num = i + step_offset
        output += f"- 操作步骤{step_num}：{step['description']}\n\n"

    # 添加验证点
    if testcase["expected_result"]:
        output += f"- 验证：{testcase['expected_result']}\n\n"

    return output
```

## 关键优化点

### 1. 完整提取所有测试类型

**问题**: 可能只提取部分测试类型

**解决**: 遍历所有已知测试类型，确保不遗漏

```python
for test_type in ALL_TEST_TYPES:
    testcases = extract_testcases_under_type(content, test_type)
    all_testcases.extend(testcases)
```

### 2. 正确处理多级标题

**问题**: 标题层级复杂，容易遗漏

**解决**: 明确定义每级标题的含义

```markdown
# 根节点
## 测试类型 (功能测试、业务流程测试等)
### 功能点 (F01 - 订阅列表展示)
#### 测试用例 (TC-F01-001 - 列表页展示)
```

### 3. 灵活的模块分组

**问题**: 固定分组方式可能不适用所有场景

**解决**: 多种分组策略，按优先级选择

1. **优先级1**: 保留原文档的模块结构
2. **优先级2**: 根据用例内容智能推断
3. **优先级3**: 按测试类型分组
4. **优先级4**: 扁平化列表

### 4. 步骤编号重置

**问题**: 步骤编号应该在每个用例内从1开始

**解决**: 明确重置逻辑

```python
for testcase in testcases:
    step_offset = 1 if has_executable_precondition(testcase) else 0

    for i, step in enumerate(testcase["steps"], 1):
        step_num = i + step_offset
        # 生成步骤
```

### 5. 前置条件智能处理

**问题**: 不是所有前置条件都需要转换为步骤

**解决**: 判断前置条件是否可执行

```python
def is_executable(precondition):
    """
    判断前置条件是否可执行
    """
    # 包含动作动词 → 可执行
    action_verbs = ["登录", "进入", "点击", "输入", "选择", "勾选"]

    for verb in action_verbs:
        if verb in precondition:
            return True

    # 只是状态描述 → 不可执行
    status_descriptions = ["正常运行", "已就绪", "准备好", "存在"]

    for desc in status_descriptions:
        if desc in precondition:
            return False

    # 默认: 可执行
    return True
```

## 测试用例计数验证

### 验证方法

```python
def validate_extraction(input_file, output_file):
    """
    验证提取是否完整
    """
    # 统计输入文件中的测试用例数量
    input_count = count_testcases(input_file)

    # 统计输出文件中的测试用例数量
    output_count = count_testcases(output_file)

    if input_count != output_count:
        print(f"❌ 测试用例数量不匹配: 输入={input_count}, 输出={output_count}")
        return False
    else:
        print(f"✅ 测试用例数量匹配: {output_count}")
        return True
```

### 按测试类型统计

```python
def count_by_test_type(content):
    """
    按测试类型统计测试用例数量
    """
    counts = {}

    for test_type in ALL_TEST_TYPES:
        testcases = extract_testcases_under_type(content, test_type)
        counts[test_type] = len(testcases)

    return counts
```

## 常见问题和解决方案

### 问题1: 只提取了功能测试，其他测试类型缺失

**原因**: 测试类型名称不匹配

**解决**:
1. 检查测试类型名称是否完全匹配
2. 支持模糊匹配（如包含"功能"就识别为功能测试）
3. 打印未识别的测试类型

### 问题2: 某些测试用例被遗漏

**原因**: 功能点标题层级不正确

**解决**:
1. 检查是使用 `###` 还是 `####`
2. 支持多种标题层级
3. 记录所有扫描到的标题

### 问题3: 模块分组结果不合理

**原因**: 分组策略不适合当前文档

**解决**:
1. 提供多种分组策略供选择
2. 允许用户提供自定义分组映射
3. 保留原始结构作为备选

### 问题4: 步骤编号不连续

**原因**: 前置条件转换后步数未调整

**解决**:
1. 明确 step_offset 的计算逻辑
2. 添加调试信息打印
3. 在生成前验证步骤编号

## 性能优化

### 1. 单次遍历

```python
# 不推荐: 多次遍历文档
for test_type in test_types:
    content.find(test_type)  # 每次都扫描整个文档

# 推荐: 单次遍历
for line in content.split('\n'):
    classify_and_process(line)
```

### 2. 延迟解析

```python
# 只在需要时才解析详细信息
testcases_meta = []  # 只存储元数据

for testcase in testcases:
    if needs_detail(testcase):
        testcase["detail"] = parse_detail(testcase)
```

### 3. 并行处理

```python
# 对大型文档，可以并行处理各个测试类型
from concurrent.futures import ThreadPoolExecutor

with ThreadPoolExecutor() as executor:
    futures = []
    for test_type in test_types:
        future = executor.submit(
            extract_testcases_under_type,
            content,
            test_type
        )
        futures.append(future)

    all_testcases = []
    for future in futures:
        all_testcases.extend(future.result())
```

## 总结

完整的测试用例提取算法应该：

1. ✅ 遍历所有测试类型
2. ✅ 提取每个类型下的所有功能点
3. ✅ 提取每个功能点下的所有测试用例
4. ✅ 解析每个测试用例的详细信息
5. ✅ 智能处理前置条件
6. ✅ 灵活的模块分组
7. ✅ 正确的步骤编号
8. ✅ 完整的字段过滤
9. ✅ 输出格式验证
10. ✅ 数量统计和验证
