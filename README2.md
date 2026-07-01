# 🛡️ Trust Before Answer: Self-Verifying AI Guardrail Agent for Webex

## Project Summary

A **Self-Verifying AI Agent** that validates its own answers against retrieved evidence before presenting them. Implements a **three-layer guardrail architecture** using Retrieval-Augmented Generation (RAG) to prevent hallucination and ensure knowledge sufficiency.

**Knowledge Source**: [help.webex.com](https://help.webex.com) — Cisco Webex documentation  
**UI Theme**: Cisco brand palette (Blue #049FD9, Teal #00BCEB, Green #6CC04A)

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
User Query
    │
    ▼
┌─────────────────────────────┐
│  RETRIEVAL                  │  Stem matching + best-match scoring
│  (KB or uploaded document)  │  Word-boundary regex for precision
└─────────────┬───────────────┘
              │
              ▼
┌─────────────────────────────┐
│  LAYER 1: HARD GUARDRAIL    │  Code-level: context == None → REFUSE
└─────────────┬───────────────┘
              │
              ▼
┌─────────────────────────────┐
│  LAYER 2: SOFT GUARDRAIL    │  Prompt: "only use context"
│  (LLM Generation)           │  Role-based prompt engineering
└─────────────┬───────────────┘
              │
              ▼
┌─────────────────────────────┐
│  POST-PROCESSING            │  build_structured_answer()
│  (Context enrichment)       │  Sentence scoring + bullet formatting
└─────────────┬───────────────┘
              │
              ▼
┌─────────────────────────────┐
│  LAYER 3: SELF-VERIFICATION │  Second LLM pass validates answer
│  (Agent audits itself)      │  against original evidence
└─────────────┬───────────────┘
              │
              ▼
┌─────────────────────────────┐
│  PRESENTATION               │  ✅ Verified / ⚠️ Unverified / 🚫 Refused
└─────────────────────────────┘
```

### Decision Logic

```python
context = retrieve_context(query)  # Stem matching + best-match scoring

if context is None:
    → REFUSE (safe denial, no LLM call)
else:
    raw_answer = guardrail_chain.invoke(context, question)

    # Fallback: if model falsely refuses with valid context
    if "INSUFFICIENT_KNOWLEDGE" in raw_answer and context exists:
        raw_answer = ""  # Use context directly

    structured = build_structured_answer(question, context, raw_answer)
    verification = verification_chain.invoke(evidence=context, answer=structured)

    if verification == "SUPPORTED":
        → PRESENT verified answer ✅
    else:
        → FLAG as unverified ⚠️
```

---

## ✨ Key Features

| Feature | Description |
|---------|-------------|
| **Hard Guardrail** | Code-level refusal when no relevant context exists |
| **Soft Guardrail** | Role-based prompt constrains LLM to context only |
| **Self-Verification** | Second LLM pass validates answer against evidence |
| **Intelligent Retrieval** | Stem matching + best-match scoring + word-boundary regex |
| **Structured Answers** | Post-processing builds bullet-point responses from context |
| **False-Refusal Fallback** | Catches when model incorrectly refuses valid queries |
| **Document Upload** | Upload `.txt`, `.pdf`, `.md`, `.csv` as custom knowledge |
| **Chunk Visualization** | Shows which chunks were retrieved and used |
| **Dual Mode** | Built-in Webex KB or user-uploaded documents |
| **Cisco-Themed UI** | Professional Gradio interface with Cisco brand colors |
| **Fully Local** | No cloud APIs, no costs, no internet after first download |

---

## 🛠️ Models and Technologies

| Component | Technology |
|-----------|-----------|
| **LLM** | Google Flan-T5-Base (250M params, local) |
| **Generation** | Beam search (4 beams), 512 max tokens, no-repeat 3-gram |
| **Framework** | LangChain LCEL (prompt templates, chains, output parsers) |
| **Text Splitting** | RecursiveCharacterTextSplitter (500 chars, 100 overlap) |
| **PDF Parsing** | PyPDF2 |
| **UI** | Gradio 4.44+ with custom CSS (Cisco palette) |
| **Paradigm** | RAG + Self-Verifying AI Agent Pattern |

---

## 🚀 How to Run

### Prerequisites

- Python 3.9+
- ~4 GB free disk space (for model cache)
- **No API key required!**

### Steps

1. Open `TrustBeforeAnswer.ipynb` in VS Code or Jupyter

2. Run cells sequentially (Block 1 through Block 8):

   | Block | Purpose | Time |
   |-------|---------|------|
   | 1 | Install packages | ~30s |
   | 2 | Import libraries | instant |
   | 3 | Load Flan-T5 model | ~5s (cached) / ~3min (first run) |
   | 4 | Setup retrieval function | instant |
   | 5 | Create guardrail chain | instant |
   | 5B | Create verification chain | instant |
   | 6 | Define agent pipeline | instant |
   | 7 | Define Gradio UI (Cisco theme) | instant |
   | 8 | Launch application | ~2s |

3. Click the Gradio URL to open the interactive demo

### Dependencies (auto-installed by Block 1)

```bash
transformers torch langchain-huggingface langchain-core
langchain-text-splitters PyPDF2 gradio
```

---

## 💡 Usage

### Knowledge Base Mode (No file uploaded)

Ask questions about Cisco Webex:

- "How do I join a meeting in Webex?"
- "How do I share content in a Webex meeting?"
- "Tell me about Webex messaging and spaces"
- "How does noise removal work in Webex?"
- "Is Webex encrypted and secure?"
- "How do I create a Webex bot?"

Out-of-scope queries are safely refused:

- "What is the weather today?" → Refused
- "How do I use Microsoft Teams?" → Refused
- "What is the capital of France?" → Refused

### Document Mode (File uploaded)

1. Upload any `.txt`, `.pdf`, `.md`, or `.csv` file
2. Ask questions about the document content
3. View retrieved chunks and verification trace

---

## 📊 Response Status Indicators

| Status | Meaning |
|--------|---------|
| ✅ **Verified** | Answer generated AND confirmed against evidence |
| ⚠️ **Unverified** | Answer generated but NOT fully confirmed — use caution |
| 🚫 **Refused** | No relevant knowledge found — safe denial |

---

## 🔬 Technical Innovations

### 1. Intelligent Retrieval (Block 4)

**Problem solved**: Basic keyword matching failed for stems ("encrypted" != "encryption") and substring collisions ("api" matched "capital").

**Solution**:
- **Stem map**: Maps word variants to KB keys (e.g., encrypted to encryption, secure to security)
- **Best-match scoring**: Scores all keywords, returns highest relevance (not first match)
- **Word-boundary regex**: Prevents false positives like "api" in "capital"
- **Context fusion**: Merges top-2 entries for multi-topic queries

### 2. Structured Post-Processing (Block 6)

**Problem solved**: Flan-T5-Base is extractive — outputs 8-60 char fragments instead of full answers.

**Solution** — `build_structured_answer()` function:
- Takes model's key insight (what it identified as most relevant)
- Scores all context sentences by relevance to the question
- Builds clean bullet-point answers from scored sentences
- Includes false-refusal fallback when model incorrectly refuses valid queries

### 3. Self-Verification Chain (Block 5B)

**Pattern**: Inspired by LangChain's RAG QA patterns — agent validates its own output:

```
Generated Answer + Original Context → [Verification LLM] → SUPPORTED / NOT_SUPPORTED
```

### 4. Tuned Generation Parameters (Block 3)

| Parameter | Value | Purpose |
|-----------|-------|---------|
| max_new_tokens | 512 | Fuller answers |
| num_beams | 4 | Better quality via beam search |
| no_repeat_ngram_size | 3 | Reduces repetition |
| do_sample | False | Deterministic outputs |
| early_stopping | True | Efficient decoding |

---

## 📚 Knowledge Base Coverage

All entries sourced from [help.webex.com](https://help.webex.com):

| Topic | Keywords | Content |
|-------|----------|---------|
| Meetings | meeting, join, schedule | Scheduling, joining, translation, Webex Assistant |
| Messaging | messaging, message | Async collaboration, file sharing, threading |
| Spaces | space, spaces | Workstreams, projects, external collaboration |
| Sharing | share, shared, content | Screen share, immersive share, attachments |
| Audio/Video | noise, noisy, background | Noise removal, virtual backgrounds |
| Security | security, secure, encryption, encrypted | E2E encryption, anti-malware |
| Developer | api, apis, bot, bots, developer | REST API, rate limits, bot accounts |
| AI Assistant | assistant | Voice commands, transcription, highlights |

---

## 🎨 UI Design

- **Theme**: Cisco brand palette
- **Primary**: Cisco Blue #049FD9
- **Secondary**: Cisco Teal #00BCEB
- **Success**: Cisco Green #6CC04A
- **Tertiary**: Cisco Indigo #5B57D1
- **Background**: Professional dark gradient (#0d1117 to #122030)
- **Typography**: Inter font, 15px, high readability
- **Framework**: Gradio 4.44+ with gr.themes.Base() + pure CSS

---

## ⚖️ Ethical AI Principles

1. **Transparency** — Clearly communicates when it cannot answer and when answers are unverified
2. **Truthfulness** — Answers strictly grounded in verified documentation
3. **Self-Accountability** — Agent audits its own responses before presenting
4. **Safety** — Errs on the side of refusal/flagging rather than fabrication
5. **Traceability** — Every answer traced back to source content
6. **Accessibility** — Runs locally without paid APIs — democratized AI safety

---

## 🔮 Future Improvements

| Area | Improvement |
|------|-------------|
| Retrieval | Vector embeddings (FAISS/ChromaDB) for semantic search |
| Knowledge | Live crawling/indexing of help.webex.com |
| Model | Upgrade to Flan-T5-Large (780M) or use free APIs (Groq/Gemini) |
| Verification | Ensemble approach — different model for verification |
| Confidence | Retrieval confidence scoring + verification confidence |
| Multi-modal | Support screenshots of Webex UI |
| Granularity | Nuanced verification (fully / partially / not supported) |

---

## 📖 References

- [LangChain Documentation](https://python.langchain.com/)
- [LangChain RAG QA Patterns](https://python.langchain.com/docs/use_cases/question_answering/)
- [HuggingFace Transformers](https://huggingface.co/docs/transformers)
- [Google Flan-T5](https://huggingface.co/google/flan-t5-base)
- [Gradio Documentation](https://www.gradio.app/docs)
- [Cisco Webex Help Center](https://help.webex.com)

---

*Built with LangChain, Hugging Face Flan-T5, Gradio, and Cisco Palette — Self-Verifying AI Agent, No API Key Required*
