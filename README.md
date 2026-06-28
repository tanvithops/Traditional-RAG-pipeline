# Traditional RAG Pipeline 📚🔍

This is my implementation of a **traditional RAG (Retrieval-Augmented Generation)** pipeline. I built it to actually understand how RAG works under the hood instead of just calling some black-box library function. So I went through the whole thing step by step — loading documents, chunking them, embedding them, storing them in a vector store, retrieving the relevant chunks for a question, and then feeding those chunks to an LLM to generate the final answer.

I learned this while following along with RAG tutorials, but I built my own version with my own data and my own structure.

> **Heads up:** Right now everything is done in **Jupyter notebooks**. I haven't refactored it into proper modular code (separate `.py` files / a clean `src/` package) yet — that's on my to-do list. The actual RAG logic is fully working though.

---

## What this project does

The whole idea of RAG is simple once it clicks:

1. You have a bunch of documents (PDFs, text files, whatever).
2. You can't just dump all of them into an LLM — too big, too expensive, and the model forgets stuff.
3. So instead, you store them as **embeddings** (vectors) in a vector store.
4. When a user asks a question, you find the **most relevant chunks** using similarity search.
5. You pass *only those chunks* to the LLM as context, and it answers based on them.

This keeps the answers grounded in my own documents instead of the model just making things up.

---

## How it works (the pipeline)

Here's the flow I built:

**1. Document Loading**
I load documents from two sources:
- PDFs (`data/pdf/`)
- Plain text files (`data/text_files/`)

**2. Chunking**
The documents get split into smaller chunks so each piece is a reasonable size for embedding and retrieval.

**3. Embeddings**
I use a sentence-transformer embedding model to turn each chunk into a 384-dimensional vector. (You can see in the output it generates embeddings with shape `(1, 384)` per text.)

**4. Vector Store**
All the embeddings get saved into a vector store (`vector_store/`) so I don't have to re-embed everything each time.

**5. Retrieval**
When I ask a question, the retriever pulls back the top-k most similar chunks. It also gives me a **similarity score**, **distance**, and a **rank** for each result so I can see *why* a chunk was retrieved.

**6. Generation (the LLM part)**
The retrieved chunks become the context. I build a prompt like:

```
Use the following context to answer the question concisely.

Context:
{context}

Question: {query}

Answer:
```

Then I send it to the LLM and get back a grounded answer. If no relevant context is found, it just says so instead of hallucinating.

---

## Tech Stack 🛠️

- **Python**
- **LangChain** (`langchain_groq`, `langchain_core`) — for the LLM integration
- **Groq API** — for fast LLM inference
- **Sentence-Transformers** — for generating the embeddings (384-dim model)
- **python-dotenv** — to keep my API key out of the code
- **Jupyter Notebook** — where the whole thing currently lives

---

## Project Structure 📁

```
trad rag/
├── data/
│   ├── pdf/                 # PDF documents to load
│   └── text_files/          # Text documents to load
├── vector_store/            # Saved embeddings live here
├── notebook/
│   ├── document.ipynb       # document loading / prep
│   └── pdf_loader.ipynb     # main RAG pipeline (loading → retrieval → generation)
├── .env                     # my API key (NOT pushed to git)
├── .gitignore
└── requirements.txt
```

---

## Getting Started 🚀

### 1. Clone the repo
```bash
git clone <your-repo-url>
cd "trad rag"
```

### 2. Create a virtual environment
```bash
python -m venv .venv
# Windows
.venv\Scripts\activate
# Mac/Linux
source .venv/bin/activate
```

### 3. Install the requirements
```bash
pip install -r requirements.txt
```

### 4. Set up your API key
This project uses **Groq** for the LLM. Get a free API key from [console.groq.com](https://console.groq.com), then create a file called `.env` in the project root:

```
GROQ_API_KEY=your_key_here
```

> ⚠️ Don't commit your `.env` file. Make sure it's listed in `.gitignore` so your key stays private.

### 5. Run the notebooks
Open the notebooks in `notebook/` and run the cells in order:
- first the document loading,
- then the embedding / vector store creation,
- then the retrieval + generation cells.

Ask it a question and it'll retrieve the relevant chunks from your documents and answer based on them.

---

## A small note on the LLM model

Groq retires older models from time to time, so if you ever get a "model has been decommissioned" error, just swap the model name in the `ChatGroq(...)` call for a currently-supported one (for example `llama-3.3-70b-versatile`). You can always check the live list at [console.groq.com/docs/models](https://console.groq.com/docs/models).

---

## What I'd like to do next 📝

- [ ] Refactor the notebook code into clean, modular `.py` files (a proper `src/` package)
- [ ] Add a simple UI (Streamlit) so I can ask questions without opening a notebook
- [ ] Try out different chunking strategies and compare retrieval quality
- [ ] Add evaluation to measure how good the answers actually are

---

## Why I built this

I wanted to *actually understand* RAG, not just use it. Building each piece myself — embeddings, vector store, retrieval, prompting — made the whole concept way clearer than just reading about it. This repo is me documenting that learning. 🙂
