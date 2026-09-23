# 04 模型权重与样例数据

> 本目录为复现与审稿核验提供**可直接加载的模型权重**与**少量样例数据**。所有文件均为复制件，未改动原始工程。

---

## 一、模型权重（weights/，8 个）

加载方式（需本包 `02_源码` 的改动源码）：
```python
from ultralytics import YOLO
m = YOLO("submission/04_权重与样例/weights/best_yolo11n_baseline_seed42.pt")
r = m.val(data="submission/02_源码/configs/data_DamCrack-Sub5.yaml", split="test", imgsz=640)
print(r.box.map, r.box.map50)
```

| 权重文件 | 对应实验 | 训练数据 | 参数量 | 测试集 mAP50-95 | 测试集 mAP50 |
|---|---|---|---|---|---|
| `best_yolo11n_baseline_seed42.pt` | **主基线（表 2、表 3）** | DamCrack-Sub5 train（3902 幅） | 2.583 M | **34.34%** | 57.08% |
| `best_yolo11n_baseline_seed123.pt` | 主基线复现性（表 2） | 同上 | 2.583 M | 35.06% | 57.66% |
| `best_yolo11n_baseline_seed456.pt` | 主基线复现性（表 2） | 同上 | 2.583 M | 34.59% | 57.04% |
| `best_yolo11n_skeleton020.pt` | 骨架辅助监督 λ=0.2（表 4） | 同上 + 骨架伪标签 | 2.583 M | 34.51% | 57.23% |
| `best_yolo26n.pt` | 检测器对比（表 9） | 同上 | 2.376 M | 33.50% | 54.92% |
| `best_yolo26n_p2.pt` | 结构对比：+P2（表 9） | 同上 | 2.402 M | 30.88% | 52.35% |
| `best_yolo26n_strip.pt` | 结构对比：+条带注意力（表 9） | 同上 | 2.616 M | 27.97% | 49.32% |
| `best_kshot10_full.pt` | **K-shot（K=10）全量微调（表 6）** | 每子类 10 幅（共 50 幅） | 2.583 M | 5.84% | — |

> 说明：主基线 3 种子文件可用于核对表 2 的均值±标准差（34.66 ± 0.37）。权重为 Ultralytics 既有格式（含模型结构、权重与训练元数据）。

---

## 二、样例数据（samples/）

| 子目录 | 内容 | 数量 |
|---|---|---|
| `images/` | 测试集样例图像（每子类 4 幅，命名 `cls{类别号}_{原文件名}.jpg`） | 20 |
| `labels/` | 与上表图像一一对应的 YOLO 标注（`类别 cx cy w h`，5 列归一化） | 20 |
| `骨架示例/` | 训练集的骨架伪标签 PNG（0/255）+ 对应原图（文件名 `*_原图.jpg`） | 12 |
| `退化条件示例/` | 成像仿真的退化样例：GSD 0.35×（`gsd035_*`）与运动模糊 7 px（`blur7_*`），均为 640×640、标签不变 | 6 |

类别编号对照（`data.yaml` 中一致）：

| 编号 | 英文名 | 中文名 |
|---|---|---|
| 0 | crack_fine | 细长裂缝 |
| 1 | crack_network | 网状裂缝 |
| 2 | crack_blocky | 块状裂缝 |
| 3 | spalling_small | 小型剥落 |
| 4 | spalling_large | 大型剥落 |

---

## 三、快速核验（3 条命令）

```bash
# 1) 主基线在完整测试集上的指标（应得 mAP50-95 ≈ 34.34%）
python -c "import sys; sys.path.insert(0,'.'); from ultralytics import YOLO; m=YOLO('submission/04_权重与样例/weights/best_yolo11n_baseline_seed42.pt'); r=m.val(data='submission/02_源码/configs/data_DamCrack-Sub5.yaml', split='test', imgsz=640, workers=0); print(r.box.map, r.box.map50)"

# 2) 单张样例推理（可视化）
python -c "import sys; sys.path.insert(0,'.'); from ultralytics import YOLO; m=YOLO('submission/04_权重与样例/weights/best_yolo11n_baseline_seed42.pt'); m.predict('submission/04_权重与样例/samples/images/', imgsz=640, conf=0.25, save=True)"

# 3) 骨架辅助模型（需骨架标签支持）在测试集上的指标
python -c "import sys; sys.path.insert(0,'.'); from ultralytics import YOLO; m=YOLO('submission/04_权重与样例/weights/best_yolo11n_skeleton020.pt'); r=m.val(data='submission/02_源码/configs/data_DamCrack-Sub5.yaml', split='test', imgsz=640, workers=0); print(r.box.map)"
```

> 注：`data_DamCrack-Sub5.yaml` 中的 `path` 指向作者环境（`E:\...\datasets\DamCrack-Sub5`），核验前请按实际数据位置修改该行。

---

## 四、未包含的内容

| 项目 | 原因 | 获取方式 |
|---|---|---|
| 图像本体（5500 幅，约 500 MB） | 体积过大且为公开数据 | Zenodo DOI 10.5281/zenodo.17274707（CC BY 4.0）；或用 `scripts_damsegment/data/14_build_merged_5sub.py` 重建 |
| 全部中间实验权重（约 40 个） | 体积过大，且论文主结论仅依赖上表 8 个 | `runs/` 目录保留完整产物，可按需补充 |
| 数据集缓存（`*.cache`） | 机器/路径相关 | 首次运行时自动生成 |
