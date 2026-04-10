# Skill: metric-establish

## Scenario
**Metric Establish（内置指标融合版）**：直接使用本 skill 内置的 VLMEvalKit 指标定义，完成 capability -> metric 映射、scoring rules 和 metric library 规范输出。

## Skill定位
- 本 skill 可独立部署，但**默认不再依赖外部任意指标库扫描**。
- 指标集合已融合进 skill（来自当前仓库已实现指标）。
- 如需扩展，只在“附加指标区”增量注册，不改主规范。

## Objectives
1. 直接复用已融合指标；
2. 建立 capability -> metric 映射；
3. 标准化 scoring rules（sample/field/query/task/summary）；
4. 产出可机读+可审计的 metric library 规范；
5. 为每个主指标提供计算方法与源码定位。

## Inputs
- `capability_taxonomy` (optional): 业务能力分类。
- `task_types` (optional): 任务类型集合。
- `allow_llm_judge` (optional): 是否允许 LLM-as-a-Judge。

## Standard Outputs
1. `metric_library_spec.md`
2. `capability_metric_map.yaml`
3. `scoring_rules.yaml`
4. `metric_method_catalog.yaml`

## Workflow (Bundled)
1. 读取 skill 内置指标目录（method catalog + mapping）。
2. 根据任务能力选择 primary/secondary metric。
3. 应用统一 scoring rules 计算并聚合。
4. 输出规范化报告（ALL/VALID + method trace）。

## Quality Gates
- 每个 capability 至少一个 primary metric。
- 每个 primary metric 具备 method_definition 与 source_locator。
- scoring rules 包含 failure_policy 与 ALL/VALID 双视图。

## Constraints
- 优先使用内置指标，不重复造轮子。
- Judge 类指标必须给 rubric，不伪造闭式公式。
