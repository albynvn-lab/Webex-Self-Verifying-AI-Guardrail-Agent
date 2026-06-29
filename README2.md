# 🛡️ Trust Before Answer: Ethical AI Guardrail Agent

## Project Summary

This project implements a **"Trust Before Answer"** system — a Self-Verifying AI Agent that incorporates ethical guardrails to **prevent hallucination** and ensure **knowledge sufficiency** before generating responses. The system uses a **Retrieval-Augmented Generation (RAG)** pipeline combined with a three-layer guardrail mechanism and internal self-verification to guarantee that every answer is grounded in verified knowledge.

**Knowledge Source**: [help.webex.com](https://help.webex.com) — Cisco Webex documentation covering meetings, messaging, security, and more.

> **No API key required** — all models run locally on your machine!

---

## 📂 Project Structure

```
Proj 1/
├── TrustBeforeAnswer.ipynb    # Main notebook — Self-Verifying AI Guardrail Agent
├── BRKCOL-2990.pdf            # Reference document (can be uploaded for Document Mode)
└── README2.md                 # This file
```

---

## 🏗️ Architecture

### Three-Layer Protection System

```
User Query -> [Retrieval] -> [Hard Guardrail] -> [Soft Guardrail (LLM)] -> [Self-Verification] -> Response
                                  | (if no context)
                            SAFE REFUSAL
```

### Detailed Flow

```
Layer 1: HARD GUARDRAIL    -> Code-level check (context == None -> refuse)
Layer 2: SOFT GUARDRAIL    -> Prompt instructs "only use context"
Layer 3: SELF-VERIFICATION -> LLM validates its own answer against evidence
```

### Decision Logic

```
if context == None:
    -> REFUSE (safe denial, no LLM call needed)
else:
    answer = generate(context, question)
    verification = verify(evidence=context, answer=answer)
    if verification == "SUPPORTED":
        -> PRESENT verified answer
    else:
        -> FLAG as unverified
```

---

## Key Features

| Feature | Description |
|---------|-------------|
| **Hard Guardrail** | Code-level check — refuses immediately if no relevant context is found |
| **Soft Guardrail** | Prompt-level constraint — LLM instructed to only use provided context |
| **Self-Verification** | A second LLM pass validates the generated answer against evidence |
| **Hallucination Prevention** | Three-layer protection ensures no fabricated answers |
| **Document Upload** | Upload `.txt`, `.pdf`, `.md`, or `.csv` files as custom knowledge sources |
| **Chunk Visualization** | Displays which document chunks were retrieved and used for verification |
| **Dual Mode** | Works with built-in Webex KB or user-uploaded documents |
| **Explicit Refusals** | Clear communication when knowledge is insufficient |
| **Fully Local** | Runs entirely on your machine — no cloud APIs, no costs |

---

## Models & Technologies

| Component | Technology |
|-----------|-----------|
| **LLM** | Google Flan-T5-Base (250M params, runs locally) |
| **Framework** | LangChain (LCEL chains, prompt templates, output parsers) |
| **Text Splitting** | LangChain RecursiveCharacterTextSplitter |
| **PDF Parsing** | PyPDF2 |
| **UI** | Gradio (interactive web interface with file upload) |
| **Paradigm** | RAG + Self-Verifying AI Agent Pattern |

---

## How to Run

### Prerequisites

- Python 3.9+
- pip (Python package manager)
- **No API key required!**

### Steps

1. Open `TrustBeforeAnswer.ipynb` in Jupyter or VS Code

2. Run cells sequentially (Block 1 through Block 8):
   - **Block 1**: Installs all required packages automatically
   - **Block 2**: Imports libraries
   - **Block 3**: Downloads & loads Flan-T5 model (~990MB on first run, cached after)
   - **Block 4**: Sets up the Webex knowledge retrieval function
   - **Block 5**: Creates the guardrail prompt template & chain
   - **Block 5B**: Creates the self-verification chain
   - **Block 6**: Implements the full self-verifying agent pipeline
   - **Block 7**: Defines the Gradio web interface
   - **Block 8**: Launches the application

3. A Gradio URL will appear — click it to open the interactive demo

### Dependencies (auto-installed by Block 1)

```bash
pip install transformers torch langchain-huggingface langchain-core
pip install langchain-text-splitters PyPDF2 gradio
```

---

## Usage Modes

### Knowledge Base Mode (No file uploaded)

Ask questions about Cisco Webex — the system queries its built-in knowledge base sourced from help.webex.com:

- "How do I join a meeting in Webex?"
- "How do I share content in a Webex meeting?"
- "Tell me about Webex messaging and spaces"
- "How does noise removal work in Webex?"
- "Is Webex encrypted and secure?"
- "How do I create a Webex bot?"

Out-of-scope queries are safely refused:
- "What is the weather today?" -> Refusal
- "How do I use Microsoft Teams?" -> Refusal

### Document Mode (File uploaded)

Upload any `.txt`, `.pdf`, `.md`, or `.csv` file to use as a custom knowledge source:

1. Upload your document via the file upload widget
2. Ask questions about the document content
3. View which chunks were retrieved and how the answer was verified

---

## Response Status Indicators

| Icon | Status | Meaning |
|------|--------|---------|
| VERIFIED | **Verified** | Answer generated AND confirmed against evidence |
| UNVERIFIED | **Unverified** | Answer generated but NOT fully confirmed — use caution |
| REFUSED | **Refused** | No relevant knowledge found — safe denial |

---

## Self-Verifying AI Agent Technique

Inspired by [LangChain's RAG QA patterns](https://python.langchain.com/docs/use_cases/question_answering/), this project implements internal answer verification:

| Step | Action | Purpose |
|------|--------|---------|
| 1. Retrieve | Search knowledge base / document chunks | Get relevant evidence |
| 2. Guard | Hard guardrail check (None -> refuse) | Catch missing knowledge |
| 3. Generate | LLM creates answer from context only | Produce candidate response |
| 4. **Verify** | LLM checks answer against evidence | Catch hallucinations |
| 5. Present | Show verified or flagged result | Ensure user trust |

---

## Knowledge Base Coverage

All entries sourced from [help.webex.com](https://help.webex.com):

| Topic | Keywords |
|-------|----------|
| Meetings | `meeting` — scheduling, joining, features, translation, Webex Assistant |
| Messaging & Spaces | `messaging`, `space` — collaboration, file sharing, threading |
| Content Sharing | `share`, `content` — screen share, immersive share, attachments |
| Audio & Video | `noise`, `background` — noise removal, virtual backgrounds |
| Security | `security`, `encryption` — end-to-end encryption, anti-malware |
| Developer | `api`, `bot` — REST API, pagination, rate limits, bot accounts |
| AI Assistant | `assistant` — voice commands, transcription, highlights |

---

## Key Design Decisions

| Decision | Rationale |
|----------|----------|
| Local Flan-T5 model | No API key, no costs, no internet dependency |
| `do_sample=False` | Ensures deterministic, reproducible outputs |
| Three-layer guardrails | Hard (code) + Soft (prompt) + Verification (self-check) |
| Self-verification chain | Agent validates its own output before presenting |
| Explicit refusal messages | User understands WHY the system can't answer |
| Context-only generation | LLM cannot use training data, only provided evidence |
| Document chunking | RecursiveCharacterTextSplitter (500 chars, 100 overlap) |

---

## Ethical AI Principles Demonstrated

1. **Transparency** — System clearly communicates when it cannot answer and when an answer is unverified
2. **Truthfulness** — Answers are strictly grounded in verified documentation
3. **Self-Accountability** — Agent audits its own responses before presenting them
4. **Safety** — Errs on the side of refusal/flagging rather than fabrication
5. **Traceability** — Every answer can be traced back to source content
6. **Accessibility** — Runs locally without paid APIs — democratized AI safety

---

## Limitations & Future Improvements

| Limitation | Possible Improvement |
|-----------|---------------------|
| Keyword matching for KB | Use vector embeddings (FAISS/ChromaDB) for semantic search |
| Static knowledge base | Live crawling/indexing of help.webex.com articles |
| No confidence scoring | Add retrieval confidence threshold + verification confidence |
| Same model for generation & verification | Use different model for verification (ensemble) |
| Flan-T5-base (250M) | Use larger model (flan-t5-large/xl) for better answers |
| Text only | Support multi-modal inputs (screenshots of Webex UI) |
| Binary verification | Add nuanced levels (fully / partially / not supported) |

---

## References

- [LangChain Documentation](https://python.langchain.com/)
- [LangChain RAG QA Patterns](https://python.langchain.com/docs/use_cases/question_answering/)
- [HuggingFace Transformers](https://huggingface.co/docs/transformers)
- [Google Flan-T5](https://huggingface.co/google/flan-t5-base)
- [Gradio Documentation](https://www.gradio.app/docs)
- [Cisco Webex Help Center](https://help.webex.com)

---

*Built with LangChain, Self-Verifying AI Agent Pattern, Hugging Face Flan-T5, Gradio, and knowledge from help.webex.com — No API key required!*
