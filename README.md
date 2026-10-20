cat << 'EOF' > README.md

\# 🤖 SMARRtino RAG System - Local LLM \& Vector Database Architecture



Official repository for the \*\*Retrieval-Augmented Generation (RAG)\*\* system designed for \*\*SMARRtino\*\*, the social and educational robot developed by \*\*Robotics 3D\*\*.



This project enables the robot to dynamically learn from external \*\*PDF documents\*\* using a \*\*FAISS vector database\*\* and provide accurate, context-aware responses in real time.



\---



\## 🌐 System Architecture \& Remote SSH Offloading



Running modern Large Language Models (LLMs) and vector similarity searches requires significant computational resources that exceed the onboard hardware capabilities of a standalone mobile robot. 



To solve this, the system implements a \*\*Remote Compute Offloading Architecture\*\*:

\* \*\*The Robot (Client / ROS2 Node):\*\* Captures voice/text queries from users and children, connects via \*\*SSH / Local Network\*\* to a dedicated high-performance host machine, sends the query to the Flask/RAG server, and receives the generated response to be spoken aloud via Text-to-Speech (TTS) or executed as a physical movement command.

\* \*\*The Host Machine (Server):\*\* A powerful workstation equipped with a dedicated \*\*NVIDIA GPU\*\* that runs \*\*Ollama\*\*, the \*\*FAISS vector store\*\*, and the \*\*Flask web application\*\* (`port 8060`).



\---



\## 🧠 What is RAG and How Does It Work Here?



\*\*Retrieval-Augmented Generation (RAG)\*\* is an AI framework that enhances the capabilities of a Large Language Model by connecting it to an external, authoritative knowledge base before generating a response:



1\. \*\*PDF Ingestion \& Chunking:\*\* Users upload PDF documents through the `/manage` web interface\[cite: 1, 5, 6]. The system extracts the text and splits it into structured semantic chunks\[cite: 2, 4, 7].

2\. \*\*Vector Embeddings \& FAISS Indexing:\*\* Each text chunk is converted into high-dimensional numerical vectors using HuggingFace's `sentence-transformers/all-MiniLM-L6-v2` embedding model and stored locally in a \*\*FAISS (Facebook AI Similarity Search)\*\* vector database\[cite: 2, 4, 7].

3\. \*\*Semantic Retrieval \& Generation:\*\* When a user asks a question, the system searches the FAISS database for the most relevant text chunks and feeds them—alongside the user's question—to the local LLM via \*\*Ollama\*\*, ensuring accurate, grounded, and hallucination-free answers\[cite: 2, 4, 7].



\---



\## 🛠️ Custom Fine-Tuned Model (`smarrtino`)



To ensure the robot maintains its authentic persona (a friendly, 3D-printed educational robot created in Rome by Robotics 3D) while answering questions\[cite: 4, 7], this RAG pipeline utilizes a custom fine-tuned \*\*Gemma 3 (4B)\*\* model named \*\*`smarrtino`\*\*\[cite: 5, 6].



> 🔗 \*\*Model Training Repository:\*\* The complete QLoRA fine-tuning pipeline, dataset formatting, Unsloth training scripts, and GGUF export instructions used to build the `smarrtino` model are available in the dedicated repository on my GitHub profile: \*\*\[SMARRtino Fine-Tuning Repository](https://github.com/YOUR\_USERNAME/Smarrtino\_llm)\*\* \*(replace with your exact repo link)\*.



\---



\## 💻 Hardware \& Software Prerequisites



Because all LLM inference and vector indexing are offloaded to the host machine, the server requires capable hardware:

\* \*\*GPU:\*\* NVIDIA GPU with at least \*\*8 GB of VRAM\*\* (12 GB+ recommended for fast inference and low latency).

\* \*\*CPU \& RAM:\*\* Modern multi-core processor (Intel Core i7/i9 or AMD Ryzen 7/9) and at least \*\*16 GB of RAM\*\*.

\* \*\*OS:\*\* Windows 10/11 or Linux (Ubuntu 22.04/24.04).

\* \*\*Software:\*\* 

&#x20; \* \*\*Python\*\* `3.10+`

&#x20; \* \*\*Ollama\*\* installed and running locally on port `11434`\[cite: 2, 4, 7].



\---



\## 🚀 1. Environment Setup \& Installation



Clone this repository on the host workstation and navigate into the project directory:



```bash

git clone \[https://github.com/YOUR\_USERNAME/SMARRtino\_RAG.git](https://github.com/YOUR\_USERNAME/SMARRtino\_RAG.git)

cd SMARRtino\_RAG

```



\### Create and Activate the Python Virtual Environment (`venv`)



\*\*On Linux / WSL2:\*\*

```bash

python3 -m venv venv

source venv/bin/activate

```



\*\*On Windows (PowerShell):\*\*

```powershell

python -m venv venv

.\\venv\\Scripts\\Activate.ps1

```



\### Install Required Dependencies

Upgrade `pip` and install all necessary Python packages for Flask, LangChain, FAISS, HuggingFace Embeddings, and PDF processing:



```bash

pip install --upgrade pip

pip install flask requests fpdf pypdf

pip install langchain langchain-community langchain-core langchain-text-splitters

pip install langchain-huggingface langchain-ollama

pip install sentence-transformers faiss-cpu

```



\### Verify Ollama Models

Make sure Ollama is running and that your custom \*\*`smarrtino`\*\* model (as well as any base fallback models like `gemma3:4b` or `mistral`) is registered in Ollama\[cite: 1, 5, 6]:



```bash

ollama list

```



\---



\## 🖥️ 2. Application Versions \& How to Run



This repository includes \*\*three distinct versions\*\* of the application, tailored for different use cases and operational environments:



\### 🔹 Version 1: `app2.py` — Standard PDF Q\&A (Base LLM)

\* \*\*Description:\*\* Designed purely for standard document Question-Answering over uploaded PDFs. It uses a general-purpose base LLM (without the custom `smarrtino` fine-tuned persona).

\* \*\*How to run:\*\*

&#x20; ```bash

&#x20; python app2.py

&#x20; ```



\### 🔹 Version 2: `app3.py` — SMARRtino Interactive Robot RAG (Fine-Tuned LLM)

\* \*\*Description:\*\* The core interactive version for the \*\*SMARRtino\*\* robot\[cite: 5, 6]. It integrates the custom fine-tuned \*\*`smarrtino`\*\* model from Ollama, handles direct physical robot movement commands (`avanti`, `dietro`, `destra`, `sinistra`, arm movements), retrieves children's rhymes/poems from the FAISS vector database, and enforces concise, speech-friendly responses for ROS2 Text-to-Speech integration\[cite: 1, 2, 4, 5, 6, 7].

\* \*\*How to run:\*\*

&#x20; ```bash

&#x20; python app3.py

&#x20; ```



\### 🔹 Version 3: `app4.py` — Librarian Assistant ("Assistente Bibliotecario")

\* \*\*Description:\*\* Specialized application mode tailored to act as an intelligent \*\*Librarian Assistant\*\*, helping users browse, catalog, and query book collections, summaries, and library resources indexed in the vector store.

\* \*\*How to run:\*\*

&#x20; ```bash

&#x20; python app4.py

&#x20; ```



\---



\## 🌐 3. Web Interface \& API Endpoints



Once you launch any of the application versions, the Flask server will start listening on `http://0.0.0.0:8060`\[cite: 1, 5, 6]:



\* \*\*Main Chat Interface:\*\* `http://localhost:8060/` — Interactive web UI to test questions and select installed Ollama models\[cite: 1, 5, 6].

\* \*\*PDF \& Vector Database Manager:\*\* `http://localhost:8060/manage` — Upload new PDFs, trigger FAISS chunk ingestion, or delete existing documents\[cite: 1, 5, 6].

\* \*\*Inspect Indexed Chunks:\*\* `http://localhost:8060/chunks` — View, search, and export the text chunks currently stored in FAISS\[cite: 1, 5, 6].

\* \*\*Robot / External Client Endpoints:\*\*

&#x20; \* `GET /get?msg=<your\_question>\&model=smarrtino` (Plain-text response)\[cite: 1, 5, 6]

&#x20; \* `GET /json?query=<your\_question>\&model=smarrtino` (JSON response for ROS2 integration)\[cite: 1, 5, 6]

EOF

