# Customer Support Case Investigator - Conversational RAG with Pattern Analysis

## Problem Statement

Customer support teams in SaaS environments handle large volumes of incident tickets,
logs, and diagnostic reports. Engineers spend significant time identifying recurring
issues because historical cases are scattered and lack contextual access.

This project builds an AI-powered Customer Support Case Investigator that leverages
historical case data to diagnose issues, detect recurring patterns, and provide
structured, actionable insights using conversational RAG.

---

## What This System Can Do

| Capability                      | How                                                     |
| ------------------------------- | ------------------------------------------------------- |
| Retrieve similar past cases     | RAG over ChromaDB vector store (PDF-based cases)        |
| Maintain conversational context | ConversationBufferWindowMemory (k=5)                    |
| Analyze multiple cases together | Prompt-driven pattern detection across retrieved chunks |
| Identify recurring issues       | LLM reasoning over top-k retrieved cases                |
| Provide structured diagnosis    | Root cause, resolution, recurrence, confidence          |

---

## Architecture

AWS Diagnostic Case PDF
↓
PyPDFLoader → RecursiveTextSplitter (chunk=1000, overlap=100)
↓
HuggingFace Embeddings (all-MiniLM-L6-v2)
↓
ChromaDB — Local Vector Store
↓
Top-K Retrieval (k=5)
↓
ConversationalRetrievalChain
↓
Groq LLM (llama-3.1-8b-instant)
↓
Conversation Memory Window (k=5)
↓
Pattern-aware diagnostic response

---

## Key Design Decisions

| Decision                    | Reason                                                                  |
| --------------------------- | ----------------------------------------------------------------------- |
| PDF-based case ingestion    | Simulates real-world historical incident data                           |
| Larger chunk size (1000)    | Preserves full case context (issue + cause + resolution)                |
| Top-k retrieval set to 5    | Enables pattern detection across multiple similar cases                 |
| Structured system prompt    | Forces analytical output instead of generic answers                     |
| Memory window k=5           | Maintains troubleshooting continuity across follow-ups                  |
| No agent layer in core flow | Keeps system deterministic and focused on reasoning over internal cases |

---

## Sample Scenarios This Handles

**Scenario 1 — Recurring issue detection:**

> "Why is our system experiencing high CPU usage?"
> → Retrieves multiple similar cases → identifies common pattern → outputs dominant root cause

**Scenario 2 — Root cause inference:**

> "We are seeing intermittent timeout errors in database connections"
> → Matches across cases → infers likely cause (e.g., connection pool exhaustion)

**Scenario 3 — Conversational follow-up:**

> Turn 1: "Our system is showing latency spikes"
> Turn 2: "Was this seen before?"
> Turn 3: "What was the fix?"
> → Memory maintains context → system answers without re-specifying issue

---

## How to Run

1. Open notebook in Google Colab
2. Add Groq API key to Colab Secrets as `GROQ_API_KEY`
3. Upload diagnostic PDF (e.g., AWS troubleshooting guide)
4. Run all cells sequentially
5. Use `chat()` function to interact with the system

---

## Production Scaling Notes

This is a prototype. For production use:

* Replace ChromaDB with managed vector store (OpenSearch / Pinecone)
* Ingest structured ticket data instead of raw PDFs
* Add metadata filtering (service, severity, timestamp)
* Introduce evaluation layer for answer validation
* Add agent layer for external knowledge fallback (if required)

---

## Tech Stack

Python · LangChain · ChromaDB · HuggingFace Embeddings ·
Groq (llama-3.1-8b) · PyPDFLoader · RecursiveTextSplitter · Google Colab
