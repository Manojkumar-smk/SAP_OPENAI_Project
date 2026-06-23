# SAP OpenAI Project - SAP HANA Cloud Integration

![Status](https://img.shields.io/badge/status-Production%20Ready-brightgreen)
![Python](https://img.shields.io/badge/python-3.11%2B-blue)
![License](https://img.shields.io/badge/license-MIT-green)

## 🚀 Quick Start

A production-ready integration of **OpenAI GPT-4** with **SAP HANA Cloud** featuring:
- 🤖 Natural Language SQL Generation
- 📚 Retrieval Augmented Generation (RAG)
- 💬 Multi-Mode Conversational AI
- 🔍 Vector Search with Semantic Similarity
- 🎨 Streamlit Web Interface

### 30 Seconds to Running

```bash
# 1. Clone
git clone https://github.com/Manojkumar-smk/SAP_OPENAI_Project.git
cd SAP_OPENAI_Project

# 2. Install
pip install -r sample_project_openai/requirements.txt

# 3. Configure
cat > sample_project_openai/.env << EOF
HANA_HOST=your-host.hanacloud.ondemand.com
HANA_PORT=443
HANA_USER=DBADMIN
HANA_PASSWORD=your-password
OPENAI_API_KEY=sk-your-key
VECTOR_TABLE_NAME=VECTOR_STORE
EOF

# 4. Initialize
cd sample_project_openai && python setup_sample_data.py

# 5. Run
streamlit run app.py
```

**Access**: http://localhost:8501

---

## 📋 Templates Overview

| Template | Purpose | Use Case |
|----------|---------|----------|
| **hana_db_connection** | Database connectivity | Connect to HANA Cloud/BTP |
| **hana_ai_query_openai** | SQL generation | "Who is the highest paid employee?" |
| **hana_vector_store_openai** | Vector embeddings | Store & search documents |
| **hana_rag_openai** | Document Q&A | Answer questions from documents |
| **sap_chatbot_openai** | Multi-mode chatbot | Combine SQL + RAG + General chat |
| **sample_project_openai** | Complete app | 4 interactive tabs |

---

## 📖 Documentation

- **[USER_MANUAL.md](USER_MANUAL.md)** - Complete guide (setup, usage, commands, troubleshooting)
- **[Quick Reference](#quick-reference)** - Below

---

## 🎯 What You Can Do

### Tab 1: Connection Test
```
✓ Test HANA connectivity
✓ View employee data
✓ Check vector store status
```

### Tab 2: SQL Assistant
```
Questions → LLM → SQL Query → Results + Summary

Examples:
- "Who is the highest paid employee?"
- "List employees in Engineering"
- "Average salary by department?"
```

### Tab 3: Document Q&A
```
Questions → Search Embeddings → Retrieve Chunks → LLM → Answer + Citations

Examples:
- "What is the work from home policy?"
- "How many vacation days?"
- "What's the onboarding process?"
```

### Tab 4: Full Chatbot
```
Multi-turn conversation with 4 modes:
- GENERAL: Plain chat
- RAG: Document search + memory
- SQL: Database queries + memory
- FULL: Everything combined
```

---

## 🛠️ Key Commands

### Setup
```bash
# Install dependencies
pip install -r sample_project_openai/requirements.txt

# Initialize sample data
cd sample_project_openai
python setup_sample_data.py

# Create .env file
cp sample_project_openai/.env.example sample_project_openai/.env
# Edit with your credentials
```

### Run
```bash
# Web app (recommended)
streamlit run sample_project_openai/app.py

# CLI chatbot
python sap_chatbot_openai/cli_runner.py

# Test individual modules
python hana_db_connection/connect_env.py
python hana_ai_query_openai/sql_executor.py
python hana_vector_store_openai/similarity_search.py
```

### Debug
```bash
# UTF-8 encoding for Windows
export PYTHONIOENCODING=utf-8

# Debug logging
streamlit run app.py --logger.level=debug

# Test connection
python hana_db_connection/health_check.py
```

---

## 📊 Architecture

```
┌─────────────────────────────────────────────┐
│         Streamlit Web Interface             │
│  (Connection | SQL | RAG | Chatbot Tabs)    │
└────────────────┬────────────────────────────┘
                 │
        ┌────────┼────────┐
        │        │        │
   ┌────▼──┐ ┌──▼──┐ ┌───▼───┐
   │ SQL   │ │ RAG │ │Chatbot│
   │Engine │ │Pipe │ │Engine │
   └────┬──┘ └──┬──┘ └───┬───┘
        │       │        │
        └───────┼────────┘
                │
      ┌─────────▼──────────┐
      │   SAP HANA Cloud   │
      │  - SQL Queries     │
      │  - Vector Engine   │
      │  - Embeddings      │
      └────────────────────┘
                │
        ┌───────┴────────┐
        │                │
    ┌───▼──┐        ┌───▼──┐
    │OpenAI│        │OpenAI│
    │ GPT-4│        │ Embed│
    └──────┘        └──────┘
```

---

## 📁 Project Structure

```
SAP_OPENAI_Project/
├── hana_db_connection/          # HANA connectivity
│   ├── connect_env.py           # Local .env connection
│   ├── connect_btp_cf.py        # BTP Cloud Foundry
│   └── health_check.py          # Connection test
│
├── hana_ai_query_openai/        # SQL generation
│   ├── llm_setup.py             # OpenAI LLM config
│   ├── schema_inspector.py      # Table schema reading
│   ├── sql_generator.py         # NL to SQL
│   ├── sql_executor.py          # Safe SQL execution
│   └── query_with_summary.py    # End-to-end query
│
├── hana_vector_store_openai/    # Vector embeddings
│   ├── embedding_setup.py       # OpenAI embeddings config
│   ├── vector_table.py          # HANA vector table schema
│   ├── embed_and_store.py       # Embed & store documents
│   ├── similarity_search.py     # Vector similarity search
│   ├── text_chunker.py          # Document chunking
│   └── document_loaders.py      # Load various formats
│
├── hana_rag_openai/             # RAG pipeline
│   ├── rag_pipeline.py          # End-to-end RAG
│   ├── context_retriever.py     # Chunk retrieval
│   ├── llm_setup.py             # RAG-specific LLM
│   ├── conversation_memory.py   # Multi-turn memory
│   └── rag_prompts.py           # System prompts
│
├── sap_chatbot_openai/          # Multi-mode chatbot
│   ├── chatbot_core.py          # Main chatbot engine
│   ├── chatbot_config.py        # Configuration & presets
│   ├── prompt_builder.py        # Dynamic prompts
│   ├── rag_retriever.py         # RAG for chatbot
│   ├── sql_retriever.py         # SQL for chatbot
│   └── cli_runner.py            # CLI interface
│
├── sample_project_openai/       # Complete Streamlit app
│   ├── app.py                   # Main Streamlit app
│   ├── setup_sample_data.py     # Initialize database
│   ├── pages/                   # Tab implementations
│   └── utils/                   # Utilities
│
├── USER_MANUAL.md               # Complete documentation
├── README.md                    # This file
└── .gitignore                   # Exclude secrets
```

---

## 🔐 Security

- ✅ Secrets excluded from git (.gitignore)
- ✅ SQL injection protection (parameterized queries)
- ✅ API key validation
- ✅ HTTPS for HANA Cloud connection
- ⚠️ Never commit .env files with credentials

---

## ⚙️ Configuration

### .env File
```
# HANA Cloud
HANA_HOST=your-instance.hanacloud.ondemand.com
HANA_PORT=443
HANA_USER=DBADMIN
HANA_PASSWORD=your-password
HANA_CERTIFICATE=          # Leave blank for BTP Trial

# OpenAI
OPENAI_API_KEY=sk-your-key-here

# Vector Store
VECTOR_TABLE_NAME=VECTOR_STORE
VECTOR_DIMENSION=1536      # For text-embedding-3-small
```

### Chatbot Modes
```python
config = ChatbotConfig(
    llm_model="gpt-4o-mini",
    temperature=0.2,
    memory_window=10,       # Conversation turns to remember
    enable_rag=True,        # Enable document search
    enable_sql=True,        # Enable database queries
)
```

---

## 🚀 Deployment

### Local Development
```bash
streamlit run sample_project_openai/app.py
```

### Docker
```bash
docker build -t sap-openai .
docker run -p 8501:8501 --env-file .env sap-openai
```

### SAP BTP
Use `connect_btp_cf.py` with Cloud Foundry environment variables

---

## 📚 Examples

### Query Employee Data
```python
from sample_project_openai.connect_env import connect
from sample_project_openai.sql_executor import execute_sql

conn = connect()
sql = "SELECT * FROM DBADMIN.SAMPLE_EMPLOYEES WHERE DEPARTMENT = 'Engineering'"
df = execute_sql(conn, sql)
print(df)
```

### Search Documents
```python
from sample_project_openai.similarity_search import similarity_search
from sample_project_openai.embedding_setup import get_embedding_model

model = get_embedding_model()
results = similarity_search(
    conn, 
    query="work from home policy",
    embedding_model=model,
    top_k=3
)
```

### Chat with Bot
```python
from sample_project_openai.chatbot_core import SAPChatbot
from sample_project_openai.chatbot_config import ChatbotConfig

config = ChatbotConfig(enable_rag=True, enable_sql=True)
bot = SAPChatbot(conn, config)

response = bot.chat("What's the highest salary?")
print(response["answer"])
```

---

## 🐛 Troubleshooting

| Issue | Solution |
|-------|----------|
| HANA connection fails | Check HANA_HOST format (no https://) |
| SQL syntax error | Use LIMIT instead of FETCH FIRST |
| Source shows "unknown" | Check metadata has "source_name" key |
| Encoding errors | Set `export PYTHONIOENCODING=utf-8` |
| Rate limit exceeded | Reduce top_k or increase delay |

See **[USER_MANUAL.md](USER_MANUAL.md)** for detailed troubleshooting.

---

## 📖 Learn More

- [Complete User Manual](USER_MANUAL.md)
- [SAP HANA Cloud Documentation](https://help.sap.com/docs/hana-cloud)
- [OpenAI API Documentation](https://platform.openai.com/docs)
- [LangChain Documentation](https://python.langchain.com)
- [Streamlit Documentation](https://docs.streamlit.io)

---

## 👥 Contributing

Contributions welcome! Please:
1. Fork the repository
2. Create a feature branch
3. Commit your changes
4. Push to the branch
5. Create a Pull Request

---

## 📄 License

This project is provided as-is for educational and commercial use.

---

## 📞 Support

- **GitHub Issues**: Report bugs and request features
- **Documentation**: See [USER_MANUAL.md](USER_MANUAL.md)
- **Examples**: Check `sample_project_openai/` for working examples

---

**Ready to get started?** Follow the [Quick Start](#-quick-start) above or read the [USER_MANUAL.md](USER_MANUAL.md) for detailed instructions!

**Last Updated**: June 2026 | **Version**: 1.0 | **Status**: Production Ready
