# DamCrack-Sub5 代码包使用说明书

> 配套论文：*Subclass-level damage detection on concrete dams: difficulty follows imaging conditions,
> not class frequency*（投 Arabian Journal for Science and Engineering）
>
> 配套文档：`README.md`（英文速览）｜`docs/REPRODUCE.md`（复现步骤清单）｜`docs/RESULTS_MAP.md`（论文 ↔ 脚本 ↔ 结果对照表）
> 本文件是**手把手版**，按"想干什么"分三条路径，照着做即可。

---

## 0. 一分钟认识这个包

| 你想…… | 走哪条路径 | 需要 GPU 吗 |
|---|---|---|
| 查论文里的某个数字是哪来的 | 路径 A（第 2 节） | 不需要 |
| 重新生成论文的图 / 表 | 路径 B（第 3 节） | 不需要 |
| 训练、评估、完整复现 | 路径 C（第 4 节） | 需要（本论文用单张 RTX 4060 Laptop 8GB） |

**包里有什么**：论文用到的全部代码（框架改动、配置、图表脚本、评估代码、训练脚本、数据构建、少样本基线）＋ 全部标注文件（19,428 个 YOLO label）＋ 论文引用的 28 个结果文件。

**包里没有什么**：原始图像（公开数据集，自己下载）、skeleton 伪标签 PNG（可用脚本重建）、模型权重（在 Zenodo 复现包）。

**相关材料地图**：

| 材料 | 位置 | 用途 |
|---|---|---|
| 本代码包（19,633 文件 / ~10 MB） | GitHub（待推送）+ 本地 `damcrack-sub5-code/` | 代码 + 标注 + 结果 |
| 复现包 zip（61 MB：标注 + 结果 + 权重 + 脚本） | Zenodo（待上传，拿 DOI） | 归档、引用、审稿人索取 |
| 论文交付包（18,718 文件） | 本地 `论文交付包_20260922/` | 作者自用：投稿材料、源码、证据 |
| 论文 PDF / LaTeX 工程 | 本地 `submission_ajse/` | 投稿系统上传 |

---

## 1. 目录结构（每个文件夹是干什么的）

| 目录 | 文件数 | 说明 | 从哪里开始看 |
|---|---|---|---|
| `framework/` | 10 | 改动的 7 个框架文件（`ultralytics/` 原路径）+ `changes.patch` + 改动说明（中文）+ 英文 README | `framework/README.md` |
| `configs/` | 99 | `datasets/` 数据集描述 11 个｜`models/` 结构消融模型 2 个｜`runs_args/` 每个实验的实测 `args.yaml`（85 个） | `configs/README.md` |
| `figures/` | 2 | `55_render_figures_en.py` 生成全部图；`56_build_ajse_tables.py` 生成 LaTeX 表 | 文件头部注释 |
| `evaluation/` | 13 | 指标套件、K-shot 诊断、成像仿真、可分离性分析等 | `docs/RESULTS_MAP.md` |
| `training/` | 8 | 主实验训练脚本（基线/增强/K-shot/重配平/分辨率/标注预算/增量） | 同上 |
| `data/` | 11 | 数据集构建（划分、子类映射、骨架伪标签、批次/增量/DSI） | `docs/REPRODUCE.md` |
| `fewshot/` | 9 | 表 S1 的 patch 级少样本基线（ProtoNet / FOMAML / Meta-Baseline） | `fewshot/README.md` |
| `annotations/` | 19,446 | **全部标注**（labels + data.yaml + manifest），不含图像 | `annotations/README.md` |
| `results/` | 28 | 论文引用的结果 JSON/CSV（论文数字的唯一出处） | `docs/RESULTS_MAP.md` |
| `docs/` | 2 | 复现指南 + 结果映射表 | — |

---

## 2. 路径 A：查数字（1 分钟，不需要装环境）

1. 打开 `docs/RESULTS_MAP.md`，找到论文里那一节/那张表对应的**结果文件**；
2. 用任意文本编辑器或 Excel 打开 `results/` 下对应文件，即可看到原始数字（JSON 缩进格式，字段名与论文术语一致）。

> 原则：论文中的每个数字都能在 `results/` 里溯源。如果哪天论文和 `results/` 对不上，**以 `results/` 为准**并回头改论文。

---

## 3. 路径 B：重新生成图 / 表（5 分钟，不需要 GPU）

```bash
# 1) 安装两个画图依赖即可
pip install matplotlib numpy pyyaml

# 2) 把结果文件摆成脚本期望的布局
cp -r results/* <你的项目根>/runs/          # 脚本按 runs/xxx/yyy.json 读取

# 3) 改脚本顶部的 ROOT 常量（两种办法）
#    办法一：直接编辑 figures/55_render_figures_en.py 第一屏的 ROOT = r"E:\..."
#    办法二：把项目放到与作者相同的路径（不推荐）

# 4) 跑
python figures/55_render_figures_en.py      # 输出 19 张图到 <ROOT>/paper/en/figures/
python figures/56_build_ajse_tables.py      # 输出 LaTeX 表格
```

说明：
- 图里没有任何手写数字，全部从 `results/` 实时读取并计算（均值 ± 标准差也是脚本算的）；
- 想只重画某几张：`55_render_figures_en.py --only fig3,fig19`。

---

## 4. 路径 C：完整跑实验（需要 NVIDIA GPU）

### 4.1 建环境（约 10 分钟）

```bash
conda create -n damcrack python=3.10 -y
conda activate damcrack
pip install torch==2.5.1 torchvision==0.20.1 --index-url https://download.pytorch.org/whl/cu121
pip install -r requirements.txt
```

### 4.2 应用框架改动（关键一步）

本包**不含完整框架**（那是别人维护的基座），只含我们改动/新增的 7 个文件。做法：

```bash
# 1) 准备好基座源码树（Ultralytics 8.4 / YOLO-PEFT 基座发行版）
# 2) 把本包的改动文件覆盖进去：
cp -r framework/ultralytics/* <基座根>/ultralytics/
# 3) 以基座根为工作目录安装：
pip install -e <基座根>

# 4) 验证改动生效（应输出 True）：
python -c "import ultralytics, pathlib; p = pathlib.Path(ultralytics.__file__).parent/'cfg'/'default.yaml'; print('skeleton_aux' in p.read_text())"
```

7 个文件分别是干什么的、为什么要改，见 `framework/README.md`（英文）或 `framework/改动说明.md`（中文，含两个设计踩坑记录）。**所有改动默认关闭**（`skeleton_aux=0.0`、`copy_paste_classes=None`），不影响原生行为。

### 4.3 准备数据

```bash
# 1) 下载公开图像：
#    - DamCrack（doi:10.1016/j.dib.2026.112737）→ 两批分别放 datasets/DamSegment/ 与 "Damage Detection/"
#    - DSI 数据集 → 按 data/20_dsi_mask2yolo.py 头部说明放置
# 2) 直接用本包标注（推荐，省一步）：
cp -r annotations/DamCrack-Sub5 <项目根>/datasets/
#    然后把下载的图像放进 datasets/DamCrack-Sub5/images/{train,val,test}/
#    （其余数据集同理：Batches / Incremental / DSI 两种）
# 3) 或者从原始数据重建标注：
python data/14_build_merged_5sub.py
python data/16_gen_skeleton_merged.py      # skeleton 伪标签（可选，用于骨架辅助实验）
```

### 4.4 冒烟测试（先小跑 1 个 epoch 确认全链路通）

```bash
cd <项目根>
yolo detect train data=datasets/DamCrack-Sub5/data.yaml model=yolo11n.pt epochs=1 imgsz=640 batch=8 name=smoke
```

跑通（`runs/smoke/` 里出现 `weights/best.pt`）再进下一步。

### 4.5 各章节复现速查

| 论文内容 | 命令 |
|---|---|
| §4.1 基线（3 种子） | `python training/19_full_seeds.py --seeds 42,123,456 --name e6full` |
| §4.2 增强消融多种子 | 同上 + `runs_args/e7_aug`、`e8_multi` 里的参数 |
| §4.4 K-shot 矩阵 | `python training/15_kshot_matrix.py --seeds 42` |
| §4.5/4.6 成像与补救 | `python evaluation/21_sim_imaging.py --device 0` → `python evaluation/24_remedy_experiment.py --device 0` |
| §4.7 分辨率 | `python training/57_resolution_experiment.py --imgsz 960` |
| §4.9 跨批次 | `python data/41_build_batch_datasets.py` → `python evaluation/42_cross_batch.py --device 0` |
| §4.11 增量 | `python training/30_incremental_experiment.py --device 0` |
| §4.12 重配平（DACW/DAAA/DBJT） | `python training/49_method_experiments.py --seed 42,123,456` |
| §4.13 标注预算曲线 | `python training/62_annot_budget.py` |
| §4.14 DSI 验证 | `python evaluation/54_metric_suite.py` |
| 全部图表 | 见路径 B |

（每个脚本的 `--help` 都有参数说明；报错时先看 `docs/REPRODUCE.md` 的 Notes。）

### 4.6 常见问题速查

| 问题 | 原因 | 解法 |
|---|---|---|
| `FileNotFoundError: E:\Arceno\...` | 脚本里的 `ROOT` 常量是作者机器路径 | 改成你自己的项目根（每个脚本顶部一处） |
| `WinError 5` / DataLoader 起不来 | 受限环境不支持命名管道 | 用正常权限的终端，或训练时 `workers=0` |
| 显存不够（8GB） | batch 太大 | `batch=8` 或 `4`，保持 `imgsz=640` |
| 想开骨架辅助监督 | 默认关闭 | 训练参数加 `skeleton_aux=0.5`，且数据要有 `skeleton/` 目录 |
| 类感知 CopyPaste | 默认关闭 | 加 `copy_paste=0.3 copy_paste_classes=[3,4]` |
| 图脚本读不到数据 | `runs/` 布局不对 | 先 `cp -r results/* <ROOT>/runs/` |
| Git 提示 CRLF/LF | Windows 行尾 | 已用 `.gitattributes` 统一，忽略警告即可 |
| 训练中途 AMP 报 inf/nan | 梯度缩放溢出 | 本包 `trainer.py` 已带保护补丁（7 个文件之一），确认已覆盖 |

---

## 5. 发布与上传

1. 用 GitHub 账号登录 [zenodo.org](https://zenodo.org) → Settings → GitHub，开启对应仓库的自动归档（打 release 即自动存档），或直接上传 `reproducibility_package.zip`；
2. 拿到 DOI 后，回填到论文的 Data availability 段落和本包 README 的 "Data and results" 一节。

---

## 6. 许可与引用

- 许可：**AGPL-3.0**（包内含修改过的 Ultralytics 代码，必须沿用其许可；全仓统一最省事）。
- 引用：引用本论文；若使用了标注或结果文件，再引用 Zenodo 归档（DOI 待补）。

---

## 7. 数据集
DamSegment：https://doi.org/10.17632/z5z6gtt5t4.1
DamCrack：https://doi.org/10.5281/zenodo.17274706
均为公开数据集

---
