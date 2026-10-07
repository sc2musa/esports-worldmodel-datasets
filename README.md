---
license: Apache License 2.0
---

# Esports World Model Dataset

[English](README.md) | [简体中文](README.zh-CN.md) | [ModelScope dataset](https://www.modelscope.cn/datasets/meisah111/Esports_world_model_datasets)

Replay-decoded human gameplay across **StarCraft II, Dota 2, and Counter-Strike 2**, covering real-time strategy (RTS), multiplayer online battle arena (MOBA), and tactical first-person shooter (FPS) environments.

The dataset pairs rendered player-view videos with game-native actions, engine-derived state and event records, and temporal alignment information. It supports research on action-conditioned world models, video understanding, vision-language-action models, and multi-agent reasoning under partial observability.

## Why esports replays?

Competitive replays connect player decisions to their consequences over an entire match. Decoding a replay provides structured supervision alongside visual observations, making it possible to study both visual prediction and changes in the underlying game state.

- **Human competitive play:** ladder, matchmaking, and professional replay sources.
- **Multiple modalities:** video, actions, states, events, and game or match metadata.
- **Multiple perspectives:** two player views in SC2, up to ten player slots in Dota 2, and player POV clips in CS2.
- **Multiple time scales:** immediate control, second-level tactics, and strategic decisions spanning minutes.
- **Cross-genre dynamics:** macro/micro control in RTS, team coordination in MOBA, and embodied tactical control in FPS.

Actions are decoded game commands or recorded engine inputs. They should not be assumed to share a common keyboard/mouse representation across games. Likewise, a structured state record is not necessarily a complete global state: its visibility and entity coverage depend on the game and extraction format.

## Dataset scale

### Reference corpus snapshot

The project presentation reports the following snapshot. These figures describe that corpus, rather than a live count of every file currently hosted.

| Metric | StarCraft II | Dota 2 | Counter-Strike 2 |
|---|---:|---:|---:|
| Genre / players | RTS / 1v1 | MOBA / 5v5 | Tactical FPS / 5v5 |
| Matches | 2,145 | 273 | 50 logical matches |
| Gameplay hours | 395.8 | 195.5 | 34.8 |
| Player-view hours | 791.5 | 1,955.0 | 138.4 |
| Additional coverage | 4,290 player views; 18 maps | 381 professional players; 82 teams | 7,623 POV clips; 1,057 rounds |
| Structured scale | Approximately 59 million trajectory steps | Approximately 7.7 billion state changes; 220 million events | Video/state alignment at 16 FPS and 64 ticks/s |

**Reference total:** 2,468 matches, 626.1 gameplay hours, and 2,884.9 player-view hours (approximately 2,885 hours).

Gameplay hours count each match timeline once. Player-view hours sum the durations of individual rendered perspectives. They are different measures; CS2 POV coverage also varies with available players, rounds, and death-truncated clips. State changes, event records, trajectory steps, and clips are different units and should not be added together.

### Published metadata observed on 2026-10-07

The online release has evolved beyond the presentation snapshot:

- **SC2 standard release `v1.0.0`:** `meta/info.json` records 2,320 indexed games, **2,314 training-ready games**, and **4,628 completed player views**. The index separately reports a more restrictive default eligible subset of **2,295 games**, 430.635 gameplay hours, 861.271 player-view hours, and 69,452,882 two-view trajectory steps. The standard release and default eligible subset use different filters. These metadata files were generated on 2026-07-28. [Release metadata](https://www.modelscope.cn/datasets/meisah111/Esports_world_model_datasets/resolve/master/esports_world_model_full_20260901/starcraft2/publish/standard/v1.0.0/meta/info.json), [index statistics](https://www.modelscope.cn/datasets/meisah111/Esports_world_model_datasets/resolve/master/esports_world_model_full_20260901/starcraft2/publish/index/stats.json).
- **Dota 2:** both `gdrive/matches/` and `gdrive2/matches/` contain match packages, with overlapping match IDs. The presentation's 273-match figure remains the reference statistic; directory presence alone does not establish a complete, aligned ten-view training sample. Deduplicate by match ID and check each package's metadata and assets.
- **CS2:** the published measurement report records 50 logical matches, 52 decoded demo units, and 7,623 canonical POV clips. Its stricter training-ready subset contains **7,341 paired clips**, 134.69 video hours, and approximately 617.40 GB of video plus canonical Parquet data. This report was generated on 2026-07-27. [Measurement report](https://www.modelscope.cn/datasets/meisah111/Esports_world_model_datasets/resolve/master/esports_world_model_full_20260901/counter_strike2/reports/dataset_scale_summary.md).

Do not combine these independently filtered snapshots into a new cross-game total. Use the metadata accompanying the subset selected for an experiment.

## Visual examples

Player-view examples from the project presentation:

| StarCraft II | Dota 2 | Counter-Strike 2 |
|---|---|---|
| ![StarCraft II player-view example](docs/assets/starcraft2-preview.png) | ![Dota 2 player-view example](docs/assets/dota2-preview.png) | ![Counter-Strike 2 POV example](docs/assets/counter-strike2-preview.png) |

## Repository organization

The payload root is `esports_world_model_full_20260901/`. Selected directories observed in the published repository are shown below; this is not an exhaustive inventory.

```text
esports_world_model_full_20260901/
├── starcraft2/
│   ├── publish/
│   │   ├── index/                  # Inventory, source labels, and statistics
│   │   ├── standard/v1.0.0/        # Typed trajectories, video, and metadata
│   │   └── derived/               # Derived artifacts
│   ├── dataset_prep/              # Preparation scripts and format documentation
│   └── extracted/                 # Extracted source assets
├── dota2/
│   ├── gdrive/
│   │   ├── matches/               # Per-match multimodal packages
│   │   ├── demos/                 # Replay sources
│   │   ├── decoded/               # Decoder outputs
│   │   ├── delta/                 # State-change outputs
│   │   └── messages/              # Message outputs
│   └── gdrive2/matches/           # Additional / overlapping match packages
└── counter_strike2/
    ├── extracted/                 # Decoded match and round assets
    ├── reports/                   # Inventory, quality, and scale measurements
    ├── scripts/                   # Data preparation utilities
    └── variants/                  # Alternative source versions
```

The repository also contains source archives and intermediate artifacts. Its total storage size is not the size of a deduplicated training set. Prefer indexed standard outputs and verified video/state pairs over recursively loading every file.

## Formats and temporal alignment

### StarCraft II

![StarCraft II replay decoding pipeline](docs/assets/starcraft2-decoding-pipeline.png)

*Original replay decoding workflow. The diagram shows raw decoder outputs; the standard release uses the Parquet layout documented below.*

The decoding pipeline matches the replay's game version, launches the corresponding SC2 engine, and reconstructs both player perspectives with fog of war preserved. It extracts RGB video, minimap video, actions, unit information, resources, scores, upgrades, and camera information on a shared `game_loop` timeline.

1. Read replay metadata, including the game version, base build, and data version, and launch the matching engine.
2. Replay the same match from each player's perspective with fog of war preserved.
3. Sample observations on the shared `game_loop` timeline and extract RGB/minimap frames, actions, unit information, resources, and camera data.
4. Convert raw decoder records into typed Parquet trajectories and associate each view with its MP4 streams.
5. Validate required assets, trajectory timing, media indices, and game-level split consistency.

The standard release stores one typed Parquet trajectory per player view, alongside MP4 video:

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

Key trajectory fields include:

| Field group | Fields / meaning |
|---|---|
| Identity | `game_id`, `view_id`, `player_index`, `step_index` |
| Timing | `frame_index`, `game_loop`, `prev_game_loop`, `timestamp_s` |
| Observation | Typed `observation` struct, including camera, player resources, score, upgrades, and units |
| Action | Ragged `action` list of camera moves and unit commands |
| Boundaries | `is_first`, `is_last`, `is_terminal`, `is_truncated` |
| Derived terminal signal | `reward_terminal`: win `+1`, loss `-1`, tie/draw `0` at the last step; zero otherwise |

**Action alignment matters.** Raw decoder actions describe commands leading into the current observation. The standard converter shifts this convention: `step[t].action` leads from `observation[t]` to `observation[t+1]`. Commands before the first observation are retained in `views.reset_actions`; the final step has no action. Do not shift standard-release actions a second time.

Entity tags are local to a player view and decoder run, so they cannot directly identify the same unit across views. Unit, ability, upgrade, and buff IDs are build-specific. SC2 structured observations also retain player-specific visibility; paired views should not be presented as an automatically complete omniscient state.

The standard release includes game-level split metadata and keeps both views of a game together. Consult [the standard-release specification](https://www.modelscope.cn/datasets/meisah111/Esports_world_model_datasets/resolve/master/esports_world_model_full_20260901/starcraft2/dataset_prep/STANDARD_RELEASE.md) and the published `meta/schema.json` for the full contract.

### Dota 2

![Dota 2 replay decoding pipeline](docs/assets/dota2-decoding-pipeline.png)

*Replay parsing, multi-player-view rendering, and frame/tick/action alignment.*

The decoding workflow connects replay-derived engine records to rendered player perspectives:

1. Parse the `.dem` replay into actions, entity state changes, and event/message records.
2. Render available player perspectives in segments along the match timeline.
3. Assemble each player's video and map its frames to replay ticks.
4. Associate frames with player actions and engine records through per-slot alignment tables.
5. Package match metadata, shared structured tables, per-slot videos, and missing-asset information.

A representative published match has this layout:

```text
dota2/gdrive/matches/<match_id>/
├── match.json
├── delta.parquet
├── messages.parquet
├── delta_summary.json             # Present in some packages
├── message_summary.json           # Present in some packages
└── slot00/ ... slot09/
    ├── video.mp4
    ├── alignment.parquet
    └── frame_actions.parquet
```

`match.json` records the schema version, match and league information, duration, replay fingerprint, tick-rate information, per-slot files, and missing assets. `delta.parquet` contains engine state changes, while `messages.parquet` stores decoded messages. Per-slot tables connect rendered frames to replay timing and actions. State-change rows are delta records rather than independent complete world states; reconstruct state using the applicable decoder semantics.

Read each package's timing and alignment tables rather than assuming a universal frame rate or a fixed frame-to-tick ratio. Match IDs can occur in both source roots; choose a canonical complete package before counting or splitting data. See [a representative match manifest](https://www.modelscope.cn/datasets/meisah111/Esports_world_model_datasets/resolve/master/esports_world_model_full_20260901/dota2/gdrive/matches/8447645427/match.json).

### Counter-Strike 2

![Counter-Strike 2 replay decoding pipeline](docs/assets/counter-strike2-decoding-pipeline.png)

*Replay parsing, POV recording, and round/player packaging with frame-to-tick alignment.*

The pipeline parses `.dem` replays into structured state, input, and event tables; records player POV video; and packages aligned data by round and player. The presentation describes `demoparser2` for parsing and a CS2 replay recording pipeline using `csdm` and FFmpeg.

1. Parse the replay into player states, player inputs, and event tables using `demoparser2`.
2. Replay and record available player POVs, then encode the captured frames into video.
3. Split output by round and player, retaining the corresponding replay timeline.
4. Align video frames with replay ticks, state records, and events using packaged timing information.
5. Record match metadata and classify complete, partial, or unpaired outputs for subset selection.

Representative outputs include:

| Scope | Assets |
|---|---|
| Match | `metadata.json` and structured tables such as `player_states.parquet`, `player_inputs.parquet`, and event tables |
| Round / player | `video.mp4`, `state.parquet`, and round-level `events.parquet` where available |
| Reports | `dataset_scale_stats.json`, `dataset_scale_summary.md`, and decoded-demo inventories |

The reference capture setting is **16 FPS / 64 ticks per second**, corresponding to four replay ticks per video frame. Actual correspondence must come from the packaged timing information, including clip offsets and any truncation. CS2 packages do not guarantee ten complete views for every round.

The measurement report distinguishes canonical output from training-ready paired clips and lists missing metadata, state-only outputs, video-only outputs, and damaged archives. Use those quality categories when choosing an experimental subset.

## Quick start

### Download and inspect small metadata files first

```bash
python -m pip install pandas pyarrow
```

The following example uses the public ModelScope file endpoint to download only three SC2 metadata files, without downloading trajectories, video, or a recursive repository inventory:

```python
import json
import shutil
from pathlib import Path
from urllib.parse import urlencode
from urllib.request import urlopen

import pandas as pd

DATASET_ID = "meisah111/Esports_world_model_datasets"
REVISION = "master"  # Record the revision used for your experiment.
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

### Load one training trajectory

Continue from the example above. Paths in the SC2 view index are relative to the SC2 standard-release directory:

```python
import pyarrow.parquet as pq

view = views.loc[views["split"] == "train"].iloc[0]
trajectory_path = download_file(f"{SC2_ROOT}/{view['data_path']}")

# Read a row group rather than loading an entire match into memory.
trajectory = pq.ParquetFile(trajectory_path)
sample = trajectory.read_row_group(
    0,
    columns=["step_index", "game_loop", "timestamp_s", "observation", "action"],
)
print(sample.select(["step_index", "game_loop", "timestamp_s"]).slice(0, 2).to_pylist())
```

RGB and minimap video can be downloaded separately using the index's `rgb_path` and `minimap_path`. Check file sizes in the index before downloading. For Dota 2, start with `match.json`; for CS2, start with the inventory and quality reports. Use [ModelScope's single-file download API](https://github.com/modelscope/modelscope/blob/master/modelscope/hub/file_download.py) for individual assets, or [snapshot download](https://github.com/modelscope/modelscope/blob/master/modelscope/hub/snapshot_download.py) with explicit file filters for a chosen subset.

## Research uses

| Research direction | Example task |
|---|---|
| Action-conditioned world modelling | Predict future observations, state changes, and events from a history and action sequence |
| Partial-observability reasoning | Infer unseen opponents or future threats from the information available to a player |
| Multi-agent and multi-view modelling | Study cross-view consistency, opponent behaviour, and teammate coordination |
| Video understanding and VQA | Answer temporally grounded questions about movement, engagements, abilities, and tactical outcomes |
| Cross-game transfer | Test whether temporal representations transfer across RTS, MOBA, and FPS |
| Skill and version adaptation | Compare available competition/MMR strata and adapt across builds or maps |
| Long-horizon reasoning | Connect immediate actions to tactical and strategic consequences minutes later |

These are supported research directions, rather than claims of completed benchmark results. VLA work needs a game-specific action representation and an execution adapter. The presentation also describes an SC2 VQA prototype with 90 player-view clips and six English questions per clip; this card does not establish those annotations as a published benchmark package.

## Evaluation and known limitations

- **Prevent split leakage:** all views, rounds, segments, and duplicates from one match belong to one split. Respect published SC2 split metadata. Group by series or source when evaluating generalization; do not randomly split frames from the same replay.
- **Respect information boundaries:** separate player-visible observations from privileged engine records. Hidden information used for supervision must not silently become an input in a partial-observability evaluation.
- **Check alignment:** preserve timestamps, replay clocks, frame indices, and the game-specific action convention. Raw SC2 decoder output and SC2 standard trajectories use different action alignment.
- **Select complete pairs:** verify required videos and structured data, and exclude or explicitly evaluate incomplete packages. A source directory or a successful conversion record alone does not certify the complete hosted payload.
- **Account for versions and identifiers:** engine builds, maps, game-native IDs, and action spaces differ. Cross-game use requires an explicit normalization layer.
- **Interpret counts correctly:** storage includes archives and intermediates; multiple tables may describe the same timeline. Directory counts and Parquet row totals are not unique transition counts.

SC2 release metadata redacts nonprofessional identities and leaves professional identities pending a reviewed identity table. This behavior applies to the SC2 standard format; it should not be assumed for every raw or game-specific asset in this repository.

## License and attribution

The ModelScope repository declares **Apache License 2.0**. This is the repository's stated license; it does not establish ownership of third-party game assets, trademarks, or all upstream replay content. Consult the applicable source terms when redistributing those materials.

If you use the dataset, cite the repository and record the payload subset and revision used:

```bibtex
@misc{esports_world_model_dataset,
  author       = {Ma, Weiyu},
  title        = {Esports World Model Dataset},
  howpublished = {ModelScope dataset repository},
  url          = {https://www.modelscope.cn/datasets/meisah111/Esports_world_model_datasets},
  note         = {Repository citation; include the subset and revision used}
}
```

This is a repository citation, not a paper citation. Questions and issue reports can be raised through the dataset's ModelScope discussion page.
