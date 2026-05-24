# 🤖 RAG Chatbot — Chat With Your Documents

A production-style Retrieval-Augmented Generation chatbot built with **Streamlit**, **LangChain**, **Groq**, and **FAISS**. Upload PDFs or text files, build a local vector index, and chat with your documents using a fast 70B-parameter LLM — with streaming responses, conversation memory, and source citations.

## ✨ Features

- 💬 **Streaming chat** — tokens appear in real time, no waiting for the full response
- 📄 **PDF & TXT support** — upload your own files or use the built-in sample knowledge base
- 🧠 **Conversation memory** — remembers the last 6 turns and rephrases follow-up questions into standalone queries
- 🔍 **Source citations** — every answer shows which documents it pulled from
- 🚀 **Fast inference** via Groq's `llama-3.3-70b-versatile`
- 🔒 **Local embeddings** — `all-MiniLM-L6-v2` runs on CPU, no embeddings API key needed
- 🎨 **Polished UI** — custom-styled chat bubbles, avatars, and navigation

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                       Streamlit Frontend                        │
│  Upload Page  →  Chat Page (streaming bubbles + source tags)    │
└─────────────────────────────┬───────────────────────────────────┘
                              │
              ┌───────────────┴────────────────┐
              ▼                                ▼
   ┌──────────────────────┐         ┌──────────────────────┐
   │   document_loader    │         │     rag_pipeline     │
   │  • PDF / TXT loaders │         │  • Vector store      │
   │  • Recursive chunker │         │  • Memory buffer     │
   │  • Overlap handling  │         │  • Question condense │
   └──────────┬───────────┘         │  • Retrieval + LLM   │
              │                     └──────────┬───────────┘
              ▼                                │
        ┌──────────┐                           ▼
        │  Chunks  │──── embed ────►   ┌─────────────────┐
        └──────────┘                   │  FAISS Index    │
                                       └────────┬────────┘
                                                │ top-k=3
                                                ▼
                                   ┌─────────────────────────┐
                                   │  Groq llama-3.3-70b     │
                                   │  (streaming response)   │
                                   └─────────────────────────┘
```

## 📁 Project Structure

```
.
├── app.py                  # Streamlit UI (upload + chat pages)
├── rag_pipeline.py         # RAGChatbot class — retrieval, memory, streaming
├── document_loader.py      # PDF/TXT loaders + recursive text splitter
├── config.py               # Model names, chunk sizes, API key loader
├── sample_docs/            # Built-in knowledge base (AI / ML / Python)
├── vectorstore_index/      # FAISS index (auto-generated)
├── requirements.txt
├── .env.example
└── README.md
```

## ⚙️ Configuration

All knobs live in `config.py`:

| Setting | Default | Description |
|---|---|---|
| `GROQ_MODEL` | `llama-3.3-70b-versatile` | LLM used for answer generation |
| `EMBEDDING_MODEL` | `all-MiniLM-L6-v2` | Local sentence-transformer for embeddings |
| `CHUNK_SIZE` | `1000` | Characters per document chunk |
| `CHUNK_OVERLAP` | `200` | Overlap between adjacent chunks |
| `MAX_CONTEXT_TURNS` | `6` | Conversation history window |
| `RETRIEVER_K` | `3` | Number of chunks retrieved per query |

## 🚀 Setup

### 1. Clone & install
```bash
git clone https://github.com/<your-username>/rag-chatbot.git
cd rag-chatbot
pip install -r requirements.txt
```

### 2. Get a Groq API key
Sign up at [console.groq.com](https://console.groq.com/) and generate a free API key.

### 3. Configure your key
Create a `.env` file in the project root:
```
GROQ_API_KEY=gsk_your_key_here
```
Or, for Streamlit Cloud deployment, add it to **App Settings → Secrets**:
```toml
GROQ_API_KEY = "gsk_your_key_here"
```

### 4. Run the app
```bash
streamlit run app.py
```

The app opens at `http://localhost:8501`.

## 💻 Usage

1. **Upload page** — Click **⚡ Load Sample Docs** to use the built-in knowledge base, or upload your own PDF / TXT files.
2. **Chat page** — Click **💬 Start Chatting**, type a question, and watch the answer stream in. Source documents appear beneath each response.
3. **Clear chat** — Reset the conversation memory anytime with the 🗑️ button.

## 🧩 How It Works

### Document Ingestion
PDFs and text files are loaded with LangChain's `PyPDFLoader` / `TextLoader`, then split into ~1000-character chunks with a custom recursive splitter that preserves paragraph boundaries and adds 200-char overlap between chunks for context continuity.

### Embedding & Indexing
Each chunk is embedded locally using `all-MiniLM-L6-v2` (384-dim, normalized) and indexed in FAISS. The embedding model is loaded once per process via a singleton pattern to avoid re-initialization overhead.

### Question Answering
For each user message:
1. **Condense** — If chat history exists, the follow-up is rephrased into a standalone question.
2. **Retrieve** — FAISS returns the top-3 most semantically similar chunks.
3. **Generate** — The chunks + history + question are stuffed into a prompt template and streamed through Groq's Llama 3.3 70B.
4. **Remember** — The Q&A pair is saved to a sliding-window memory buffer (last 6 turns).

### Streaming
Tokens arrive incrementally via `llm.stream()` and update a Streamlit placeholder in place, with a blinking cursor (`▌`) during generation that disappears on completion.

## 📦 Requirements

```
streamlit
langchain
langchain-classic
langchain-community
langchain-core
langchain-groq
langchain-huggingface
faiss-cpu
sentence-transformers
pypdf
python-dotenv
```

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| Frontend | Streamlit + custom CSS |
| Orchestration | LangChain |
| LLM | Groq Cloud (Llama 3.3 70B Versatile) |
| Embeddings | HuggingFace `all-MiniLM-L6-v2` (local, CPU) |
| Vector Store | FAISS |
| Document Parsing | pypdf, TextLoader |

## 🐛 Troubleshooting

**Windows error 1455 / paging file too small**
The embedding model needs ~500MB of memory. Open **System Properties → Advanced → Performance → Settings → Advanced → Virtual Memory** and increase your page file size, then restart.

**"Groq API key is required"**
Make sure `.env` is in the project root, the variable is named exactly `GROQ_API_KEY`, and you've restarted the Streamlit app.

**Slow first run**
The first time you load documents, sentence-transformers downloads the embedding model (~90MB). Subsequent runs use the cache.

## 📝 License

MIT
