# Assignment 3 - RAG Machine Learning Knowledge Assistant

## Objective

Build a Retrieval-Augmented Generation (RAG) application that retrieves relevant passages from an authorized machine-learning book PDF and uses Gemini to answer questions grounded in those passages.

## Files

- `rag_app.py` - PDF extraction, chunking, Gemini embeddings, Chroma vector search, and interactive question answering.
- `rag_knowledge_assistant_report.md` - Assignment overview, setup, and usage instructions.
- `requirements.txt` - Required Python packages.
- `ml_knowledge.pdf` - Add your authorized, text-searchable ML book PDF here. This input file is not included. An example source is the [ISL with Python PDF from the authors' official download page](https://hastie.su.domains/ISLP/ISLP_website.pdf.download.html).

## Technology

- **PDF extraction:** pypdf
- **Embeddings:** Gemini `gemini-embedding-001`, using `RETRIEVAL_DOCUMENT` and `RETRIEVAL_QUERY` task types.
- **Vector store:** Persistent local Chroma collection with cosine-distance search.
- **Answer generation:** Gemini `gemini-3.5-flash-lite` by default. Override with the `GEMINI_MODEL` environment variable if needed.

## Preprocessing and Indexing

The app extracts text page by page and prints progress every 25 pages. It reports the number of pages, extracted character count, chunk count, pages with no extractable text, and pages with unusually little text. Scanned PDFs without an OCR text layer are not silently treated as usable; run OCR first.

Text is split into approximately 3,500-character chunks with a 450-character overlap. These are practical approximations of the assignment's suggested token ranges; actual token counts vary by content. Page number and chunk ID are retained as metadata for citations.

Gemini embeddings are generated in batches. Chroma stores the vectors, text chunks, source filename, page number, and chunk index in `.chroma_db/`. The collection name is derived from the PDF's SHA-256 hash, so a changed PDF gets a separate index. The index and source PDF are excluded from Git by `.gitignore`.

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

Keep the API key in your environment; do not commit it to source control. Download an authorized ML book PDF and save it in this folder as `ml_knowledge.pdf` to use the commands below unchanged. If you keep the downloaded filename or save it elsewhere, pass its path with `--pdf` when running the app.

## Run the Assistant

Inspect the PDF and report text-quality issues without an API key:

```powershell
python rag_app.py --inspect-only
```

If the PDF has a different name, pass that filename:

```powershell
python rag_app.py --pdf ".\ISLP_website.pdf" --inspect-only
```

Start interactive question answering:

```powershell
python rag_app.py
```

Ask a single question:

```powershell
python rag_app.py --question "What is overfitting, and how can regularization help?" --top-k 5
```

For a differently named PDF, add `--pdf ".\ISLP_website.pdf"` to the command:

```powershell
python rag_app.py --pdf ".\ISLP_website.pdf" --question "What is overfitting, and how can regularization help?" --top-k 5
```

Compare retrieved passages for different `k` values:

```powershell
python rag_app.py --question "How does backpropagation train a neural network?" --compare-k
```

The app prints the retrieved source passages and page numbers before the generated answer. The generation instructions require answers to use retrieved excerpts only, cite pages, and say when the excerpts do not contain enough information. A citation check warns if a cited page was not among the retrieved passages or if no page citations were returned.

## Evaluation

Try several questions spanning different chapters and concepts. Run `--compare-k` for `k = 3, 5, 7`, then compare:

1. Whether the retrieved passages are relevant to the question.
2. Whether the answer is supported by those passages.
3. Whether the answer's page citations match the retrieved context.
4. Whether the assistant abstains when the retrieved passages do not answer the question.

To compare answer quality as well, run the same question separately with `--top-k 3`, `--top-k 5`, and `--top-k 7`.

Other useful test questions include "How does gradient descent update model parameters?" and "What is the difference between bagging and boosting?" Confirm that the selected book covers each topic.

Do not report sample answers or retrieval results until the app has been run with the chosen book PDF and a valid Gemini API key. The assignment brief PDF is not itself the ML knowledge base.

## Limitations

- Quality depends on the PDF's text layer, chunk size, retrieval quality, and model output.
- OCR is not built in.
- Gemini receives the text sent for embedding and generation. Use a PDF you are authorized to process and avoid uploading confidential material.
- A page citation is a traceability aid, not proof that every answer claim is correct; verify important answers against the cited source.
