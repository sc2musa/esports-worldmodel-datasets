---
license: Apache License 2.0
---

# 电子竞技世界模型数据集

[English](README.md) | [简体中文](README.zh-CN.md) | [ModelScope 数据集](https://www.modelscope.cn/datasets/meisah111/Esports_world_model_datasets)

本数据集通过解码真实人类玩家的比赛回放，构建覆盖 **StarCraft II（星际争霸 II）、Dota 2 和 Counter-Strike 2（CS2）** 的多模态游戏轨迹，分别对应即时战略（RTS）、多人在线战术竞技（MOBA）和战术第一人称射击（FPS）环境。

数据将玩家视角视频与游戏原生动作、引擎状态及事件记录、时间对齐信息结合，可用于动作条件世界模型、视频理解、视觉语言动作模型（VLA），以及部分可观测条件下的多智能体推理研究。

## 为什么使用电竞回放？

比赛回放能够将玩家决策与整局比赛中的后续结果联系起来。通过回放解码，可以在获得视觉观测的同时获取结构化监督，从而研究未来画面与底层游戏状态的变化。

- **真实人类竞技行为：** 数据来源包括天梯、匹配和职业赛事回放。
- **多模态监督：** 视频、动作、状态、事件，以及游戏版本或比赛元数据。
- **多个玩家视角：** SC2 的双玩家视角、Dota 2 最多十个玩家槽位，以及 CS2 的玩家第一人称片段。
- **多个时间尺度：** 瞬时控制、秒级战术交互和分钟级战略决策。
- **跨游戏类型：** RTS 的宏观运营与微观控制、MOBA 的团队协作，以及 FPS 的第一人称战术控制。

这里的动作指解码得到的游戏指令或引擎输入，三款游戏并不共享统一的键盘鼠标动作表示。结构化状态也不一定等同于完整全局状态，其可见性与实体覆盖范围取决于游戏和具体提取格式。

## 数据规模

### 汇报材料中的统计快照

项目汇报材料给出了以下统计。这些数值对应汇报中的数据快照，并非当前线上全部文件的实时统计。

| 指标 | StarCraft II | Dota 2 | Counter-Strike 2 |
|---|---:|---:|---:|
| 游戏类型 / 玩家规模 | RTS / 1v1 | MOBA / 5v5 | 战术 FPS / 5v5 |
| 对局数量 | 2,145 | 273 | 50 局逻辑对局 |
| 比赛时间轴时长 | 395.8 小时 | 195.5 小时 | 34.8 小时 |
| 玩家视角视频时长 | 791.5 小时 | 1,955.0 小时 | 138.4 小时 |
| 其他覆盖范围 | 4,290 个玩家视角；18 张地图 | 381 名职业选手；82 支战队 | 7,623 个 POV 视频片段；1,057 个回合 |
| 结构化数据规模 | 约 5,900 万个轨迹步 | 约 77 亿次状态变化；2.2 亿条事件 | 16 FPS 视频与每秒 64 tick 的回放时间轴对齐 |

**汇报快照合计：** 2,468 局对局、626.1 小时比赛时间轴，以及 2,884.9 小时玩家视角视频（约 2,885 小时）。

比赛时间轴时长对每局比赛只统计一次；玩家视角时长对不同视角的视频时长求和。二者不能混用。CS2 的实际 POV 覆盖还受可用视角、回合划分及玩家死亡截断影响。状态变化、事件、轨迹步和视频片段也是不同计量单位，不能直接相加。

### 2026-10-07 查看到的线上发布元数据

线上发布内容已超出汇报中的统计快照：

- **SC2 标准版 `v1.0.0`：** `meta/info.json` 记录了 2,320 局已索引对局、**2,314 局可训练对局**和 **4,628 个已完成玩家视角**。索引还单独报告了筛选更严格的默认可用子集：**2,295 局**、430.635 小时比赛时间轴、861.271 小时玩家视角视频，以及 69,452,882 个双视角轨迹步。标准版与默认可用子集采用不同筛选条件。这些元数据生成于 2026-07-28。[标准版元数据](https://www.modelscope.cn/datasets/meisah111/Esports_world_model_datasets/resolve/master/esports_world_model_full_20260901/starcraft2/publish/standard/v1.0.0/meta/info.json)、[索引统计](https://www.modelscope.cn/datasets/meisah111/Esports_world_model_datasets/resolve/master/esports_world_model_full_20260901/starcraft2/publish/index/stats.json)。
- **Dota 2：** `gdrive/matches/` 与 `gdrive2/matches/` 均包含比赛数据包，且存在重复比赛 ID。文档保留汇报中的 273 局作为参考统计；目录存在本身不能证明其是具有十个完整、对齐视角的训练样本。使用时应按比赛 ID 去重，并核对元数据和文件。
- **CS2：** 线上实测报告记录了 50 局逻辑对局、52 个解码 demo 单元和 7,623 个标准 POV 片段。采用更严格的训练可用条件后，子集包含 **7,341 个配对片段**、134.69 小时视频，以及约 617.40 GB 视频和标准 Parquet 数据。该报告生成于 2026-07-27。[实测报告](https://www.modelscope.cn/datasets/meisah111/Esports_world_model_datasets/resolve/master/esports_world_model_full_20260901/counter_strike2/reports/dataset_scale_summary.md)。

这些快照具有不同的筛选口径，不应相加形成新的跨游戏总量。实验应以实际选用子集附带的元数据为准。

## 视觉示例

以下玩家视角示例来自项目汇报材料：

| StarCraft II | Dota 2 | Counter-Strike 2 |
|---|---|---|
| ![StarCraft II 玩家视角示例](docs/assets/starcraft2-preview.png) | ![Dota 2 玩家视角示例](docs/assets/dota2-preview.png) | ![Counter-Strike 2 第一人称示例](docs/assets/counter-strike2-preview.png) |

## 仓库组织

数据主体位于 `esports_world_model_full_20260901/`。下面展示已核对的部分线上目录，并非完整文件清单。

```text
esports_world_model_full_20260901/
├── starcraft2/
│   ├── publish/
│   │   ├── index/                  # 文件索引、来源标签及统计
│   │   ├── standard/v1.0.0/        # 类型化轨迹、视频及元数据
│   │   └── derived/               # 派生结果
│   ├── dataset_prep/              # 数据准备脚本及格式说明
│   └── extracted/                 # 解压后的源数据
├── dota2/
│   ├── gdrive/
│   │   ├── matches/               # 按比赛组织的多模态数据包
│   │   ├── demos/                 # 回放源文件
│   │   ├── decoded/               # 解码输出
│   │   ├── delta/                 # 状态变化输出
│   │   └── messages/              # 消息输出
│   └── gdrive2/matches/           # 补充或重复的比赛数据包
└── counter_strike2/
    ├── extracted/                 # 解码后的比赛与回合数据
    ├── reports/                   # 清单、质量及规模实测报告
    ├── scripts/                   # 数据处理脚本
    └── variants/                  # 不同来源的替代版本
```

仓库中还包含源压缩包和处理中间产物。因此，仓库总存储量不等于去重后的训练集体积。建议优先使用标准索引和已核验的视频/状态配对数据，避免直接递归加载所有文件。

## 数据格式与时间对齐

### StarCraft II

![StarCraft II 回放解码流程](docs/assets/starcraft2-decoding-pipeline.png)

*原始回放解码流程。图中为原始解码输出；标准版采用下文所述的 Parquet 布局。*

解码流程先匹配回放的游戏版本，再启动对应 SC2 引擎，保留战争迷雾，分别重建两个玩家视角。在共享的 `game_loop` 时间轴上提取 RGB 视频、小地图视频、动作、单位、资源、分数、升级和镜头信息。

1. 读取回放中的游戏版本、base build 和 data version，启动匹配的引擎。
2. 保留战争迷雾，从两个玩家视角分别重放同一局比赛。
3. 沿共享的 `game_loop` 时间轴采样，提取 RGB/小地图帧、动作、单位、资源和镜头信息。
4. 将原始解码记录转换为类型化 Parquet 轨迹，并关联各视角的 MP4 视频。
5. 核验必需文件、轨迹时钟、视频帧索引及比赛级数据划分的一致性。

标准版为每个玩家视角保存一份类型化 Parquet 轨迹，并配套 MP4 视频：

```text
starcraft2/publish/standard/v1.0.0/
├── meta/
│   ├── info.json
│   ├── schema.json
│   ├── games.parquet
│   ├── players.parquet
│   ├── views.parquet
│   ├── clips.parquet
│   ├── files.parquet
│   └── vocab/build=<data_build>/ids.parquet
├── data/build=<data_build>/context=<context>/split=<split>/
│   └── <game_id>.p<player_index>.trajectory.parquet
└── videos/build=<data_build>/context=<context>/split=<split>/
    ├── <game_id>.p<player_index>.rgb.mp4
    └── <game_id>.p<player_index>.minimap.mp4
```

主要轨迹字段包括：

| 字段类别 | 字段 / 含义 |
|---|---|
| 标识 | `game_id`、`view_id`、`player_index`、`step_index` |
| 时间 | `frame_index`、`game_loop`、`prev_game_loop`、`timestamp_s` |
| 观测 | 类型化 `observation` 结构，包含镜头、玩家资源、分数、升级和单位等 |
| 动作 | 变长 `action` 列表，包含镜头移动和单位指令 |
| 轨迹边界 | `is_first`、`is_last`、`is_terminal`、`is_truncated` |
| 派生终局信号 | `reward_terminal`：终局胜 `+1`、负 `-1`、平 `0`；其他步为零 |

**动作对齐必须区分原始输出与标准版。** 原始解码器将动作记录在这些动作执行后到达的当前观测上；标准版转换器已平移一次，使 `step[t].action` 对应从 `observation[t]` 到 `observation[t+1]` 的动作。首个观测之前的动作保存在 `views.reset_actions`，最后一步不含动作。使用标准版时，不应再次平移动作。

实体 tag 仅在单个玩家视角和一次解码运行中有效，不能直接用于跨视角匹配同一单位。单位、技能、升级和 buff ID 与游戏 build 相关。SC2 结构化观测也具有玩家可见性边界，不能将双视角数据自动视为完整全知状态。

标准版提供比赛级划分信息，同一比赛的两个玩家视角保持在同一划分中。完整约定请参阅[标准版格式说明](https://www.modelscope.cn/datasets/meisah111/Esports_world_model_datasets/resolve/master/esports_world_model_full_20260901/starcraft2/dataset_prep/STANDARD_RELEASE.md)及线上 `meta/schema.json`。

### Dota 2

![Dota 2 回放解码流程](docs/assets/dota2-decoding-pipeline.png)

*回放解析、多玩家视角渲染，以及帧、tick 与动作的对齐流程。*

解码流程将回放中的引擎记录与渲染后的玩家视角关联起来：

1. 解析 `.dem` 回放，提取动作、实体状态变化和事件/消息记录。
2. 沿比赛时间轴分段渲染可用的玩家视角。
3. 合成各玩家的视频，并建立视频帧与回放 tick 的对应关系。
4. 通过各槽位的对齐表，将视频帧与玩家动作及引擎记录关联。
5. 打包比赛元数据、共享结构化表、各槽位视频和缺失文件信息。

已核对的代表性比赛数据包采用以下布局：

```text
dota2/gdrive/matches/<match_id>/
├── match.json
├── delta.parquet
├── messages.parquet
├── delta_summary.json             # 部分数据包包含
├── message_summary.json           # 部分数据包包含
└── slot00/ ... slot09/
    ├── video.mp4
    ├── alignment.parquet
    └── frame_actions.parquet
```

`match.json` 记录 schema 版本、比赛与联赛信息、时长、回放指纹、tick rate、各槽位文件及缺失项。`delta.parquet` 保存引擎状态变化，`messages.parquet` 保存解码消息。各槽位的表将视频帧与回放时间及动作关联起来。状态变化行是增量记录，并不是相互独立的完整世界状态；重建状态时需遵循相应解码器语义。

应使用数据包自身的时钟和对齐表，不能假定所有样本具有统一帧率或固定的帧/tick 比例。同一比赛 ID 可能出现在两组源目录中，应先选择完整的标准数据包，再统计或划分数据。可参阅[代表性比赛 manifest](https://www.modelscope.cn/datasets/meisah111/Esports_world_model_datasets/resolve/master/esports_world_model_full_20260901/dota2/gdrive/matches/8447645427/match.json)。

### Counter-Strike 2

![Counter-Strike 2 回放解码流程](docs/assets/counter-strike2-decoding-pipeline.png)

*回放解析、第一人称录制，以及按回合和玩家打包并进行帧/tick 对齐的流程。*

解码流程将 `.dem` 回放解析为状态、输入和事件表，同时录制玩家 POV 视频，再按回合和玩家打包对齐。汇报中使用 `demoparser2` 进行解析，通过包含 `csdm` 和 FFmpeg 的回放录制流程生成视频。

1. 使用 `demoparser2` 解析回放，提取玩家状态、玩家输入和事件表。
2. 重放并录制可用的玩家第一人称视角，将采集帧编码成视频。
3. 按回合和玩家切分输出，并保留对应的回放时间轴。
4. 根据数据包中的时间信息，将视频帧与回放 tick、状态和事件对齐。
5. 保存比赛元数据，将完整、部分可用和未配对输出分类，供训练子集筛选。

代表性输出包括：

| 范围 | 文件 |
|---|---|
| 比赛级 | `metadata.json`、`player_states.parquet`、`player_inputs.parquet` 及事件表等 |
| 回合 / 玩家级 | `video.mp4`、`state.parquet`，以及可用的回合级 `events.parquet` |
| 报告 | `dataset_scale_stats.json`、`dataset_scale_summary.md` 和 demo 清单 |

参考录制设置为 **16 FPS / 每秒 64 tick**，即一帧视频对应四个回放 tick。实际对齐需读取数据包中的时间信息，并考虑片段起点和截断。CS2 数据不保证每个回合均具备十个完整玩家视角。

实测报告区分标准输出与训练可用的配对片段，并列出了缺元数据、仅有状态、仅有视频和损坏压缩包等情况。选择实验子集时应使用这些质量分类。

## 快速开始

### 先下载小型元数据文件

```bash
python -m pip install pandas pyarrow
```

以下示例只下载三份 SC2 元数据文件，不下载轨迹或视频：

```python
import json
import shutil
from pathlib import Path
from urllib.parse import urlencode
from urllib.request import urlopen

import pandas as pd

DATASET_ID = "meisah111/Esports_world_model_datasets"
REVISION = "master"  # 实验时记录所使用的 revision。
SC2_ROOT = "esports_world_model_full_20260901/starcraft2/publish/standard/v1.0.0"
LOCAL_DIR = "./esports-world-model"

def download_file(file_path):
    query = urlencode({"Revision": REVISION, "FilePath": file_path})
    url = f"https://www.modelscope.cn/api/v1/datasets/{DATASET_ID}/repo?{query}"
    target = Path(LOCAL_DIR) / file_path
    target.parent.mkdir(parents=True, exist_ok=True)
    with urlopen(url, timeout=60) as response, target.open("wb") as output:
        shutil.copyfileobj(response, output)
    return target


metadata = {
    name: download_file(f"{SC2_ROOT}/meta/{name}")
    for name in ("info.json", "schema.json", "views.parquet")
}

info = json.loads(Path(metadata["info.json"]).read_text(encoding="utf-8"))
views = pd.read_parquet(metadata["views.parquet"])
print(info["dataset_release"], info["status"])
print(views[["game_id", "view_id", "split", "step_count"]].head())
```

### 读取一个训练轨迹

接着运行以下代码。SC2 视角索引中的文件路径相对于 SC2 标准版目录：

```python
import pyarrow.parquet as pq

view = views.loc[views["split"] == "train"].iloc[0]
trajectory_path = download_file(f"{SC2_ROOT}/{view['data_path']}")

# 按 row group 读取，避免一次将整局轨迹载入内存。
trajectory = pq.ParquetFile(trajectory_path)
sample = trajectory.read_row_group(
    0,
    columns=["step_index", "game_loop", "timestamp_s", "observation", "action"],
)
print(sample.select(["step_index", "game_loop", "timestamp_s"]).slice(0, 2).to_pylist())
```

RGB 与小地图视频可以通过索引中的 `rgb_path` 和 `minimap_path` 单独下载，下载前先检查索引中的文件大小。Dota 2 可先读取 `match.json`，CS2 可先读取清单与质量报告。单个文件可使用 [ModelScope 单文件下载 API](https://github.com/modelscope/modelscope/blob/master/modelscope/hub/file_download.py)，批量子集可使用带明确文件过滤条件的 [snapshot download](https://github.com/modelscope/modelscope/blob/master/modelscope/hub/snapshot_download.py)。

## 支持的研究方向

| 研究方向 | 示例任务 |
|---|---|
| 动作条件世界模型 | 根据观测历史和动作序列预测未来观测、状态变化及事件 |
| 部分可观测推理 | 根据玩家可获取的信息推断隐藏对手及潜在威胁 |
| 多智能体与多视角建模 | 研究跨视角一致性、对手行为和队友协作 |
| 视频理解与 VQA | 回答依赖时间过程的移动、交战、技能释放和战术结果问题 |
| 跨游戏迁移 | 检验时序表征能否在 RTS、MOBA 和 FPS 之间迁移 |
| 玩家水平与版本适配 | 利用可用的赛事/MMR 分层信息，对不同 build 或地图进行适配 |
| 长时序推理 | 将即时操作与数分钟后的战术和战略结果关联起来 |

这些内容是数据支持的研究方向，并不表示已有相应 benchmark 实验结果。VLA 研究需要游戏专用的动作表示和执行适配器。汇报还描述了一个 SC2 VQA 原型，包含 90 个玩家视角片段、每个片段六个英文问题；本文档不将其视为已确认公开发布的评测数据包。

## 评测建议与已知限制

- **避免划分泄漏：** 同一比赛的视角、回合、片段和重复数据必须归入同一划分。遵循已发布的 SC2 划分元数据。测试泛化时可按系列赛或来源分组，不应将同一回放的帧随机拆到训练集和测试集中。
- **保留信息边界：** 区分玩家可见观测和引擎特权信息。用于监督的隐藏信息不能在部分可观测评测中悄然成为模型输入。
- **核对时间对齐：** 保留时间戳、回放时钟、帧索引及游戏特定的动作约定。SC2 原始解码输出与标准版轨迹使用不同的动作对齐方式。
- **选择完整配对：** 核验所需视频与结构化数据；对不完整样本应排除或单独评测。源目录存在或转换记录成功，本身不能证明线上完整数据均已到位。
- **处理版本与标识差异：** 引擎 build、地图、游戏原生 ID 和动作空间不同，跨游戏使用需要明确的标准化层。
- **正确理解规模：** 存储量包含压缩包和中间产物，不同表也可能描述同一时间轴。目录数和 Parquet 总行数不等于互不重复的转移样本数。

SC2 标准版元数据对非职业玩家身份做了脱敏，职业身份等待单独核验的身份表。该行为针对 SC2 标准格式，不应推广为所有原始文件及其他游戏数据均采用了相同处理。

## 许可与引用

ModelScope 仓库声明的许可证为 **Apache License 2.0**。这反映仓库的许可声明，并不等于对第三方游戏素材、商标或所有上游回放内容拥有权利；再分发相关材料时应查阅其来源条款。

使用数据集时，可引用仓库，并记录实验使用的子集及 revision：

```bibtex
@misc{esports_world_model_dataset,
  author       = {Ma, Weiyu},
  title        = {Esports World Model Dataset},
  howpublished = {ModelScope dataset repository},
  url          = {https://www.modelscope.cn/datasets/meisah111/Esports_world_model_datasets},
  note         = {Repository citation; include the subset and revision used}
}
```

这是仓库引用，并非论文引用。数据问题与使用反馈可通过 ModelScope 数据集讨论页提出。
