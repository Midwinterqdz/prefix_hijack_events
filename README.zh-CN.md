# BGP 劫持 90 天前缀起源数据集（序号版）

本目录中的每条记录只用一个整数 `id` 标识。原始标识信息（AS 号与窗口起止时间）已统一映射为序号，数据文件中不再出现 AS 号和具体日期。

## 数据概览

| | `hijacker` | `legal` |
|---|---|---|
| 图像数量 | 11,758 | 6,240 |
| 特征行数 | 13,123 | 5,903 |
| 同时具备图像与特征行 | 11,406 | 5,821 |

## 目录结构

```
.
├── hijacker_charts/          # 11,758 张，文件名为 <id>.png
├── legal_charts/             #  6,240 张，文件名为 <id>.png
├── hijack_struc.csv          # 13,123 行 × 30 列
├── legal_struc.csv           #  5,903 行 × 30 列
├── README.md                 # English
└── README.zh-CN.md           # 本文件
```

## 序号规则

- `id` 为 `1`–`19797` 的整数，全局唯一，两个类别之间不重复；
- 图片文件名为六位零填充的 `id`，例如 `000001.png`，与 CSV 中的 `id` 一一对应；
- 17,227 条记录同时具有 CSV 行与图片；1,799 条只有 CSV 行；771 张图片没有对应行；
- CSV 按 `id` 升序排列。

## 字段说明

两个 CSV 结构一致，共 30 列：`id` + 29 个特征。计数类字段以 `ln(1 + x)` 存储。

### 活跃度与变更

| 列名 | 说明 |
|---|---|
| `appear_share` | 该记录占窗口内全部前缀-出现记录的比例 |
| `insert_ratio` | 通告数 ÷（通告数 + 撤销数） |
| `remove_ratio` | 撤销数 ÷（通告数 + 撤销数） |
| `turnover` | 总体变更量，`ln(1 + insert_cnt + remove_cnt)` |
| `prefix_coverage_ratio_for_as` | 该记录的前缀占窗口内全部观测前缀的比例 |
| `adj_change_ratio` | 涉及相邻前缀的变更占比 |

### 位置偏移

| 列名 | 说明 |
|---|---|
| `pos_shift_mean` | 窗口内位置/名次的平均有符号偏移 |
| `pos_shift_std` | 该偏移的标准差 |

### 短命通告

| 列名 | 说明 |
|---|---|
| `short_lived_insert_ratio` | 被判定为短命的通告占比 |
| `short_lived_remove_ratio` | 被判定为短命的撤销占比 |

### 计数字段（`ln(1 + x)`）

| 列名 | 说明 |
|---|---|
| `appear_cnt` | 前缀-出现记录数 |
| `insert_cnt` | 前缀通告数 |
| `remove_cnt` | 前缀撤销数 |
| `persist_cnt` | 整个窗口内持续存在的前缀数 |
| `short_ins_cnt` | 短命通告数 |
| `short_rem_cnt` | 短命撤销数 |
| `total_events` | 窗口内全部 BGP 事件数（全局背景） |
| `total_prefix_cnt` | 窗口内全部不同前缀数（全局背景） |

### MOAS 特征（多起源 AS）

| 列名 | 说明 |
|---|---|
| `k_used` | MOAS 特征使用的 `k` 参数（固定为 `3`） |
| `moas_event_cnt` | MOAS 相关事件数，`ln(1 + x)` |
| `moas_prefix_cnt` | 涉及 MOAS 的前缀数，`ln(1 + x)` |
| `moas_days_active` | 有 MOAS 活动的天数，`ln(1 + x)` |
| `moas_days_active_ratio` | 活跃天数 ÷ 90 |
| `moas_multi_origin_ratio_k` | 起源数超过 `k` 的前缀占比 |
| `moas_more_specific_ratio` | 以更具体前缀形式通告的 MOAS 前缀占比 |
| `moas_origin_cnt_mean` | 每个 MOAS 前缀的平均起源数 |
| `moas_prefix_run_max` | 连续一起通告的最长前缀序列长度 |
| `moas_top_peer_share` | 来自主导对等体/采集器的 MOAS 事件占比 |
| `moas_peer_entropy` | 对等体/采集器分布的熵 |

## 加载数据

```python
import pandas as pd

df = pd.read_csv("hijack_struc.csv")
df["image_path"] = df["id"].map(lambda i: f"hijacker_charts/{i:06d}.png")
print(df.shape)
```

部分行没有对应图片，使用图像时请先检查文件是否存在。

## 计数还原

计数类字段为 `ln(1 + x)`，可用 `numpy.expm1()` 还原原始计数：

```python
import numpy as np
raw = np.expm1(df[["appear_cnt", "insert_cnt", "remove_cnt", "turnover"]])
```

## 划分建议

`id` 为顺序编号，可按连续 `id` 区间划分训练集和测试集；如需要类别均衡，可在每个类别内部按 `id` 区间抽样。

## 数据来源

**TODO：** 补充采集器与归档、标签来源、抽取时间段和预处理代码。

## 许可证

**TODO：** 添加 `LICENSE` 文件。

## 引用

**TODO：** 补充论文或数据集引用信息。
