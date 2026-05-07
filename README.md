# AutoMed 🩺
**Autonomous multimodal diagnostic AI agent for remote healthcare triage**
 
![Status](https://img.shields.io/badge/status-in%20active%20development-yellow)
![Python](https://img.shields.io/badge/python-3.10+-blue)
![PyTorch](https://img.shields.io/badge/PyTorch-2.x-orange)
![License](https://img.shields.io/badge/license-MIT-green)
 
AutoMed is a multimodal AI system for preliminary medical triage. It accepts natural language symptom descriptions, audio input, and chest X-ray images — runs them through a three-agent pipeline (symptom analysis, image classification, RAG-grounded reasoning) — and returns a structured diagnostic suggestion with confidence scores, risk level, and next-step recommendations.
 
> ⚠️ **Disclaimer:** AutoMed is an academic research project. It is not validated for clinical use and should never substitute professional medical judgment.
 
---
 
## Motivation
 
Primary care access remains limited across much of the MENA region and the developing world. Patients often wait days for basic triage. AutoMed explores how multimodal AI can assist in early-stage symptom triage — not replacing doctors, but helping patients understand urgency and next steps faster, particularly in under-resourced settings.
 
---
 
## System Architecture
 
AutoMed is structured as a **three-agent pipeline** where each agent handles a distinct modality, and a fusion engine combines their outputs into a final diagnostic response.
 
```
┌─────────────────────────────────────────────────────────────┐
│                        Input Layer                          │
│                                                             │
│   Text symptoms    Audio input     X-ray image    History   │
│        │               │               │              │     │
│        │         Whisper ASR           │              │     │
│        │               │               │              │     │
└────────┼───────────────┼───────────────┼──────────────┼─────┘
         │               │               │              │
         ▼               ▼               │              │
┌─────────────────────┐  │  ┌────────────▼───────────┐  │
│    Agent 1          │  │  │    Agent 2             │  │
│  Symptom Analyzer   │◄─┘  │  Image Classifier      │  │
│                     │     │                        │  │
│  LLM extracts:      │     │  ViT (BEiT) outputs:   │  │
│  - symptom list     │     │  - disease class       │  │
│  - severity flags   │     │  - confidence score    │  │
│  - risk keywords    │     │                        │  │
└────────┬────────────┘     └───────────┬────────────┘  │
         │                              │               │
         └──────────────┬───────────────┘               │
                        ▼                               │
              ┌─────────────────────┐                   │
              │    Agent 3          │◄──────────────────┘
              │  RAG Reasoning      │
              │                     │
              │  FAISS retrieval    │
              │  + PubMed / WHO     │
              │  + patient history  │
              └────────┬────────────┘
                       │
                       ▼
            ┌──────────────────────┐
            │   Diagnosis Output   │
            │                      │
            │  Ranked diagnoses    │
            │  Confidence scores   │
            │  Risk level          │
            │  Next-step actions   │
            └────────┬─────────────┘
                     │
          ┌──────────┴──────────┐
          │   FastAPI Backend   │
          │   Streamlit UI      │
          └─────────────────────┘
```
 
---
 
## Sample Output (Target)
 
```
Input:
  Symptoms: "chest tightness, shortness of breath, dry cough"
  Image:    chest_xray_001.jpg
 
─────────────────────────────────────────
AutoMed Diagnostic Report
─────────────────────────────────────────
Extracted symptoms:
  - Chest tightness       [HIGH severity]
  - Shortness of breath   [HIGH severity]
  - Dry cough             [MODERATE severity]
 
Image analysis:
  - Bilateral infiltrates detected
  - ViT confidence: 0.81
 
Ranked diagnoses:
  1. Pneumonia        — 81%
  2. Bronchitis       — 54%
  3. COVID-19         — 38%
 
Risk level:   MODERATE–HIGH
Action:       Seek medical review within 24 hours.
              Suggested tests: CBC, chest CT, SpO2 monitoring.
─────────────────────────────────────────
```
> This output is illustrative. Actual model performance depends on training data and validation.
 
---
 
## Design Decisions
 
**Why a multi-agent structure instead of one model?**
Each modality (text, image, structured history) has different data characteristics and requires different model architectures. Separating them into agents makes the system modular — each agent can be improved, swapped, or evaluated independently without breaking the others. It also mirrors how real clinical reasoning works: a physician weighs imaging results separately from a verbal history before synthesizing a conclusion.
 
**Why ViT / BEiT over CNN for X-ray classification?**
Vision Transformers capture long-range spatial dependencies via self-attention across the full image, which is important for diffuse pathologies (e.g., bilateral pneumonia) that CNNs with local receptive fields can miss. BEiT's masked image modelling pre-training also makes it more data-efficient when fine-tuning on limited medical datasets.
 
**Why RAG over a fine-tuned medical LLM?**
Fine-tuning a domain-specific LLM requires large labeled clinical corpora and significant compute. RAG lets us ground outputs in a retrievable, auditable knowledge base (PubMed abstracts, WHO/CDC guidelines) that can be updated without retraining. Every claim in the output traces to a retrieved source — which matters for explainability in medical AI.
 
**Why MongoDB for patient history?**
Patient records are semi-structured — different patients have different fields, conditions, and history depth. MongoDB's flexible document schema handles this better than a rigid relational model and integrates naturally with the context injection pattern used in the RAG step.
 
---
 
## Tech Stack
 
| Layer | Technology |
|---|---|
| Language | Python 3.10+ |
| ML Framework | PyTorch |
| Vision Model | HuggingFace BEiT / ViT (`microsoft/beit-base-patch16-224`) |
| LLM | HuggingFace Transformers (BioMedLM / Mistral-7B) |
| Speech-to-Text | OpenAI Whisper |
| RAG | FAISS + LangChain + sentence-transformers |
| Image Processing | OpenCV |
| API | FastAPI |
| Database | MongoDB |
| Frontend | Streamlit (MVP) |
 
---
 
## Repository Structure
 
```
automed/
├── agents/
│   ├── symptom_agent.py         # Agent 1: LLM symptom extraction
│   ├── image_agent.py           # Agent 2: ViT/BEiT image classification
│   └── rag_agent.py             # Agent 3: RAG retrieval + reasoning
├── core/
│   ├── fusion.py                # Combines agent outputs → diagnosis
│   ├── risk_scorer.py           # Risk level classification
│   └── preprocessors/
│       ├── image_prep.py        # OpenCV X-ray preprocessing
│       └── audio_prep.py        # Whisper audio transcription
├── api/
│   └── main.py                  # FastAPI routes
├── frontend/
│   └── app.py                   # Streamlit UI
├── db/
│   └── patient_history.py       # MongoDB patient context store
├── notebooks/
│   ├── 01_beit_exploration.ipynb    # BEiT model loading + inference
│   ├── 02_rag_prototype.ipynb       # FAISS retrieval prototype
│   └── 03_symptom_extraction.ipynb # LLM symptom parsing tests
├── data/
│   └── sample_xrays/            # Sample test images (not committed)
├── requirements.txt
├── .env.example
└── README.md
```
 
---
 
## Getting Started
 
### Prerequisites
 
- Python 3.10+
- MongoDB (local or Atlas free tier)
- HuggingFace account (for model access)
- GPU recommended for ViT inference — CPU works for development
### Installation
 
```bash
git clone https://github.com/ManarIbrahim206/automed-ai.git
cd automed-ai
 
python -m venv venv
source venv/bin/activate       # Windows: venv\Scripts\activate
 
pip install -r requirements.txt
cp .env.example .env
# Add MONGO_URI and HUGGINGFACE_TOKEN to .env
```
 
### Run
 
```bash
# Backend
uvicorn api.main:app --reload
 
# Frontend
streamlit run frontend/app.py
```
 
---
 
## Datasets
 
| Dataset | Use |
|---|---|
| NIH Chest X-ray14 | Primary X-ray classification training |
| COVID-19 Radiography Database | Supplemental validation data |
| PubMedQA | RAG knowledge base for diagnostic retrieval |
| WHO / CDC clinical guidelines | RAG grounding for recommendation engine |
 
---
 
## Real-World Deployment Considerations
 
AutoMed is designed with a realistic path from research prototype to deployed product:
 
- **Phase 1 — Direct users:** Freemium web app. Free tier for basic triage, paid tier for detailed reports, history tracking, and priority analysis.
- **Phase 2 — B2B:** API access for telemedicine platforms and private clinics needing AI-assisted pre-triage.
- **Phase 3 — Enterprise:** Integration with hospital and government health systems in markets with limited specialist access (MENA, Sub-Saharan Africa).
The multi-agent architecture was chosen partly with this in mind — each agent can be licensed, updated, or replaced independently as deployment requirements change.
 
---
 
## Ethical Considerations
 
- **Not clinically validated.** All outputs are probabilistic suggestions, not diagnoses.
- **Dataset bias.** Medical imaging datasets are skewed toward certain demographics and equipment types. Model performance varies across populations.
- **Hallucination risk.** RAG reduces but does not eliminate LLM hallucination. All outputs should be treated as a starting point for professional review.
- **Data privacy.** Production deployment requires compliance with applicable health data regulations (UAE Health Data Law, HIPAA, GDPR equivalents).
---
 
## Roadmap
 
- [ ] BEiT fine-tuning pipeline on NIH Chest X-ray14
- [ ] FAISS RAG retrieval with PubMedQA corpus
- [ ] LLM symptom extraction agent
- [ ] Fusion engine combining all three agents
- [ ] Risk scoring module
- [ ] FastAPI endpoints for multimodal input
- [ ] MongoDB patient history integration
- [ ] Streamlit demo interface
- [ ] End-to-end integration test
- [ ] Deploy demo to HuggingFace Spaces
---
 
## Author
 
**Manar Ibrahim**
BSc AI & Business Information Technology — Murdoch University Dubai
 
[![LinkedIn](https://img.shields.io/badge/LinkedIn-connect-blue)](https://linkedin.com/in/YOUR_HANDLE)
[![Email](https://img.shields.io/badge/email-albadawe.manar%40gmail.com-lightgrey)](mailto:albadawe.manar@gmail.com)
 
---
 
## License
 
MIT License — see [LICENSE](LICENSE) for details.
 

