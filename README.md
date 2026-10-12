# AudioSet-1961094: a PANNs-based recovered and cleaned snapshot

English | [简体中文](README.zh-CN.md)

This project documents a personal AudioSet snapshot derived from the [downloaded dataset provided by PANNs](https://github.com/qiuqiangkong/audioset_tagging_cnn#1-download-dataset), followed by missing-audio recovery, audio-quality checks, replacement of defective recordings, and cleanup. It contains **1,961,094 WAV files**, retains the original AudioSet v1 527-class weak labels, and follows the PANNs directory layout for PANNs/HTS-AT workflows.

“Improvement” here means recovery and data curation. Missing recordings and weak-label uncertainty remain. This is an independent snapshot, not an official Google, PANNs, or HTS-AT release, and no training-accuracy improvement is claimed.

## Download

- Baidu Netdisk: [AudioSet_1961094](https://pan.baidu.com/s/1YRtWpQzuWsWMykfSVgnORQ?pwd=xmsq)
- Access code: `xmsq`
- **125 archive/volume files**, totaling **1,241,013,140,141 bytes**, approximately 1.241 TB in decimal units.
- The share contains WAV archives; HDF5 files, training indexes, model weights, and official metadata CSV files are not included.
- See [archive_manifest.json](archive_manifest.json) for filenames, exact sizes, MD5, and SHA-256.

Local MD5/SHA-256 calculation for all 125 upload-source files completed on 2026-10-12. These are **content hashes of the local upload source**. Cloud filename/size checks do not establish content identity; full cloud-side content verification is not claimed. Verify your downloads against the hashes below.

## Final size

The denominators are the official v1 CSVs used for this snapshot.

| Split | Official CSV segments | Available WAVs | Missing |
| --- | ---: | ---: | ---: |
| Balanced train | 22,160 | 20,715 | 1,445 |
| Evaluation | 20,371 | 19,044 | 1,327 |
| Unbalanced train | 2,041,789 | 1,921,335 | 120,454 |
| **Total** | **2,084,320** | **1,961,094** | **123,226** |

Overall coverage is approximately **94.09%**. Website summary counts can differ from the v1 CSVs; this table uses the actual CSVs associated with this snapshot.

The final curation report records the following mutually exclusive provenance categories, counted by final identity.

| Recorded provenance category | Retained files |
| --- | ---: |
| Local PANNs-base provenance category | 1,951,260 |
| YouTube recovery provenance category | 8,424 |
| Muno459/AudioSet HF-mirror provenance category | 1,410 |
| **Total** | **1,961,094** |

The retained YouTube and HF categories total **9,834 files**.

## Changes to the PANNs-derived dataset

### 1. Recover missing audio using original segment identities

Missing entries were identified from official YouTube IDs, segment timestamps, and labels. An adapted PANNs download workflow retried YouTube downloads. Matching missing entries were also extracted individually from the [Muno459/AudioSet](https://huggingface.co/datasets/Muno459/AudioSet) mirror.

Recovered/replacement audio follows the PANNs download settings: **32 kHz, mono, 16-bit PCM WAV**, with the corresponding segment timing. Official IDs, labels, and split assignments were preserved. Mirror conversion and later source downloads can differ from historical YouTube encodings; recovery does not imply sample-for-sample waveform identity with the original publication.

### 2. Audit audio integrity and distinguish failure types

Checks covered format, decoding, duration, zero-frame files, all-zero/near-silent signals, silent-frame fraction, and clipping for both the PANNs base and recovered audio. All-zero candidates were decoded again and checked against the official `Silence` label.

Cleanup targeted format/decoding failures, empty recordings, recordings shorter than 0.96 seconds when their nominal duration was at least 9 seconds, and all-zero or peak ≤ −70 dBFS recordings **without a Silence label**. Unresolved failures were excluded from the final WAV tree and treated as missing. Mild duration deficits and clipping warnings were not indiscriminately deleted.

Recordings carrying the official Silence label were retained when they were genuine silent examples. Workflows requiring exclusively nonzero inputs should handle them in a separate training view; the released WAV snapshot must not be described as entirely nonzero.

### 3. Screen label relevance and similarity to official features

Official pretrained HTS-AT models were used for label-relevance screening. Extracted VGGish embeddings were compared with Google's released AudioSet features to assist review of newly downloaded and anomalous recordings.

These models and features were used to identify anomalous candidates for review, not to certify AudioSet labels. Predictions were not written into the labels, and normal audio was not deleted solely because of a model score.

### 4. Review disputed candidates and consolidate valid replacements

Anomalous candidates were screened automatically and then listened to when needed. Confirmed replacements were consolidated at their original relative path, replacing the corresponding defective WAV rather than appending a duplicate sample. Label-mismatched or uncertain candidates were excluded.

### 5. Restore the standard layout and preserve official labels

Final filenames use `Y<YouTube_ID>.wav`. Recordings remain in their original balanced, evaluation, or unbalanced split; unbalanced data retain 41 directories, `part00`–`part40`. Accepted recovered/replacement files are consolidated directly at their corresponding location.

All five official CSVs remain byte-for-byte unchanged, preserving the original 527-class indexes, MIDs, and weak labels. No relabeling or label optimization was applied.

## Archive layout and extraction destinations

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

Balanced and evaluation data each use one ordinary ZIP. Each unbalanced part uses **split ZIP**: download the matching `.z01`, `.z02`, and `.zip` files and keep all three in the same directory. Each `.z01`/`.z02` is 10,485,760,000 bytes. Do not rename them to `.7z.001` or extract `.z01` as an independent archive.

Use extraction software supporting `.z01/.z02/.zip` volumes and open the group's `.zip` entry. Balanced/evaluation archives already contain their respective directory. Each unbalanced archive already contains `unbalanced_train_segments_partXX/`. Extract balanced/evaluation archives into `dataset_root/audios/` and unbalanced parts into `dataset_root/audios/unbalanced_train_segments/` to avoid an extra nested directory. Being able to list an archive with ordinary `unzip` does not imply support for extracting a multi-volume ZIP.

After extraction and separate official metadata download, the PANNs layout is:

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

## Download verification and official metadata

Run inside the downloaded `AudioSet_1961094/` directory, substituting the actual repository path:

```bash
sha256sum -c /path/to/audioset-panns-cleaned/checksums/archives.sha256
```

The list covers all 125 files. Matching hashes establish byte identity with the local upload source. Extraction-time CRC checks provide another integrity check, not label-semantic verification.

[metadata_manifest.json](metadata_manifest.json) contains official download URLs and the MD5/SHA-256 of the five CSVs associated with this snapshot. Save the downloaded files under `metadata/`. Checksums were calculated locally by this project; see the [official AudioSet download page](https://research.google.com/audioset/download.html) for their source.

## Scope and attribution

The evaluation split has 19,044 recordings and is incomplete. Model comparisons should fix actual sample identities and evaluation protocols; mAP values from different downloaded snapshots are not directly comparable solely because both are called AudioSet. Original PANNs packing can leave missing CSV entries as blank-name, zero-waveform, zero-target placeholder rows. HDF5 row counts are not valid WAV counts; check that sampling and evaluation exclude missing entries.

This repository provides curation documentation, checksums, and download information, not new ground-truth annotations or a performance guarantee.

Audio originated from YouTube/PANNs and the listed mirror. Curation does not change ownership of the underlying audio. PANNs/HTS-AT code licenses do not automatically cover audio, and this project does not assign a blanket MIT or other license to third-party audio.

- [Google AudioSet](https://research.google.com/audioset/): original dataset, labels, and reference features.
- [PANNs / audioset_tagging_cnn](https://github.com/qiuqiangkong/audioset_tagging_cnn): download base, audio format, layout, and packing method.
- [Official HTS-AT repository](https://github.com/RetroCirce/HTS-Audio-Transformer): pretrained classifiers and subsequent training workflow reference.
- [Muno459/AudioSet](https://huggingface.co/datasets/Muno459/AudioSet): HF mirror actually used for recovery.

