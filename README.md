# Improved Yoruba Traineddata for Tesseract OCR

### 📝 Purpose

For decades, progress in Yoruba digital text processing and preservation has been significantly slowed by the lack of accurate Optical Character Recognition (OCR) resources. Researchers, linguists, and historians have faced substantial barriers when scanning Yoruba documents — particularly due to poor OCR support for complex characters and diacritics. Many are forced to rely on expensive commercial services just to digitize Yoruba texts with acceptable accuracy.

This repository offers a powerful, open-source alternative: an **improved `yor.traineddata`** for Tesseract OCR, designed specifically to enhance the accuracy of Yoruba text recognition and support long-term development of Yoruba in the digital world.

---

### ✅ Improvements Over the Official Model

The official Yoruba OCR model included in Tesseract’s `tessdata_best` suffers from **critical limitations in recognizing combined diacritic characters**, especially when both an underdot and a tonal accent are present.

#### Observed problems in the official model:
- ✅ Scans basic tonal characters (e.g., `È`, `Ó`)
- ✅ Scans underdot characters (e.g., `Ṣ`, `Ẹ`)
- ❌ Fails or produces poor accuracy for combined characters with both underdot and tone (e.g., `Ẹ́`, `Ọ̀`)

#### Root cause:
Our research suggests that the original model was likely trained with **unnormalized (non-NFC)** text. This causes issues during `.box` file generation, where decomposed character sequences are not visually or structurally aligned in OCR training. As a result, the model fails to recognize composite forms reliably.

#### Our solution:
We designed a **custom normalization and character composition system**:
- All training texts were normalized to **NFC (Normalization Form C)**.
- A script was written to **automatically reconstruct complex character compositions** in `.box` files.
- Each base character was merged with its respective diacritic and/or underdot by **adjusting bounding box positions** (top, bottom, left, right) to fully encapsulate the combined glyph.
- This correction was applied to **~50,000 `.box` files** — ensuring every training sample accurately represents the full character.

As a result, this model:
- Accurately recognizes **all tonal vowels and consonants**, including `Ẹ́`, `Ọ̀`, `Á`, `Ǹ`, etc.
- Performs significantly better on **historical, scanned, or degraded Yoruba documents**.
- Provides a valuable resource for the long-term digital preservation of Yoruba literature.

---

### 🧠 Training Methodology

This model was trained using [Tesseract OCR](https://github.com/tesseract-ocr/tesseract) v5 with the LSTM engine.

- **Training corpus**: 50,000+ individual text-image pairs (`.tif` + `.gt.txt`)
- **Corpus content**: Each sample contains a unique sentence with diverse coverage of:
  - All Yoruba vowel/consonant combinations
  - Accented vowels (acute, grave)
  - Underdot consonants
  - Combined forms with both diacritics and underdots
  - Special characters and punctuation
- **Model initialization**: Although we used `yor.traineddata` from `tessdata_best` as a start model, the training was **not a fine-tune** — we **retrained from scratch**, using it only for bootstrapping glyph shape data.
- **Normalization**: All input ground truth was passed through strict Unicode NFC normalization before generating training files.
- **Training iteration**: 60,000 iterations

#### 🚀 Training command:
```bash
gmake -f Makefile training \
  MODEL_NAME=yor \
  START_MODEL=yor \
  TESSDATA=/usr/local/share/tessdata \
  MAX_ITERATIONS=60000 \
  NORM_MODE=1
