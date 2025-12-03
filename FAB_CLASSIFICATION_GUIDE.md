# FAB Classification System - Leukemia Detection

## Overview

This system detects and classifies leukemia into FAB (Franco-American-British) morphological subtypes:

### ALL (Acute Lymphoblastic Leukemia) Classifications:
- **L1**: Small uniform cells with high nuclear-to-cytoplasmic ratio, scanty cytoplasm, clumpy chromatin
- **L2**: Large heterogeneous cells with moderate cytoplasm and dispersed chromatin pattern
- **L3**: Large cells with extensive vacuolization (Burkitt-like appearance)

### AML (Acute Myeloid Leukemia) Classifications:
- **M0**: Undifferentiated AML with minimal cytological features
- **M1**: Minimal myeloid differentiation with myeloblasts
- **M2**: Myeloid with maturation and differentiation
- **M3**: Acute Promyelocytic Leukemia (APL) with abundant Auer rods and heavy granulation
- **M4**: Acute Myelomonocytic Leukemia with mixed myeloid and monocytic features
- **M5**: Acute Monocytic/Monocytoid Leukemia
- **M6**: Acute Erythroid Leukemia (rare, erythroid predominance)
- **M7**: Acute Megakaryocytic Leukemia (rare, megakaryocytic features)

## How FAB Classification Works

### 1. Morphological Feature Extraction
The system analyzes 8 key morphological features from blood smear images:

| Feature | Description | Range | Usage |
|---------|-------------|-------|-------|
| **Staining Intensity** | Overall brightness/hemoglobin content | 0-1 | Normal vs abnormal cells |
| **Chromatin Density** | Darkness of nucleus (DNA content) | 0-1 | Cell maturity level |
| **Cell Size** | Estimated size of cells | 0-1 | Normal vs leukemic cells |
| **Size Uniformity** | Consistency of cell sizes | 0-1 | L1 (uniform) vs L2 (variable) |
| **Vacuolization** | Light spots in cytoplasm | 0-1 | L3 (high) vs others (low) |
| **N:C Ratio** | Nuclear-to-cytoplasmic ratio | 0-2+ | Immature (high) vs mature (low) |
| **Circularity** | Shape regularity | 0-1 | Cell morphology classification |
| **Contrast** | Intensity variation | 0-1 | Edge definition and clarity |

### 2. Leukemia Type Classification
First, the system determines if the sample contains:
- **ALL** (Acute Lymphoblastic Leukemia)
- **AML** (Acute Myeloid Leukemia)
- **CLL** (Chronic Lymphocytic Leukemia)
- **CML** (Chronic Myeloid Leukemia)
- **NORMAL** (Healthy/Non-leukemic)

### 3. FAB Subtype Classification
Based on the leukemia type, the system further classifies into specific FAB subtypes:

#### ALL FAB Classification Logic
```
L1: IF cell_size < 0.4 AND uniformity > 0.6 AND vacuoles < 0.15
L2: IF cell_size 0.4-0.65 AND uniformity < 0.7 AND vacuoles 0.1-0.25
L3: IF cell_size > 0.5 AND vacuoles > 0.25 (heavy vacuolization)
```

#### AML FAB Classification Logic
```
M0: IF cell_size < 0.5 AND nc_ratio > 0.75 (undifferentiated)
M1: IF cell_size 0.4-0.6 AND nc_ratio 0.6-0.8
M2: IF cell_size 0.5-0.7 AND nc_ratio 0.5-0.7
M3: IF chromatin > 0.6 AND staining < 0.4 (heavy granulation - APL)
M4: IF cell_size 0.45-0.65 AND nc_ratio 0.5-0.75
M5: IF cell_size > 0.55 AND nc_ratio < 0.65 AND chromatin < 0.5 (monocytic)
M6: IF staining > 0.55 AND cell_size > 0.5 (erythroid features)
M7: IF cell_size > 0.65 AND circularity < 0.6 (irregular shape)
```

## Database Fields

### DetectionResult Model - FAB Fields
```python
# Leukemia type
leukemia_type = CharField(choices=['ALL', 'AML', 'CLL', 'CML', 'NORMAL'])

# ALL specific FAB classification
all_fab_classification = CharField(choices=['L1', 'L2', 'L3'])

# AML specific FAB classification
aml_fab_classification = CharField(choices=['M0', 'M1', 'M2', 'M3', 'M4', 'M5', 'M6', 'M7'])

# Confidence score for FAB classification
fab_confidence = FloatField(0-1 range)
```

## Usage in Views

### Detection View Integration
```python
from detection_app.fab_classification import FABClassifier

# Perform FAB classification
fab_results = FABClassifier.detect_fab_classification(image_file)

# Results include:
# - leukemia_type: The main leukemia type detected
# - leukemia_confidence: Confidence score for type
# - fab_classification: Specific FAB subtype (L1/L2/L3 or M0-M7)
# - fab_confidence: Confidence score for FAB classification
# - morphological_features: Dictionary of all extracted features
```

### Template Display
The patient detail template automatically displays:
- Leukemia type with full name
- FAB classification with morphological description
- Classification confidence percentage

Example output:
```
Leukemia Type: Acute Lymphoblastic Leukemia (ALL)
ALL FAB Classification: L2 (Large heterogeneous cells)
Classification Confidence: 87.3%
```

## Report Generation

Reports now include FAB classification details:
```
Detection Results:
- Result: Positive
- Risk Level: MEDIUM
- Leukemia Type: Acute Myeloid Leukemia (AML)
- AML FAB Classification: M3 (Acute Promyelocytic Leukemia - APL)
- FAB Confidence: 91.2%
```

## Clinical Significance

### ALL FAB Types
- **L1** (70% of ALL in children): Better prognosis, uniform appearance
- **L2** (25% of ALL in adults): Variable features, different treatment
- **L3** (5%, Burkitt): Highly aggressive, requires special treatment

### AML FAB Types
- **M3 (APL)** (5-10% of AML): Highly responsive to ATRA/arsenic, best prognosis
- **M4-M5** (30% of AML): Monocytic features, prone to disseminated intravascular coagulation
- **M6, M7** (<5% each): Rare, different prognosis and treatment

## Accuracy Considerations

The FAB classification system uses image-based morphology analysis. Accuracy depends on:
1. **Image Quality**: Clear, well-stained smears provide better accuracy
2. **Cell Representation**: Sufficient visible cells in the field
3. **Staining Consistency**: Proper Wright-Giemsa or similar staining
4. **Blast Percentage**: Adequate abnormal cell population

Confidence scores above 70% generally indicate reliable classifications.

## Troubleshooting

### Low Confidence Scores
- Check image quality and staining
- Ensure sufficient cell population in image
- Verify proper sample preparation

### Misclassifications
- Compare with multiple fields on the same slide
- Consider clinical context (CBC, bone marrow aspirate)
- Consult with hematopathologist for confirmation

## Implementation Files

1. **fab_classification.py**: Core classification engine with FABClassifier class
2. **models.py**: Database fields for DetectionResult (leukemia_type, *_fab_classification, fab_confidence)
3. **views.py**: Integration in detect_leukemia() view
4. **patient_detail.html**: Template display with descriptions
5. **generate_report()**: PDF report generation with FAB details

## Future Enhancements

1. **Deep Learning Model**: Train CNN specifically for FAB classification
2. **Morphometric Analysis**: Automated measurement of nuclear and cytoplasmic features
3. **Cytochemistry Integration**: Add special staining results (MPO, PAS, etc.)
4. **Multi-slide Analysis**: Process and average results from multiple fields
5. **Confidence Intervals**: Provide statistical confidence ranges
6. **Feature Visualization**: Highlight detected features in annotated images
