# Skill: vlmeval-metric-library

## 目的
将 VLMEvalKit 中可复用评测指标组织为统一 **Metric Library**，并输出：
- capability -> metric 映射
- scoring rules
- 标准化 metric 规范

## 何时使用
当你需要：
1. 盘点仓库可复用指标；
2. 给新 benchmark 选择评分方法；
3. 统一多任务评测口径（sample/field/query/task/summary）；
4. 制定评测治理规范（主指标、失败样本策略、ALL/VALID 双口径）。

## 输入
- 目标能力清单（可为空，默认自动归纳）
- 数据集输出形态（mcq / yes-no / open qa / structured / grounding / generation）
- 是否允许 LLM-as-a-Judge

## 输出
1. `metric_library_spec.md`（规范文档）
2. `capability_metric_map.yaml`（可机读映射）
3. `scoring_rules.yaml`（统一计分规则）
4. `metric_formula_library.yaml`（核心指标公式库，建议默认产出）

## 执行流程（最小闭环）
1. 扫描 `vlmeval/dataset/utils` 与 `megabench` 的 metric 注册。
2. 按能力维度聚类指标。
3. 产出主指标 + 辅指标推荐。
4. 生成统一评分规则（sample/field/query/task）。
5. 显式写出每个主指标的计算公式（能写公式就写；不能写则给 rubric + score range）。
6. 生成标准 schema 与失败策略。

## 约束
- 优先复用已有实现，不重复造轮子。
- 所有 metric 必须声明方向（higher/lower is better）与分值区间。
- 所有聚合必须显式声明权重与无效值策略。

## 快速检查清单
- [ ] 每个 capability 至少有 1 个主指标
- [ ] 每个指标有明确输入输出定义
- [ ] query/task 汇总公式明确
- [ ] 失败样本是否进入 ALL/VALID 已定义
- [ ] 主指标选择规则已定义


## 关于“是否需要手写公式”
- **推荐**：直接提供 `metric_formula_library.yaml`，让智能体“读取即用”，减少推断误差。
- **可自动**：若未提供，智能体可依据仓库实现自动归纳，但对 LLM-as-a-judge 类指标只能产出“评分 Rubric”，不可能得到严格闭式公式。
