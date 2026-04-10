# Portable Metric Library 规范（Capability -> Metric -> Scoring Rules）

## 1. 目标
本规范用于构建**与具体仓库解耦**的评测指标库：
1. **Capability（能力维度）**：评测目标。
2. **Metric（指标）**：量化函数。
3. **Scoring Rules（计分规则）**：样本到汇总的聚合方法。

> 该规范可在任意仓库复用；只需实现适配层，不依赖固定目录结构。

---

## 2. 通用可复用指标族
- 分类/识别：Accuracy, Precision, Recall, F1
- 文本匹配：Exact Match, ANLS, Near String Match
- 集合/结构化：Set Equality, Jaccard, Dict Equality
- 空间定位：IoU, Point-in-BBox, Point Distance
- 数值误差：Relaxed Accuracy, Relative Diff Ratio, RMSE
- 生成质量：BLEU/GLEU、FID/CLIP/LPIPS/SSIM/PSNR、FVD/ViCLIP
- Judge 指标：LLM/VLM-as-a-Judge（含 rubric）

---

## 3. Capability -> Metric 映射原则
- 每个 capability 至少一个主指标（primary）。
- 推荐为每个 capability 追加 1~3 个辅指标（secondary），用于诊断。
- 主指标需满足：
  - 可解释；
  - 可稳定复现；
  - 对业务目标单调一致。

---

## 4. Scoring Rules（统一）

### 4.1 Sample-level
- 输出标准分值 `score in [0,1]`。
- 无法评估时记 `-1` 并记录失败原因。

### 4.2 Field-level
- 结构化任务按字段打分：`field_scores[field]`。
- 默认权重 `1.0`，可覆盖。

### 4.3 Query-level
\[
query\_score = \frac{\sum_i w_i s_i}{\sum_i w_i}
\]

### 4.4 Task-level
- `mean_task_score = mean(query_scores)`

### 4.5 Summary-level
- `macro_mean_score = mean(task_mean_score)`
- `micro_mean_score = mean(all_query_scores)`

### 4.6 Reporting Views
- **ALL**：包含失败样本。
- **VALID**：仅统计有效样本（score >= 0）。

---

## 5. 适配层（Adapter Contract）

```yaml
adapter_contract:
  MetricProvider:
    input: [codebase_root]
    output: [metric_id_list]
  MetricExecutor:
    input: [metric_id, prediction, answer, context]
    output: [score, info]
  ScoreAggregator:
    input: [field_scores, field_weights]
    output: [query_score, task_score, summary]
  FailurePolicy:
    input: [error_type]
    output: [sentinel_score, include_in_all, include_in_valid]
```

---

## 6. 部署建议（任意位置可用）
1. 将本目录复制到任意路径。
2. 在目标仓库新增 `adapter.metric_bindings`（metric_id -> 实现路径）。
3. 用 `metric_formula_library.yaml` 做公式强约束。
4. 用 `scoring_rules.yaml` 做聚合与报表强约束。

---

## 7. 公式治理建议
- 可闭式公式：必须显式写入公式库。
- Judge 类指标：必须定义 rubric、score range、failure policy。
