# VLMEvalKit Metric Library 规范（Capability -> Metric -> Scoring Rules）

## 1. 目标
本规范沉淀 VLMEvalKit 中可直接复用的评测指标，统一三层抽象：
1. **Capability（能力维度）**：我们评什么能力。
2. **Metric（指标）**：使用什么量化函数。
3. **Scoring Rules（计分规则）**：样本级、字段级、任务级如何聚合。

---

## 2. 可直接复用的指标库（Metric Library）

### 2.1 通用 QA / 分类 / 抽取指标
- **Accuracy / Hit Rate**：多选、判断、抽取任务基础正确率（`hit` / `score` 均值）。
- **Precision / Recall / F1**：二分类（如 Yes/No）和 hallucination 检测。
- **ANLS**（Approximate Normalized Levenshtein Similarity）：DocVQA/InfoVQA 类 OCR 文本相似。
- **Relaxed Accuracy**：数值答案允许相对误差（默认 5%）。
- **Exact Match（EM）**：严格字符串相等。

### 2.2 集合 / 序列 / 结构化答案指标（MEGABench）
- **Set / Dict / Sequence 系列**：
  - Set Equality, Set Precision, Jaccard
  - Dict Equality, Dict-Jaccard 聚合
  - Sequence Equality / Prefix Ratio
- **字符串近似匹配**：
  - Damerau-Levenshtein 归一化相似度
  - Near String Match
- **数值类**：
  - Relative Diff Ratio
  - General/Boxed Numerical Match
  - RMSE / Angle RMSE
- **空间定位类**：
  - BBox IoU（single/tuple/sequence）
  - Point in BBox / Point Distance
- **代码与规划类**：
  - Program Judge
  - Symbolic Planning
- **语言生成类**：
  - BLEU / GLEU
- **LLM-as-a-Judge**：
  - VLM/LLM 裁判指标（如 `gpt_4o_as_judge`）

### 2.3 图像生成 / SVG 指标（SArena）
- FID / FID-C
- CLIP-Score（T2I, I2I）
- DINO-Score
- LPIPS
- SSIM
- PSNR
- Token Length

### 2.4 视频生成指标（SArena Video）
- FVD
- ViCLIP（T2V, V2V）
- DINO-Video
- SSIM-Video
- LPIPS-Video
- PSNR-Video
- Token Length

---

## 3. Capability -> Metric 映射

| Capability | 推荐主指标 | 备选/辅助指标 | 适用输出形态 |
|---|---|---|---|
| 多选/单选理解 | Accuracy | Macro Accuracy（按类别平均） | A/B/C/D 等离散标签 |
| 二分类判断（Yes/No, True/False） | Accuracy + F1 | Precision/Recall | 二元文本标签 |
| 开放式短答案 QA | EM / VQA Score | ANLS / Near Match | 短文本、词组 |
| OCR/文档问答 | ANLS | Relaxed Accuracy（数字） | OCR 字符串、数字 |
| 表格问答 | Denotation Accuracy | Exact Match / Relaxed Accuracy | 单值/多值（分隔） |
| 数值推理 | Relaxed Accuracy | Relative Diff Ratio / RMSE | 数字/百分比/单位 |
| 集合抽取 | Set Equality | Jaccard / Set Precision / Set Recall | 集合、多标签 |
| 结构化字段抽取（JSON/Dict） | Dict Equality | Dict-Jaccard / Dict-Recall | 字段化输出 |
| 空间定位（检测/grounding） | IoU | Point Distance / Point-in-BBox | bbox / point / xml |
| 序列规划 /动作序列 | Sequence Equality | Prefix Ratio / Sequence Accuracy | token/step 序列 |
| 文本生成质量 | BLEU/GLEU | LLM-as-a-Judge | 句子/段落 |
| 代码执行/约束满足 | Program Judge | Constrained Generation | 程序输出/规则 |
| 图像生成质量 | FID / CLIP-Score | LPIPS / SSIM / PSNR / DINO | 图像 |
| 视频生成质量 | FVD / ViCLIP | DINO-Video / LPIPS-Video / SSIM-Video / PSNR-Video | 视频 |

---

## 4. Scoring Rules（统一计分规则）

### 4.1 样本级（Sample-level）
- 每个样本按其 `metric` 输出 `score`，建议标准化到 **[0,1]**：
  - 离散正确类：`correct ∈ {0,1}`。
  - 距离/误差类：映射为相似度（例如 `1 - normalized_distance`）。
  - Judge 类：要求返回可解析数值，失败记 `-1`（无效）。

### 4.2 字段级（Field-level）
- 结构化任务以字段维度打分：`field_scores[field]`。
- 支持权重：`field_weights[field]`，未设置默认 1.0。

### 4.3 查询级（Query-level）
- 使用加权平均：
  \[
  query\_score = \frac{\sum_i w_i s_i}{\sum_i w_i}
  \]
- 无有效字段时：返回 0 或 -1（根据任务约定；建议用 -1 表示无效）。

### 4.4 任务级（Task-level）
- Task Score = 所有 query score 的平均。
- 推荐同时报告：
  - **micro mean**（按样本平均）
  - **macro mean**（按任务平均）

### 4.5 主指标选择（Primary Metric）
- 自动优先级建议：`overall` > `acc` > `score`。
- 若包含 split（test/val/dev），优先 test。

### 4.6 失效与异常处理
- 无法评估/解析失败：
  - 记录 `judge/info/log`。
  - 样本记 `-1`，并在汇总时区分：
    - **ALL**：包含失败样本（可按 0 处理）
    - **VALID**：仅统计有效样本

---

## 5. Metric Library 数据模型（建议）

```yaml
metric_id: string
capability: string
family: [classification|similarity|distance|generation|judge]
direction: [higher_is_better|lower_is_better]
range: [min, max]
sample_score_fn: string
normalization: string
aggregation:
  field: weighted_mean
  query: weighted_mean
  task: mean
failure_policy:
  score_on_failure: -1
  include_in_all: true
  include_in_valid: false
required_inputs:
  - prediction
  - answer
optional_inputs:
  - eval_context
  - images
```

---

## 6. 落地建议
1. 优先复用 `vlmeval/dataset/utils/megabench` 的 `MetricType` 作为统一注册中心。
2. 对老 benchmark（MCQ、Y/N、VQA）增加 `metric_id` 显式声明，避免隐式逻辑分叉。
3. 统一输出 schema：`sample -> field -> query -> task -> summary`。
4. 把 `ALL/VALID` 双视图固化为标准报表字段。


## 7. 公式库文件（建议新增）
- 建议维护 `metric_formula_library.yaml` 作为“强约束层”。
- 对可确定义指标（Acc/F1/IoU/ANLS 等）写明公式。
- 对 Judge 型指标写明：评分区间、rubric 维度、失败策略。

