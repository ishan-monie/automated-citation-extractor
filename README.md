
# Citation Extraction Pipeline (Multilingual Humanities Texts)

This repository contains a full NLP pipeline for extracting and reconstructing inline citations from scanned, multilingual academic texts. The system is designed for structurally inconsistent sources like Assyriological publications in English and French.

---

## 📁 Project Structure

```
.
├── combined_text.txt              # Raw OCR text input
├── sentences.csv                 # Sentence-level fragments from OCR text
├── outputs/
│   ├── candidates.json           # Inline citation spans predicted by LLaMA
│   ├── reconstructed.csv        # Fully structured citation metadata
│   └── zotero_export.bib        # Final export file for bibliographic tools
├── extract_sentences.py         # Cleans and segments raw text into sentences
├── extract_candidates_llama.py  # Prompts LLaMA for citation candidate spans
├── ner_bilstm_train.py          # Trains the BiLSTM + NER tagger
├── ner_bilstm_predict.py        # Applies BIO tags to complete citations
└── README.md
```

---

## 🧠 Step 1: Sentence Segmentation

We begin with a raw OCR `.txt` file. `extract_sentences.py` performs:

- Hyphen correction and line flattening
- Regex-based sentence splitting
- Filtering short/noisy segments

**Output:** `sentences.csv` — one sentence per row.

---

## 🤖 Step 2: Citation Candidate Extraction (LLaMA 3)

The script `extract_candidates_llama.py` prompts `Meta-LLaMA-3-8B-Instruct-iMatrix` with few-shot examples to detect citation-like spans.

**Model:**
- `meta-llama/Meta-LLaMA-3-8B-Instruct-iMatrix`
- Few-shot, instruction-following format

**Parameters:**
- `temperature = 0.0`
- `max_new_tokens = 100`
- `do_sample = False`

**Prompt Format:**
```
Extract all citation references from the following paragraph.
Return each citation as a separate line.

Text:
<Your Sentence Here>
```

**Output:** JSON list of sentences + extracted citations.

---

## 🧩 Step 3: Full Citation Reconstruction (BiLSTM + NER)

### Model Architecture:
- **Embedding layer:** 100 dimensions, trained on the OCR corpus
- **BiLSTM:** 64 hidden units (bidirectional)
- **Dense output:** TimeDistributed layer with softmax

### Training:
- **Data:** 1,200 manually labeled citation fragments
- **Tag format:** BIO (e.g., B-AUTHOR, I-TITLE)
- **Loss:** Categorical Cross-Entropy
- **Optimizer:** Adam
- **Epochs:** 10–15 with early stopping

**Output:** Tagged token sequences are reassembled into structured citation records (CSV or BibTeX).

---

## 📊 Evaluation

### Step 2 (LLaMA):
- **English Accuracy:** 81%
- **French Accuracy:** 79%

### Step 3 (BiLSTM+NER):
- **English Accuracy:** 96.5%
- **French Accuracy:** 89.4%

---

## 📤 Export to Zotero

Final output can be exported as:

- `.csv`: structured table of fields
- `.bib`: for LaTeX use
- `.json`: Zotero import format

---

## 📚 Citation Format Fields

Each final record contains:

- `Author`
- `Title`
- `Journal / Book`
- `Volume`
- `Year`
- `Pages`
- `Notes`

---

## 🛠 Dependencies

- Python ≥ 3.8
- `transformers`
- `pandas`
- `torch`
- `scikit-learn`
- (Optional) GPU for inference

---

## 📎 Notes

- French citation accuracy is slightly lower due to abbreviation density, OCR noise, and variable typographic norms.
- Embeddings were trained from scratch rather than using GloVe to better capture domain-specific vocabulary.

---

For questions or contributions, feel free to open an issue or reach out.
