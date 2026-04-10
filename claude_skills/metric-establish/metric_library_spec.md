# Metric Library Spec (Bundled Metrics)

## 1) 场景定义
本规范用于 **Metric Establish（内置指标融合）**：
- 直接使用 skill 内置指标（已融合自当前库实现）；
- 建立 capability -> metric 映射；
- 统一 scoring rules；
- 输出标准化 metric library 规范。

## 2) 独立部署 + 内置复用
- skill 可独立拷贝部署。
- 指标来源固定为内置 bundle（非运行时任意库扫描）。
- 每个指标保留 `source_locator` 指向原始实现位置。

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
```

## 5) 运行流程
1. 读取内置 `metric_method_catalog.yaml`。
2. 读取 `capability_metric_map.yaml` 选择指标。
3. 按 `scoring_rules.yaml` 执行聚合。
4. 输出 ALL/VALID 双视图结果。

## 6) 验收规则
- 每个 capability 至少一个 primary metric；
- 所有 metric 必须存在 source_locator；
- 所有 primary metric 具备 method_definition；
- Judge 指标必须使用 rubric 类型方法。
