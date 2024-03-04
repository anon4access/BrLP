<h1 align="center">
  <br>
  Brain Latent Progression
  <br>
</h1>

<h4 align="center">Enhancing Spatiotemporal Disease Progression Models via Latent Diffusion and Prior Knowledge</h4>

<p align="center">
  <a href="#installation">Installation</a> •
  <a href="#training">Training</a> •
  <a href="#citing">Cite</a>
</p>

https://github.com/anon4access/BrLP/assets/121947111/303e0fc7-d617-41af-b60e-e218fd64d608

## Table of Contents
- [Installation](#installation)
- [Data preparation](./REPR-DATA.md)
- [Training](#training)
- [Pretrained models](#pretrained-models)
- [Acknowledgements](#acknowledgements)
- [Citing our work](#citing)

## Installation

Download the repository, `cd` into the project folder and install the `brlp` package:

```bash
pip install -e .
```
We recommend using a separate environment (see [Anaconda](https://www.anaconda.com/)). The code has been tested using python 3.9, however we expect it to work also with newer versions.

## Data preparation

Check out our document on [*Data preparation and study reproducibility*](./REPR-DATA.md). This file will guide you in organizing your data and creating the required CSV files to run the training pipelines.


## Training
![](assets/pipeline.png)


Training BrLP has 3 main phases that will be discribed in the subsequent sections. Every training (except for the auxiliary model) can be monitored using `tensorboard` as follows:

```bash
tensorboard --logdir runs
```



### Train the autoencoder

Follow the commands below to train the autoencoder.

```bash
# Create an output and a cache directory
mkdir ae_output ae_cache

# Run the training script
python scripts/training/train_autoencoder.py \
  --dataset_csv /path/to/A.csv \
  --cache_dir   ./ae_cache \
  --output_dir  ./ae_output
```

Then extract the latents from your MRI data:

```bash
python scripts/prepare/extract_latents.py \
  --dataset_csv /path/to/A.csv \
  --aekl_ckpt   ae_output/autoencoder-ep-XXX.pth
```

Replace `XXX` to select the autoencoder checkpoints of your choice.

### Train the UNet

Follow the commands below to train the diffusion UNet. Replace `XXX` to select the autoencoder checkpoints of your choice.


```bash
# Create an output and a cache directory:
mkdir unet_output unet_cache

# Run the training script
python scripts/training/train_diffusion_unet.py \
  --dataset_csv /path/to/A.csv \
  --cache_dir   unet_cache \
  --output_dir  unet_output \
  --aekl_ckpt   ae_output/autoencoder-ep-XXX.pth
```

### Train the ControlNet

Follow the commands below to train the ControlNet. Replace `XXX` to select the autoencoder and UNet checkpoints of your choice.

```bash
# Create an output and a cache directory:
mkdir cnet_output cnet_cache

# Run the training script
python scripts/training/train_controlnet.py \
  --dataset_csv /path/to/B.csv \
  --cache_dir   unet_cache \
  --output_dir  unet_output \
  --aekl_ckpt   ae_output/autoencoder-ep-XXX.pth \
  --diff_ckpt   unet_output/unet-ep-XXX.pth
```

### Auxiliary models

Follow the commands below to train the DCM auxiliary model.

```bash
# Create an output directory
mkdir aux_output

# Run the training script
python scripts/training/train_aux.py \
  --dataset_csv /path/to/A.csv \
  --output_path aux_output
```

We emphasize that any disease progression model capable of predicting volumetric changes over time is also viable as an auxiliary model for BrLP.

## Inference

Our package comes with a `brlp` command to use BrLP for inference. Check:
```
brlp --help
```

The `--input` parameter requires a CSV file where you list all available data for your subjects. For an example, check to `examples/input.example.csv`. If you haven't segmented your input scans, `brlp` can perform this task for you using [SynthSeg](https://surfer.nmr.mgh.harvard.edu/fswiki/SynthSeg), but it requires that FreeSurfer >= 7.4 be installed. The `--confs` parameter specifies the paths to the models and other inference parameters, such as LAS $m$. For an example, check `examples/confs.example.yaml`. 

Running the program looks like this:

![inference-preview](assets/inference.gif)


## Pretrained models
The pretrained models will be published upon acceptance.

| Model                | Weights URL  |
| -------------------- | ------------ |
| Autoencoder          | - |
| Diffusion Model UNet | - |
| ControlNet           | - |

## Acknowledgements

We thank the maintainers of open-source libraries for their contributions to accelerating the research process, with a special mention of [MONAI](https://monai.io/) and its [GenerativeModels](https://github.com/Project-MONAI/GenerativeModels/tree/main) extension.

## Citing

```
Currently under MICCAI 2024 review process.
```
