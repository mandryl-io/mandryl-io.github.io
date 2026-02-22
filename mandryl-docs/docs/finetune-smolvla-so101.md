# Finetuning SmolVLA for the SO-101 Arm

This guide walks through finetuning a [SmolVLA](https://huggingface.co/lerobot/smolvla_base) imitation learning policy using the [LeRobot](https://github.com/huggingface/lerobot) library on Google Colab. A full working notebook is available at [`notebooks/mandryl_smolvla.ipynb`](https://github.com/mandryl-io/mandryl-docs/blob/main/notebooks/mandryl_smolvla.ipynb).

## Prerequisites

- A Google Colab instance with GPU runtime (NVIDIA A100 recommended)
- A Hugging Face account and access token
- A [Weights & Biases](https://wandb.ai/) account (optional, for training visualisation)
- A LeRobot v3.0 dataset uploaded to the Hugging Face Hub

## Expected Training Time

Training SmolVLA for 20,000 steps typically takes **about 3.5 hours on an NVIDIA A100**. On less powerful GPUs or CPU, training will take significantly longer.

## Overview

The training process has six steps:

1. **Install Conda** — bootstrap a Conda environment inside Colab
2. **Install LeRobot** — clone the repo, install FFmpeg, and install the package
3. **Log in** — authenticate with Weights & Biases and Hugging Face Hub
4. **Install SmolVLA dependencies** — install the extra dependencies for SmolVLA
5. **Configure and train** — set your training arguments and run the training script
6. **Retrieve your model** — find your finetuned model on the Hub

## Step 1: Install Conda

Install `condacolab` to set up a full Conda environment in Google Colab:

```python
!pip install -q condacolab
import condacolab
condacolab.install()
```

The kernel will restart automatically after installation.

## Step 2: Install LeRobot

Clone the LeRobot repository, install FFmpeg, and install the package in editable mode:

```bash
!git clone https://github.com/huggingface/lerobot.git
!conda install ffmpeg=7.1.1 -c conda-forge
!cd lerobot && pip install -e .
```

## Step 3: Log in to Weights & Biases and Hugging Face Hub

### Weights & Biases

```bash
!wandb login
```

Paste your API key when prompted. You can generate one at [wandb.ai/authorize](https://wandb.ai/authorize).

### Hugging Face Hub

```bash
!huggingface-cli login
```

Paste your Hugging Face access token when prompted. You can generate one at [huggingface.co/settings/tokens](https://huggingface.co/settings/tokens).

> ⚠️ **Don't skip this step.** If you're not logged in to the Hub, the training will fail when it tries to upload the final model. Checkpoints are saved locally as a safety net, but it's better to be safe than sorry.

## Step 4: Install SmolVLA Dependencies

```bash
!cd lerobot && pip install -e ".[smolvla]"
```

## Step 5: Configure and Train

Run the training script with the following command, adjusting the arguments to match your setup:

```bash
!cd /content/lerobot && lerobot-train \
  --policy.path=lerobot/smolvla_base \
  --policy.repo_id=${HF_USER}/my_smolvla \
  --dataset.repo_id=${HF_USER}/mydataset \
  --batch_size=64 \
  --steps=20000 \
  --save_checkpoint=true \
  --save_freq=2000 \
  --output_dir=outputs/train/my_smolvla \
  --job_name=my_smolvla_training \
  --policy.device=cuda \
  --wandb.enable=true \
  --rename_map='{"observation.images.up": "observation.images.camera1", "observation.images.side": "observation.images.camera2"}'
```

### Key Arguments

| Argument | Description |
|---|---|
| `--policy.path` | Base policy to finetune from (e.g., `lerobot/smolvla_base`) |
| `--policy.repo_id` | Hub repo ID where the finetuned model will be uploaded |
| `--dataset.repo_id` | Hub repo ID of your training dataset |
| `--batch_size` | Number of samples per gradient update. Reduce if you run out of GPU memory |
| `--steps` | Total number of training steps |
| `--save_checkpoint` | Whether to save intermediate checkpoints |
| `--save_freq` | Save a checkpoint every N steps |
| `--output_dir` | Local directory for logs and checkpoints |
| `--job_name` | Name for this training run (used in wandb logging) |
| `--policy.device` | `cuda` for NVIDIA GPUs, `mps` for Apple Silicon, `cpu` otherwise |
| `--wandb.enable` | Set to `true` to log training metrics to Weights & Biases |
| `--rename_map` | JSON mapping of dataset observation keys to policy-expected keys (see below) |

### Renaming Observation Keys with `--rename_map`

If your dataset uses different observation key names than what the policy expects, use `--rename_map` to map them at training time. The argument takes a JSON dictionary where keys are the names **in your dataset** and values are the names **the policy expects**.

For example, if your dataset has `observation.images.up` and `observation.images.side` but the policy expects `observation.images.camera1` and `observation.images.camera2`:

```
--rename_map='{"observation.images.up": "observation.images.camera1", "observation.images.side": "observation.images.camera2"}'
```

To find the right mapping, check your dataset's features on its Hub page and compare them against the policy's expected key names.

> ⚠️ If your dataset keys already match the policy's expected names, you can omit `--rename_map` entirely.

## Step 6: Retrieve Your Model

Once training finishes, the finetuned model will be uploaded to your Hugging Face Hub repo at the `--policy.repo_id` you specified. Model checkpoints, logs, and training plots are also saved to the local `--output_dir`.