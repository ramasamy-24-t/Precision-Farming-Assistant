# Agricultural RAG Assistant

A retrieval-augmented generation (RAG) system built for the **36 Hours Hackathon** that answers crop advisory questions.

The project ingests agricultural PDFs, extracts text/tables/images, enriches chunks with GPT-4o-mini summaries, stores embeddings in ChromaDB, and serves answers through an interactive notebook chat loop and a Flask REST API.

## Features

- **Multimodal PDF ingestion** — Parses PDFs with `unstructured` (hi-res strategy) to capture text, tables, and images
- **Smart chunking** — Title-based chunking with AI-enhanced summaries for mixed-content sections
- **Vector search** — OpenAI `text-embedding-3-small` embeddings stored in ChromaDB (cosine similarity)
- **Multi-query retrieval** — Reciprocal Rank Fusion (RRF) across LLM-generated query paraphrases
- **Conversational RAG** — Multi-turn chat with conversation history support
- **REST API** — Flask endpoints for health checks and question answering

## Architecture

```
PDFs (Docs/)
  → unstructured hi-res partition
  → title-based chunking
  → GPT-4o-mini chunk enrichment
  → OpenAI embeddings → ChromaDB (dbv1/chroma_db)
  → multi-query RRF retrieval
  → GPT-4o-mini answer generation
  → interactive chat OR Flask /ask API
```

## Prerequisites

- Python 3.11+
- [OpenAI API key](https://platform.openai.com/api-keys)
- Jupyter Notebook or JupyterLab
- Source PDFs placed in the `Docs/` folder

## Installation

1. Clone or download this repository.

2. Create and activate a virtual environment:

```powershell
cd "d:\Projects\36 Hours Hackathon"
python -m venv venv
.\venv\Scripts\Activate.ps1
```

3. Install Jupyter (if not already installed):

```powershell
pip install jupyter
```

4. Open the notebook:

```powershell
jupyter notebook RAG.ipynb
```

5. Run the dependency installation cells at the top of the notebook:

```
unstructured[all-docs]
unstructured-inference
langchain-chroma
langchain
langchain-community
langchain-openai
python-dotenv
flask
```

> **Note:** `unstructured[all-docs]` pulls in heavy dependencies (PyTorch, OpenCV, spaCy, etc.) and may take several minutes to install.

## Configuration

Create a `.env` file in the project root with your OpenAI API key:

```env
OPENAI_API_KEY=your_openai_api_key_here
```

The notebook loads this automatically via `python-dotenv`. Never commit your `.env` file — it is listed in `.gitignore`.

## Usage

### 1. Build the vector store (first run)

Run the notebook cells in order:

1. Install dependencies
2. Import libraries and load environment variables
3. Partition PDFs — `partition_all_pdfs()`
4. Chunk documents — `create_chunks_by_title()`
5. Summarize and embed — `summarise_chunks()` then `create_vector_store()`

The vector store is persisted to `dbv1/chroma_db/` and can be reused on subsequent runs without re-processing PDFs.

> **Cost warning:** `summarise_chunks()` makes one GPT-4o-mini call per chunk. A full PDF like `pp_rabi.pdf` (~326 chunks) can take significant time and incur OpenAI API costs.

### 2. Query interactively

Use the `rag_multi_query_chat()` function in the notebook to ask questions with conversation history:

```python
questions = ["Which PBW variety is best for south western Punjab?"]
answers, history = rag_multi_query_chat(questions)
print(answers[0])
```

### 3. Start the Flask API

Run the final notebook cell to start the API server on port 5000:

```
http://127.0.0.1:5000
```

#### Health check

```bash
curl http://127.0.0.1:5000/health
```

Response:

```json
{"status": "ok"}
```

#### Ask a question

```bash
curl -X POST http://127.0.0.1:5000/ask \
  -H "Content-Type: application/json" \
  -d "{\"question\": \"Which PBW variety is best for south western Punjab?\"}"
```

Response:

```json
{"answer": "..."}
```

If the `question` field is missing or empty, the API returns a `400` error:

```json
{"error": "question is required"}
```

## Project Structure

```
.
├── RAG.ipynb          # Main application (ingestion, RAG pipeline, Flask API)
├── Docs/              # Source PDF documents
│   └── pp_rabi.pdf    # PAU Package of Practices, Rabi 2024-25
├── dbv1/              # Persisted ChromaDB vector store (gitignored)
│   └── chroma_db/
├── rag_results.json   # Sample retrieval export (gitignored)
├── .env               # OpenAI API key (gitignored — create your own)
└── .gitignore
```

## Data Source

The included document is the PAU *Package of Practices for Crops of Punjab, Rabi 2024-25* (`Docs/pp_rabi.pdf`), covering topics such as:

- Wheat variety recommendations by region and sowing window
- Sowing dates, seed rates, and irrigation practices
- Pest and disease management for Punjab crops

Add more PDFs to the `Docs/` folder and re-run the ingestion pipeline to expand the knowledge base.

## Tech Stack

| Component | Technology |
|-----------|------------|
| PDF parsing | `unstructured` |
| LLM | OpenAI GPT-4o-mini |
| Embeddings | OpenAI `text-embedding-3-small` |
| Vector database | ChromaDB |
| Orchestration | LangChain |
| API | Flask |
| Environment | `python-dotenv` |

## Limitations

- All application logic lives in a single Jupyter notebook — there is no separate production deployment setup
- No web frontend; the API is intended for curl, Postman, or programmatic clients
- Initial PDF processing and chunk summarization is slow and incurs OpenAI API costs
- Answers are limited to the content of ingested PDFs

## License

This is a hackathon project. Verify licensing for PAU source documents before redistribution.
