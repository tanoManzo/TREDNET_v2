# TREDNet Summer 2026

Two-phase deep learning pipeline for transcription factor binding prediction from genomic sequences.

This repository contains an updated TREDNet workflow using TensorFlow 2.20 and Keras 3, with support for Python 3.11+ (including Python 3.13).

## Colab Notebook

Open the shared notebook directly:

- https://colab.research.google.com/drive/1Ud_DNLwrt5wdAuNdRrnxRKbUu_PQfB2a?usp=sharing

Recommended runtime:

- Runtime -> Change runtime type -> Hardware accelerator: GPU

## Repository Structure

- `TREDNet_v2.py`: end-to-end pipeline (dataset creation + phase-two training)
- `input_training_data/`: BED files for positive/control regions
- `fasta/`: reference genome FASTA (for local runs)
- `model_phase_I/`: phase-one model architecture/weights
- `model_phase_II/`: legacy phase-two model file input
- `models_output/`: generated outputs and trained artifacts
- `pyproject.toml`: project metadata and dependencies (managed by `uv`)

## Requirements

- Python 3.11+
- `uv` package manager
- For GPU training: NVIDIA GPU with compatible drivers

## Local Setup and Run

From repository root:

```bash
uv sync
uv run TREDNet_v2.py
```

What this does:

1. Creates the phase-two dataset from BED + FASTA using the phase-one model embeddings.
2. Trains the phase-two model.
3. Writes metrics and model artifacts under `models_output/<EID>/`.

## Expected Outputs

Under `models_output/<EID>/` (default `<EID>` in code is `K562_Enhancer_DHS_x2`):

- `auc.txt`
- `prc.txt`
- `roc_values.txt`
- `fpr_threshold_scores.txt`
- `prc_values.txt`
- `<EID>_phase_two_dataset.hdf5`
- `<EID>_phase_two_weights.weights.h5`
- `phase_two_model.keras`

## Run in Google Colab

Use this single Colab cell when assets are hosted in Google Drive:

```bash
%%bash
set -euo pipefail

cd /content
rm -rf TREDNET_v2 TREDNet_assets
git clone https://github.com/tanoManzo/TREDNET_v2.git
cd TREDNET_v2

python -m pip -q install uv gdown

# Download all required data/models from shared Drive folder
gdown --folder "https://drive.google.com/drive/folders/16KJapskdOPgrrReCOWJW17sU-M29_tl4?usp=sharing" -O /content/TREDNet_assets

# Link downloaded assets into repository layout
rm -rf fasta model_phase_I model_phase_II
ln -s /content/TREDNet_assets/fasta fasta
ln -s /content/TREDNet_assets/model_phase_I model_phase_I
ln -s /content/TREDNet_assets/model_phase_II model_phase_II

# Fresh environment
rm -rf .venv .python-version

# Ensure uv uses public PyPI (fixes custom index DNS failures)
export UV_INDEX_URL="https://pypi.org/simple"
unset UV_EXTRA_INDEX_URL PIP_INDEX_URL PIP_EXTRA_INDEX_URL 2>/dev/null || true

# Python 3.13 path (change to 3.11 if desired)
uv python pin 3.13
uv venv --python 3.13 --clear
uv sync --python 3.13

uv run --python 3.13 python -V
uv run --python 3.13 TREDNet_v2.py
```

## Troubleshooting

### `Failed to download ml-dtypes` with Artifactory/NCBI URL in Colab

Cause: Colab session is using a custom package index that is unreachable from your runtime.

Fix: force `uv` to use official PyPI before `uv sync`:

```bash
export UV_INDEX_URL="https://pypi.org/simple"
unset UV_EXTRA_INDEX_URL PIP_INDEX_URL PIP_EXTRA_INDEX_URL 2>/dev/null || true
```

### GPU not detected

In Colab, enable GPU runtime first. Then verify:

```python
import tensorflow as tf
print(tf.__version__)
print(tf.config.list_physical_devices("GPU"))
```

### Missing chromosome warnings

If you see warnings such as skipped positive/negative regions due to missing chromosomes in FASTA, the pipeline will continue. This indicates BED chromosome names/regions do not fully match available FASTA entries.

## Notes

- Current implementation runs with fixed constants from `TREDNet_v2.py` (for example, `EID`, epochs, batch size).
- To run a different experiment ID, update `EID` in `TREDNet_v2.py` and ensure matching BED files exist in `input_training_data/`.
