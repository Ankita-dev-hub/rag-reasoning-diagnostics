# Cloud Operations Assistant - Conversational RAG with Agentic Fallback

## Problem Statement

Modern cloud-based applications generate large volumes of documentation, 
runbooks, and troubleshooting guides. Engineers waste significant time 
diagnosing issues because knowledge is fragmented across sources and lacks 
contextual, conversational access.

This project builds an AI-powered Cloud Operations Assistant that solves this 
by combining internal document retrieval (RAG), conversational memory, and 
an agentic decision layer — simulating a real-world DevOps support assistant.

---

## What This System Can Do

| Capability | How |
|---|---|
| Answer from internal docs | RAG over ChromaDB vector store |
| Remember conversation context | ConversationBufferWindowMemory (k=5) |
| Fall back to live web search | DuckDuckGo agent when docs have no answer |
| Rank retrieved chunks by relevance | FlashrankRerank before LLM call |
| Decide which tool to use | LangChain AgentExecutor with tool routing |

---

## Architecture
Internal Docs (URLs/PDFs)
↓
WebBaseLoader → RecursiveTextSplitter (chunk=500, overlap=50)
↓
HuggingFace Embeddings (all-MiniLM-L6-v2)
↓
ChromaDB — Local Vector Store
↓
FlashrankRerank — Top-K Reranking
↓
AgentExecutor
├── Tool 1: search_docs       → ChromaDB retriever (primary)
└── Tool 2: web_search        → DuckDuckGo (fallback)
↓
Groq LLM (llama-3.1-8b-instant)
↓
ConversationalRetrievalChain + Memory Window
↓
Context-aware response to engineer

---
---

## Key Design Decisions

| Decision | Reason |
|---|---|
| ChromaDB over Pinecone | Zero cloud dependency, runs free in Colab |
| FlashrankRerank before LLM | Improves answer quality vs raw top-k retrieval |
| Separate memory objects for chain vs agent | AgentExecutor expects `output` key; chain expects `answer` key — different memory configs required |
| Window memory k=5 | Prevents token overflow on long troubleshooting sessions |
| Groq + llama-3.1-8b | Fast inference on free tier, sufficient for RAG-grounded answers |
| Agent decides tool routing | LLM chooses between internal docs vs web search — not hardcoded logic |

---

## Sample Scenarios This Handles

**Scenario 1 — Internal doc exists:**
> "What is the snapshot lifecycle behavior in OpenSearch?"
→ Agent picks `search_docs` → retrieves from ChromaDB → reranked → answered

**Scenario 2 — Internal doc does not exist:**
> "What is the current pricing for OpenSearch Serverless?"
→ Agent picks `web_search` → DuckDuckGo → answered from live web

**Scenario 3 — Multi-turn troubleshooting:**
> Turn 1: "Our OpenSearch cluster is throwing JVM memory errors"
> Turn 2: "What heap size is recommended for our data volume?"
> Turn 3: "How do I apply that change without downtime?"
→ Memory window maintains full context across all three turns

---

## How to Run

1. Open in Google Colab
2. Add your Groq API key to Colab Secrets as `GROQ_API_KEY`
3. Replace `doc_base` URLs with your own internal documentation links
4. Run all cells top to bottom
5. Use the agent query cell to ask troubleshooting questions

---

## Production Scaling Notes

This is a prototype. For production use, the following refinements apply:
- Replace ChromaDB with AWS OpenSearch or Pinecone for persistent vector storage
- Replace Groq with AWS Bedrock (Claude / Titan) for enterprise governance
- Add Bedrock Guardrails for PII masking and prompt injection protection
- Replace DuckDuckGo with a controlled internal search API
- Add logging and tracing via AWS CloudWatch or LangSmith

---

## Tech Stack

Python · LangChain · ChromaDB · HuggingFace Embeddings · 
Groq (llama-3.1-8b) · FlashrankRerank · DuckDuckGo Search · Google Colab
