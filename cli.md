# RoboPianist Command Line Interface

## 查看帮助

```bash
robopianist --help
```

---

## 命令列表

### 1. `--version`

打印 robopianist 的版本号。

```bash
robopianist --version
```

| 参数 | 类型 | 说明 |
|---|---|---|
| `--version` | flag | 输出当前版本号 |

---

### 2. `--check-pig-exists`

检查 PIG 数据集是否正确下载并完成预处理。已验证通过则输出绿色提示，未通过则输出红色警告。

```bash
robopianist --check-pig-exists
```

| 参数 | 类型 | 说明 |
|---|---|---|
| `--check-pig-exists` | flag | 检查 PIG 数据集状态（应有 150 首） |

---

### 3. `player`

播放指定的 MIDI 文件（通过 FluidSynth 合成音频后用 PyAudio 播放）。

```bash
robopianist player --midi-file <path> [--stretch <float>] [--shift <int>]
```

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|---|---|---|---|---|
| `--midi-file` | `str` | 是 | - | MIDI 文件路径（`.mid` 或 `.proto`） |
| `--stretch` | `float` | 否 | `1.0` | 时间拉伸系数。>1 变慢，<1 变快 |
| `--shift` | `int` | 否 | `0` | 移调量（半音数），正数升调，负数降调 |

示例：
```bash
robopianist player --midi-file robopianist/music/data/rousseau/twinkle-twinkle-trimmed.mid
robopianist player --midi-file song.mid --stretch 0.8 --shift 2
```

---

### 4. `preprocess`

将 PIG 原始数据集（CSV + 指法文本文件）转换为 `.proto` protobuf 格式。这是离线的**一次性预处理**，处理完后 150 首曲目将保存在指定目录中。

```bash
robopianist preprocess --dataset-dir <path> [--save-dir <path>]
```

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|---|---|---|---|---|
| `--dataset-dir` | `str` | 是 | - | PIG 数据集根目录，须包含 `FingeringFiles/` 子目录和 `List.csv` |
| `--save-dir` | `str` | 否 | `robopianist/music/data/pig_single_finger/` | 输出 `.proto` 文件的目录 |

示例：
```bash
robopianist preprocess --dataset-dir /path/to/PIG/dataset
robopianist preprocess --dataset-dir /path/to/PIG/dataset --save-dir ./my_pig_output
```

预处理逻辑（[cli.py#L231-288](file:///home/jz/robopianist/robopianist/cli.py#L231-288)）：
- 从 `FingeringFiles/` 下读取所有 `.txt` 指法文件
- 每首曲只保留第一个指法版本（去重后共 150 首）
- 从 `List.csv` 获取曲名
- 解析 TSV 格式的指法文件，将音名转为 MIDI 编号，指法编码写入 `part` 字段
- 序列化为 `NoteSequence` protobuf 格式的 `.proto` 文件

---

### 5. `soundfont`

管理音色库（SoundFont `.sf2` 文件），用于 MIDI 音频合成。

```bash
# 查看可用音色库
robopianist soundfont --list

# 切换默认音色库
robopianist soundfont --change-default

# 下载额外的音色库
robopianist soundfont --download
```

| 参数 | 类型 | 说明 |
|---|---|---|
| `--list` | flag | 列出 `robopianist/soundfonts/` 下所有可用音色库，标注当前默认 |
| `--change-default` | flag | 交互式切换默认音色库，会列出所有选项供用户选择编号 |
| `--download` | flag | 下载额外音色库（TimGM6mb、SalamanderGrandPiano），交互式选择 |

可下载的音色库：

| 名称 | 说明 |
|---|---|
| `TimGM6mb` | 轻量通用音色库 |
| `SalamanderGrandPiano` | 高质量大钢琴采样音色库 |

示例：
```bash
robopianist soundfont --list
robopianist soundfont --change-default
robopianist soundfont --download
```
