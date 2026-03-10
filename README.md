<div align="center">

# 🇮🇳 Vani — AI Voice Helpline for Indian Government Schemes

**Ask about any government scheme in your language — by voice or WhatsApp.**

[![Python](https://img.shields.io/badge/Python-3.12-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://python.org)
[![AWS](https://img.shields.io/badge/AWS-Bedrock%20%7C%20Lambda%20%7C%20Transcribe-FF9900?style=for-the-badge&logo=amazon-aws&logoColor=white)](https://aws.amazon.com)
[![Qdrant](https://img.shields.io/badge/Qdrant-Hybrid%20Vector%20DB-DC244C?style=for-the-badge&logo=qdrant&logoColor=white)](https://qdrant.tech)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.110-009688?style=for-the-badge&logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com)
[![Twilio](https://img.shields.io/badge/Twilio-Voice%20%7C%20WhatsApp-F22F46?style=for-the-badge&logo=twilio&logoColor=white)](https://twilio.com)
[![Azure](https://img.shields.io/badge/Azure-Neural%20TTS-0078D4?style=for-the-badge&logo=microsoft-azure&logoColor=white)](https://azure.microsoft.com)
[![Docker](https://img.shields.io/badge/Docker-Container%20Lambda-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://docker.com)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge)](LICENSE)

*Built for the **AWS AI for Bharat Hackathon** — making government welfare accessible to every Indian citizen*

</div>

---

## 🎯 Problem Statement

Millions of Indians are eligible for government welfare schemes but never receive benefits — not because the schemes don't exist, but because **finding and understanding them is too hard**. Language barriers, low digital literacy, and complex eligibility rules lock out the people who need help most.

**Vani** solves this with a simple phone call or WhatsApp message — no app, no internet literacy, no English required.

---

## ✨ What it does

Vani is a **multilingual AI voice helpline** that lets citizens query 500+ Indian government welfare schemes (central + state) over a phone call or WhatsApp message and receive a spoken answer in their own language — in under 5 seconds.

```
📱 User calls Twilio number / sends WhatsApp message
            │
            ▼
    🔗 Twilio webhook  (FastAPI / AWS Lambda)
            │
            ▼
    🎙️ AWS Transcribe  ──→  question text (any Indian language)
            │
            ▼
    🔍 Qdrant Hybrid Search  (BM25 + BGE-large dense, RRF fusion)
       22,000+ scheme chunks across 500+ schemes
            │
            ▼
    🧠 AWS Bedrock Nova Lite  ──→  concise, citizen-friendly answer
            │
            ▼
    🔊 Azure Neural TTS  ──→  spoken reply in user's language
            │
            ▼
    📲 Twilio delivers audio back to the user
```

---

## 🌐 Languages Supported

| 🗣️ Language | 🔊 Azure Neural Voice |
|---|---|
| 🇬🇧 English (India) | `en-IN-NeerjaNeural` |
| 🇮🇳 Hindi | `hi-IN-SwaraNeural` |
| Telugu | `te-IN-ShrutiNeural` |
| Tamil | `ta-IN-PallaveNeural` |
| Kannada | `kn-IN-SapnaNeural` |
| Malayalam | `ml-IN-SobhanaNeural` |
| Marathi | `mr-IN-AarohiNeural` |
| Bengali | `bn-IN-TanishaaNeural` |
| Gujarati | `gu-IN-DhwaniNeural` |
| Punjabi | `pa-IN-VaaniNeural` |
| Odia | `or-IN-SubhasiniNeural` |
| Assamese | `as-IN-YashicaNeural` |

---

## 🏗️ Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                       ☁️  AWS Cloud                          │
│                                                              │
│  🌐 API Gateway ──► ⚡ Lambda (vani-jan-webhook / main.py)   │
│                          │                                   │
│                          │  boto3.invoke                     │
│                          ▼                                   │
│              📦 Lambda Container (lambda_function.py)        │
│              ├── 🧮 fastembed  (BGE-large + BM25)            │
│              ├── 🗄️  Qdrant Cloud  (22k vectors, RRF)        │
│              ├── 🧠 Bedrock Nova Lite  (answer generation)   │
│              └── 🔊 Azure Speech  (Neural TTS)               │
│                                                              │
│  🪣 S3  ──► 🎙️ AWS Transcribe  (voice-to-text)               │
└──────────────────────────────────────────────────────────────┘

        ▲  📞 Twilio Voice & 💬 WhatsApp webhooks
        │
   👤 User (phone / WhatsApp)
```

### 🚀 Deployment Options

| Mode | Entry Point | When to Use |
|---|---|---|
| ⚡ **Lambda container** | `lambda_voice_rag/lambda_function.py` | Production — serverless, zero cold-start (model baked in) |
| 🖥️ **FastAPI server** | `voice_rag_server.py` | Local dev / ECS Fargate / App Runner |
| 💬 **WhatsApp Lambda** | `main.py` | Twilio WhatsApp webhook (zip-deployed Lambda) |

---

## 📂 Repository Structure

```
.
├── 🖥️  voice_rag_server.py          # FastAPI server (Voice + WhatsApp)
├── 💬  main.py                      # WhatsApp Lambda handler
├── ⚡  lambda_voice_rag/
│   └──     lambda_function.py       # Container Lambda (RAG engine)
├── 🧠  rag/                         # Reusable RAG pipeline modules
│   ├──     config.py
│   ├──     embedder.py
│   ├──     retriever.py
│   ├──     reranker.py
│   ├──     llm_client.py
│   ├──     pipeline.py
│   ├──     indexer.py
│   └──     query_processor.py
├── 📥  upsert_telangana_schemes.py  # Data ingestion — add schemes to Qdrant
├── ✂️   add_text_for_embedding.py    # Chunk & enrich scheme text before upsert
├── 🔎  probe_qdrant.py              # CLI tool to inspect / query Qdrant
├── 🏗️   preload_models.py            # Bakes BGE model into Docker image
├── 🐳  Dockerfile.lambda            # Lambda container image
├── 🐳  Dockerfile.server            # ECS / App Runner server image
├── 🐳  docker-compose.yml           # Local dev stack (Qdrant + FastAPI)
├── 🚀  deploy_lambda.ps1            # One-command ECR push + Lambda update
├── 📦  build_lambda.py              # Builds zip packages for Lambdas
├── 📋  requirements.txt             # Full dev dependencies
├── 📋  requirements_lambda.txt      # Slim Lambda / container dependencies
└── 🎬  ffmpeg_linux_amd64           # ffmpeg binary (audio conversion in Lambda)
```

---

## ⚙️ Prerequisites

| ☁️ Service | 🎯 Purpose |
|---|---|
| ![AWS](https://img.shields.io/badge/AWS-FF9900?logo=amazon-aws&logoColor=white) AWS account | Bedrock (Nova Lite), Transcribe, Lambda, ECR, S3 |
| ![Qdrant](https://img.shields.io/badge/Qdrant-DC244C?logo=qdrant&logoColor=white) Qdrant Cloud | Vector database (22k+ scheme vectors) |
| ![Twilio](https://img.shields.io/badge/Twilio-F22F46?logo=twilio&logoColor=white) Twilio account | Phone number / WhatsApp sender |
| ![Azure](https://img.shields.io/badge/Azure-0078D4?logo=microsoft-azure&logoColor=white) Azure Cognitive Services | Neural TTS (12 Indian languages) |

---

## 🚀 Quick Start

### 1️⃣ Clone & Install

```bash
git clone https://github.com/your-username/vani-gov-schemes.git
cd vani-gov-schemes

python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate

pip install -r requirements.txt
```

### 2️⃣ Configure Environment

```bash
cp .env.example .env
# Fill in all values in .env (Twilio, Azure, AWS, Qdrant)
```

### 3️⃣ Run Locally (FastAPI + local Qdrant)

```bash
# Start Qdrant locally via Docker
docker-compose up qdrant-local

# Run the voice RAG server
uvicorn voice_rag_server:app --host 0.0.0.0 --port 8000

# Expose to internet for Twilio webhooks
ngrok http 8000
# Set Twilio Voice webhook → https://<ngrok-url>/voice/incoming
```

### 4️⃣ Deploy to AWS Lambda (container)

```bash
# Build & push container image, update Lambda function
.\deploy_lambda.ps1
```

---

## 🗄️ Vector Database

Schemes are stored in **Qdrant Cloud** as a hybrid collection (`schemes_hybrid`) with two named vectors per chunk:

| 🔢 Vector | 🤖 Model | 📐 Dimension | 🏷️ Type |
|---|---|---|---|
| `dense` | `BAAI/bge-large-en-v1.5` | 1024 | Dense |
| `sparse` | `Qdrant/bm25` | variable | Sparse |

Retrieval uses **Reciprocal Rank Fusion (RRF)** over both vectors to combine semantic similarity and keyword matching — especially important for scheme names, eligibility criteria, and income limits.

To add new schemes:

```bash
# Edit upsert_telangana_schemes.py with your scheme data, then:
python upsert_telangana_schemes.py
```

---

## 🧠 RAG Pipeline (`rag/`)

| 📄 Module | 🎯 Role |
|---|---|
| `config.py` | Centralised config (models, Qdrant URL, retrieval knobs) |
| `embedder.py` | Dense + sparse embedding via fastembed |
| `retriever.py` | Qdrant hybrid search with RRF fusion |
| `reranker.py` | Cross-encoder reranking (`BGE-reranker-v2-m3`) |
| `llm_client.py` | Bedrock / Groq LLM abstraction |
| `pipeline.py` | End-to-end query → answer orchestration |
| `query_processor.py` | Language detection, query normalisation |
| `indexer.py` | Batch upsert utilities |

---

## 🔐 Environment Variables

See [.env.example](.env.example) for the full list. Key variables:

```env
# 📞 Twilio
TWILIO_ACCOUNT_SID=ACxxx
TWILIO_AUTH_TOKEN=xxx

# 🔊 Azure TTS
AZURE_SPEECH_KEY=xxx
AZURE_SPEECH_REGION=centralindia

# 🗄️ Qdrant Cloud
QDRANT_CLOUD_URL=https://xxx.aws.cloud.qdrant.io:6333
QDRANT_CLOUD_API_KEY=xxx
QDRANT_COLLECTION=schemes_hybrid

# ☁️ AWS
AWS_REGION_NAME=eu-north-1
BEDROCK_LLM_MODEL=eu.amazon.nova-lite-v1:0
TRANSCRIBE_S3_BUCKET=government-scheme
```

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| 🗄️ **Vector DB** | [Qdrant Cloud](https://qdrant.tech/) — hybrid dense + sparse with RRF fusion |
| 🧮 **Embeddings** | [fastembed](https://github.com/qdrant/fastembed) — `BAAI/bge-large-en-v1.5` + `Qdrant/bm25` |
| 🧠 **LLM** | [AWS Bedrock](https://aws.amazon.com/bedrock/) — Amazon Nova Lite |
| 🎙️ **Speech-to-Text** | [AWS Transcribe](https://aws.amazon.com/transcribe/) — multi-language |
| 🔊 **Text-to-Speech** | [Azure Neural TTS](https://azure.microsoft.com/en-us/products/cognitive-services/speech-services/) — 12 Indian voices |
| 📞 **Telephony** | [Twilio](https://www.twilio.com/) — Voice & WhatsApp |
| ⚡ **Serverless** | [AWS Lambda](https://aws.amazon.com/lambda/) — container image (10 GB limit) |
| 🖥️ **Web Framework** | [FastAPI](https://fastapi.tiangolo.com/) — async webhook server |
| 🐳 **Containerisation** | [Docker](https://www.docker.com/) — model baked in at build time |

---

## 📄 License

MIT — built with ❤️ for Bharat 🇮🇳
