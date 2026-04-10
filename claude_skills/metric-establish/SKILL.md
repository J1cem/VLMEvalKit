# Skill: metric-establish

## Scenario
**Metric Establish**：当你需要从任意“指标库/评测库”中直接复用已有指标，建立 capability -> metric 映射，统一 scoring rules，并输出标准化 metric library 规范。

## Skill定位（独立可部署）
- 本 skill **独立于任何具体仓库**。
- 通过参数 `library_root` 指向目标库，实现“就地抽取 + 直接复用”。
- 可复制到任意目录后运行，不依赖固定项目结构。

## Objectives
1. 发现并复用库中已有测试指标（不重复造轮子）；
2. 建立 capability -> metric 映射；
3. 标准化 scoring rules（sample/field/query/task/summary）；
4. 产出可机读+可审计的 metric library 规范；
5. 为每个主指标提供计算方法（公式或算法步骤）。

## Inputs
- `library_root` (required): 目标指标库根目录。
- `capability_taxonomy` (optional): 业务能力分类。
- `task_types` (optional): 任务类型集合。
- `allow_llm_judge` (optional): 是否允许 LLM-as-a-Judge。

## Standard Outputs
1. `metric_library_spec.md`：规范文档（治理与流程）。
2. `capability_metric_map.yaml`：capability -> metric 映射。
3. `scoring_rules.yaml`：统一聚合与失败策略。
4. `metric_method_catalog.yaml`：指标计算方法目录（公式/算法）。

## Workflow (Standard)
1. **Discover**: 扫描 `library_root` 识别可复用指标及实现入口。
2. **Normalize**: 统一命名、方向（higher/lower is better）、分值区间。
3. **Map**: 生成 capability 的 primary/secondary metric。
4. **Methodize**: 为每个 primary metric 写计算方法（公式优先，算法其次）。
5. **Ruleize**: 生成 sample->summary 的 scoring rules。
6. **Specify**: 生成最终 metric library 规范文档。

## Quality Gates
- 每个 capability 至少一个 primary metric。
- 每个 primary metric 必须有 `method_type` + `method_definition`。
- scoring rules 必须包含 `failure_policy` 与 `ALL/VALID` 双视图。
- 映射中的 metric 必须能追溯到 `library_root` 中实现（source_locator）。

## Constraints
- 优先“直接复用”已有实现；
- 禁止硬编码某个私有仓库路径；
- 对 Judge 类指标必须给 rubric（不能伪造闭式公式）。
