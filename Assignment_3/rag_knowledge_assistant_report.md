# Assignment 3 - RAG Troubleshooting Runbook Assistant

## Objective

Build a Retrieval-Augmented Generation (RAG) application that retrieves relevant passages from a troubleshooting runbook PDF and uses Gemini to answer questions grounded in those passages.

## Files

- `rag_app.py` - PDF extraction, chunking, Gemini embeddings, Chroma vector search, and interactive question answering.
- `rag_knowledge_assistant_report.md` - Assignment overview, setup, and usage instructions.
- `requirements.txt` - Required Python packages.
- [`troubleshooting_runbook.pdf`](./troubleshooting_runbook.pdf) - Synthetic sample runbook used by default and included with this project.

## Technology

- **PDF extraction:** pypdf
- **Embeddings:** Gemini `gemini-embedding-001`, using `RETRIEVAL_DOCUMENT` and `RETRIEVAL_QUERY` task types.
- **Vector store:** Persistent local Chroma collection with cosine-distance search.
- **Answer generation:** Gemini `gemini-3.5-flash-lite` by default. Override with the `GEMINI_MODEL` environment variable if needed.

## Preprocessing and Indexing

The app extracts text page by page and prints progress every 25 pages. It reports the number of pages, extracted character count, chunk count, pages with no extractable text, and pages with unusually little text. Scanned PDFs without an OCR text layer are not silently treated as usable; run OCR first.

Text is split into approximately 3,500-character chunks with a 450-character overlap. These are practical approximations of the assignment's suggested token ranges; actual token counts vary by content. Page number and chunk ID are retained as metadata for citations.

Gemini embeddings are generated in batches. Chroma stores the vectors, text chunks, source filename, page number, and chunk index in `.chroma_db/`. The collection name is derived from the PDF's SHA-256 hash, so a changed PDF gets a separate index. The local index is excluded from Git by `.gitignore`; the included runbook PDF is synthetic demo data.

## Setup on Windows

Use Python 3.11-3.14. Chroma depends on `onnxruntime`, which does not currently have wheels for Python 3.15. With `uv`, install Python 3.14 and create the environment:

```powershell
uv python install 3.14
uv venv .venv --python 3.14
.\.venv\Scripts\Activate.ps1
uv pip install -r requirements.txt
$env:GEMINI_API_KEY = "your-Gemini-API-key"
```

If Python 3.14 is already installed and available through the Windows Python Launcher, you can instead use `py -3.14 -m venv .venv` and `python -m pip install -r requirements.txt`.

Keep the API key in your environment; do not commit it to source control. The app uses `troubleshooting_runbook.pdf` in this folder by default. If you use another runbook, pass its path with `--pdf` when running the app.

## Run the Assistant

Inspect the PDF and report text-quality issues without an API key:

```powershell
python rag_app.py --inspect-only
```

If you use a different runbook filename, pass its path:

```powershell
python rag_app.py --pdf ".\another_runbook.pdf" --inspect-only
```

Start interactive question answering:

```powershell
python rag_app.py
```

Ask a question about an incident in the runbook:

```powershell
python rag_app.py --question "What should I check for ERR-401 on payment-gateway?" --top-k 5
```

For a differently named PDF, add its path with `--pdf`:

```powershell
python rag_app.py --pdf ".\another_runbook.pdf" --question "What should I check for ERR-401 on payment-gateway?" --top-k 5
```

Compare retrieved passages for different `k` values:

```powershell
python rag_app.py --question "How should I investigate an ERR-500?" --compare-k
```

The app prints the retrieved source passages and page numbers before the generated answer. The generation instructions require answers to use retrieved excerpts only, cite pages, and say when the excerpts do not contain enough information. A citation check warns if a cited page was not among the retrieved passages or if no page citations were returned.

## Evaluation

Try several questions spanning different chapters and concepts. Run `--compare-k` for `k = 3, 5, 7`, then compare:

1. Whether the retrieved passages are relevant to the question.
2. Whether the answer is supported by those passages.
3. Whether the answer's page citations match the retrieved context.
4. Whether the assistant abstains when the retrieved passages do not answer the question.

To compare answer quality as well, run the same question separately with `--top-k 3`, `--top-k 5`, and `--top-k 7`.

Other useful test questions include "What are the first steps for a Sev1 incident?", "How should I investigate DB-1205 lock wait timeouts?", and "What diagnostics are listed for MEM-OOM?".

Do not report sample answers or retrieval results until the app has been run with the runbook PDF and a valid Gemini API key. The included runbook is synthetic demo data, not a source of live service status.

## Limitations

- Quality depends on the PDF's text layer, chunk size, retrieval quality, and model output.
- OCR is not built in.
- Gemini receives the text sent for embedding and generation. Use a PDF you are authorized to process and avoid uploading confidential material.
- A page citation is a traceability aid, not proof that every answer claim is correct; verify important answers against the cited source.
