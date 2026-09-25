An AI-powered Telegram assistant that helps post-surgery patients remember their aftercare instructions (diet, medication, mobility, wound care) — reducing repetitive calls to the hospital and flagging emergencies early.
 
---
 
## The Problem (Before)
 
After being discharged from the hospital, patients often:
- Forget the instructions given to them (what to eat, when to take medication, how to move safely).
- Call the hospital repeatedly with the same, already-answered questions.
- Sometimes miss real warning signs because they don't know which symptoms are urgent.
This creates unnecessary load on hospital staff and, in some cases, delays needed medical attention.
 
## The Solution (Now)
 
A Telegram bot built with **n8n** that uses **RAG (Retrieval-Augmented Generation)** to answer patients strictly from verified medical documents (per surgery type), and escalates to emergency care when needed.
 
The bot:
- Answers only using retrieved, hospital-approved documents (no hallucinated medical advice).
- Cites which document/surgery type an answer came from.
- Replies "I don't have confirmed information about this — please ask your doctor" when the answer isn't in the knowledge base.
- Immediately tells the patient to go to the ER or call the hospital if they mention a dangerous symptom (heavy bleeding, difficulty breathing, sudden severe pain, etc.).
- Clarifies at all times that it's an informational assistant, not a substitute for medical evaluation.
---
 
## Architecture
 
Two independent n8n workflows, connected only through a shared Supabase table:
 
### 1. Ingestion Workflow (run manually / on document update)
```
Google Drive (source document)
   → Default Data Loader
   → Recursive Character Text Splitter
   → Embeddings (Google Gemini)
   → Supabase Vector Store (insert)
```
 
### 2. Chat Workflow (always active)
```
Telegram Trigger (patient message)
   → AI Agent
       ├── Chat Model: Google Gemini
       ├── Memory: Simple Memory (session = Telegram chat ID)
       └── Tool: Supabase Vector Store (retrieve)
   → Telegram (send reply)
```
 
Both workflows read/write the **same Supabase table**, which is the only link between them.
 
---
 
## Tech Stack
 
- **n8n** — workflow orchestration
- **Telegram Bot API** — patient-facing chat interface
- **Google Gemini** — chat model + embeddings
- **Supabase (pgvector)** — vector store for document retrieval
- **Google Drive** — source document storage for the knowledge base
---
 
## Safety & Guardrails
 
The AI Agent's system prompt enforces:
1. Answer only from retrieved documents — never from general/model knowledge.
2. If the information isn't found, say so clearly instead of guessing.
3. Always mention the source/surgery type behind an answer.
4. Detect dangerous symptoms and instruct the patient to seek emergency care immediately.
5. Remind the patient this is an informational tool, not a replacement for a doctor.
---
 
## Status
 
- ✅ Ingestion pipeline built and tested (Google Drive → Supabase Vector Store)
- ✅ Chat pipeline built (Telegram → AI Agent → Supabase retrieval → reply)
- 🔄 Currently validating the full pipeline with a neutral test document before loading real, doctor-reviewed medical content
- ⏳ Medical source documents pending clinical review before going live
---
 
## Disclaimer
 
This project is an informational support tool only. It is **not** a medical device and does **not** replace professional medical advice, diagnosis, or treatment. All medical content must be reviewed and approved by qualified healthcare professionals before deployment.
 
