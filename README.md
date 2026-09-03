# Mortgage Document RAG Chatbot

A Retrieval-Augmented Generation (RAG) chatbot that reads mortgage-related PDF documents — including scanned pages requiring OCR — and answers natural language questions about them, grounded strictly in the document's actual content, with source citations and a confidence score for every answer.

Built as part of an AI/ML externship focused on applying document intelligence and RAG techniques to real-world mortgage document processing.

## Problem Statement

Mortgage document packages are messy: multi-page PDFs mixing digital and scanned pages, inconsistent layouts across lenders, and dense fee/loan tables that are tedious to search manually. This project builds an end-to-end pipeline that lets a user simply ask a question in plain English (e.g. "What is the total loan amount?") and get back an accurate, cited answer pulled directly from the document — instead of manually scanning pages by hand.

## Features

- Handles both digital PDFs (text-based) and scanned PDFs (image-based, via OCR)
- Splits documents into chunks with page-level metadata (page number, source file, digital vs. OCR)
- Uses an open-source embedding model to convert text into searchable vectors
- Indexes and retrieves relevant chunks using semantic similarity search
- Uses grounded, few-shot prompting to keep answers factual and reduce hallucination
- Generates answers with an open-source LLM (no paid API required)
- Returns a confidence score and page-level source citations with every answer
- Interactive chat interface built with Gradio, deployable with a public shareable link

## Architecture

```
PDF Upload
    │
    ▼
Text Extraction (PyMuPDF)
    │
    ├── Digital page ──► extract text directly
    └── Scanned page ──► render as image ──► OCR (Tesseract) ──► extract text
    │
    ▼
Chunking + Metadata Tagging (LlamaIndex SentenceSplitter)
    │  (page number, source file, digital/OCR tag attached to every chunk)
    ▼
Embedding (sentence-transformers/all-MiniLM-L6-v2)
    │
    ▼
Vector Index (LlamaIndex VectorStoreIndex)
    │
    ▼
User Question ──► Embed Question ──► Retrieve Top-K Similar Chunks
    │
    ▼
Grounded Prompt Construction (context + citation rules + few-shot example)
    │
    ▼
Open-Source LLM Generation (TinyLlama-1.1B-Chat)
    │
    ▼
Answer + Sources + Confidence Score ──► Gradio Chat UI
```

## Component Table

| Component | Choice Used | Notes / Rationale |
|---|---|---|
| PDF Text Extraction | PyMuPDF (fitz) | Fast, accurate text + layout extraction for digital pages |
| OCR Engine | Tesseract (pytesseract) | Open-source, handles scanned/image-only pages |
| Chunking Strategy | LlamaIndex `SentenceSplitter` (chunk_size=300, overlap=50) | Keeps sentences intact, avoids cutting facts mid-sentence |
| Embedding Model | `sentence-transformers/all-MiniLM-L6-v2` | Small, fast, strong performance on short document text, fully open-source |
| Vector Store / Index | LlamaIndex `VectorStoreIndex` | Simple in-memory setup, sufficient for single/small document sets |
| Retriever | LlamaIndex `as_retriever(similarity_top_k=3)` | Returns top 3 most relevant chunks per question |
| LLM | `TinyLlama/TinyLlama-1.1B-Chat-v1.0` (Hugging Face) | Open-source, lightweight enough to run in Colab without a paid API |
| Prompt Strategy | Grounded + few-shot prompting | Forces answers from context only, includes a worked example, enforces exact citation format |
| UI | Gradio `Blocks` + `Chatbot` | Interactive chat interface, deployable via `share=True` |

## How It Works

1. **Ingestion** — the PDF is opened page by page. Pages with a real text layer are extracted directly; pages that come back empty (scanned images) are rendered as high-resolution images and passed through Tesseract OCR instead.
2. **Metadata tagging** — every page is wrapped as a `Document` with metadata: source filename, page number, and whether it was extracted digitally or via OCR.
3. **Chunking + embedding** — LlamaIndex splits each page into overlapping ~300-token chunks and embeds each chunk into a vector using the MiniLM embedding model.
4. **Indexing** — all chunk vectors are stored in a searchable `VectorStoreIndex`.
5. **Retrieval** — when a question comes in, it's embedded the same way, and the 3 most semantically similar chunks are retrieved.
6. **Prompting** — the retrieved chunks are formatted into a labeled context block and inserted into a strict, few-shot grounded prompt that instructs the model to answer only from that context, cite the page, and say "I don't have enough information" if the answer isn't present.
7. **Generation** — the prompt is passed to the open-source LLM, which generates a short, deterministic answer.
8. **Confidence + sources** — the average similarity score of the retrieved chunks is used as a simple confidence score, and each source page is listed alongside the answer.
9. **UI** — all of the above is wrapped in a Gradio chat interface, so a user can upload a PDF, ask questions, and see the answer, confidence, and sources for each turn.

## Setup / Installation

This project is designed to run in Google Colab.

```bash
pip install pymupdf pytesseract pillow sentence-transformers llama-index llama-index-embeddings-huggingface transformers accelerate gradio
apt-get install -y tesseract-ocr
```

## Usage

1. Open the notebook in Google Colab
2. Run all cells in order — you'll be prompted to upload a mortgage PDF
3. Once the index is built, run the final Gradio cell
4. Use the chat interface to ask questions about the uploaded document
5. `demo.launch(share=True)` also prints a public shareable link (valid ~72 hours) so others can try it without running the notebook themselves

## Example

**Question:** What is the total loan amount?

**Answer:** The total loan amount is $380,000. (Source: LenderFeesWorksheet.pdf, Page: 1)

**Confidence:** 0.87

**Sources:** Page 1 (digital, score: 0.87)

## Known Limitations

- Small open-source LLMs (like TinyLlama) are less capable than larger commercial models, so answer quality on complex or ambiguous questions can be inconsistent
- OCR accuracy drops on low-resolution scans, handwriting, or skewed pages
- The current pipeline is built and tested on a single PDF at a time; scaling to a large multi-document library would require persistent vector storage and metadata-based routing across files
- Runs noticeably faster with a GPU runtime enabled in Colab; on CPU, generation is slower

## Possible Next Steps

- Add a reranker to improve retrieval precision on similar/overlapping clauses
- Add structured table parsing for fee schedules instead of relying on plain text chunks
- Swap in a larger open-source LLM (e.g. Mistral-7B or Phi-3) for higher answer quality where compute allows
- Deploy permanently on Hugging Face Spaces with an in-browser PDF upload component
- Extend to support multiple documents at once with metadata-based query routing

## Tech Stack

Python, PyMuPDF, Tesseract OCR, Sentence-Transformers, LlamaIndex, Hugging Face Transformers, Gradio

