# 目录

- [目录](#contents)
    - [CTSDG描述](#ctsdg-description)
    - [Model框架结构](#model-architecture)
    - [数据集](#dataset)
    - [环境需求](#environment-requirements)
    - [快速开始](#quick-start)
    - [脚本描述](#script-description)
        - [脚本和示例代码](#script-and-sample-code)
        - [脚本参数](#script-parameters)
        - [训练流程](#training-process)
        - [测试流程](#evaluation-process)
        - [输出MINDIR](#export-mindir)
    - [模型描述](#model-description)
        - [在GPU上的训练表现](#training-performance-gpu)
    - [随机情况描述](#description-of-random-situation)
    - [ModelZoo主页](#modelzoo-homepage)

## [CTSDG description](#contents)

近年来，通过引入结构先验知识，深度生成方法在图像修复领域取得了长足进展。然而，由于在结构重建过程中缺乏与图像纹理的适当交互，现有的解决方案无法处理具有较大腐蚀的情况，并且通常会导致结果失真。这是一种新颖的用于图像修复的双流网络，它以耦合方式对结构约束的纹理合成和纹理引导的结构重建进行建模，以便它们更好地相互利用，生成更合理的图像。此外，为了增强全局一致性，设计了双向选通特征融合（Bi-GFF）模块来交换和组合结构和纹理信息，并开发了上下文特征聚合（CFA）模块来通过区域关联学习和多尺度特征聚合来细化生成的内容。

> [沦为](https://arxiv.org/pdf/2108.09760.pdf):  Image Inpainting via Conditional Texture and Structure Dual Generation
> Xiefan Guo, Hongyu Yang, Di Huang, 2021.
> [补充资料](https://openaccess.thecvf.com/content/ICCV2021/supplemental/Guo_Image_Inpainting_via_ICCV_2021_supplemental.pdf)

## [模型框架结构](#contents)

## [数据集](#contents)

使用的数据集: [CELEBA](http://mmlab.ie.cuhk.edu.hk/projects/CelebA.html), [NVIDIA Irregular Mask Dataset](https://nv-adlr.github.io/publication/partialconv-inpainting)

- 你需要从 **CELEBA** 下载 (板块 *Downloads -> Align&Cropped Images*):
    - `img_align_celeba.zip`
    - `list_eval_partitions.txt`
- 你需要从 **NVIDIA Irregular Mask Dataset** 下载:
    - `irregular_mask.zip`
    - `test_mask.zip`
- 目录结构如下:

  ```text
    .
    ├── img_align_celeba            # images folder
    ├── irregular_mask              # masks for training
    │   └── disocclusion_img_mask
    ├── mask                        # masks for testing
    │   └── testing_mask_dataset
    └── list_eval_partition.txt     # train/val/test splits
  ```

## [环境需求](#contents)

- 硬件（GPU）
    - 使用GPU处理器准备硬件环境。
- 框架
    - [MindSpore1.8.1](https://gitee.com/mindspore/mindspore)
- 有关更多信息，请查看以下资源：
    - [MindSpore Tutorials](https://www.mindspore.cn/tutorials/en/master/index.html)
    - [MindSpore Python API](https://www.mindspore.cn/docs/en/master/index.html)
- 下载数据集

## [快速开始](#contents)

### [预训练VGG16](#contents)

您需要将火炬VGG16模型转换为训练CTSDG模型的感知损失。

1. [下载预训练VGG16](https://download.pytorch.org/models/vgg16-397923af.pth)
2. 将torch模型转换为mindspore:

```shell
python converter.py --torch_pretrained_vgg=/path/to/torch_pretrained_vgg
```

转换后的mindspore检查点将与名为“vgg16_feat_extr_ms.ckpt”的torch模型保存在同一目录中。

准备好数据集并转换VGG16后，您可以开始训练和评估，如下所示：

### [在GPU上运行](#contents)

#### 训练

```shell
# 单设备训练
bash scripts/run_standalone_train_gpu.sh [DEVICE_ID] [CFG_PATH] [SAVE_PATH] [VGG_PRETRAIN] [IMAGES_PATH] [MASKS_PATH] [ANNO_PATH]

# 分布式训练
bash scripts/run_distribute_train_gpu.sh [DEVICE_NUM] [CFG_PATH] [SAVE_PATH] [VGG_PRETRAIN] [IMAGES_PATH] [MASKS_PATH] [ANNO_PATH]
```

示例:

```shell
# 单设备训练
# DEVICE_ID - device number for training
# CFG_PATH - path to config
# SAVE_PATH - path to save logs and checkpoints
# VGG_PRETRAIN - path to pretrained VGG16
# IMAGES_PATH - path to CELEBA dataset
# MASKS_PATH - path to masks for training
# ANNO_PATH - path to file with train/val/test splits
bash scripts/run_standalone_train_gpu.sh 0 ./default_config.yaml /path/to/output /path/to/vgg16_feat_extr.ckpt /path/to/img_align_celeba /path/to/training_mask /path/to/list_eval_partitions.txt

# 分布式训练 (8p)
# DEVICE_NUM - number of devices for training
# other parameters as for standalone train
bash scripts/run_distribute_train_gpu.sh 8 ./default_config.yaml /path/to/output /path/to/vgg16_feat_extr.ckpt /path/to/img_align_celeba /path/to/training_mask /path/to/list_eval_partitions.txt
```

#### 评测

```shell
# 评测
bash scripts/run_eval_gpu.sh [DEVICE_ID] [CFG_PATH] [CKPT_PATH] [IMAGES_PATH] [MASKS_PATH] [ANNO_PATH]
```

Example:

```shell
# evaluate
# DEVICE_ID - device number for evaluating
# CFG_PATH - path to config
# CKPT_PATH - path to ckpt for evaluation
# IMAGES_PATH - path to img_align_celeba dataset
# MASKS_PATH - path to masks for testing
# ANNO_PATH - path to file with train/val/test splits
bash scripts/run_eval_gpu.sh 0 ./default_config.yaml /path/to/ckpt /path/to/img_align_celeba /path/to/testing_mask /path/to/list_eval_partitions.txt  
```

## [脚本描述](#contents)

### [脚本和示例代码](#contents)

```text
.
└── CTSDG
    ├── model_utils
    │   ├── __init__.py                     # init file
    │   └── config.py                       # parse arguments
    ├── scripts
    │   ├── run_distribute_train_gpu.sh     # launch distributed training(8p) on GPU
    │   ├── run_eval_gpu.sh                 # launch evaluating on GPU
    │   ├── run_export_gpu.sh               # launch export mindspore model to mindir
    │   └── run_standalone_train_gpu.sh     # launch standalone traininng(1p) on GPU
    ├── src
    │   ├── discriminator
    │   │   ├── __init__.py                 # init file
    │   │   ├── discriminator.py            # discriminator
    │   │   └── spectral_conv.py            # conv2d with spectral normalization
    │   ├── generator
    │   │   ├── __init__.py                 # init file
    │   │   ├── bigff.py                    # bidirectional gated feature fusion
    │   │   ├── cfa.py                      # contextual feature aggregation
    │   │   ├── generator.py                # generator
    │   │   ├── pconv.py                    # partial convolution
    │   │   ├── projection.py               # feature to texture and texture to structure parts
    │   │   └── vgg16.py                    # VGG16 feature extractor
    │   ├── __init__.py                     # init file
    │   ├── callbacks.py                    # callbacks
    │   ├── dataset.py                      # celeba dataset
    │   ├── initializer.py                  # weight initializer
    │   ├── losses.py                       # model`s losses
    │   ├── trainer.py                      # trainer for ctsdg model
    │   └── utils.py                        # utils
    ├── __init__.py                         # init file
    ├── converter.py                        # convert VGG16 torch checkpoint to mindspore
    ├── default_config.yaml                 # config file
    ├── eval.py                             # evaluate mindspore model
    ├── export.py                           # export mindspore model to mindir format
    ├── README.md                           # readme file
    ├── requirements.txt                    # requirements
    └── train.py                            # train mindspore model
```

### [脚本参数](#contents)

训练参数可以在`default_config.yaml`中配置

```text
"gen_lr_train": 0.0002,                     # learning rate for generator training stage
"gen_lr_finetune": 0.00005,                 # learning rate for generator finetune stage
"dis_lr_multiplier": 0.1,                   # discriminator`s lr is generator`s lr multiply by this parameter
"batch_size": 6,                            # batch size
"train_iter": 350000,                       # number of training iterations
"finetune_iter": 150000                     # number of finetune iterations
"image_load_size": [256, 256]               # input image size
```

有关更多参数，请参阅“default_config.yaml”的内容

### [训练流程](#contents)

#### [在GPU上运行](#contents)

##### 单设备运行 (1p)

```shell
# DEVICE_ID - device number for training (0)
# CFG_PATH - path to config (./default_config.yaml)
# SAVE_PATH - path to save logs and checkpoints (/path/to/output)
# VGG_PRETRAIN - path to pretrained VGG16 (/path/to/vgg16_feat_extr.ckpt)
# IMAGES_PATH - path to CELEBA dataset (/path/to/img_align_celeba)
# MASKS_PATH - path to masks for training (/path/to/training_mask)
# ANNO_PATH - path to file with train/val/test splits (/path/to/list_eval_partitions.txt)
bash scripts/run_standalone_train_gpu.sh 0 ./default_config.yaml /path/to/output /path/to/vgg16_feat_extr.ckpt /path/to/img_align_celeba /path/to/training_mask /path/to/list_eval_partitions.txt
```

日志将会储存在路径`/path/to/output/log.txt`

结果:

```text
...
DATE TIME iter: 250, loss_g: 19.7810001373291, loss_d: 1.7710000276565552, step time: 570.67 ms
DATE TIME iter: 375, loss_g: 20.549999237060547, loss_d: 1.8650000095367432, step time: 572.09 ms
DATE TIME iter: 500, loss_g: 25.295000076293945, loss_d: 1.8630000352859497, step time: 572.23 ms
DATE TIME iter: 625, loss_g: 24.059999465942383, loss_d: 1.812999963760376, step time: 573.33 ms
DATE TIME iter: 750, loss_g: 26.343000411987305, loss_d: 1.8539999723434448, step time: 573.18 ms
DATE TIME iter: 875, loss_g: 21.774999618530273, loss_d: 1.8509999513626099, step time: 573.0 ms
DATE TIME iter: 1000, loss_g: 18.062999725341797, loss_d: 1.7960000038146973, step time: 572.41 ms
...
```

##### 分布式训练 (8p)

```shell
# DEVICE_NUM - number of devices for training (8)
# other parameters as for standalone train
bash scripts/run_distribute_train_gpu.sh 8 ./default_config.yaml /path/to/output /path/to/vgg16_feat_extr.ckpt /path/to/img_align_celeba /path/to/training_mask /path/to/list_eval_partitions.txt
```

日志将会储存在路径`/path/to/output/log.txt`

结果:

```text
...
DATE TIME iter: 250, loss_g: 26.28499984741211, loss_d: 1.680999994277954, step time: 757.67 ms
DATE TIME iter: 375, loss_g: 21.548999786376953, loss_d: 1.468000054359436, step time: 758.02 ms
DATE TIME iter: 500, loss_g: 17.89299964904785, loss_d: 1.2829999923706055, step time: 758.57 ms
DATE TIME iter: 625, loss_g: 18.750999450683594, loss_d: 1.2589999437332153, step time: 759.95 ms
DATE TIME iter: 750, loss_g: 21.542999267578125, loss_d: 1.1829999685287476, step time: 759.45 ms
DATE TIME iter: 875, loss_g: 27.972000122070312, loss_d: 1.1629999876022339, step time: 759.62 ms
DATE TIME iter: 1000, loss_g: 18.03499984741211, loss_d: 1.159000039100647, step time: 759.51 ms
...
```

### [测评结果](#contents)

#### GPU

```shell
bash scripts/run_eval_gpu.sh [DEVICE_ID] [CFG_PATH] [CKPT_PATH] [IMAGES_PATH] [MASKS_PATH] [ANNO_PATH]
```

示例:

```shell
# DEVICE_ID - device number for evaluating (0)
# CFG_PATH - path to config (./default_config.yaml)
# CKPT_PATH - path to ckpt for evaluation (/path/to/ckpt)
# IMAGES_PATH - path to img_align_celeba dataset (/path/to/img_align_celeba)
# MASKS_PATH - path to masks for testing (/path/to/testing/mask)
# ANNO_PATH - path to file with train/val/test splits (/path/to/list_eval_partitions.txt)
bash scripts/run_eval_gpu.sh 0 ./default_config.yaml /path/to/ckpt /path/to/img_align_celeba /path/to/testing_mask /path/to/list_eval_partitions.txt
```

日志将会储存在路径 `./logs/eval_log.txt`.

结果:

```text
PSNR:
0-20%: 38.04
20-40%: 29.39
40-60%: 24.21
SSIM:
0-20%: 0.979
20-40%: 0.922
40-60%: 0.83
```

### [Export MINDIR](#contents)

如果要推断Ascend 310上的网络，应将模型转换为MINDIR。

#### GPU

```shell
bash scripts/run_export_gpu.sh [DEVICE_ID] [CFG_PATH] [CKPT_PATH]
```

示例:

```shell
# DEVICE_ID - device number (0)
# CFG_PATH - path to config (./default_config.yaml)
# CKPT_PATH - path to ckpt for evaluation (/path/to/ckpt)
bash scripts/run_export_gpu.sh 0 ./default_config.yaml /path/to/ckpt
```

如果要推断Ascend 310上的网络，应将模型转换为MINDIR。`./logs/export_log.txt`, converted model will have the same name as ckpt except extension.

