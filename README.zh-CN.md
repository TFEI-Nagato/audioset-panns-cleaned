# AudioSet-1961094：基于 PANNs 的补全与清理版本

[English](README.md) | 简体中文

本项目以 [PANNs 项目提供的 AudioSet 下载版本](https://github.com/qiuqiangkong/audioset_tagging_cnn#1-download-dataset)为基底，进行缺失音频补全、音频质量检查、异常音频替换和清理，共保留 **1,961,094 条 WAV**。数据集沿用该项目公开的 AudioSet 目录结构和 AudioSet v1 原始 527 类弱标签，可用于 HTS-AT 等音频分类工作流。

这里的“优化”指音频覆盖补全和数据整理。该版本仍有缺失和弱标签不确定性；本项目不代表 Google、PANNs 或 HTS-AT 官方发布，也不宣称已经证明训练精度提升。

## 下载

- 百度网盘：[AudioSet_1961094](https://pan.baidu.com/s/1YRtWpQzuWsWMykfSVgnORQ?pwd=xmsq)
- 提取码：`xmsq`
- 压缩包/分卷共 **125 个文件**，总计 **1,241,013,140,141 字节**，约 1.241 TB（十进制）。
- 分享内容为 WAV 音频压缩包；不包含 HDF5、训练索引、模型权重或官方 CSV 标签文件。
- 完整文件名、字节数、MD5、SHA-256 见 [archive_manifest.json](archive_manifest.json)。

125 个上传源文件已于 2026-10-12 完成本地 MD5/SHA-256 计算。这些值是**本地上传源的内容哈希**。网盘文件名/大小核对不能替代内容哈希核对，目前不宣称云端全部文件已完成内容一致性校验。下载后请按下面的方法自行验证。

## 最终数据规模

统计以本版本对应的官方 v1 CSV 行数为分母。

| 划分 | 官方 CSV 片段数 | 本版本 WAV 数 | 缺失数 |
| --- | ---: | ---: | ---: |
| balanced train | 22,160 | 20,715 | 1,445 |
| eval | 20,371 | 19,044 | 1,327 |
| unbalanced train | 2,041,789 | 1,921,335 | 120,454 |
| **合计** | **2,084,320** | **1,961,094** | **123,226** |

总覆盖率约 **94.09%**。官网页面展示的汇总数字可能与 v1 CSV 不同，本表使用实际 CSV，不将网页数字混作本版本分母。

最终整理记录中的来源分类如下；按最终身份计数，三行互斥。

| 记录的来源类别 | 最终保留条数 |
| --- | ---: |
| 本地 PANNs 基底来源类别 | 1,951,260 |
| YouTube 补全来源类别 | 8,424 |
| Muno459/AudioSet HF 镜像来源类别 | 1,410 |
| **合计** | **1,961,094** |

YouTube 与 HF 两类最终保留合计 **9,834 条**。

## 基于 PANNs 做了哪些改动

### 1. 按原始官方身份补回缺失音频

以官方 CSV 的 YouTube ID、片段时间和原标签为依据识别缺失条目，使用适配后的 PANNs 下载流程重试 YouTube；对可匹配的缺失条目，从 [Muno459/AudioSet](https://huggingface.co/datasets/Muno459/AudioSet) 镜像按单条成员提取音频。

新增/替代音频沿用 PANNs 下载脚本的 **32 kHz、单声道、16-bit PCM WAV** 设置，并按对应片段时间处理。官方 ID、标签和划分不因补全而改变。镜像转码及重新获取的源音频可能与早期 YouTube 编码存在差异，补回不意味着波形与最初发布时逐样本相同。

### 2. 全量检查音频完整性，细分异常

对 PANNs 基底和补回音频检查格式、可解码性、时长、零帧、全零/近静音、静音占比及削波；逐条复验全零候选，并结合官方 `Silence` 标签判断是否属于可保留静音。

清理规则针对格式/解码错误、空音频、名义时长至少 9 秒但实际不足 0.96 秒的片段，以及**没有 Silence 标签**的全零或峰值不高于 −70 dBFS 的音频。尝试恢复后仍不合格的条目从最终 WAV 树中排除，作为缺失处理。轻微时长不足、削波等警告没有被一概当成错误删除。

上述规则保留了带官方 `Silence` 标签的真实静音片段；训练时若需要全部非零输入，应在独立训练视图中处理，不能把发布 WAV 描述为全部非零。

### 3. 检查标签相关性和官方特征相似性

使用 HTS-AT 官方预训练模型进行标签相关性筛查，并提取 VGGish 嵌入与 Google 发布的官方 AudioSet 特征比较，辅助复核新下载和异常音频。

这些模型和特征只用于定位需要复核的异常候选，不是 AudioSet 官方认证标准。没有将模型预测写回标签，也没有仅凭模型分数批量删除正常音频。

### 4. 人工复核争议候选，整合有效替代音频

需要复核的异常候选经过自动筛查和人工听检；确认有效的替代音频按照原相对路径整合，替换对应异常 WAV，未将同一身份追加为重复训练样本。标签不符或无法确认的候选没有进入最终 WAV 树。

### 5. 恢复标准目录，保持标签原样

最终音频文件名为 `Y<YouTube_ID>.wav`；balanced、eval 与 unbalanced 各归原划分，unbalanced 保留 `part00`–`part40` 共 41 个目录。有效补全和替代音频已直接归入对应位置。

五个官方 CSV 保持字节不变，沿用原始 527 类类别索引、MID 和弱标签，不进行重新标注或标签优化。

## 压缩包结构与解压位置

```text
AudioSet_1961094/
├── balanced_train_segments.zip
├── eval_segments.zip
└── unbalanced_train_segments/
    ├── unbalanced_train_segments_part00.z01
    ├── unbalanced_train_segments_part00.z02
    ├── unbalanced_train_segments_part00.zip
    ├── ...
    ├── unbalanced_train_segments_part40.z01
    ├── unbalanced_train_segments_part40.z02
    └── unbalanced_train_segments_part40.zip
```

balanced 和 eval 各一个普通 ZIP；每个 unbalanced part 是一组 **split ZIP**，必须同时下载同名 `.z01`、`.z02`、`.zip` 三个文件并放在同一目录。每个 `.z01`/`.z02` 为 10,485,760,000 字节。不要把分卷改名为 `.7z.001`，也不要将 `.z01` 当成独立压缩包解压。

使用支持 `.z01/.z02/.zip` 分卷格式的解压软件，从该组 `.zip` 打开/解压。balanced/eval 压缩包内部已经包含各自目录；每个 unbalanced part 内部已经包含 `unbalanced_train_segments_partXX/` 目录。因此 balanced/eval 的解压目标是 `dataset_root/audios/`，每个 part 的目标是 `dataset_root/audios/unbalanced_train_segments/`，避免多套一层目录。普通 `unzip` 能列出目录并不保证能直接解压多卷 ZIP。

完成解压并按下文下载官方 CSV 后，数据集应按 PANNs 项目公开的 AudioSet 数据集目录结构放置：

```text
dataset_root/
├── audios/
│   ├── balanced_train_segments/Y<YouTube_ID>.wav
│   ├── eval_segments/Y<YouTube_ID>.wav
│   └── unbalanced_train_segments/
│       ├── unbalanced_train_segments_part00/Y<YouTube_ID>.wav
│       ├── ...
│       └── unbalanced_train_segments_part40/Y<YouTube_ID>.wav
└── metadata/
    ├── balanced_train_segments.csv
    ├── class_labels_indices.csv
    ├── eval_segments.csv
    ├── qa_true_counts.csv
    └── unbalanced_train_segments.csv
```

## 下载后校验与官方标签

在下载所得 `AudioSet_1961094/` 目录内运行，校验清单路径改为本仓库的实际路径：

```bash
sha256sum -c /path/to/audioset-panns-cleaned/checksums/archives.sha256
```

清单覆盖全部 125 个文件。哈希匹配证明下载文件与本地上传源字节相同；解压软件的 CRC 检查是另一层检查，不代表标签语义检验。

五个 CSV 的官方下载链接及本版本对应的 MD5/SHA-256 见 [metadata_manifest.json](metadata_manifest.json)。下载后保存进 `metadata/`。校验值由本项目本地计算，官方来源见 [AudioSet 下载页](https://research.google.com/audioset/download.html)。

## 使用边界与致谢

本版本 eval 只有 19,044 条，不能称作完整官方评估集。比较模型时应固定实际样本身份和评估协议；不同下载版本的 mAP 不能仅凭数据集名称直接比较。按原始 PANNs 打包逻辑，缺失 CSV 行可能成为空名、零波形、零标签占位，HDF5 行数不等于有效 WAV 数，使用者应检查采样和评估是否跳过缺失行。

本仓库提供整理说明、校验值和下载信息，不提供重新标注真值或性能保证。

音频源于 YouTube/PANNs 与所列镜像；此整理不改变原音频的权利归属。PANNs/HTS-AT 代码许可证不自动覆盖音频。本项目不为第三方音频另行声明 MIT 等统一许可证。

- [Google AudioSet](https://research.google.com/audioset/)：原始数据集、标签和参考特征。
- [PANNs / audioset_tagging_cnn](https://github.com/qiuqiangkong/audioset_tagging_cnn)：下载基底、音频格式、目录与打包方法。
- [HTS-AT 官方仓库](https://github.com/RetroCirce/HTS-Audio-Transformer)：预训练分类模型和后续训练工作流参考。
- [Muno459/AudioSet](https://huggingface.co/datasets/Muno459/AudioSet)：本项目实际使用的 HF 补全镜像。

