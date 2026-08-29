# EduMentor AI v2.0 — PDF-Powered RAG Learning Assistant

> **Version 2.0** — Complete rebuild with real PDF knowledge base, multimodal extraction (text + tables + images), Gemini 3.5 Flash, and an MRR-led evaluation framework.

EduMentor AI v2.0 is a production-grade **Retrieval-Augmented Generation (RAG)** educational assistant. It ingests your PDF textbooks and lecture notes, extracts all content types (paragraphs, tables, diagrams), and answers student questions grounded exclusively in those documents — with citations, configurable generation parameters, and a comprehensive evaluation pipeline.

**Kaggle Notebook**: [edumentor-ai on Kaggle](https://www.kaggle.com/code/shivakantkurmi/edumentor-ai) //only have Version 1 for version 2 check the github 

---

## What's New in v2.0

| Feature | v1.0 | v2.0 |
|---------|------|------|
| Knowledge source | Static CSV dataset | **Real PDF files** |
| Content extracted | Text only | **Text + Tables + Images** |
| Image understanding | None | **Gemini 3.5 Flash Vision** |
| LLM model | `gemini-1.5-flash` (deprecated) | **`gemini-3.5-flash`** |
| SDK | `google-generativeai` (legacy) | **`google-genai`** (official current SDK) |
| Evaluation | Basic | **MRR + Hit Rate + ROUGE-L + BLEU-1 + Faithfulness + Bias/Fairness** |
| Primary eval metric | None | **MRR (Mean Reciprocal Rank)** |
| Context window | 1M tokens | 1M tokens |

---

## Architecture Overview

```
Your PDFs
    │
    ▼
┌─────────────────────────────────────────────────────────┐
│  Phase 1 — Multimodal PDF Extraction (Steps 1–7)        │
│                                                         │
│  pdfplumber ──► Text paragraphs (page-by-page)          │
│             ──► Tables (structured rows)                │
│  PyMuPDF   ──► Images (embedded rasterisation)          │
│                      │                                  │
│             Gemini 3.5 Flash Vision                     │
│                      ▼                                  │
│             Image Descriptions                          │
│                      │                                  │
│  all-MiniLM-L6-v2 Embeddings (384-dim)                  │
│                      │                                  │
│         FAISS Vector Store                              │
│         Similarity Search (Top-K)                       │
└─────────────────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────────────────┐
│  Phase 2 — LLM Generation (Steps 8–12)                  │
│                                                         │
│  RAG Prompt (system + context + question)               │
│         │                                               │
│  gemini-3.5-flash (google-genai SDK)                    │
│  temperature / max_tokens / top_p / top_k               │
│         │                                               │
│  Answer + Citations (filename, page, content_type)      │
└─────────────────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────────────────┐
│  Phase 3 — Evaluation Framework (Steps 13–18)           │
│                                                         │
│  MRR  ◄── Primary retrieval metric                      │
│  Hit Rate @K / Precision @K                             │
│  ROUGE-L / BLEU-1 (answer quality)                      │
│  Faithfulness (lexical + LLM judge)                     │
│  Bias / Fairness (lexical + LLM reviewer)               │
└─────────────────────────────────────────────────────────┘
```

---

## Model Stack & Justification

### Embedding — `sentence-transformers/all-MiniLM-L6-v2`
- **Dimensions**: 384 | **Context**: 512 tokens
- **Why**: 5x faster than `all-mpnet-base-v2`; STS Spearman correlation ~0.88; CPU-only; free
- **Chunk size**: 400 chars (~80–110 tokens) — safely within the 512-token limit

### Vector Store — FAISS
- **Why**: In-memory, zero server overhead, exact cosine search, native LangChain integration

### LLM — `gemini-3.5-flash` via `google-genai` SDK
- **Why**: Modern generation model with optimal latency, low cost, native multimodality, and high quality reasoning
- **Context window**: 1,000,000 tokens — only ~0.05% used per call at K=5
- **SDK**: `google-genai` (`from google import genai`)
- **Vision**: Same model describes embedded PDF images (diagrams, charts, equations)

### PDF Extraction Libraries
| Library | Role |
|---------|------|
| `pdfplumber` | Text paragraphs + table cell detection |
| `PyMuPDF` (`fitz`) | Embedded image rasterisation |
| `Gemini 3.5 Flash` Vision | Image-to-text searchable descriptions |

### Primary Evaluation Metric — MRR
**Mean Reciprocal Rank** measures how quickly the RAG system surfaces the correct source:

```
MRR = (1/N) * SUM( 1 / rank_of_first_relevant_result )
```

- MRR = 1.0 means the correct source is always ranked first (perfect retrieval)
- MRR = 0.5 means the correct source is on average at rank 2
- Composite score weights: **MRR 40%** + Hit Rate 20% + ROUGE-L 25% + BLEU-1 15%

---

## Notebook Structure (18 Steps)

### Phase 1 — PDF / RAG Pipeline
| Step | Description |
|------|-------------|
| 1 | PDF Upload API — `load_pdfs(paths)` for local or Kaggle dataset PDFs |
| 2 | Extraction — text (`pdfplumber`), tables (`pdfplumber`), images (`PyMuPDF` + `Gemini 3.5 Flash` Vision) |
| 3 | Chunking — 400-char / 80-overlap; tables and image descriptions kept atomic |
| 4 | Embedding — `all-MiniLM-L6-v2`, 384-dim, L2-normalised |
| 5 | FAISS vector store — built from all chunks, saved to disk |
| 6 | Similarity search — cosine similarity across text, tables, and images |
| 7 | Configurable Top-K — `retrieve_context(query, top_k=3)` |

### Phase 2 — LLM Integration
| Step | Description |
|------|-------------|
| 8 | `gemini-3.5-flash` via `google-genai` SDK |
| 9 | RAG prompt — cite sources, use only context, admit uncertainty, adapt to student level |
| 10 | Temperature presets — 0.0 (factual QA) to 0.9 (creative/quiz) |
| 11 | Full parameter control — `max_output_tokens`, `top_p`, `top_k`, `stop_sequences` |
| 12 | Citation extraction — filename, page, content_type, relevance score, snippet |

### Phase 3 — Evaluation
| Step | Description |
|------|-------------|
| 13 | Gold-standard test Q+A dataset (customise to match your PDFs) |
| 14 | K-sweep — run all questions at K in {1, 2, 3, 5} |
| 15 | **MRR** (primary) + Hit Rate @K + Precision @K |
| 16 | ROUGE-L + BLEU-1 answer quality |
| 17 | Faithfulness — lexical token overlap + Gemini LLM self-judge (0.0–1.0) |
| 18 | Bias/Fairness — lexical scan + Gemini fairness reviewer (0.0–1.0) |

---

## Quick Start

### Option 1: Kaggle (Recommended)

1. Fork the notebook: [Kaggle Notebook](https://www.kaggle.com/code/shivakantkurmi/edumentor-ai) → **Copy & Edit**
2. Add your API key in Kaggle Secrets:
   - Label: `GEMINI_API_KEY`
   - Value: your key from [Google AI Studio](https://aistudio.google.com/)
3. Add your PDF files as a Kaggle Dataset input
4. Set `PDF_PATHS` in Step 1:
   ```python
   PDF_PATHS = [
       "/kaggle/input/your-dataset/textbook.pdf",
       "/kaggle/input/your-dataset/lecture_slides.pdf",
   ]
   ```
5. Run All

### Option 2: Local

```bash
git clone https://github.com/shivakantkurmi/Edumentor-AI.git
cd Edumentor-AI
pip install google-genai langchain langchain-community langchain-huggingface \
            faiss-cpu pdfplumber pymupdf Pillow rouge-score nltk pandas matplotlib seaborn
```

Set your API key:
```bash
# Linux / Mac
export GEMINI_API_KEY="your-api-key"

# Windows PowerShell
$env:GEMINI_API_KEY = "your-api-key"
```

Run:
```bash
jupyter notebook edumentor-ai.ipynb
```

---

## Key API Changes (v1 to v2)

```python
# ── v1 (DEPRECATED) ─────────────────────────────────────────────────────────
import google.generativeai as genai
genai.configure(api_key=os.getenv("GOOGLE_API_KEY"))
model = genai.GenerativeModel("gemini-1.5-flash")
response = model.generate_content(prompt, generation_config=GenerationConfig(...))

# ── v2 (CURRENT) ────────────────────────────────────────────────────────────
from google import genai
from google.genai import types as genai_types
client = genai.Client(api_key=os.getenv("GEMINI_API_KEY"))
response = client.models.generate_content(
    model="gemini-3.5-flash",
    contents=prompt,
    config=genai_types.GenerateContentConfig(temperature=0.3, max_output_tokens=512),
)
```

---

## Evaluation Dashboard Output

The final dashboard cell executes the evaluation and saves `mrr_dashboard.png`:

1. **MRR bar chart** — primary metric, optimal K highlighted in green
2. **All retrieval metrics** — grouped bars (MRR / Hit Rate @K / Precision @K)
3. **Composite score** — MRR 40% + Hit Rate 20% + ROUGE-L 25% + BLEU-1 15%

Plus a complete tabular summary of retrieval, answer quality, faithfulness, and fairness scores.

---

## License

[MIT License](LICENSE)

---

Built by **Shivakant Kurmi** | EduMentor AI v2.0
