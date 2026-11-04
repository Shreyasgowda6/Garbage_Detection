# Garbage Detection Using YOLOv8

## Overview

This project focuses on developing an object detection system for automated waste sorting using deep learning.

The objective is to detect and classify different types of waste in images using the YOLOv8 object detection architecture. The model predicts the waste category and its location through bounding boxes.

The project is part of **Portfolio 1 – Computer Vision**, at Technical University of Applied Sciences Würzburg-Schweinfurt (THWS).

## Objectives

- Analyse and prepare a waste detection dataset.
- Design a suitable training and validation pipeline.
- Train a YOLOv8 object detection model.
- Evaluate model performance using object detection metrics.
- Visualise predictions and analyse detection errors.

## Dataset

The dataset contains **10,464 images**, with annotations in YOLO format.

The six object categories used in the project are:

| Class ID | Category |
|---|---|
| 0 | Biodegradable |
| 1 | Cardboard |
| 2 | Glass |
| 3 | Metal |
| 4 | Paper |
| 5 | Plastic |

*Note: The class-name mapping is based on the dataset filename conventions and should be verified against the original dataset documentation.*

### Dataset Split

The dataset was divided into three subsets using iterative multilabel stratification.

| Split | Images | Percentage |
|---|---:|---:|
| Training | 7,305 | 69.8% |
| Validation | 2,113 | 20.2% |
| Testing | 1,046 | 10.0% |
| **Total** | **10,464** | **100%** |

The test dataset was kept separate from training and validation and used for final model evaluation.

## Model and Technologies

### Model

**YOLOv8s (You Only Look Once – Small)** was selected for object detection.

YOLO is a single-stage object detector that predicts object classes and bounding boxes in one forward pass.

### Technologies Used

- Python
- PyTorch
- Ultralytics YOLOv8
- OpenCV
- NumPy
- Pandas
- Matplotlib
- Seaborn
- Jupyter Notebook

### Hardware

- Apple MacBook Air M5
- 16 GB unified memory
- Apple Metal Performance Shaders (MPS) acceleration

## Training Configuration

The initial YOLOv8s training configuration was:

| Parameter | Value |
|---|---|
| Model | YOLOv8s |
| Image size | 416 × 416 |
| Epochs | 10 |
| Batch size | 16 |
| Device | MPS |
| Optimizer | Auto |
| Pretrained weights | Enabled |
| Workers | 2 |
| Random seed | 42 |

Data augmentation included geometric transformations, horizontal flipping, HSV colour augmentation, and mosaic augmentation settings.

Training completed successfully in approximately one hour.

## Model Evaluation

The trained model was evaluated on the separate test dataset.

### Overall Test Results

| Metric | Score |
|---|---:|
| Precision | 0.688 |
| Recall | 0.488 |
| mAP@50 | 0.559 |
| mAP@50–95 | 0.362 |

### Class-wise Results

| Category | Precision | Recall | mAP@50 | mAP@50–95 |
|---|---:|---:|---:|---:|
| Biodegradable | 0.780 | 0.515 | 0.625 | 0.331 |
| Cardboard | 0.528 | 0.440 | 0.484 | 0.336 |
| Glass | 0.746 | 0.536 | 0.584 | 0.405 |
| Metal | 0.631 | 0.656 | 0.689 | 0.449 |
| Paper | 0.764 | 0.301 | 0.402 | 0.293 |
| Plastic | 0.676 | 0.481 | 0.569 | 0.357 |

### Initial Observations

- Metal achieved the highest mAP@50 of 0.689.
- Paper had the lowest recall at 0.301, indicating that many paper objects were missed.
- Cardboard had the lowest precision at 0.528.
- The difference between precision and recall suggests that the model needs improvement in detecting all objects present in an image.

## Project Structure

```text
Garbage_Detection/
│
├── data/
│   ├── raw/
│   │   └── GarbageDataset/
│   │
│   └── processed/
│       └── GarbageDataset/
│           ├── train/
│           │   ├── images/
│           │   └── labels/
│           ├── val/
│           │   ├── images/
│           │   └── labels/
│           ├── test/
│           │   ├── images/
│           │   └── labels/
│           └── data.yaml
│
├── notebooks/
│   └── P1.ipynb
│
├── results/
│   ├── plots/
│   └── predictions/
│
├── weights/
│
├── requirements.txt
├── .gitignore
└── README.md
```

*Note: Training outputs are currently saved under `notebooks/runs/`. The structure above represents the intended organisation; the final output folders will be organised during project cleanup.*

## Installation

Clone the repository:

```bash
git clone https://github.com/Shreyasgowda6/Garbage_Detection.git
```

Navigate to the project directory:

```bash
cd Garbage_Detection
```

Create a virtual environment:

```bash
python3 -m venv .venv
```

Activate it:

```bash
source .venv/bin/activate
```

Install the required dependencies:

```bash
python -m pip install -r requirements.txt
```

Open the notebook:

```bash
jupyter notebook
```

Run `notebooks/P1.ipynb` to explore the dataset, training pipeline, evaluation, and visualisations.

**Dataset and trained weights:** The dataset and model checkpoint files are not included in the Git repository. They must be obtained separately.