# Metric Library Spec for Metric Establish

## 1) 场景定义
本规范服务于 **Metric Establish** 场景：
- 从目标库中直接复用已有测试指标；
- 建立 capability -> metric 映射；
- 形成统一 scoring rules；
- 输出标准化 metric library 规范。

## 2) 独立性要求
- skill 目录可独立拷贝到任意位置。
- 通过 `library_root` 接入目标库，不绑定固定仓库结构。
- 所有指标必须可追溯到 `source_locator`。

## 3) 标准产物
- `capability_metric_map.yaml`
- `scoring_rules.yaml`
- `metric_method_catalog.yaml`
- `metric_library_spec.md`

## 4) 核心数据模型
```yaml
metric_record:
  metric_id: string
  capability: string
  role: [primary, secondary]
  source_locator: string
  method_type: [closed_form, algorithmic, rubric]
  method_definition: string
  direction: [higher_is_better, lower_is_better]
  range: [min, max]
  failure_policy:
    sentinel_score: -1
    include_in_all: true
    include_in_valid: false
```

## 5) 建立流程
1. Discover: 从 `library_root` 提取可复用指标。
2. Classify: 将指标映射到 capability taxonomy。
3. Bind: 为每个映射补齐 `source_locator`。
4. Methodize: 从实现或论文补齐方法定义。
5. Ruleize: 统一 sample/field/query/task/summary 计分。
6. Validate: 检查 primary 完整性和可追溯性。

## 6) 验收规则
- 每个 capability >= 1 个 primary metric。
- 所有映射项包含 source_locator。
- 所有 primary metric 具备 method_definition。
- scoring rules 含 ALL/VALID 双视图。

## 7) Judge类指标规范
对于无法闭式定义的指标（如 LLM-as-a-Judge）：
- 使用 `method_type: rubric`；
- 必须定义评分维度、分值范围、解析规则、失败回退。
