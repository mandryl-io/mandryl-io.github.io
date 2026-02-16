# Porting LeRobot Datasets to v3.0

This guide walks through converting existing LeRobot datasets from v2.1 to v3.0 using the built-in conversion utilities in the LeRobot library. A full working notebook is available at [`notebooks/port-so101-table-cleanup-to-v3.ipynb`](https://github.com/mandryl-io/mandryl-docs/blob/main/notebooks/port-so101-table-cleanup-to-v3.ipynb).

## Prerequisites

- Python 3.11+
- A Hugging Face account and access token (for downloading/uploading datasets)
- LeRobot v3 (pre-release build or a release that includes v3.0 support)

Install the LeRobot build with v3 support:

```bash
pip install "https://github.com/huggingface/lerobot/archive/33cad37054c2b594ceba57463e8f11ee374fa93c.zip"
```

## Overview

The conversion process has four steps:

1. **Download** the v2.1 dataset from Hugging Face Hub
2. **Convert** it to v3.0 format using `lerobot.datasets.v30.convert_dataset_v21_to_v30`
3. **Upload** the converted dataset back to the Hub (optional)
4. **Verify** the conversion by loading the new dataset

## Step 1: Login to Hugging Face

```python
import huggingface_hub
huggingface_hub.login()
```

## Step 2: Download the v2.1 Dataset

```python
from huggingface_hub import snapshot_download
from pathlib import Path

local_dataset_dir = Path("./datasets")
local_dataset_dir.mkdir(exist_ok=True)

repo_id = "youliangtan/so101-table-cleanup"  # replace with your dataset
local_path = snapshot_download(
    repo_id=repo_id,
    repo_type="dataset",
    revision="v2.1",
    local_dir=local_dataset_dir / repo_id,
)
```

Set `repo_id` to the Hugging Face repo ID of the dataset you want to convert.

## Step 3: Run the Conversion

```python
from lerobot.datasets.v30.convert_dataset_v21_to_v30 import convert_dataset

convert_dataset(
    repo_id=repo_id,
    branch=None,
    data_file_size_in_mb=None,   # defaults to 100 MB
    video_file_size_in_mb=None,  # defaults to 500 MB
    root=str(local_dataset_dir),
    push_to_hub=False,
    force_conversion=False,
)
```

This converts data files to parquet format and re-encodes any video files for v3.0 compatibility. The conversion happens in-place within the local dataset directory.

## Step 4: Upload to Hugging Face Hub (Optional)

```bash
huggingface-cli repo create <your-username>/<dataset-name> --type dataset
huggingface-cli upload <your-username>/<dataset-name> ./datasets/<original-repo-id> --repo-type dataset --revision v3.0
```

Replace the paths and names to match your setup.

## Step 5: Verify the Conversion

```python
from lerobot.datasets.lerobot_dataset import LeRobotDataset

dataset = LeRobotDataset(
    repo_id="<your-username>/<dataset-name>",
    force_cache_sync=True,
)

print(dataset.meta)
```

A successful load confirms the dataset is valid v3.0. The metadata will show episode counts, frame counts, and available features.
