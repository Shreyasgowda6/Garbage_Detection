# Waste Detection with YOLOv5s

Object detection for automated waste sorting. The model detects waste items in images and assigns each bounding box to one of six material categories: biodegradable, cardboard, glass, metal, paper, or plastic.

The primary workflow is in [`notebooks/Portfolio1_YOLOv5s.ipynb`](notebooks/Portfolio1_YOLOv5s.ipynb). It analyzes and prepares the data, trains a pretrained YOLOv5s-family detector, evaluates it, and visualizes prediction errors.

## Features

- Dataset and annotation inspection
- Annotation cleanup and duplicate-image analysis
- Group-aware, multi-label train/validation/test splitting
- Transfer learning with pretrained YOLOv5s-family weights
- Per-class precision, recall, and mAP evaluation
- Confusion matrices, precision-recall plots, and example detections
- Error analysis by class, object size, and error type

## Dataset

The source dataset contains 10,464 JPEG images at 416 × 416 pixels, with YOLO-format labels (`class_id x_center y_center width height`). Class names are inferred from image filename prefixes and checked against the labels; the dataset does not include an official class-name file.

| ID | Category |
|---:|---|
| 0 | Biodegradable |
| 1 | Cardboard |
| 2 | Glass |
| 3 | Metal |
| 4 | Paper |
| 5 | Plastic |

The dataset archive is not included in this repository. The notebook expects an archive containing a folder with `images/` and `labels/` subfolders. By default, it looks for:

```text
/content/drive/MyDrive/GarbageDataset.zip
```

Change `ZIP_PATH` in the notebook if the archive is stored elsewhere.

## Run in Google Colab

1. Open `notebooks/Portfolio1_YOLOv5s.ipynb` in Google Colab.
2. Select **Runtime → Change runtime type → GPU**.
3. Put the dataset archive in Google Drive and confirm `ZIP_PATH` in the notebook.
4. Run the notebook from top to bottom. It installs its Python packages, mounts Drive, prepares the dataset, trains the detector, evaluates the validation and test splits, and displays visualizations.
5. The notebook copies the model and result files to `MyDrive/Portfolio_1_results` so they remain available after the Colab runtime ends.

The notebook uses Ultralytics' YOLOv5s-family `yolov5su.pt` pretrained checkpoint. It currently targets Google Colab; running it locally requires replacing its Google Drive setup and archive path with local paths.

## Training configuration

The saved run used:

- Model: pretrained `yolov5su.pt` (YOLOv5u variant)
- Image size: 416
- Epochs: 100; early-stopping patience: 20
- Batch size: 16
- Optimizer: SGD with cosine learning-rate schedule
- Augmentation: mosaic, HSV variation, horizontal flip, scale, translation, small rotation, and erasing
- Seed: 42; deterministic mode enabled
- Hardware: Google Colab Tesla T4

The best checkpoint was selected using validation mAP@0.5:0.95 at epoch 86. Training ran for 100 epochs.

## Evaluation results

Results below are from the saved checkpoint in `Portfolio_1_results/best.pt`.

| Category | Test boxes | Precision | Recall | mAP@0.5 | mAP@0.5:0.95 |
|---|---:|---:|---:|---:|---:|
| Biodegradable | 4,709 | 0.711 | 0.541 | 0.602 | 0.330 |
| Cardboard | 484 | 0.657 | 0.583 | 0.645 | 0.463 |
| Glass | 689 | 0.809 | 0.808 | 0.869 | 0.672 |
| Metal | 605 | 0.767 | 0.702 | 0.755 | 0.524 |
| Paper | 524 | 0.775 | 0.512 | 0.613 | 0.453 |
| Plastic | 691 | 0.769 | 0.520 | 0.643 | 0.423 |
| **Overall** | **7,702** | **0.748** | **0.611** | **0.688** | **0.478** |

Glass achieved the strongest result. Small and crowded objects, especially biodegradable items, were the main challenge. At a separate confidence threshold of 0.25 and matching IoU of 0.5, the error analysis counted 4,631 true positives, 2,164 false positives, and 3,071 missed objects. Recall by object size was 0.328 for small, 0.627 for medium, and 0.827 for large objects.

## Data-split caveat

The split grouped images with perceptual-hash distance ≤ 4, yielding 7,331 training, 2,022 validation, and 1,029 test images. This prevented cross-split matches at that selected hash threshold. A separate thumbnail-correlation check nevertheless found 53 similar image pairs across splits at correlation ≥ 0.92, including 14 at ≥ 0.95. The evaluation may therefore retain some near-duplicate leakage; treat the reported test metrics as estimates rather than a fully independent benchmark.

The processed dataset currently in `data/processed/GarbageDataset` is from an earlier split (7,305 / 2,113 / 1,046). Running the primary notebook regenerates the split used for the results above.

## Results and artifacts

The exported run is stored in `Portfolio_1_results/`:

- `best.pt` — best model checkpoint
- `test_metrics_per_class.csv` — validation and test metrics by category
- `runs/yolov5s_garbage/` — training configuration, curves, sample batches, and checkpoint history
- `runs/eval_val/` and `runs/eval_test/` — confusion matrices, PR/F1 curves, and labeled/predicted batch visualizations
- `duplicate_groups.json` and `duplicates/` — duplicate-analysis groups, images, and reports

## Repository layout

```text
.
├── notebooks/
│   ├── Portfolio1_YOLOv5s.ipynb  # primary end-to-end workflow
│   ├── P1.ipynb                  # earlier YOLOv8 experiment
│   ├── p2.ipynb                  # structured notebook workflow
│   └── Pipeline.ipynb            # additional notebook
├── data/
│   ├── raw/GarbageDataset/       # local source images and labels
│   └── processed/GarbageDataset/ # generated train/val/test dataset
├── Portfolio_1_results/         # exported model and run artifacts
├── weights/
├── requirements.txt
└── README.md
```

## Limitations and next steps

- Class names are inferred from filenames rather than an official dataset specification.
- Some labels have inconsistent box granularity, especially for small or overlapping food items.
- Small objects have substantially lower recall than large objects.
- Near-duplicate images may remain across the data splits.
- The evaluation uses a single dataset and fixed 416-pixel image resolution; performance on production sorting-line images has not been measured.

Potential improvements include reviewing the remaining similar-image pairs, standardizing ambiguous annotations, and evaluating higher-resolution inputs or tiling for small objects.
