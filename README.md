# Langchain RAG Tutorial Offline

**100% open-source, fully offline RAG template** using **LangChain + Chroma + BGE (BAAI/bge-base-en-v1.5)** for embeddings and **Mistral 7B Instruct** via **Ollama** for answers.  
Tested on **Windows 11**, **Python 3.11**, and safe for **commercial use** (Apache-2.0 / MIT licenses).

## Features
- 🔌 **Offline** after one-time downloads (or use the full offline prep below)
- 🧠 **Embeddings:** BGE base v1.5 (Apache-2.0)
- 💬 **LLM:** Mistral 7B Instruct (Apache-2.0) via Ollama runtime
- 📚 **Vector DB:** Chroma (local)
- 🧱 **LangChain**-based RAG pipeline
- 🪟 **Windows-friendly**, Python 3.11

## Stack
- **Embeddings:** `BAAI/bge-base-en-v1.5` (Hugging Face / sentence-transformers)
- **Vector store:** `chromadb`
- **RAG glue:** `langchain`, `langchain-huggingface`, `langchain-ollama`
- **Runtime:** `ollama` (local)
- **Document loader:** `unstructured[md]`

## Project structure
```
.
├─ create_database.py      # builds Chroma DB with BGE embeddings
├─ query_data.py           # queries Chroma, calls Mistral via Ollama
├─ requirements.txt
├─ data/
│  └─ books/               # put your .md files here
├─ chroma/                 # generated vector DB (created on first run)
└─ models/
   └─ bge-base-en-v1.5/    # optional local copy of the embedding model
```

## Prerequisites
- **Windows 11**, **Python 3.11 (64-bit)**
- **Microsoft Visual C++ Build Tools** (helps with `onnxruntime` on Windows)
- **Ollama** (Windows installer) – to run Mistral locally

## Quick Start (online once, then offline)
> If you need a **fully offline** setup from the start, skip to the next section.

1) Create & activate a venv (PowerShell):
```powershell
python -m venv venv
venv\Scripts\activate
pip install --upgrade pip
```

2) Install dependencies:
```powershell
pip install -r requirements.txt
pip install "unstructured[md]"
pip install -U langchain-huggingface langchain-ollama
```

3) (Optional) Download BGE locally for offline use:
```powershell
pip install -U sentence-transformers
mkdir models
python -c "from sentence_transformers import SentenceTransformer as S; S('BAAI/bge-base-en-v1.5').save(r'models\bge-base-en-v1.5')"
```

4) Build your vector DB:
```powershell
$env:EMBED_MODEL="models\bge-base-en-v1.5"   # or omit to download on first run
python create_database.py
```

5) Pull the LLM (one time, online), then you’re offline:
```powershell
ollama pull mistral:7b-instruct
```

6) Query (offline works after step 5):
```powershell
$env:HF_HUB_OFFLINE="1"                    # optional: force HF offline
$env:ANONYMIZED_TELEMETRY="False"          # optional: silence Chroma telemetry
python query_data.py "How does Alice meet the Mad Hatter?" --k 5 --min_score 0.2
```

## Fully Offline Setup

### A) Build an offline wheelhouse (on any **online** Windows 11 + Python 3.11 machine)
1) Create a clean venv and download wheels:
```powershell
python -m venv venv && venv\Scripts\activate
pip install --upgrade pip
mkdir wheelhouse
pip download -r requirements.txt -d wheelhouse
pip download -d wheelhouse "unstructured[md]" langchain-huggingface langchain-ollama
```

> If PyTorch wheels aren’t pulled automatically by dependencies, follow PyTorch’s site to fetch the correct wheel and add it to `wheelhouse`.

2) Prepare **local embeddings** folder:
```powershell
pip install -U sentence-transformers
mkdir models
python -c "from sentence_transformers import SentenceTransformer as S; S('BAAI/bge-base-en-v1.5').save(r'models\bge-base-en-v1.5')"
```

3) Prepare **Ollama model files**:
```powershell
ollama pull mistral:7b-instruct
ollama list   # confirm it's present
```
Copy the Ollama model store to transfer later (typically under `%USERPROFILE%\.ollama\models\` → includes `blobs\` and `manifests\`).

4) Move these to your **offline** machine:
- `wheelhouse\` (all wheels)
- `models\bge-base-en-v1.5\` (embedding model)
- Ollama’s `models\blobs\` and `models\manifests\` folders

### B) Install **offline** on the target machine
1) Create venv & install from wheelhouse:
```powershell
python -m venv venv && venv\Scripts\activate
pip install --no-index --find-links=wheelhouse -r requirements.txt
pip install --no-index --find-links=wheelhouse "unstructured[md]" langchain-huggingface langchain-ollama
```

2) Place the **embedding model**:
```
<project>\models\bge-base-en-v1.5\
```

3) Place the **Ollama model store**:
- Default: `%USERPROFILE%\.ollama\models\`
- Or set a custom path (e.g., `D:\ollama-models`) via env var:
  - System Properties → Environment Variables → New:
    - Name: `OLLAMA_MODELS`
    - Value: `D:\ollama-models`
  - Put `blobs\` and `manifests\` inside that folder.
  - Restart the Ollama service (or `ollama serve`).

4) Set env vars and run:
```powershell
$env:EMBED_MODEL="models\bge-base-en-v1.5"
$env:HF_HUB_OFFLINE="1"
$env:ANONYMIZED_TELEMETRY="False"

python create_database.py

ollama list               # should show mistral:7b-instruct
python query_data.py "How does Alice meet the Mad Hatter?" --k 5 --min_score 0.2
```

## Usage
1) Put Markdown files in `data\books\` (e.g., `alice_in_wonderland.md`)
2) Build the DB:
```powershell
python create_database.py
```
3) Ask questions:
```powershell
python query_data.py "How does Alice meet the Mad Hatter?"
```

**Flags** (query):
- `--k` *(int)*: top-k chunks to retrieve (default 3)
- `--min_score` *(float 0–1)*: minimum relevance score (default 0.7)
- `--model` *(str)*: Ollama model tag (default `mistral:7b-instruct`)
- `--temperature` *(float)*: LLM creativity (default 0.2)

## Configuration
Environment variables:
- `EMBED_MODEL` → path or name (e.g., `models\bge-base-en-v1.5`)
- `HF_HUB_OFFLINE=1` → force Hugging Face offline
- `ANONYMIZED_TELEMETRY=False` → silence Chroma telemetry
- `OLLAMA_MODELS` → custom folder for Ollama model store
- `OLLAMA_HOST` → point to a remote/local Ollama server

## Troubleshooting
- **“Unable to find matching results.”**
  - Ensure `data\books\` contains `.md` files.
  - Rebuild DB with the **same** `EMBED_MODEL` used at query time.
  - Lower threshold: `--min_score 0.2` while testing.
- **Chroma telemetry spam** → set `ANONYMIZED_TELEMETRY=False`.
- **Ollama not found / not running**
  - Install Ollama; ensure the service is running or run `ollama serve`.
  - `ollama list` should show `mistral:7b-instruct`.
- **ONNXRuntime issues on Windows**
  - Install **Microsoft C++ Build Tools**.
- **Deprecation warnings**
  - Use `langchain-huggingface` and `langchain-ollama` imports (already in code).

## Licenses
- **Template code:** choose your own (e.g., MIT).
- **Models:**
  - BAAI/bge-base-en-v1.5 → Apache-2.0
  - Mistral 7B Instruct → Apache-2.0
Both allow **commercial use**.

## Credits
LangChain, Chroma, Sentence-Transformers, Hugging Face, Mistral, Ollama, BAAI.
