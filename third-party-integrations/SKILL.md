---
name: third-party-integrations
description: Integrate Claude with third-party services — Pinecone vector DB, MongoDB, LlamaIndex, VoyageAI embeddings, Deepgram speech, ElevenLabs TTS, Wikipedia, WolframAlpha
origin: anthropics/claude-cookbooks/third_party
tools: Read, Write, Edit, Bash
---

# Third-Party Integrations Skill

Connect Claude to external services and databases: vector stores, knowledge bases, speech processing, text-to-speech, and specialized APIs.

## When to Activate

- Adding a vector database (Pinecone, MongoDB Atlas) to a RAG pipeline
- Using VoyageAI for best-in-class embeddings
- Transcribing audio with Deepgram before sending to Claude
- Generating speech from Claude's text output with ElevenLabs
- Querying Wikipedia or WolframAlpha as Claude tools
- Building with LlamaIndex as an orchestration layer

## Pinecone + Claude (Vector RAG)

```python
import anthropic
import voyageai
from pinecone import Pinecone

client = anthropic.Anthropic()
vo = voyageai.Client()
pc = Pinecone(api_key="PINECONE_KEY")
index = pc.Index("my-index")

def upsert_documents(docs: list[str], namespace: str = "default"):
    embeddings = vo.embed(docs, model="voyage-3", input_type="document").embeddings
    vectors = [(f"doc-{i}", emb, {"text": doc}) for i, (doc, emb) in enumerate(zip(docs, embeddings))]
    index.upsert(vectors=vectors, namespace=namespace)

def rag_query(question: str, top_k: int = 5) -> str:
    q_emb = vo.embed([question], model="voyage-3", input_type="query").embeddings[0]
    results = index.query(vector=q_emb, top_k=top_k, include_metadata=True, namespace="default")
    context = "\n\n".join(r.metadata["text"] for r in results.matches)
    response = client.messages.create(
        model="claude-sonnet-4-6", max_tokens=1024,
        system="Answer using only the provided context. Cite sources as [1], [2], etc.",
        messages=[{"role": "user", "content": f"Context:\n{context}\n\nQuestion: {question}"}]
    )
    return response.content[0].text
```

## MongoDB Atlas + Claude

```python
from pymongo import MongoClient

mongo = MongoClient("mongodb+srv://...")
db = mongo["mydb"]

def store_with_embedding(collection_name: str, doc: dict, text_field: str):
    """Store a document with its embedding for vector search."""
    text = doc[text_field]
    embedding = vo.embed([text], model="voyage-3", input_type="document").embeddings[0]
    doc["embedding"] = embedding
    db[collection_name].insert_one(doc)

def vector_search(collection_name: str, query: str, top_k: int = 5) -> list[dict]:
    q_emb = vo.embed([query], model="voyage-3", input_type="query").embeddings[0]
    results = db[collection_name].aggregate([{
        "$vectorSearch": {
            "index": "vector_index",
            "path": "embedding",
            "queryVector": q_emb,
            "numCandidates": 100,
            "limit": top_k
        }
    }])
    return list(results)
```

## VoyageAI Embeddings

```python
import voyageai

vo = voyageai.Client(api_key="VOYAGE_KEY")

# General text (best quality)
embeddings = vo.embed(texts, model="voyage-3", input_type="document").embeddings

# Code (specialized)
code_embeddings = vo.embed(code_snippets, model="voyage-code-3", input_type="document").embeddings

# Query vs document (asymmetric)
query_emb = vo.embed([query], model="voyage-3", input_type="query").embeddings[0]
doc_embs = vo.embed(docs, model="voyage-3", input_type="document").embeddings

# Reranking (improves RAG precision)
reranked = vo.rerank(query, docs, model="rerank-2", top_k=5)
```

## Deepgram (Speech → Claude)

```python
from deepgram import DeepgramClient

dg = DeepgramClient(api_key="DG_KEY")

def transcribe_and_analyze(audio_path: str, analysis_prompt: str) -> str:
    with open(audio_path, "rb") as f:
        audio_data = f.read()
    response = dg.listen.prerecorded.v("1").transcribe_file(
        {"buffer": audio_data},
        {"model": "nova-2", "language": "en", "punctuate": True}
    )
    transcript = response["results"]["channels"][0]["alternatives"][0]["transcript"]

    # Send transcript to Claude
    return client.messages.create(
        model="claude-sonnet-4-6", max_tokens=1024,
        messages=[{"role": "user", "content": f"{analysis_prompt}\n\nTranscript:\n{transcript}"}]
    ).content[0].text
```

## ElevenLabs (Claude → Speech)

```python
from elevenlabs import ElevenLabs, Voice

el = ElevenLabs(api_key="EL_KEY")

def claude_speaks(prompt: str, voice_id: str = "21m00Tcm4TlvDq8ikWAM") -> bytes:
    # Get Claude's response
    text = client.messages.create(
        model="claude-haiku-4-5", max_tokens=512,
        messages=[{"role": "user", "content": prompt}]
    ).content[0].text

    # Convert to speech
    audio = el.generate(text=text, voice=Voice(voice_id=voice_id), model="eleven_multilingual_v2")
    return b"".join(audio)
```

## Wikipedia Tool

```python
import httpx

def wikipedia_search(query: str, sentences: int = 3) -> str:
    resp = httpx.get("https://en.wikipedia.org/api/rest_v1/page/summary/" + query.replace(" ", "_"))
    if resp.status_code == 200:
        return resp.json().get("extract", "Not found")[:sentences * 200]
    return f"No Wikipedia article found for '{query}'"

wikipedia_tool = {
    "name": "wikipedia_search",
    "description": "Search Wikipedia for factual information about a topic.",
    "input_schema": {
        "type": "object",
        "properties": {"query": {"type": "string", "description": "The topic to search for"}},
        "required": ["query"]
    }
}
```

## WolframAlpha Tool

```python
def wolfram_query(query: str, app_id: str) -> str:
    resp = httpx.get(f"https://api.wolframalpha.com/v1/result?i={query}&appid={app_id}")
    return resp.text if resp.status_code == 200 else "Could not compute"
```

## Links

- [Cookbook: third_party/](https://github.com/anthropics/claude-cookbooks/tree/main/third_party)
- [VoyageAI docs](https://docs.voyageai.com)
- [Pinecone docs](https://docs.pinecone.io)
- [Deepgram docs](https://developers.deepgram.com)
- [ElevenLabs docs](https://elevenlabs.io/docs)
