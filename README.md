# RAG Reasoning Diagnostics

Two production-style RAG systems built to understand where retrieval ends 
and reasoning begins.

Most RAG tutorials answer one question from one document. These systems 
do something harder — one diagnoses recurring patterns across historical 
incidents, the other decides in real time whether its own answer is 
good enough or whether it needs to go looking for more.

Built with AWS Bedrock, ChromaDB, Groq, and Snowflake Cortex.

---

## What's in here

**01 — Web RAG with Chroma and Groq**  
A cloud operations assistant that retrieves from internal documentation, 
maintains conversational memory, and falls back to web search when 
internal context is insufficient. The agent decides when to search — 
not the user.

**02 — PDF RAG with Chroma and Groq**  
An incident case investigator that retrieves multiple historical cases, 
reasons across them collectively to detect patterns, and validates its 
own output against the retrieved context before returning a response.

---

## The design question these systems answer

RAG is the data access layer. The system behavior — what gets retrieved, 
how it's used, whether the output is validated, and when external 
enrichment is triggered — is controlled entirely by how the problem 
is framed and what the model is asked to do with the data.

Both systems use the same core pipeline. They behave completely differently.

---

## Architecture

**Retrieval → Prompt design → Validation → Decision layer → Output**

The decision layer is where agents actually live — not at retrieval, 
not at generation, but at the point where the system evaluates whether 
its current reasoning is complete or whether another step is needed.

---

## Read the full breakdown

I wrote about the thinking behind both systems and how the design evolved 
on Medium:  
[I Built the Same RAG Pipeline Twice. They Behave Nothing Alike.](https://medium.com/@sharmankita0604)

---

## Stack

- AWS Bedrock (embeddings + LLM)
- ChromaDB (vector store)
- Groq (fast inference)
- Snowflake Cortex (semantic search layer)
- LangChain (agent + memory)
- Python, Jupyter Notebooks
