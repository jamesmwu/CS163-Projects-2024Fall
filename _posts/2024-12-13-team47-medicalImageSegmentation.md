---
layout: post
comments: true
title: Medical Image Segmentation
author: James Wu, Shiyu Ye, Yun Zhang, Nelson Lu
date: 2024-12-11
---

> Medical image segmentation leverages deep learning to partition medical images
> into meaningful regions like organs, tissues, and abnormalities. This report
> explores key segmentation models such as **U-Net**, **U-Net++**, and
> **nnU-Net**, detailing their architectures, challenges, comparative
> performance, and practical applications in clinical and research settings.

<!--more-->

{: class="table-of-content"}

- TOC  
  {:toc}

## Introduction

Our group explored medical image segmentation, an increasingly critical tool in
healthcare that has been used for decades to help doctors and researchers
analyze medical scans like MRIs, CT scans, and X-rays. In the past, segmentation
was done either manually — requiring painstaking effort from medical
professionals — or using basic computer algorithms that often lacked precision.
These traditional methods struggled with issues like the variability between
different patients' scans, the complexity of human anatomy, and the quality of
the images, which could be blurry or noisy.

However, the rise of deep learning in recent years has transformed the field of
medical image segmentation. Deep learning models can automatically analyze
thousands of images and learn to identify patterns, making the segmentation
process faster and more accurate. This has led to significant improvements in
the ability to detect tumors, delineate organs, and monitor diseases. Deep
learning can adapt to different types of medical images and handle the natural
variability found in human tissues more effectively than older methods.

While challenges like image noise and medical uncertainty still exist, deep
learning has pushed the boundaries of what medical image segmentation can
achieve. With continued research and better data, this technology holds great
promise for improving patient care and advancing medical knowledge.

### Definition

Medical image segmentation is a technique used to partition medical images, such
as MRI or CT scans, into regions of interest. These regions can include organs,
tissues, tumors, and other abnormalities. However, regardless of what region,
accurate segmentation is essential for a wide range of applications in clinical
practice and medical research.

Medical image segmentation differs from standard image classification because it
requires high precision and accuracy due to the nuanced nature of medical
diagnosis. Misclassifications can lead to incorrect treatment plans or flawed
research conclusions.

Given a 2D or 3D medical image, the objective is to generate a segmentation mask
with the same dimensions. Each pixel or voxel in the mask is assigned a label
corresponding to predefined categories, such as organs, lesions, or background.

### Applications

This kind of segmentation has a profound impact on both clinical and research
settings. In clinical practice, it enhances the precision of diagnostic
procedures, treatment planning, and disease monitoring. For instance, accurate
tumor segmentation can significantly improve the outcomes of radiotherapy by
targeting cancerous tissues while sparing healthy ones. In research,
segmentation facilitates the creation of high-quality datasets for training
machine learning models, advancing our understanding of various diseases, and
developing new therapeutic strategies.

- **Clinical Applications**:

  - **Tumor Detection**: Identifying the presence and extent of tumors.
  - **Treatment Planning**: Assisting in radiotherapy and surgical procedures.
  - **Disease Monitoring**: Tracking changes in pathology over time.

- **Research Applications**:
  - **Machine Learning**: Preparing datasets for training and evaluating AI
    models.
  - **Disease Analysis**: Understanding the progression of diseases such as
    cancer or neurodegenerative disorders. **Monitoring Disease Progression**:
    In oncology, segmentation helps track changes in tumor size or shape across
    multiple scans. For instance, sequential MRI scans can be segmented to
    visualize and quantify how a tumor responds to chemotherapy.

Other examples include:

- **Surgical Planning**: Segmenting organs and tissues to assist surgeons in
  planning minimally invasive procedures.
- **Radiotherapy**: Accurately delineating tumor boundaries to target radiation
  precisely while sparing healthy tissue.

## Deep Learning-Based Solutions

### U-Net

**U-Net** is a widely used convolutional neural network (CNN) architecture
designed for image segmentation tasks. It has become a cornerstone in medical
image segmentation due to its effectiveness, especially with limited datasets.

#### Key Features

- **Data Augmentation**: Enhances performance on small datasets.
- **Detailed Masks**: Produces segmentation masks with high accuracy.
- **Flexibility**: Applicable to both 2D and 3D image segmentation tasks.
- **Simplicity and Robustness**: Easy to implement and train.

#### Architecture

U-Net follows an **encoder-decoder** structure that captures both spatial
context and fine-grained details:

1. **Encoder (Contracting Path)**:

   - The encoder uses convolutional and pooling layers to systematically extract
     features from the input image while progressively reducing the spatial
     dimensions. This process allows the network to capture essential details at
     different levels of abstraction. As the spatial dimensions decrease, the
     feature depth increases, enabling the model to learn and represent more
     complex, hierarchical information about the image.

1. **Decoder (Expanding Path)**:

   - The decoder restores the spatial dimensions of the feature maps by using
     upsampling, specifically through transposed convolutions. During this
     process, the decoder integrates features from the encoder using skip
     connections, which help retain important details lost during the
     downsampling phase. This combination of upsampling and skip connections
     refines the segmentation mask, ensuring that fine-grained details and
     spatial context are preserved.

![U-Net Architecture]({{ '/assets/images/group47/unet_architecture.png' | relative_url }}){:
style="width: 400px; max-width: 100%;"} _Fig 1. U-Net Encoder-Decoder
Architecture_

#### Training Process

1. **Data Preparation**: Labeled images are preprocessed through resizing,
   normalization, and augmentation.
2. **Forward Pass**: The input image is passed through the network to produce a
   segmentation map.
3. **Loss Calculation**: The predicted segmentation map is compared with the
   ground truth using a loss function such as Dice Loss.
4. **Backward Pass**: The model's parameters are updated using gradient descent
   and an optimizer like Adam.
5. **Training Loop**: The process is repeated over multiple epochs to minimize
   loss.
6. **Validation and Testing**: The model's accuracy is evaluated on unseen data.

### Limitations of U-Net

- **Conceptual Gap**: Encoder captures low-level details, while the decoder
  captures high-level semantics, making integration challenging.
- **Skip Connection Issues**: Direct skip connections may lead to poor feature
  fusion, affecting segmentation accuracy.

### Implementation

```
import torch
import torch.nn as nn
import torch.nn.functional as F

class UNet(nn.Module):
    def __init__(self, in_channels=1, out_channels=1):
        super(UNet, self).__init__()

        # Encoder layers
        self.enc1 = nn.Sequential(
            nn.Conv2d(in_channels, 64, kernel_size=3, padding=1),
            nn.ReLU(),
            nn.Conv2d(64, 64, kernel_size=3, padding=1),
            nn.ReLU()
        )
        self.pool1 = nn.MaxPool2d(2)

        self.enc2 = nn.Sequential(
            nn.Conv2d(64, 128, kernel_size=3, padding=1),
            nn.ReLU(),
            nn.Conv2d(128, 128, kernel_size=3, padding=1),
            nn.ReLU()
        )
        self.pool2 = nn.MaxPool2d(2)

        # Decoder layers
        self.up2 = nn.ConvTranspose2d(128, 64, kernel_size=2, stride=2)
        self.dec2 = nn.Sequential(
            nn.Conv2d(128, 64, kernel_size=3, padding=1),
            nn.ReLU(),
            nn.Conv2d(64, 64, kernel_size=3, padding=1),
            nn.ReLU()
        )

        self.final = nn.Conv2d(64, out_channels, kernel_size=1)

    def forward(self, x):
        # Encoder
        e1 = self.enc1(x)
        p1 = self.pool1(e1)

        e2 = self.enc2(p1)
        p2 = self.pool2(e2)

        # Decoder
        up2 = self.up2(e2)
        merge2 = torch.cat([up2, e1], dim=1)
        d2 = self.dec2(merge2)

        out = self.final(d2)
        return out
```

This implementation of U-Net creates the essential encoder-decoder structure of
U-Net with skip connections. It can be expanded with various optimization
techniques, such as adding more layers, data augmentation, and further.

### U-Net++

**U-Net++** improves upon U-Net by introducing **dense, nested skip
connections**:

- **Feature Refinement**: Intermediate convolutional layers progressively refine
  encoder features before passing them to the decoder.
- **Modular Design**: Easily integrates with other deep learning techniques for
  enhanced performance.

![U-Net++ Architecture]({{ '/assets/images/group47/unetplusplus_architecture.png' | relative_url }}){:
style="width: 400px; max-width: 100%;"} _Fig 2. U-Net++ with Nested Skip
Connections_

### Multi-Dimensional U-CNN

The **Multi-Dimensional U-Convolutional Neural Network** (2024) further refines
U-Net++ by:

- **Horizontal Refinement**: Adding convolution layers between encoder and
  decoder features to improve feature alignment.
- **Vertical Alignment**: Feeding feature maps from each layer into the
  horizontal convolution path for enhanced feature extraction.

### Implementation

```
import torch
import torch.nn as nn

class UNetPlusPlus(nn.Module):
    def __init__(self, in_channels=1, out_channels=1, num_filters=64):
        super(UNetPlusPlus, self).__init__()

        # Initial convolutional layers
        self.conv1 = nn.Sequential(
            nn.Conv2d(in_channels, num_filters, kernel_size=3, padding=1),
            nn.ReLU(),
            nn.Conv2d(num_filters, num_filters, kernel_size=3, padding=1),
            nn.ReLU()
        )
        self.pool1 = nn.MaxPool2d(2)

        self.conv2 = nn.Sequential(
            nn.Conv2d(num_filters, num_filters * 2, kernel_size=3, padding=1),
            nn.ReLU(),
            nn.Conv2d(num_filters * 2, num_filters * 2, kernel_size=3, padding=1),
            nn.ReLU()
        )
        self.pool2 = nn.MaxPool2d(2)

        # Decoder layers
        self.up2 = nn.ConvTranspose2d(num_filters * 2, num_filters, kernel_size=2, stride=2)
        self.conv2_1 = nn.Sequential(
            nn.Conv2d(num_filters * 2, num_filters, kernel_size=3, padding=1),
            nn.ReLU(),
            nn.Conv2d(num_filters, num_filters, kernel_size=3, padding=1),
            nn.ReLU()
        )

        self.final = nn.Conv2d(num_filters, out_channels, kernel_size=1)

    def forward(self, x):
        # Encoder
        e1 = self.conv1(x)
        p1 = self.pool1(e1)

        e2 = self.conv2(p1)

        # Decoder with nested skip connection
        up2 = self.up2(e2)
        merge2 = torch.cat([up2, e1], dim=1)
        d2 = self.conv2_1(merge2)

        out = self.final(d2)
        return out
```

This basic implementation captures the nested skip connections of U-Net++, which
refine features more effectively than the standard U-Net. The model can also be
further expanded by adding more layers or blocks to match the full complexity of
U-Net++.

Reference:
[Srinivasan et al., 2024](https://doi.org/10.1186/s12880-024-01197-5).

## nnU-Net

**nnU-Net** ("no-new-U-Net") automates the process of adapting U-Net to new
datasets, providing a robust and standardized pipeline. It streamlines the
workflow by automatically configuring network architecture, preprocessing steps,
and training strategies based on the characteristics of the input dataset. This
automation minimizes the need for manual intervention and ensures that the model
is optimized for a wide range of medical imaging tasks. nnU-Net also supports
both 2D and 3D image segmentation, making it highly versatile.

### Configurations

1. **Fixed Configurations**:

   - Learning rate, loss function, and optimizer remain consistent across
     datasets.

2. **Rule-Based Configurations**:

   - Adjustments for patch size, normalization, and batch size based on dataset
     properties.

3. **Empirical Configurations**:
   - Fine-tuning and post-processing based on validation performance.

### Pipeline

1. **Data Fingerprinting**: Extract image properties like spacing, intensity,
   and shape.
2. **Configuration Decisions**: Apply rule-based adjustments to network
   topology.
3. **Empirical Optimization**: Post-processing and ensembling to improve
   performance.
4. **Validation**: Ensures model robustness and generalization.

![nnU-Net Pipeline]({{ '/assets/images/group47/nnunet_pipeline.png' | relative_url }}){:
style="width: 400px; max-width: 100%;"} _Fig 3. nnU-Net Pipeline_

### Evaluation

- **State-of-the-Art Performance**: nnU-Net excels in competitions like BraTS
  and KiTS.
- **Generalization**: Works well across diverse datasets without manual tuning.
- **Resource Demands**: 3D configurations require significant computational
  resources.

### Implementation

```
import torch
import torch.nn as nn
import torch.nn.functional as F

class nnUNet(nn.Module):
    def __init__(self, in_channels=1, out_channels=1, num_filters=64):
        super(nnUNet, self).__init__()

        # Encoder blocks
        self.enc1 = self.conv_block(in_channels, num_filters)
        self.enc2 = self.conv_block(num_filters, num_filters * 2)
        self.enc3 = self.conv_block(num_filters * 2, num_filters * 4)

        # Pooling layers
        self.pool = nn.MaxPool2d(2)

        # Decoder blocks
        self.up3 = nn.ConvTranspose2d(num_filters * 4, num_filters * 2, kernel_size=2, stride=2)
        self.dec3 = self.conv_block(num_filters * 4, num_filters * 2)

        self.up2 = nn.ConvTranspose2d(num_filters * 2, num_filters, kernel_size=2, stride=2)
        self.dec2 = self.conv_block(num_filters * 2, num_filters)

        # Final output layer
        self.final = nn.Conv2d(num_filters, out_channels, kernel_size=1)

    def conv_block(self, in_channels, out_channels):
        return nn.Sequential(
            nn.Conv2d(in_channels, out_channels, kernel_size=3, padding=1),
            nn.ReLU(),
            nn.Conv2d(out_channels, out_channels, kernel_size=3, padding=1),
            nn.ReLU()
        )

    def forward(self, x):
        # Encoder
        e1 = self.enc1(x)
        p1 = self.pool(e1)

        e2 = self.enc2(p1)
        p2 = self.pool(e2)

        e3 = self.enc3(p2)

        # Decoder
        up3 = self.up3(e3)
        d3 = self.dec3(torch.cat([up3, e2], dim=1))

        up2 = self.up2(d3)
        d2 = self.dec2(torch.cat([up2, e1], dim=1))

        # Final output
        out = self.final(d2)
        return out

```

## Conclusion

Deep learning-based medical image segmentation models like **U-Net**,
**U-Net++**, and **nnU-Net** provide robust and efficient tools for clinical and
research applications. Despite challenges related to dataset variability and
computational resources, these models represent significant advancements in
medical imaging. Continued research and innovation will further improve
segmentation accuracy and accessibility.

## References

[1] Redmon, Joseph, et al.