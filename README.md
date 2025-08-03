## 1. Objectives  
- Build an OCR system to extract text from scanned documents or images.
- Preprocess images to enhance OCR accuracy.
- Use OCR engines like Tesseract or transformer-based models like TrOCR.
- Present the extracted text in a structured and user-friendly format (e.g., JSON, CSV, 
plain text).

## 2. Dataset Selection & Overview  

In this task We chose the **TextOCR** dataset from Kaggle because it reflects real‑world challenges: printed text on street signs, product labels, documents and more. With 21 778 images and diverse layouts, it’s perfect for testing both classical and modern OCR methods.
<br/><br/>
<img width="1798" height="366" alt="image" src="https://github.com/user-attachments/assets/194e4557-4e8d-4102-99b1-fff8004c36a5" />

- **Source & Size**  
  - Kaggle “TextOCR – Text Extraction from Images” [1] 
  - **21 778** JPEG images, each paired with a ground‑truth text file  
<br/><br/>
<img width="1256" height="276" alt="image" src="https://github.com/user-attachments/assets/6ef816bc-4f4b-48a0-a0e0-b9cffba708e0" />
<img width="1128" height="1081" alt="image" src="https://github.com/user-attachments/assets/b7c5333e-9b99-45e1-aeea-b73fd306a974" />

- **Text Types & Length**  
  - **Printed scene text**: street signs, packaging, magazine pages  
  - **Word counts** range from **1** (logos, stamps) to **600** (full-page spreads)  
  - **Median** ≈ 21 words; **75th percentile** ≈ 49 words; **mean** ≈ 48 words  
<br/><br/>
<img width="777" height="269" alt="image" src="https://github.com/user-attachments/assets/0c16cf36-9b28-484a-9dd5-0a5b5bf676b0" />

- **Image Formats & Resolutions**  
  - All images in **.jpg** format  
  - Top resolutions:  
    - 1024×768 with 5 373 images  
    - 1024×683 with 2 395 images  
    - 768×1024 with 1 644 images  
    - 1024×1024 with 1 506 images  
    - 1024×680 with   657 images  
<br/><br/>
<img width="684" height="334" alt="image" src="https://github.com/user-attachments/assets/f6012ff7-f994-403a-bdd7-02fab80577ca" />

- **Quality Metrics (sample of 1 000 images)**  
  - **Contrast**: mean ≈ 61 (std ≈ 15)  
  - **Noise level**: mean ≈ 6.9 (std ≈ 3.9)  
<br/><br/>
<Figure size 1400x1200 with 4 Axes><img width="1349" height="1126" alt="image" src="https://github.com/user-attachments/assets/bfe2f342-98d0-4e33-adea-ea252a80d162" />

- **Representative Examples** from the dataset and its annotations
  - **1 word**: brand logos or date stamps  
  - **≈ 9 words**: watch face labels, directional signs  
  - **21 words**: product descriptions on packaging  
  - **> 2 000 words**: book or magazine spreads  

This variety in text shape, layout and image quality ensures our OCR pipeline is robust across simple and complex scenarios.  
 
## 3. Image Preprocessing  

<img width="1489" height="859" alt="image" src="https://github.com/user-attachments/assets/9b0baeab-a821-4527-a902-db939736e529" />

- **Convert to grayscale**  
  We collapse RGB into a single intensity map—this speeds up processing and makes text strokes stand out against busy backgrounds.

- **Denoise with edge-preserving filter**  
  We smooth away speckles and camera noise while keeping sharp edges intact, so letters stay crisp even on textured surfaces.

- **Detect and correct skew**  
  We find dominant lines, compute the tiny rotation angle, and straighten the image—no more slanted text throwing off segmentation.

- **Adaptive binarization**  
  We turn the deskewed gray image into a high-contrast black-and-white map, boosting text pixels and fading out uneven lighting.

- **Locate text regions**  
  We use morphology and contour analysis to draw tight boxes around signs, labels, and logos—isolating every bit of text in complex scenes.
  
## 4. OCR Engine Integration 

<img width="1928" height="1294" alt="image" src="https://github.com/user-attachments/assets/603d6421-65c1-413f-a9ec-60e93b79b54d" />


- **Initialize EasyOCR reader**  
  We spin up `easyocr.Reader(['en','ar'])` so our pipeline handles both English and Arabic out of the box.

- **Iterate with a progress bar**  
  Wrapping `tqdm` around the file list gives real-time feedback as each image is processed.

- **Read text & filter by confidence**  
  We call `reader.readtext()` to detect text regions then drop any result with confidence < 0.5 to avoid false positives.

- **Prepare image for drawing**  
  Convert OpenCV’s BGR to RGB and make a copy so we can overlay without mutating the original.

- **Draw boxes and text overlays**  
  For each high-confidence detection, we draw a semi-transparent rectangle and render the recognized string at the top-left corner.

- **Save annotated outputs**  
  Finally, we convert back to BGR and write so the image now showing exactly where and what the OCR engine read.
  <br/>
- Here is an example of the output:
  <img width="1533" height="1015" alt="image" src="https://github.com/user-attachments/assets/a38c820a-d181-4673-aefd-d15269d2cc9b" />
<br/>
***Note that we only apply OCR on 1000 files (images) due to the huge dataset size and the large amount of time reqiured to process the OCR.***

## 5. Post-processing 

<img width="1297" height="1324" alt="image" src="https://github.com/user-attachments/assets/37840da8-4e9d-4236-addf-52273ff5a4f6" />

- **Advanced text cleanup**  
  - Normalize Unicode (NFKC) so ligatures and diacritics become consistent  
  - Strip out any non-Arabic/English characters with a regex filter  
  - Collapse repeated punctuation (e.g. “…,” → “.”) and trim extra spaces  
  - Convert any stray A–Z letters to lowercase for uniformity

- **Spell-checking**  
  We run each cleaned line through a SpellChecker:  
  - Only alphabetic tokens get checked  
  - Corrections are applied when a valid suggestion exists  
  - Numbers and mixed tokens pass through unchanged

- **Finalize & export**  
  The pipeline writes out `final_extracted_text.csv` with columns `[image_name, final_text, avg_confidence, line_count]`.  


## 6. Evaluation  
<img width="1464" height="583" alt="image" src="https://github.com/user-attachments/assets/51178da4-06e5-4094-94b0-4d7300400e01" />
<img width="1189" height="390" alt="image" src="https://github.com/user-attachments/assets/e19d2f27-80e0-4c42-ab27-428464057f2b" />

- **Overall accuracy**  
  - **Mean CER:** ~30.8%  
  - **Mean WER:** ~5.7%

- **Top performers** (lowest WER = 0)  
  - Pages with crisp, short headers/logos had perfect reconstruction.

- **Challenging pages** (highest WER > 200)  
  - Extremely dense text or complex layouts led to WER > 200 on a few pages.

- **Error distributions**  
  - Most pages cluster at **WER < 20** and **CER < 100**.  
  - A long tail of higher-error pages corresponds to very long paragraphs or unusual formatting.

- **Visual diagnostics**  
  - Histograms show a tight peak at low error rates, confirming reliable extraction on the bulk of pages.  
  - Outliers highlight pages for targeted post-processing or manual review.  
 

## 7. Visualization & Examples  
<img width="1876" height="1302" alt="image" src="https://github.com/user-attachments/assets/5a68c121-ceee-4f67-8643-d990b377d94e" />
<img width="1875" height="1285" alt="image" src="https://github.com/user-attachments/assets/dde85033-03f8-4e6a-8a87-b19643d82621" />


- **Side-by-side comparison**  
  We overlay detected text boxes (red) on the original image, show the binarized/thresholded result and render the final OCR string—so you can see exactly which regions were read.

- **High-contrast processing**  
  The middle column’s black-white mask highlights how adaptive thresholding isolates text areas, even against busy backgrounds.

- **Accurate text capture**  
  - “ROAD WORK AHEAD” signs yield clean, lowercase “road work ahead.”  
  - Album covers like “Quadromania” get mostly correct OCR (“gene ammo’s you can depend on me”).  
  - Street signs (“WESTON RD. 1149”) are picked out perfectly.

- **Edge cases**  
  When no text is found, we display `nan`—making it easy to spot images that need manual review.

These examples demonstrate our pipeline’s ability to localize, enhance and extract text from diverse real-world scenes.  

## 8. References  
- Dataset used [1]:https://www.kaggle.com/datasets/robikscube/textocr-text-extraction-from-images-dataset?select=train_val_images  
- OCR model used: https://github.com/JaidedAI/EasyOCR


## 9. Project Structure

```
TASK6/
├── data/
│ ├── train_val_images/ # raw images for OCR
│ ├── train_val_images_cleaned/ # preprocessed binaries & deskewed images
│ ├── annot.csv # ground-truth annotations (CSV)
│ ├── annot.parquet # ground-truth annotations (Parquet)
│ ├── img.csv # image metadata (CSV)
│ ├── img.parquet # image metadata (Parquet)
│ ├── ocr_tesseract.json # raw Tesseract JSON outputs
│ ├── text_regions.json # detected text-region coordinates
│ └── TextOCR_0.1_train.json # original dataset split file
├── csv_output/
│ ├── extracted_text.csv 
│ ├── final_extracted_text.csv 
│ └── evaluation_summary.csv 
├── Text_Extraction_from_Images.ipynb 

```


## 10. How to Run

1. **Install requirements**  
   Make sure you have Python 3.7+ and Tesseract OCR installed. Then install the required packages:
   ```bash
   pip install jupyter pandas pdf2image pytesseract easyocr fitz-python matplotlib seaborn
2. **Prepare your data folder**
Place all your inputs under TASK6/data/:

TASK6/data/
  ├── train_val_images/               # raw images for OCR
  ├── train_val_images_cleaned/       # (optional) preprocessed images
  ├── annot.csv, annot.parquet        # ground-truth CSV/Parquet
  ├── img.csv, img.parquet            # image metadata
  ├── ocr_tesseract.json              # raw Tesseract outputs
  ├── text_regions.json               # detected text boxes
  └── TextOCR_0.1_train.json          # dataset split info

3. **mkdir -p TASK6/csv_output**

4. **cd TASK6**
jupyter notebook Text_Extraction_from_Images.ipynb
or open it in VS Code / Colab.

Run all cells

