# 🏥 AutoMed: Autonomous Diagnostic AI Agent for Remote Healthcare

AutoMed is an intelligent AI agent designed to assist with preliminary medical diagnostics by combining **LLMs**, **Vision Transformers**, and **Retrieval-Augmented Generation (RAG)**. It enables users to input symptoms or upload medical images (e.g., chest X-rays), then autonomously delivers a probable diagnosis, relevant info, and follow-up suggestions.

> 💡 Built during a 2-week break to demonstrate real-world, cutting-edge AI in healthcare automation.

---

## ✨ Key Features
- 🔍 **Symptom Analysis using LLMs** (e.g. GPT-4 / LLaMA)
- 🖼️ **Medical Image Classification with ViT**
- 📚 **RAG system** to fetch trusted info from medical datasets
- 🌐 **FastAPI** or **Streamlit** web interface
- 🧠 **Multimodal Fusion** for richer diagnoses

---

## 🧠 AI Concepts Used
- Large Language Models (LLMs)
- Vision Transformers (ViT)
- Retrieval-Augmented Generation (RAG)
- Autonomous Agent Workflows
- Prompt Engineering

---

## 🛠️ Tech Stack
| Tool | Use |
|------|-----|
| Python | Core language |
| Hugging Face Transformers | LLMs + Vision Transformers |
| FastAPI / Streamlit | API or frontend UI |
| OpenCV / PIL | Image processing |
| Torch / PyTorch | Model inference |
| FAISS / ChromaDB | RAG vector store |
| LangChain (optional) | Agent orchestration |

---

## 🚀 Getting Started

### 1. Clone the repo
bash
git clone https://github.com/yourusername/automed-ai.git
cd automed-ai


### 2. Install dependencies
bash
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt


### 3. Run basic image classifier (ViT)
bash
python test_vit.py


### 4. Launch web UI (optional)
bash
uvicorn app.main:app --reload  # FastAPI
# or
streamlit run app/ui.py        # Streamlit


---

## 🏗️ Project Architecture

mermaid
graph TD
A[User Input] --> B[Symptom Parser (LLM)]
B --> C[Multimodal Fusion Engine]
D[Uploaded X-ray] --> E[ViT Classifier]
E --> C
C --> F[Diagnosis + Suggested Info (RAG)]
F --> G[UI / API Response]


---

## 🎚 Dataset Info

- **Chest X-ray (Pneumonia/COVID)**: Used for initial image classification tasks.
- **PubMed QA / NIH Articles**: Used as the RAG knowledge base for information retrieval.

---

## 🧪 Sample Use Cases
- **Input**: "I'm coughing and feel short of breath."
- **Image**: Chest X-ray uploaded
- **Output**: "Possible pneumonia detected with 78% confidence. Consider a visit to a clinic. Learn more → [link]"

---

## ⚠️ Disclaimer
AutoMed is **not a replacement for medical professionals**. It is a prototype research tool for exploring AI-assisted diagnostics.

---

## 📌 To-Do
- [ ] Integrate full symptom-to-diagnosis pipeline
- [ ] Implement custom-trained ViT on Chest X-ray data
- [ ] Add RAG medical info retriever
- [ ] Finalize UI + deploy to cloud

---

## 👩‍💻 Built By
- [Manar Ibrahim](https://yourportfolio.com)
- 🗓️ April 2025 (Trimester Break Project)

---

## 📜 License
MIT License – see LICENSE file for details.

