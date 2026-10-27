**English** | [简体中文](#简体中文)

# Weibo-MM

A Chinese multimodal Weibo dataset for **social media popularity prediction** — **29,612** posts with **67,321** attached images, each post annotated with **15** post-publication interaction snapshots (reposts / comments / likes). Suitable for popularity prediction, multimodal content understanding, and temporal modeling research.

**Repository:** <https://github.com/xiangrui1108/Weibo-MM>

## Paper

This dataset is the **data implementation used in our paper**:

> **[DRIFT: Distribution-Routing with Injected Semantic Priors and Fused Temporal Encoding for Popularity Prediction](https://doi.org/10.1007/978-981-92-3394-6_49)**

If you use this dataset, please cite the paper above — see [Citation](#citation).

## Dataset statistics

| Item                           | Count   |
| ------------------------------ | ------- |
| Posts                          | 29,612  |
| Posts with images              | 24,415  |
| Images                         | 67,321  |
| Unique users                   | 5,436   |
| Interaction snapshots per post | 15      |
| Image size                     | ≈ 14 GB |

## Repository layout

```
.
├── README.md            # this dataset card
├── images               # image set (~10,000 images retained)
├── weibo_data.jsonl     # main annotation file, one post per line
```

> **This repository contains the annotation file only; images are not fully included.** The full image set (67,321 images, ≈ 14 GB) exceeds GitHub's hosting limits. To request the complete image data, contact `2026140643@bupt.cn`.

## Data format

Each line of `weibo_data.jsonl` is a JSON object whose payload is wrapped under a top-level `data` key:

```json
{
  "data": {
    "id": "5260197777901304",
    "title": "【宝妈回应1岁宝宝听数学题秒入睡：...】",
    "clean_text": "https://m.weibo.cn/detail/5260197777901304",
    "created_at": "Wed Jan 28 22:59:03 +0800 2026",
    "user": {"id": 1, "followers_count": "1.08亿"},
    "image_count": 3,
    "image_urls": ["images/000001.jpg", "images/000002.jpg", "images/000003.jpg"],
    "metrics_history": [
      {"timestamp": "1", "reposts_count": 11, "comments_count": 10, "attitudes_count": 78},
      {"timestamp": "2", "reposts_count": 15, "comments_count": 11, "attitudes_count": 208}
    ]
  }
}
```

| Field                  | Type         | Description                                                                                                                                        |
| ---------------------- | ------------ | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| `id`                   | string       | Weibo post ID                                                                                                                                      |
| `title`                | string       | Post title / topic summary                                                                                                                         |
| `clean_text`           | string       | Post body (for some records, the original post link)                                                                                               |
| `created_at`           | string       | Publication time, format `%a %b %d %H:%M:%S %z %Y`                                                                                                 |
| `user.id`              | int          | **Anonymized.** Original UID mapped to consecutive integers `1..5436`, one-to-one and stable across all records                                    |
| `user.followers_count` | string       | Follower count with Chinese units (e.g. `"1.08亿"`); parse to a number yourself                                                                     |
| `image_count`          | int          | Number of attached images, equal to `len(image_urls)`                                                                                              |
| `image_urls`           | list[string] | **Relative paths inside this repository**, pointing to files under `images/`, not remote URLs                                                      |
| `metrics_history`      | list[object] | 15 interaction snapshots; `timestamp` is the sampling round as a string (`"1"`–`"15"`), each round recording cumulative reposts / comments / likes |

## Quick start

```python
import json
from pathlib import Path
from PIL import Image

records = []
with open("weibo_data.jsonl", encoding="utf-8") as f:
    for line in f:
        records.append(json.loads(line)["data"])

print(len(records), records[0]["user"])

# Read the images of one post (normalize backslashes)
root = Path(".")
for p in records[1]["image_urls"]:
    img = Image.open(root / p.replace("\\", "/"))
    print(p, img.size)

# Parse follower counts with Chinese units
def parse_count(s: str) -> float:
    s = s.strip()
    for unit, mult in (("亿", 1e8), ("万", 1e4)):
        if s.endswith(unit):
            return float(s[:-1]) * mult
    return float(s)

# Build a popularity-prediction sample: first k rounds -> later rounds
k = 5
for r in records[:3]:
    hist = r["metrics_history"]
    past = [(h["reposts_count"], h["comments_count"], h["attitudes_count"]) for h in hist[:k]]
    future = hist[k:][-1]
    print(r["id"], "past:", past, "-> final:", future)
```

## Anonymization

`user.id` is **irreversibly replaced**: original UIDs are mapped, in order of first appearance, to consecutive integers starting from 1. The mapping is one-to-one and stable within this release; the mapping table is not distributed with the dataset.

## License and usage

- Data is derived from public Weibo content and is released for **non-commercial academic research only**; redistribution must follow the same restriction.
- Image copyrights belong to the original authors; images are included for research purposes only.
- `user.id` is anonymized and cannot be linked back to real accounts.
- `metrics_history` records interaction counts at the time of collection and does not represent the final state.
- For the complete image data, contact `2026140643@bupt.cn`.

## Citation

If you use this dataset, please cite our paper:

```bibtex
@inproceedings{jia2026drift,
  author    = {Jia, Qinqing and Xue, Zhe and Liu, Qinlan and Ou, Shilong and Liu, Xiangrui and Zhao, Sihan},
  title     = {{DRIFT}: Distribution-Routing with Injected Semantic Priors and Fused Temporal Encoding for Popularity Prediction},
  booktitle = {Advanced Intelligent Computing Technology and Applications},
  series    = {Lecture Notes in Computer Science},
  volume    = {16646},
  pages     = {581--591},
  year      = {2026},
  publisher = {Springer Nature Singapore},
  address   = {Singapore},
  doi       = {10.1007/978-981-92-3394-6_49},
  isbn      = {978-981-92-3393-9}
}
```

---

[English](#weibo-mm) | **简体中文**

## 简体中文

**Weibo-MM —— 中文微博多模态数据集**

中文微博多模态数据集，用于**社交媒体热度预测**：**29,612 条**微博帖子 + **67,321 张**配图，每条帖子附带 **15 轮**发布后的互动量快照（转发 / 评论 / 点赞），适用于社交媒体热度预测、多模态内容理解、时序建模等研究。

**仓库地址：** <https://github.com/xiangrui1108/Weibo-MM>

### 论文

本数据集是下列论文的**数据实现**：

> **[DRIFT: Distribution-Routing with Injected Semantic Priors and Fused Temporal Encoding for Popularity Prediction](https://doi.org/10.1007/978-981-92-3394-6_49)**

如使用本数据集，请引用上述论文，详见[引用](#引用)。

### 数据规模

| 项目         | 数量      |
| ---------- | ------- |
| 帖子         | 29,612  |
| 带图帖子       | 24,415  |
| 图片总数       | 67,321  |
| 唯一用户数      | 5,436   |
| 每条帖子的互动量快照 | 15 轮    |
| 图片体积       | ≈ 14 GB |

### 目录结构

```
.
├── README.md            # 本数据集卡
├── images               # 图像集（保留约 10000 张）
├── weibo_data.jsonl     # 主标注文件，每行一条帖子
```

> **本仓库仅包含标注文件，未包含完整图片。** 图片共 67,321 张（约 14 GB），受 GitHub 网站限制不能完整上传；如需要完整图像数据，请联系 `2026140643@bupt.cn`。

### 字段说明

`weibo_data.jsonl` 每行是一个 JSON 对象，全部内容包在顶层 `data` 键下：

```json
{
  "data": {
    "id": "5260197777901304",
    "title": "【宝妈回应1岁宝宝听数学题秒入睡：...】",
    "clean_text": "https://m.weibo.cn/detail/5260197777901304",
    "created_at": "Wed Jan 28 22:59:03 +0800 2026",
    "user": {"id": 1, "followers_count": "1.08亿"},
    "image_count": 3,
    "image_urls": ["images/000001.jpg", "images/000002.jpg", "images/000003.jpg"],
    "metrics_history": [
      {"timestamp": "1", "reposts_count": 11, "comments_count": 10, "attitudes_count": 78},
      {"timestamp": "2", "reposts_count": 15, "comments_count": 11, "attitudes_count": 208}
    ]
  }
}
```

| 字段                     | 类型           | 说明                                                                  |
| ---------------------- | ------------ | ------------------------------------------------------------------- |
| `id`                   | string       | 微博帖子 ID                                                             |
| `title`                | string       | 帖子标题 / 话题摘要                                                         |
| `clean_text`           | string       | 正文（部分记录为原文链接）                                                       |
| `created_at`           | string       | 发布时间，`%a %b %d %H:%M:%S %z %Y` 格式                                   |
| `user.id`              | int          | **已脱敏**。原始 UID 映射为 `1..5436` 的连续整数，一对一，同一用户在全部记录中编号一致               |
| `user.followers_count` | string       | 粉丝数，含中文单位（如 `"1.08亿"`），需自行解析为数值                                     |
| `image_count`          | int          | 配图数量，等于 `len(image_urls)`                                           |
| `image_urls`           | list[string] | **本仓库内的相对路径**，指向 `images/` 下的文件，不是远端 URL                            |
| `metrics_history`      | list[object] | 15 轮互动量快照；`timestamp` 为采样轮次序号（字符串 `"1"`~`"15"`），每轮记录累计转发 / 评论 / 点赞数 |

### 加载示例

```python
import json
from pathlib import Path
from PIL import Image

records = []
with open("weibo_data.jsonl", encoding="utf-8") as f:
    for line in f:
        records.append(json.loads(line)["data"])

print(len(records), records[0]["user"])

# 读取一条帖子的配图（注意规范化反斜杠）
root = Path(".")
for p in records[1]["image_urls"]:
    img = Image.open(root / p.replace("\\", "/"))
    print(p, img.size)

# 解析中文单位粉丝数
def parse_count(s: str) -> float:
    s = s.strip()
    for unit, mult in (("亿", 1e8), ("万", 1e4)):
        if s.endswith(unit):
            return float(s[:-1]) * mult
    return float(s)

# 构造热度预测样本：用前 k 轮预测后续轮次
k = 5
for r in records[:3]:
    hist = r["metrics_history"]
    past = [(h["reposts_count"], h["comments_count"], h["attitudes_count"]) for h in hist[:k]]
    future = hist[k:][-1]
    print(r["id"], "past:", past, "-> final:", future)
```

### 脱敏说明

`user.id` 已做**不可逆替换**：按用户首次出现顺序，将原始 UID 依次映射为从 1 开始的连续整数，映射一对一且在同一份数据内稳定。映射表不随数据集发布。

### 使用限制

- 数据来自微博公开内容，**仅限非商业学术研究使用**；二次分发请遵循同等限制。
- 图片版权归原作者所有，数据集仅作研究用途收录。
- `user.id` 已脱敏，无法还原真实账号。
- `metrics_history` 反映的是采集时刻的互动量快照，不代表最终状态。
- 需要完整图像数据请联系 `2026140643@bupt.cn`。

### 引用

如使用本数据集，请引用我们的论文：

```bibtex
@inproceedings{jia2026drift,
  author    = {Jia, Qinqing and Xue, Zhe and Liu, Qinlan and Ou, Shilong and Liu, Xiangrui and Zhao, Sihan},
  title     = {{DRIFT}: Distribution-Routing with Injected Semantic Priors and Fused Temporal Encoding for Popularity Prediction},
  booktitle = {Advanced Intelligent Computing Technology and Applications},
  series    = {Lecture Notes in Computer Science},
  volume    = {16646},
  pages     = {581--591},
  year      = {2026},
  publisher = {Springer Nature Singapore},
  address   = {Singapore},
  doi       = {10.1007/978-981-92-3394-6_49},
  isbn      = {978-981-92-3393-9}
}
```

