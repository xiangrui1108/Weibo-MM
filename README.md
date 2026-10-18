
# WeiboMM

中文微博多模态数据集：**29,612 条**微博帖子 + **67,321 张**配图，每条帖子附带 **15 轮**发布后的互动量快照（转发 / 评论 / 点赞），适用于社交媒体热度预测、多模态内容理解、时序建模等研究。

## 数据规模

| 项目 | 数量 |
| --- | --- |
| 帖子 | 29,612 |
| 带图帖子 | 24,415 |
| 图片总数 | 67,321 |
| 唯一用户数 | 5,436 |
| 每条帖子的互动量快照 | 15 日 |
| 图片体积 | ≈ 14 GB |

## 目录结构

```
.
├── README.md            # 本数据集卡
├── image                # 图像集（保留约10000张）
├── weibo_data.jsonl     # 主标注文件，每行一条帖子
```

> **本仓库仅包含标注文件，未包含图片。** 图片共 67,321 张（约 14 GB），受github网站限制不能完整上传，如需要完整图像数据请联系`2026140643@bupt.cn`

## 字段说明

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

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | 微博帖子 ID |
| `title` | string | 帖子标题 / 话题摘要 |
| `clean_text` | string | 正文（部分记录为原文链接） |
| `created_at` | string | 发布时间，`%a %b %d %H:%M:%S %z %Y` 格式 |
| `user.id` | int | **已脱敏**。原始 UID 映射为 `1..5436` 的连续整数，一对一，同一用户在全部记录中编号一致 |
| `user.followers_count` | string | 粉丝数，含中文单位（如 `"1.08亿"`），需自行解析为数值 |
| `image_count` | int | 配图数量，等于 `len(image_urls)` |
| `image_urls` | list[string] | **本仓库内的相对路径**，指向 `images/` 下的文件，不是远端 URL |
| `metrics_history` | list[object] | 15 轮互动量快照；`timestamp` 为采样轮次序号（字符串 `"1"`~`"15"`），每轮记录累计转发 / 评论 / 点赞数 |

## 加载示例

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

## 脱敏说明

`user.id` 已做**不可逆替换**：按用户首次出现顺序，将原始 UID 依次映射为从 1 开始的连续整数，映射一对一且在同一份数据内稳定。映射表不随数据集发布。

## 使用限制

- 数据来自微博公开内容，**仅限非商业学术研究使用**；二次分发请遵循同等限制。
- 图片版权归原作者所有，数据集仅作研究用途收录。
- `user.id` 已脱敏，无法还原真实账号。
- `metrics_history` 反映的是采集时刻的互动量快照，不代表最终状态。
- 需要完整图像数据请联系`2026140643@bupt.cn`

## 引用

如使用本数据集，请注明来源。
