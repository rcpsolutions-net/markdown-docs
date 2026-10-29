# More AI / SLM — What Node Can Actually Do

*Prompted and edited by Lawrence Ham*

## Framing

Fine-tuning / training small language models (SLMs) is a **Python** territory (PyTorch / TensorFlow). Node.js is not for that. The practical Node answer is **RAG** — retrieve-augmented generation — which teaches an SLM your company's information *without* retraining.

## RAG Pipeline

1. **Ingest** — scrape website (Puppeteer/Cheerio); parse docs (pdf-parse / mammoth).
2. **Chunk** — e.g. `RecursiveCharacterTextSplitter`, ~1000 chars / 200 overlap.
3. **Embed** — OpenAI / Cohere / local Ollama.
4. **Store** — vector DB: Pinecone / Weaviate / Chroma / Supabase.
5. **Query** — embed the question → top-K nearest neighbors → augment the prompt → SLM answers.

## Why RAG

- No retraining cost.
- Updates in real time when docs change (just re-embed).
- Node-friendly end to end.

## Choosing an SLM

- **Cloud** (e.g. GPT-3.5): easy, per-token API costs, data leaves your infra.
- **Local** via Ollama / Llama.cpp (`llama2:7b`, `mistral:7b`): free after setup, on-prem. Node talks to the local server — it does *not* run the model itself.

## Considerations

- Data quality (garbage in, garbage out).
- Scraping ToS / rate limits.
- Vector DB scaling.
- Per-token cost.
- Security / privacy — local keeps everything on-prem.
- **RAG ≠ fine-tuning**: RAG doesn't change model weights. Fine-tune (in Python) for brand tone / niche reasoning, not in Node.

Typical Node stack: **LangChain.js** + a vector DB + an SLM endpoint (cloud or local).
