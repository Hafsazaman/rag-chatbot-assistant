# RAG Chatbot Assistant

An intelligent chatbot built using Retrieval-Augmented Generation (RAG) that answers questions based on uploaded documents.

## Features
- Upload documents (PDF, TXT, etc.)
- Extract and process document content
- Create embeddings for semantic search
- Store vectors using FAISS
- Generate accurate answers using Groq LLM
- Simple web interface with Streamlit

## Tech Stack
- Python
- LangChain
- FAISS
- HuggingFace Embeddings
- Groq LLM
- Streamlit

## Project Structure

├── app.py  
├── rag_pipeline.py  
├── document_loader.py  
├── config.py  
├── requirements.txt  
└── README.md  

## Installation

Clone the repository:

```bash
git clone <your-repo-link>
cd rag-chatbot-assistant
pip install -r requirements.txt

#RUN PROJECT:
streamlit run app.py
