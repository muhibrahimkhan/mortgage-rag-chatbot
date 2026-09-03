# License Plate Detection and OCR in MATLAB

This project detects a vehicle license plate from an image and extracts the alphanumeric characters using MATLAB image processing and OCR.

## Why this project is strong for a resume
This project demonstrates:
- MATLAB-based algorithm development
- image preprocessing and signal enhancement
- adaptive thresholding and morphology
- candidate region scoring and geometric filtering
- OCR post-processing and confidence estimation
- debug visualization for rapid prototyping and validation

That overlaps well with roles that value **MATLAB**, **algorithms**, **signal processing**, **prototyping**, and **experimental validation**.

## Files
- `license_plate_ocr.m` - main detection + OCR pipeline
- `demo_license_plate_ocr.m` - example script for testing
- `.gitignore` - optional MATLAB Git ignore file

## Toolboxes needed
- Image Processing Toolbox
- Computer Vision Toolbox

## How it works
1. Loads an image
2. Converts to grayscale and enhances contrast using adaptive histogram equalization
3. Computes gradient magnitude to highlight plate edges and characters
4. Uses adaptive thresholding and morphology to isolate plate-like regions
5. Scores candidate bounding boxes using geometry and texture features
6. Picks the best plate region
7. Normalizes the cropped plate
8. Runs OCR with a restricted character set
9. Cleans the OCR result and returns confidence

## Example usage
```matlab
results = license_plate_ocr("sample_car.jpg", true);
disp(results.plateText)
disp(results.confidence)
```

## Output
The function returns a struct:
```matlab
results.plateText
results.boundingBox
results.confidence
results.allCandidates
```

## Good resume bullet
Built a MATLAB computer vision pipeline for license plate detection and OCR using adaptive preprocessing, morphology, candidate scoring, and post-processed OCR confidence estimation.

## How to upload to GitHub
### Option 1: GitHub website
1. Create a new repository on GitHub
2. Name it something like `license-plate-detection-matlab`
3. Click **uploading an existing file**
4. Drag and drop:
   - `license_plate_ocr.m`
   - `demo_license_plate_ocr.m`
   - `README.md`
   - `.gitignore`
5. Commit the files

### Option 2: Git command line
Run these commands inside the project folder:
```bash
git init
git add .
git commit -m "Initial commit: MATLAB license plate detection and OCR project"
git branch -M main
git remote add origin https://github.com/YOUR-USERNAME/license-plate-detection-matlab.git
git push -u origin main
```

## How to make it look even better
- Add 3 to 5 test images and screenshots of detections
- Include one image showing the debug pipeline
- Add a short section called **Future Improvements**
- Mention possible embedded deployment or real-time optimization ideas in the README

## Honest positioning note
This is a strong **MATLAB algorithms / computer vision portfolio project**. It aligns with job requirements around MATLAB, prototyping, signal processing, and algorithm development, but it is not the same as a control systems project. For a control/algorithms role, pair this with one more project involving PID, Kalman filtering, or sensor fusion.
