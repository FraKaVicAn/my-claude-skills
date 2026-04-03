---
name: rag
description: Build Retrieval-Augmented Generation pipelines with Claude — chunk documents, embed with VoyageAI, retrieve semantically, and generate grounded answers with citations
origin: anthropics/claude-cookbooks/capabilities/retrieval_augmented_generation
tools: Read, Write, Edit, Bash
---

# RAG (Retrieval-Augmented Generation) Skill

Ground Claude's answers in your own documents by building a retrieval pipeline: chunk → embed → index → retrieve → generate.

## When to Activate

- Building a Q&A system over internal docs, PDFs, or knowledge bases
- Reducing hallucinations by grounding answers in retrieved context
- Implementing semantic search over a document collection
- Adding citations to Claude's responses

## Architecture

```
Documents → Chunking → Embeddings → Vector Index
                                         ↓
User Query → Query Embedding → Retrieval → Claude → Answer + Citations
```

## Full Implementation

```python
import anthropic
import voyageai
import numpy as np

client = anthropic.Anthropic()
vo = voyageai.Client()  # pip install voyageai

# --- 1. CHUNKING ---
def chunk_text(text: str, chunk_size: int = 500, overlap: int = 50) -> list[str]:
    words = text.split()
    chunks = []
    for i in range(0, len(words), chunk_size - overlap):
        chunk = " ".join(words[i:i + chunk_size])
        if chunk:
            chunks.append(chunk)
    return chunks

# --- 2. EMBEDDING ---
def embed_chunks(chunks: list[str]) -> list[list[float]]:
    result = vo.embed(chunks, model="voyage-3", input_type="document")
    return result.embeddings

def embed_query(query: str) -> list[float]:
    result = vo.embed([query], model="voyage-3", input_type="query")
    return result.embeddings[0]

# --- 3. RETRIEVAL (cosine similarity) ---
def retrieve(query: str, chunks: list[str], embeddings: list[list[float]], top_k: int = 5) -> list[str]:
    q_emb = np.array(embed_query(query))
    doc_embs = np.array(embeddings)
    scores = (doc_embs @ q_emb) / (np.linalg.norm(doc_embs, axis=1) * np.linalg.norm(q_emb))
    top_indices = np.argsort(scores)[::-1][:top_k]
    return [chunks[i] for i in top_indices]

# --- 4. GENERATION WITH CITATIONS ---
def rag_answer(query: str, retrieved: list[str]) -> str:
    context = "\n\n".join(f"[{i+1}] {chunk}" for i, chunk in enumerate(retrieved))
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=1024,
        system="Answer using only the provided context. Cite sources as [1], [2], etc. If the answer isn't in the context, say so.",
        messages=[{
            "role": "user",
            "content": f"Context:\n{context}\n\nQuestion: {query}"
        }]
    )
    return response.content[0].text

# --- USAGE ---
docs = ["Your document text here...", "Another document..."]
all_chunks = [chunk for doc in docs for chunk in chunk_text(doc)]
embeddings = embed_chunks(all_chunks)

query = "What is the refund policy?"
retrieved = retrieve(query, all_chunks, embeddings)
answer = rag_answer(query, retrieved)
print(answer)
```

## With Vector Database (Pinecone)

```python
from pinecone import Pinecone

pc = Pinecone(api_key="YOUR_KEY")
index = pc.Index("my-rag-index")

# Upsert
vectors = [(f"chunk-{i}", emb, {"text": chunk})
           for i, (chunk, emb) in enumerate(zip(all_chunks, embeddings))]
index.upsert(vectors=vectors)

# Query
results = index.query(vector=embed_query(query), top_k=5, include_metadata=True)
retrieved = [r.metadata["text"] for r in results.matches]
```

## Evaluation

Use the `evaluation/` folder in the cookbook for Promptfoo-based RAG evaluation:
- Retrieval recall (are the right chunks being found?)
- Answer faithfulness (is the answer grounded in the context?)
- Answer relevance (does it actually answer the question?)

## Best Practices

- Chunk size 300–600 tokens; overlap 10–15% of chunk size
- Use `voyage-3` for general text, `voyage-code-3` for code
- Store original document metadata (page, source URL) with each chunk
- Re-rank retrieved chunks with a cross-encoder for higher precision
- Add a "no answer" fallback when retrieval confidence is low

## Links

- [Cookbook: RAG guide](https://github.com/anthropics/claude-cookbooks/tree/main/capabilities/retrieval_augmented_generation)
- [VoyageAI embeddings](https://docs.voyageai.com)
- [Anthropic API docs](https://docs.anthropic.com)
