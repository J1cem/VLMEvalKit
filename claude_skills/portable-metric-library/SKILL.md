# Skill: portable-metric-library

## 目的
构建一个**与具体仓库解耦**的 Metric Library，可在任意代码库、任意目录部署和测试。

## 何时使用
当你需要：
1. 在新仓库快速建立 capability -> metric 映射；
2. 给多任务评测定义统一 scoring rules；
3. 在不依赖现有项目结构的情况下复用指标治理方案。

## 输入
- `codebase_root`：待扫描仓库根目录（可选，不提供则只输出通用模板）
- `capabilities`：能力列表（可选）
- `task_types`：任务形态（mcq/yes-no/open-qa/structured/grounding/generation）
- `allow_llm_judge`：是否允许 LLM-as-a-Judge

## 输出
1. `metric_library_spec.md`（通用规范）
2. `capability_metric_map.yaml`（可机读映射）
3. `scoring_rules.yaml`（统一计分规则）
4. `metric_formula_library.yaml`（核心公式库）

## 执行流程（可移植）
1. 扫描 `codebase_root` 中的评测实现（若提供）。
2. 将发现的指标归并到通用 capability taxonomy。
3. 生成主指标 + 辅助指标映射。
4. 生成 sample/field/query/task 的统一聚合规则。
5. 明确可闭式公式；对 judge 类输出 rubric + score range。
6. 输出“适配层接口”，保证可在任意仓库挂载。

## 适配层接口（解耦关键）
- `MetricProvider`: 列举本仓库可用 metric_id。
- `MetricExecutor`: 输入 `(prediction, answer, context)` 返回 `score`。
- `ScoreAggregator`: 执行 field/query/task/summary 聚合。
- `FailurePolicy`: 定义失败分值、ALL/VALID 统计策略。

## 约束
- 禁止硬编码仓库私有路径（如 `xxx/dataset/utils/...`）。
- 每个指标必须声明：`direction`、`range`、`inputs`、`failure_policy`。
- 同一 skill 在不同仓库只允许替换适配层，不改核心规则。

## 关于“是否要手写公式”
- **推荐**：写入 `metric_formula_library.yaml`，部署最稳定。
- **可自动**：可从目标仓库自动提取/推断，但 Judge 类通常无法得到严格闭式公式。
