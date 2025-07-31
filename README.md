## 1. Objectives  
- Build an OCR system to extract text from scanned documents or images.
- Preprocess images to enhance OCR accuracy.
- Use OCR engines like Tesseract or transformer-based models like TrOCR.
- Present the extracted text in a structured and user-friendly format (e.g., JSON, CSV, 
plain text).

## 2. Dataset Selection & Overview  
The document is a 39 page PDF drafted entirely in Arabic with a formal legal register. While its body consists of pure, selectable text throughout, it features a high-resolution header logo on the cover, a subtle watermark/footer graphic on every page and a Vision 2030 logo. Aside from these embedded images, the layout relies solely on text elements—headings, subheadings, consistent margins and line spacing without any tables, charts or scanned page images.
<br/><br/>
<img width="684" height="255" alt="image" src="https://github.com/user-attachments/assets/f2918aac-8ecb-46f7-a125-bf9e216d8544" />

- **Pages**  
  39 pages of Modern Standard Arabic legal text.
<br/>
<img width="989" height="390" alt="image" src="https://github.com/user-attachments/assets/2e12350a-6d4d-47bd-baa4-f972413e6e91" />

- **Text Density**  
  - **Chars per page:** mean ≈ 1 307 (min 30, max 1 841)  
  - **Words per page:** mean ≈ 226 (min 5, max 350)

- **Layout Consistency**  
  Most pages hover around 1 200–1 500 characters (≈ 200–300 words), with only a handful of very short front/back matter pages.
<br/>
<img width="1491" height="660" alt="image" src="https://github.com/user-attachments/assets/eb1c06e4-e612-4227-b57f-7553228ff2f2" />

- **Embedded Graphics**  
  Out of 116 image objects, only 4 are unique: cover logo, footer watermark, Vision 2030 emblem, and page header mark.

- **File Properties**  
  - 39-page PDF, text-selectable (no scanned images)  
  - Formal legal register, consistent margins and spacing  


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
<img width="1489" height="823" alt="image" src="https://github.com/user-attachments/assets/701f909c-d7ff-4c8a-9297-a906dc79159e" />

- **Grayscale conversion**  
  We collapse the colored header logo into a single-channel intensity map—this sharpens the palm-and-swords emblem and Arabic text.

- **Gaussian blur**  
  We lightly smooth out JPEG artifacts and subtle page-render noise while keeping the logo’s edges crisp.

- **Adaptive thresholding**  
  We binarize the blurred gray image into a stark black-and-white mask, which makes the logo and text stand out against the white background.

- **Deskew correction**  
  We detect the tiny tilt of the extracted pixmap, rotate it upright, and ensure the Arabic script reads in the correct orientation.

- **Contour extraction**  
  We locate the largest continuous border around the emblem, draw a tight bounding box, and isolate the logo for downstream processing.  


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
  
## 4. OCR Engine Integration and Post‑processing
<img width="1420" height="567" alt="image" src="https://github.com/user-attachments/assets/5cc4fc03-5d98-4b88-9fcd-4bd6eba64c77" />

<img width="2031" height="1319" alt="image" src="https://github.com/user-attachments/assets/43cea844-7dbc-40ac-81a5-671624c31dd6" />
- **Rasterize PDF → Images**  
  Convert all 39 pages into 300 dpi PIL images via `pdf2image` for crisp OCR input.

- **Full-page OCR (Arabic + English)**  
  Run `pytesseract.image_to_string(img, lang="ara+eng", config="--psm 1")` on each page to grab every line of text in one pass.

- **Initial cleaning (`clean_text()`)**  
  - Drop lines of only digits (page numbers, artifacts)  
  - Translate Western “0–9” → Arabic “٠–٩” on Arabic text lines  
  - Fix stray ligature/unicode glitches  
  - Collapse extra spaces and trim whitespace

- **Strip boilerplate headers**  
  Remove every “وزارة التجارة Ministry of Commerce” so only the unique article content remains.

- **Normalize punctuation & spacing**  
  Merge multiple spaces, correct misplaced periods/colons, and ensure each sentence breaks cleanly.

- **Filter out noise**  
  Drop blank lines, lone punctuation, and any OCR artifacts (e.g. stray dashes).

- **Assemble & export**  
  Bundle `{page, raw_ocr, cleaned_text}` into a DataFrame and save as `ocr_all_pages.csv` (UTF-8-SIG).

- **Sanity-check snippets**  
  Print the first 200 characters of each cleaned page to verify accurate extraction and cleaning.  
<br/>
<img width="2035" height="864" alt="image" src="https://github.com/user-attachments/assets/39d5d75a-fe69-439a-bd70-83f521d8269b" />
## 4. OCR Engine Integration 

<img width="1310" height="1310" alt="image" src="https://github.com/user-attachments/assets/9431feae-dbf5-4920-97fd-e41f7602abac" />

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
***Note that we only apply OCR on 9,587 files (images) due to the huge dataset size and the large amount of time reqiured to process the OCR.***


## 6. Evaluation  
- Compare extracted text to ground‑truth annotations (if available). :contentReference[oaicite:46]{index=46}  
- Calculate evaluation metrics: Character Error Rate (CER) and Word Error Rate (WER). :contentReference[oaicite:47]{index=47}  
- Display examples of correct and incorrect outputs. :contentReference[oaicite:48]{index=48}  

## 7. Visualization & Examples  
- Show side‑by‑side comparisons of original image, processed image, and extracted text. :contentReference[oaicite:49]{index=49}  
- Highlight detected text regions with bounding boxes. :contentReference[oaicite:50]{index=50}  
- Display sample errors with explanations. :contentReference[oaicite:51]{index=51}  

## 8. References  
- Dataset used [1]:https://www.kaggle.com/datasets/robikscube/textocr-text-extraction-from-images-dataset?select=train_val_images  
- Hugging Face TrOCR. :contentReference[oaicite:53]{index=53}  
- EasyOCR. :contentReference[oaicite:54]{index=54}  
- PaddleOCR. :contentReference[oaicite:55]{index=55}


## Project Structure

```
task6/
├── RegulationsAPIs.pdf # 39-page Arabic PDF law document
├── RegulationsAPIs_text.csv # text extracted by PDF API
├── ocr_output_raw.csv # raw OCR results (per page)
├── ocr_all_pages.csv # cleaned OCR results (per page)
├── extracted_text.csv # combined text snippets
├── final_output.csv # post-processed, ready for analysis
├── final_output.json # final_output in JSON format
├── Text_Extraction_from_Images.ipynb # notebook: PDF→text API extraction
└── Text_Extraction_OCR.ipynb # notebook: OCR & cleanup pipeline
```


## How to Run

1. **Open & run “Text_Extraction_from_Images.ipynb”**  
   - Converts `RegulationsAPIs.pdf` → `RegulationsAPIs_text.csv` via the PDF text API.  
   - Generates `extracted_text.csv`.

2. **Open & run “Text_Extraction_OCR.ipynb”**  
   - Loads `RegulationsAPIs.pdf` pages as images.  
   - Runs Tesseract OCR → outputs `ocr_output_raw.csv`.  
   - Cleans & normalizes → writes `ocr_all_pages.csv`.  
   - Merges everything into `final_output.csv` and `final_output.json`.

3. **Inspect outputs**  
   - `extracted_text.csv` for API-extracted text.  
   - `ocr_output_raw.csv` / `ocr_all_pages.csv` for OCR results.  
   - `final_output.*` for your downstream tasks (NER, search, etc.).
