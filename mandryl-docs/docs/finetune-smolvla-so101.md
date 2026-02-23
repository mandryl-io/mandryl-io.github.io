# Finetuning SmolVLA for the SO-101 Arm

This guide walks through finetuning a [SmolVLA](https://huggingface.co/lerobot/smolvla_base) imitation learning policy using the [LeRobot](https://github.com/huggingface/lerobot) library on Google Colab. A full working notebook is available at [`notebooks/mandryl_smolvla.ipynb`](https://colab.research.google.com/drive/1r5sKIy0Z9cIV34YY0WdwEkp_aXGB2Gu9#scrollTo=NQUk3Y0WwYZ4).

## Why did we make this?

We want to make finetuning easy. The effort should be placed on making sure you collect the most useful, consistent data possible, not trying to get the train to run. We created this because we still experienced some workflow difficulties when running our own experiments and thought this would be a small, neat way to help people overcome early setup hassles so that you can get to what matters with your time and ambitions — finetuning a VLA model for the SO-101 arm!

## Prerequisites

- A Hugging Face account and access token (for uploading the finetuned model)
- Google Colab (or any notebook environment you prefer, but we recommend using Colab for this guide)
- A [Weights & Biases](https://wandb.ai/) account (optional, for training visualisation)
- A LeRobot v3.0 dataset uploaded to the Hugging Face Hub

## General Description

This notebook will set up a Conda environment in Google Colab, install the LeRobot library along with SmolVLA dependencies, log you in to Weights & Biases and the Hugging Face Hub, and then run the `lerobot-train` command to finetune SmolVLA on your dataset. You will need to provide your dataset repo ID (`--dataset.repo_id`) and the Hub repo ID where the finetuned model will be uploaded (`--policy.repo_id`). Ideally these should be of the format `<your-username>/<name>`.

## Expected Training Time

Training SmolVLA for 20,000 steps typically takes **about 3.5 hours on an NVIDIA A100**. On less powerful GPUs or CPU, training will take significantly longer.