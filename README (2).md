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
