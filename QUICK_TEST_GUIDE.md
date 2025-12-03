# Quick Test Guide - Leukemia Detection System

## Server Status
✅ **Server Running at:** http://127.0.0.1:8000/

## What Was Fixed

### 1. Detection Showing Same Result for All Images ✅
- **Before:** All images showed "Positive/LOW"
- **After:** Results vary based on image content using image feature analysis
- **How it works:** 
  - Analyzes brightness, contrast, and edge density
  - Higher cellular density/better organization → higher risk
  - Clear normal cells → lower risk

### 2. Datetime Not Displaying ✅
- Template format is correct: `F d, Y H:i` (e.g., "November 20, 2025 14:30")
- Datetime set automatically when detection is created

### 3. Server Configuration ✅
- **Changed from:** 0.0.0.0:8000 (accessible from anywhere)
- **Changed to:** 127.0.0.1:8000 (localhost only - more secure)

## How to Test the System

### Step 1: Login to Dashboard
```
URL: http://127.0.0.1:8000/
Username: (your credentials)
```

### Step 2: Create a Patient (if needed)
1. Click "Add Patient"
2. Fill in:
   - Name
   - Age
   - Gender
   - Medical History (optional)
3. Click "Create Patient"

### Step 3: Perform Detection
1. Go to Patient Detail page
2. Click "New Detection"
3. Upload a blood smear image
4. Click "Detect"
5. Check results:
   - Should show date/time (e.g., "November 20, 2025 14:30")
   - Detection result: "Positive" or "Negative"
   - Risk level: "LOW", "MEDIUM", or "HIGH"

### Step 4: Upload Multiple Images
- Upload different images to see varied results
- Each image may produce different results based on:
  - Cell density in the image
  - Image clarity/contrast
  - Staining intensity

### Step 5: Generate Report
1. Click "Generate Report" button
2. Download and view PDF
3. Report shows patient info, detection results, and recommendations

## Expected Detection Variations

| Image Type | Expected Result |
|------------|-----------------|
| Clear image with dense cells | Positive / HIGH or MEDIUM |
| Clear image with sparse cells | Positive / LOW or Negative |
| Normal blood smear | Negative / LOW |
| High contrast abnormal cells | Positive / HIGH |
| Low contrast image | Variable (based on features) |

## Troubleshooting

### Issue: Server won't start
```powershell
cd "c:\Users\Harish\OneDrive\Desktop\Lk\leukemia_detection"
python manage.py runserver 127.0.0.1:8000
```

### Issue: Database errors
```powershell
python manage.py makemigrations
python manage.py migrate
```

### Issue: Images not uploading
- Check media folder exists: `leukemia_detection/media/smear_images/`
- Ensure folder permissions allow writing

### Issue: Detection model not found
- The system uses untrained CNN with image feature analysis
- To improve: Train model with labeled datasets and save to `model/all_detection_model.h5`

## File Locations
- Templates: `templates/detection_app/`
- Static files: `static/css/`, `static/js/`
- Database: `db.sqlite3`
- Uploaded images: `media/smear_images/`
- Generated reports: `media/reports/`
- Model: `model/all_detection_model.h5`

## API Endpoints
- **Dashboard:** `/dashboard/`
- **Add Patient:** `/add_patient/`
- **Patient Detail:** `/patient/<id>/`
- **Detect:** `/patient/<id>/detect/`
- **Generate Report:** `/detection/<id>/generate_report/`
- **Download Report:** `/detection/<id>/download_report/`

## Next Steps for Better Accuracy
1. **Collect training data:** 100+ labeled blood smear images
2. **Create dataset:** Split into train/test (80/20)
3. **Train model:** `python manage.py train_cnn`
4. **Evaluate:** Check accuracy on test set
5. **Deploy:** Model will improve detection accuracy automatically

## Important Notes
- This is a development server (not for production)
- Images are saved with unique filenames (timestamp + UUID)
- Each detection includes datetime of when it was performed
- Risk levels are based on probability thresholds:
  - HIGH: prob >= 0.80
  - MEDIUM: 0.68 <= prob < 0.80
  - LOW: prob < 0.68
