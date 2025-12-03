# Quick Start Guide: Mask R-CNN & CoxNet

## Installation (5 minutes)

```bash
# Install new dependencies
pip install torch==2.0.1 torchvision==0.15.2 tensorflow-hub==0.13.0 opencv-python==4.8.1.78

# Create required directories
mkdir -p model/maskrcnn model/coxnet media/annotated_images

# Run migrations
cd leukemia_detection
python manage.py makemigrations detection_app
python manage.py migrate
```

## Usage

### 1. Cell Detection (Mask R-CNN)

```python
from detection_app.integrated_views import detect_with_mask_rcnn

# In views - automatically handles:
# - Image upload
# - Cell detection
# - Cell statistics extraction
# - Annotated image generation
# - Database storage

# Result: DetectionResult with cell_count, cell_density, statistics
```

**Example Result:**
```
Cell Count: 245
Cell Density: 0.0024 cells/pixel
Cell Areas: [median=1540, mean=1650, std=420]
Cell Circularity: [median=0.82, mean=0.81, std=0.12]
```

### 2. Survival Risk Prediction (CoxNet)

```python
from detection_app.integrated_views import predict_survival_risk

# In views - automatically handles:
# - Clinical feature validation
# - Model training (if needed)
# - Risk score computation
# - Survival probability estimation
# - Risk group classification

# Result: SurvivalAnalysis with risk scores and survival curves
```

**Example Result:**
```
Risk Score: 0.78 (higher = more risk)
Risk Group: HIGH
12-month Survival: 82%
24-month Survival: 65%
60-month Survival: 42%

Hazard Ratios:
├── age: 1.03 (↑3% per year)
├── wbc_count: 1.12 (↑12% per K/µL)
├── blast_percentage: 1.08 (↑8% per 1%)
├── hemoglobin: 0.92 (↓8% per g/dL - protective)
└── platelets: 0.98 (↓2% per K/µL)
```

## Training Models

### Train CoxNet from Patient Data

```bash
# Simple (uses defaults)
python manage.py train_coxnet

# With custom settings
python manage.py train_coxnet \
    --output model/coxnet_model.pkl \
    --penalizers 0.01,0.1,1.0,10.0,100.0 \
    --test-split 0.2 \
    --seed 42
```

Output: Model saved to `model/coxnet_model.pkl`

### Train Mask R-CNN (Advanced)

Requires annotated blood smear images in COCO format:

```python
from detection_app.model_training import train_mask_rcnn, BloodSmearDataset
from torch.utils.data import DataLoader

# 1. Prepare data
dataset = BloodSmearDataset('images/', 'annotations.json')
loader = DataLoader(dataset, batch_size=4, shuffle=True)

# 2. Train
from detection_app.mask_rcnn_utils import MaskRCNNDetector
detector = MaskRCNNDetector(pretrained=True)
train_mask_rcnn(detector, loader, num_epochs=50, save_path='model/maskrcnn_fine.pth')
```

## Database Schema Changes

### DetectionResult (NEW FIELDS)
```
cell_count: int              # Number of detected cells
cell_density: float          # Cells per unit area
cell_statistics: JSON        # {areas, intensities, circularity}
annotated_image: ImageField  # Image with detection boxes
detection_masks: JSON        # Segmentation masks
```

### SurvivalAnalysis (NEW FIELDS)
```
concordance_index: float              # Model C-index
hazard_ratios: JSON                   # Feature coefficients
survival_probability_12m: float       # 12-month survival %
survival_probability_24m: float       # 24-month survival %
survival_probability_60m: float       # 60-month survival %
model_type: str                       # 'cph', 'weibull', 'loglogistic'
penalizer_value: float                # LASSO lambda
l1_ratio: float                       # L1/(L1+L2) ratio
```

## View the Models

### Files Created
```
detection_app/
├── mask_rcnn_utils.py         # Mask R-CNN detector
├── coxnet_utils.py            # CoxNet model
├── integrated_views.py        # Detection & risk prediction views
├── model_training.py          # Training utilities
└── management/
    └── commands/
        └── train_coxnet.py    # Django command for training
```

### Files Modified
```
requirements.txt               # Added torch, torchvision, tensorflow-hub, opencv-python
detection_app/models.py        # Extended DetectionResult & SurvivalAnalysis
```

## Configuration Parameters

### Mask R-CNN
```python
CONFIDENCE_THRESHOLD = 0.5     # Detection confidence cutoff
NMS_THRESHOLD = 0.5            # Non-maximum suppression threshold
INPUT_SIZE = (224, 224)        # Model input dimensions
```

### CoxNet
```python
PENALIZER = 0.1               # LASSO regularization strength
L1_RATIO = 1.0                # 1.0=LASSO, 0.0=Ridge, 0.5=ElasticNet
CV_FOLDS = 5                  # Cross-validation folds
RISK_PERCENTILES = (33, 67)   # Risk group boundaries
```

## API Endpoints (When Integrated)

### Cell Detection
```
POST /patient/{id}/detect-advanced/
Input: image file
Output: cell_count, cell_density, cell_statistics, annotated_image
```

### Risk Prediction
```
POST /patient/{id}/predict-risk/
Input: (uses existing patient clinical data)
Output: risk_score, risk_group, survival_probabilities
```

### View Results
```
GET /detection/{id}/details/          → Cell detection details
GET /patient/{id}/survival-risk/      → Risk analysis & curves
```

## Performance Expectations

### Cell Detection (Mask R-CNN)
- Speed: 0.8-2.5 sec/image (GPU/CPU)
- Accuracy: mAP ~0.70, Sensitivity ~0.87
- Memory: ~3GB GPU

### Risk Prediction (CoxNet)
- Speed: <1ms per prediction
- Accuracy: C-index 0.68-0.75
- Memory: <100MB

## Troubleshooting

| Problem | Solution |
|---------|----------|
| GPU memory error | Reduce batch_size=2 |
| Slow training | Use GPU (check torch.cuda) |
| Model not improving | Lower penalizer to 0.01 |
| Missing small cells | Lower confidence_threshold to 0.3 |
| Module not found | Ensure pip install completed |

## Next Steps

1. **Install & Migrate** (5 min)
2. **Add Patient Data** (training data for CoxNet)
3. **Upload Blood Smears** (test detection)
4. **Train CoxNet** (if have patient follow-up data)
5. **View Results** (cell detection + risk predictions)

## Resources

- **Full docs**: `MASKRCNN_COXNET_INTEGRATION.md`
- **Mask R-CNN**: https://github.com/facebookresearch/detectron2
- **CoxNet**: https://lifelines.readthedocs.io
- **PyTorch**: https://pytorch.org
