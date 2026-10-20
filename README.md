<div align="center">

# 🤖 SMARRtino RAG System
### Local LLM & FAISS Vector Database Architecture for Educational Robotics

![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Ollama](https://img.shields.io/badge/Ollama-Gemma_3_(4B)-000000?style=for-the-badge&logo=ollama&logoColor=white)
![FAISS](https://img.shields.io/badge/Vector_DB-FAISS-00599C?style=for-the-badge)
![Flask](https://img.shields.io/badge/Server-Flask-000000?style=for-the-badge&logo=flask&logoColor=white)
![ROS2](https://img.shields.io/badge/Robotics-ROS2_Ready-22314E?style=for-the-badge&logo=ros&logoColor=white)

**Official Retrieval-Augmented Generation (RAG) server designed for SMARRtino, the 100% Made in Italy social and educational robot developed by Robotics 3D.**

</div>

---

## 📖 Overview

This repository provides a complete **Retrieval-Augmented Generation (RAG)** backend that allows the **SMARRtino** robot to dynamically learn from external **PDF documents** and answer user queries in real time with high accuracy and zero hallucinations.

By combining **LangChain**, a local **FAISS** vector database, and a custom fine-tuned **Ollama** language model, the system transforms static documents (school lessons, fairy tales, rhymes, and book catalogs) into an interactive conversational knowledge base.

---

## 🌐 System Architecture & Remote SSH Offloading

Running modern Large Language Models (LLMs) and high-dimensional vector searches requires substantial computational power that exceeds the onboard hardware limits of a standalone mobile robot. 

To overcome this, the project implements a **Remote Compute Offloading Architecture**:

| Component | Role | Description |
| :--- | :--- | :--- |
| 🤖 **SMARRtino Robot** | *Client / ROS2 Node* | Captures voice and text queries from students, connects via **SSH / Local Network** to the host workstation, forwards the request, and converts the server's response into speech (TTS) or physical movement commands. |
| 🖥️ **GPU Workstation** | *Host Server* | A high-performance machine equipped with an **NVIDIA GPU** that runs the **Flask API server** (`port 8060`), manages the **FAISS vector store**, and executes local LLM inference via **Ollama** (`port 11434`). |

---

## 🧠 How the RAG Pipeline Works

1. **📄 PDF Ingestion & Chunking:** Users upload PDF files via the `/manage` web dashboard. The server extracts the raw text and splits it into structured semantic chunks.
2. **🔢 Vector Embeddings & FAISS Indexing:** Each text chunk is converted into numerical vectors using HuggingFace's `sentence-transformers/all-MiniLM-L6-v2` model and saved locally inside a **FAISS** index (`data/vectors/`).
3. **🔍 Semantic Retrieval & Generation:** When a question is received, the server retrieves the most relevant chunks from FAISS and injects them as context into the local LLM prompt, generating a concise, child-friendly response.

---

## 🛠️ Custom Fine-Tuned Model (`smarrtino`)

To ensure the robot maintains its authentic personality—a friendly, 3D-printed educational robot created in Rome by **Robotics 3D**—this RAG architecture runs on a custom fine-tuned **Gemma 3 (4B)** model named **`smarrtino`**.

> 🔗 **SMARRtino Fine-Tuning Repository:**  
> The complete QLoRA fine-tuning pipeline, dataset generation, Unsloth training scripts, and GGUF export instructions are available in my dedicated repository:  
> 👉 **[fra-cir06/Smarrtino_llm_FineTuning](https://github.com/fra-cir06/Smarrtino_llm_FineTuning)**

---

## 💻 Hardware & Software Prerequisites

Because all AI inference and vector searches are offloaded to the host workstation, the server requires:

* **GPU:** NVIDIA GPU with at least **8 GB of VRAM** (*12 GB+ recommended for ultra-low latency*).
* **CPU & RAM:** Modern multi-core CPU (Intel Core i7/i9 or AMD Ryzen 7/9) and **16 GB+ RAM**.
* **OS:** Windows 10/11 or Linux (Ubuntu 22.04 / 24.04).
* **Software:** **Python 3.10+** and **Ollama** installed and running locally.

---

## 🚀 1. Installation & Environment Setup

### Step 1.1: Clone the Repository
Open your terminal on the host workstation and run:

```bash
git clone [https://github.com/fra-cir06/SMARRtino_RAG.git](https://github.com/fra-cir06/SMARRtino_RAG.git)
cd SMARRtino_RAG
```

### Step 1.2: Create and Activate the Virtual Environment

**On Windows (CMD / PowerShell):**
```powershell
python -m venv myenv
.\myenv\Scripts\activate
```

**On Linux / WSL2:**
```bash
python3 -m venv myenv
source myenv/bin/activate
```

### Step 1.3: Install Dependencies
With the virtual environment active, install all required Python packages:

```bash
pip install --upgrade pip
pip install flask requests fpdf pypdf
pip install langchain langchain-community langchain-core langchain-text-splitters
pip install langchain-huggingface langchain-ollama
pip install sentence-transformers faiss-cpu
```

### Step 1.4: Verify Ollama Models
Ensure Ollama is running and that your custom **`smarrtino`** model is installed:

```bash
ollama list
```

---

## 🖥️ 2. Application Versions & Usage

This repository includes **three specialized versions** of the application, each designed for a specific operational scenario:

| File | Version Name | LLM Used | Key Features & Use Case |
| :--- | :--- | :--- | :--- |
| **`app2.py`** | **Standard PDF Q&A** | Base LLM | Pure document Question-Answering over uploaded PDFs without the custom robot persona. |
| **`app3.py`** | **SMARRtino Robot RAG** | `smarrtino` *(Fine-Tuned)* | **Main Robot App.** Integrates the fine-tuned `smarrtino` persona, handles physical robot commands (`avanti`, `dietro`, `destra`, `sinistra`), extracts children's rhymes, and generates concise responses for ROS2 TTS. |
| **`app4.py`** | **Librarian Assistant** | Configurable LLM | **"Assistente Bibliotecario".** Tailored to help users browse, catalog, and query book collections and library resources indexed in FAISS. |

### How to Start the Server
Run the desired version from your terminal:

```bash
# To launch the standard PDF RAG:
python app2.py

# To launch the SMARRtino Fine-Tuned Robot RAG:
python app3.py

# To launch the Librarian Assistant:
python app4.py
```

---

## 🌐 3. Web Dashboard & API Endpoints

Once launched, the Flask server listens on `http://0.0.0.0:8060`:

* 💬 **Main Chat UI:** `http://localhost:8060/` — Interactive interface to test queries and switch models.
* 📂 **PDF & FAISS Manager:** `http://localhost:8060/manage` — Upload PDFs, index chunks into FAISS, or remove files.
* 🔎 **Chunk Inspector:** `http://localhost:8060/chunks` — View, search, and export indexed text chunks.
* 🤖 **ROS2 / Robot API Endpoints:**
  * `GET http://<HOST_IP>:8060/get?msg=<your_question>&model=smarrtino` *(Plain-text response)*
  * `GET http://<HOST_IP>:8060/json?query=<your_question>&model=smarrtino` *(JSON response)*
