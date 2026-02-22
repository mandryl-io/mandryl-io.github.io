# Porting LeRobot Datasets to v3.0

This guide walks through converting existing LeRobot datasets from v2.1 to v3.0 using the built-in conversion utilities in the LeRobot library. A full working notebook is available at [`notebooks/port-so101-table-cleanup-to-v3.ipynb`](https://colab.research.google.com/drive/1vrk1F9rJi5fAM2T5Kz4oY9FioFSgH1HO?usp=sharing).

## Why did we make this?

We created this because we still experienced some workflow difficulties when running our own experiments and thought this would be a small, neat way to help people overcome early setup hassles so that you can get to what matters with your time and ambitions - finetuning a VLA model for the SO-101 arm!

We kindly ask that for any datasets you port over, please ensure you attribute any of the original dataset sources :)

## Prerequisites

- A Hugging Face account and access token (for downloading/uploading datasets)
- Google Colab (or any notebook environment you prefer, but we recommend using colab for this guide)
- A LeRobot v2.1 dataset you wish to convert to v3.0

## General Description

This notebook will ask you for the repo ID of the v2.1 dataset you wish to convert to v3.0, and ALSO the repo ID of the v3.0 dataset you wish to upload to. Ideally the v3.0 repo ID should be of the format `<your-username>/<dataset-name>`.
