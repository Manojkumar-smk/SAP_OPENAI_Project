# SAP OpenAI Integration with HANA - User Manual

## Table of Contents
1. [Project Overview](#project-overview)
2. [Quick Start](#quick-start)
3. [Template Modules](#template-modules)
4. [Setup Instructions](#setup-instructions)
5. [Running the Application](#running-the-application)
6. [Using the Sample Application](#using-the-sample-application)
7. [Commands Reference](#commands-reference)
8. [Troubleshooting](#troubleshooting)

---

## Project Overview

This project integrates **OpenAI** with **SAP HANA Cloud** to build intelligent applications with multiple AI capabilities:

- **Natural Language SQL Generation**: Ask questions in plain English, get SQL queries executed
- **Retrieval Augmented Generation (RAG)**: Search documents and answer questions with citations
- **Multi-Mode Chatbot**: Conversational AI with memory, combining multiple data sources
- **Vector Search**: Semantic similarity search using OpenAI embeddings

### Key Features
✅ SAP HANA Cloud integration  
✅ OpenAI GPT-4 and embeddings (text-embedding-3-small)  
✅ Vector database with HANA Vector Engine  
✅ Multi-turn conversation with memory  
✅ SQL generation from natural language  
✅ Document Q&A with source citations  
✅ Streamlit web interface  

---

## Quick Start

### Prerequisites
- **Python 3.11+**
- **SAP HANA Cloud** instance (trial or production)
- **OpenAI API Key** (sk-...)
- **pip** (Python package manager)

### 30-Second Setup
```bash
# 1. Clone the repository
git clone https://github.com/Manojkumar-smk/SAP_OPENAI_Project.git
cd SAP_OPENAI_Project

# 2. Install dependencies
pip install -r sample_project_openai/requirements.txt

# 3. Create .env file with your credentials
cat > sample_project_openai/.env << EOF
HANA_HOST=your-hana-host.hanacloud.ondemand.com
HANA_PORT=443
HANA_USER=DBADMIN
HANA_PASSWORD=your-password
OPENAI_API_KEY=sk-your-key-here
VECTOR_TABLE_NAME=VECTOR_STORE
EOF

# 4. Initialize sample data
cd sample_project_openai
python setup_sample_data.py

# 5. Launch the app
streamlit run app.py
```

**Access the app at:** http://localhost:8501

---

## Template Modules

### 1. **hana_db_connection**
**Purpose**: Establish secure connections to SAP HANA Cloud

**What it contains**:
- `connect_env.py` - Local development using .env credentials
- `connect_btp_cf.py` - SAP BTP Cloud Foundry deployment
- `connect_hana_ml.py` - HANA ML integration
- `health_check.py` - Verify connection status

**How to use**:
```python
from connect_env import connect

conn = connect()
cursor = conn.cursor()
cursor.execute("SELECT * FROM SAMPLE_EMPLOYEES")
results = cursor.fetchall()
cursor.close()
```

**Key files**:
- `.env` - Store your HANA credentials (never commit!)
- `requirements.txt` - hdbcli, python-dotenv

---

### 2. **hana_ai_query_openai**
**Purpose**: Generate SQL queries from natural language questions

**What it contains**:
- `llm_setup.py` - Initialize OpenAI language model (GPT-4o-mini)
- `schema_inspector.py` - Inspect table structures from HANA
- `sql_generator.py` - Convert questions to SQL using LLM
- `sql_executor.py` - Execute SQL safely and return results
- `query_with_summary.py` - End-to-end query + AI summary

**How to use**:
```python
from connect_env import connect
from llm_setup import get_llm
from schema_inspector import build_schema_context
from sql_generator import generate_sql
from sql_executor import execute_sql

# Setup
conn = connect()
llm = get_llm()
schema = build_schema_context(conn, ["SAMPLE_EMPLOYEES"])

# Generate SQL from question
question = "Who is the highest paid employee?"
sql = generate_sql(question, schema, llm)

# Execute and get results
results = execute_sql(conn, sql)
print(results)
```

**Example Questions**:
- "List all employees in Engineering department"
- "What is the average salary by department?"
- "Who was hired after 2022?"
- "How many employees in each location?"

**Output**:
- DataFrame with query results
- Generated SQL (for verification)
- AI-generated summary of findings

---

### 3. **hana_vector_store_openai**
**Purpose**: Store and search documents using vector embeddings

**What it contains**:
- `embedding_setup.py` - Initialize OpenAI embeddings (text-embedding-3-small)
- `vector_table.py` - Create HANA vector table schema
- `embed_and_store.py` - Embed documents and store in HANA
- `similarity_search.py` - Find similar chunks using cosine similarity
- `text_chunker.py` - Split documents into chunks
- `document_loaders.py` - Load documents from various formats

**How to use**:
```python
from connect_env import connect
from embedding_setup import get_embedding_model
from embed_and_store import embed_and_store
from similarity_search import similarity_search
from langchain_core.documents import Document

# Setup
conn = connect()
embedder = get_embedding_model()

# Create vector table
from vector_table import create_vector_table
create_vector_table(conn, table_name="VECTOR_STORE")

# Embed and store documents
docs = [Document(
    page_content="Your document content here",
    metadata={"source_name": "document.pdf", "source_type": "pdf"}
)]
embed_and_store(conn, docs, embedder, table_name="VECTOR_STORE")

# Search for similar content
results = similarity_search(
    conn, 
    query="your search query",
    embedding_model=embedder,
    table_name="VECTOR_STORE",
    top_k=5
)

for r in results:
    print(f"[{r['rank']}] {r['source_name']} (score: {r['score']})")
    print(r['content'])
```

**Features**:
- Automatic document chunking (400 chars, 50 char overlap)
- Metadata tracking (source, type, chunk index)
- Cosine similarity scoring (0-1)
- Top-K retrieval with score filtering

---

### 4. **hana_rag_openai**
**Purpose**: Retrieval-Augmented Generation (RAG) for document Q&A

**What it contains**:
- `rag_pipeline.py` - End-to-end RAG flow
- `context_retriever.py` - Retrieve relevant chunks
- `llm_setup.py` - Configure LLM for RAG
- `conversation_memory.py` - Manage multi-turn conversations
- `rag_prompts.py` - System prompts for RAG

**How to use**:
```python
from connect_env import connect
from embedding_setup import get_embedding_model
from llm_setup import get_llm
from rag_pipeline import rag_query

conn = connect()
embedder = get_embedding_model()
llm = get_llm()

result = rag_query(
    conn=conn,
    user_query="What is the work from home policy?",
    embedding_model=embedder,
    llm=llm,
    table_name="VECTOR_STORE",
    top_k=4,
    min_score=0.3
)

print("Answer:", result["answer"])
print("Sources:", result["sources"])
```

**Output**:
- Direct answer with citations
- Source chunks with similarity scores
- Raw context sent to LLM (for debugging)

---

### 5. **sap_chatbot_openai**
**Purpose**: Multi-mode conversational AI with memory and data retrieval

**What it contains**:
- `chatbot_core.py` - Main chatbot engine
- `chatbot_config.py` - Configuration and presets
- `llm_setup.py` - LLM initialization
- `prompt_builder.py` - Dynamic prompt generation
- `rag_retriever.py` - RAG module for chatbot
- `sql_retriever.py` - SQL module for chatbot
- `cli_runner.py` - Command-line interface

**How to use**:
```python
from chatbot_core import SAPChatbot
from chatbot_config import ChatbotConfig
from connect_env import connect

conn = connect()

config = ChatbotConfig(
    llm_model="gpt-4o-mini",
    temperature=0.2,
    memory_window=10,
    enable_rag=True,
    enable_sql=True,
    system_prompt="You are a helpful TechCorp HR assistant",
    bot_name="TechCorp Bot"
)

bot = SAPChatbot(conn, config)

# Chat
response = bot.chat("What are employee benefits?")
print(response["answer"])
print(f"Mode: {response['mode']}")
print(f"Sources: {response['sources']}")

# Multi-turn conversation
response2 = bot.chat("Tell me more about health insurance")
print(response2["answer"])

# Reset conversation
bot.reset()
```

**Chatbot Modes**:
| Mode | Description | Data Source |
|------|-------------|-------------|
| **GENERAL** | Plain conversation | LLM only (no retrieval) |
| **RAG** | Document Q&A | FAQ documents |
| **SQL** | Database queries | SAMPLE_EMPLOYEES table |
| **FULL** | Combined mode | Documents + Database |

**Features**:
- Multi-turn conversation with memory (sliding window)
- Context-aware responses
- Source citations for RAG mode
- SQL queries shown for SQL mode
- Customizable system prompt

---

### 6. **sample_project_openai**
**Purpose**: Complete Streamlit application demonstrating all templates

**What it contains**:
- `app.py` - Main Streamlit application
- `setup_sample_data.py` - Initialize sample database
- `pages/` - Individual tab implementations
- `requirements.txt` - All dependencies

**Application Tabs**:

#### Tab 1: Connection Test
- Test HANA connectivity
- View SAMPLE_EMPLOYEES table
- Check vector store statistics

#### Tab 2: SQL Assistant
- Ask questions about employee data
- See generated SQL
- View results in table format

#### Tab 3: Document Q&A
- Search company handbook
- Ask HR policy questions
- View cited sources

#### Tab 4: Full Chatbot
- Multi-turn conversations
- Switch between 4 modes
- See conversation history

---

## Setup Instructions

### Step 1: Get HANA Cloud Access
1. Visit [SAP BTP Trial](https://account.hanatrial.ondemand.com/)
2. Create a trial account
3. Create SAP HANA Cloud instance
4. Note down:
   - Host: `xxx.hanacloud.ondemand.com`
   - Port: `443`
   - User: `DBADMIN`
   - Password: (your chosen password)

### Step 2: Get OpenAI API Key
1. Visit [OpenAI API Keys](https://platform.openai.com/api-keys)
2. Create new secret key
3. Copy and save (starts with `sk-`)

### Step 3: Clone Repository
```bash
git clone https://github.com/Manojkumar-smk/SAP_OPENAI_Project.git
cd SAP_OPENAI_Project
```

### Step 4: Install Dependencies
```bash
cd sample_project_openai
pip install -r requirements.txt
```

### Step 5: Create .env File
Create `sample_project_openai/.env`:
```
HANA_HOST=your-hana-host.hanacloud.ondemand.com
HANA_PORT=443
HANA_USER=DBADMIN
HANA_PASSWORD=your-password
OPENAI_API_KEY=sk-your-key-here
VECTOR_TABLE_NAME=VECTOR_STORE
```

### Step 6: Initialize Sample Data
```bash
python setup_sample_data.py
```

This will:
- Create SAMPLE_EMPLOYEES table with 10 employees
- Create VECTOR_STORE table in HANA
- Embed company FAQ document
- Store vectors in HANA

---

## Running the Application

### Method 1: Streamlit Web App (Recommended)
```bash
cd sample_project_openai
streamlit run app.py
```

**Access at**: http://localhost:8501

### Method 2: Individual Templates
```bash
# Test HANA connection
python hana_db_connection/connect_env.py

# Test SQL generation
python hana_ai_query_openai/sql_generator.py

# Test vector search
python hana_vector_store_openai/similarity_search.py

# Test RAG pipeline
python hana_rag_openai/rag_pipeline.py

# Test chatbot
python sap_chatbot_openai/cli_runner.py
```

### Method 3: Python Script
```python
from sample_project_openai.connect_env import connect
from sample_project_openai.llm_setup import get_llm

conn = connect()
llm = get_llm()

# Your code here
```

---

## Using the Sample Application

### Tab 1: Connection Test

**Purpose**: Verify HANA and OpenAI configuration

**Steps**:
1. Click "[CONN] Test Connection" button
2. Verify "Connected to HANA" message
3. Check OPENAI_API_KEY status
4. Click "[TABLE] Show SAMPLE_EMPLOYEES table" to see data
5. Click "📊 Show Vector Store stats" to check embeddings

**Expected Output**:
- Connection timestamp
- HANA version
- 10 employee records
- Vector store chunks count

---

### Tab 2: SQL Assistant

**Purpose**: Ask questions about employee data in natural English

**How to use**:
1. Select an example question or type your own
2. Click "[SEARCH] Run"
3. View generated SQL query
4. View results in table format
5. Read AI summary

**Example Queries**:
```
- Who are all employees in Engineering?
- What is the average salary by department?
- Who is the highest paid employee?
- How many employees are in each location?
- List employees hired after 2022, sorted by salary descending
- Which departments have more than 2 employees?
```

**Table Schema**:
```
SAMPLE_EMPLOYEES (
  ID          INT,
  NAME        NVARCHAR(100),
  DEPARTMENT  NVARCHAR(50),   -- Engineering / HR / Sales / Finance
  ROLE        NVARCHAR(100),
  SALARY      INT,            -- annual salary in INR
  HIRE_DATE   DATE,
  LOCATION    NVARCHAR(50)    -- Bangalore / Mumbai / Delhi / Hyderabad / Pune
)
```

---

### Tab 3: Document Q&A

**Purpose**: Search and ask questions about company FAQ

**How to use**:
1. Select an example question or type your own
2. Adjust "Chunks to retrieve" (top_k) slider
3. Adjust "Min similarity score" threshold
4. Click "[DOC] Ask"
5. View answer with cited sources

**Example Questions**:
```
- How many days of annual leave do employees get?
- What is the work from home policy?
- How does the performance rating system work?
- What are the business travel expense limits?
- What health insurance is provided?
- What is the learning budget per year?
```

**Available Documents**:
- Work From Home Policy
- Leave Policy (Annual, Sick, Maternity, Paternity)
- Performance Review & Ratings
- Salary & Bonus Structure
- Employee Benefits
- Onboarding Process
- Travel Policy
- IT & Security Rules

---

### Tab 4: Full Chatbot

**Purpose**: Multi-turn conversation with different data retrieval modes

**How to use**:
1. Select mode from dropdown:
   - **[CHAT] GENERAL** — Plain conversation
   - **[DOC] RAG** — Document Q&A with memory
   - **[BOT] SQL** — Employee database Q&A
   - **⚡ FULL** — Combined mode (documents + database)

2. Type your question
3. View answer with:
   - Source citations (if RAG enabled)
   - SQL queries (if SQL enabled)
   - Turn counter and mode indicator

4. Follow-up questions maintain conversation memory

**Example Conversations**:

**Mode: GENERAL**
```
User: What is TechCorp?
Bot: TechCorp is a technology company...

User: Where are you located?
Bot: We have offices in Bangalore, Mumbai, Delhi...
```

**Mode: RAG**
```
User: What is the onboarding process?
Bot: New employees go through a 2-week onboarding program...
[Sources: company_faq - score: 0.75]

User: How long is it?
Bot: The program lasts 2 weeks...
[Sources: company_faq - score: 0.82]
```

**Mode: SQL**
```
User: How many engineers do we have?
Bot: We have 3 employees in the Engineering department.
[SQL: SELECT COUNT(*) FROM DBADMIN.SAMPLE_EMPLOYEES 
      WHERE DEPARTMENT = 'Engineering']

User: What's their average salary?
Bot: The average salary in Engineering is ₹86,667
[SQL: SELECT AVG(SALARY) FROM DBADMIN.SAMPLE_EMPLOYEES 
      WHERE DEPARTMENT = 'Engineering']
```

---

## Commands Reference

### Environment Setup
```bash
# Set Python encoding to UTF-8
export PYTHONIOENCODING=utf-8

# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Install specific package
pip install hdbcli python-dotenv langchain-openai streamlit
```

### Running Applications
```bash
# Launch Streamlit app
streamlit run app.py

# Run with specific port
streamlit run app.py --server.port 8502

# Run with debug logging
streamlit run app.py --logger.level=debug

# Initialize sample data
python setup_sample_data.py

# Test individual modules
python hana_db_connection/connect_env.py
python hana_ai_query_openai/sql_executor.py
python hana_vector_store_openai/similarity_search.py
python sap_chatbot_openai/cli_runner.py
```

### Git Commands
```bash
# Clone repository
git clone https://github.com/Manojkumar-smk/SAP_OPENAI_Project.git

# Check status
git status

# Add changes
git add -A

# Commit changes
git commit -m "Your message"

# Push to GitHub
git push origin main
```

### Database Operations
```bash
# Connect to HANA
from connect_env import connect
conn = connect()

# Query database
cursor = conn.cursor()
cursor.execute("SELECT * FROM SAMPLE_EMPLOYEES")
results = cursor.fetchall()
cursor.close()

# Clear vector store
from vector_table import clear_vector_table
clear_vector_table(conn, "VECTOR_STORE")

# Get vector store stats
from vector_table import get_table_stats
stats = get_table_stats(conn, "VECTOR_STORE")
print(stats)
```

---

## Troubleshooting

### Connection Issues

**Error**: `Connection failed (RTE:[1000013] Invalid flags specified.)`
```
Solution: Update connect_env.py to not pass encrypt/certificate flags
```

**Error**: `Could not find HANA host`
```
Solution: Check HANA_HOST in .env is correct format:
xxx.hanacloud.ondemand.com (not https://...)
```

### SQL Generation Issues

**Error**: `invalid schema name: INFORMATION_SCHEMA`
```
Solution: Use SYS.TABLE_COLUMNS instead of INFORMATION_SCHEMA.COLUMNS
(HANA-specific system view)
```

**Error**: `FETCH FIRST syntax error`
```
Solution: Use LIMIT instead of FETCH FIRST (HANA syntax)
```

### Vector Store Issues

**Error**: `Table not found: VECTOR_STORE`
```
Solution: Run python setup_sample_data.py first
```

**Error**: `Source unknown`
```
Solution: Check metadata in embed_and_store has "source_name" key
```

### Encoding Issues

**Error**: `charmap codec can't encode character`
```
Solution: Set PYTHONIOENCODING=utf-8 before running
export PYTHONIOENCODING=utf-8
```

### Memory Issues

**Error**: `Conversation history too large`
```
Solution: Reduce memory_window in ChatbotConfig (default: 10 turns)
```

**Error**: `Too many embeddings in batch`
```
Solution: Reduce batch_size in embed_and_store (default: 50)
```

### API Key Issues

**Error**: `Invalid OpenAI API Key`
```
Solution: Check OPENAI_API_KEY starts with "sk-" and is valid
```

**Error**: `Rate limit exceeded`
```
Solution: Reduce top_k in RAG queries or batch_size in embedding
```

---

## Performance Tips

1. **Optimize Queries**
   - Use LIMIT to reduce results
   - Filter by date/department early
   - Use indexed columns

2. **Improve RAG**
   - Increase min_score threshold to 0.5+
   - Reduce top_k to 3-4
   - Pre-filter by source_name

3. **Faster Embeddings**
   - Use batch processing
   - Reduce document chunk size
   - Cache embeddings

4. **Streamlit Performance**
   - Use @st.cache_resource for LLM/embedder
   - Avoid recalculating embeddings
   - Minimize database queries

---

## Support & Resources

- **GitHub**: https://github.com/Manojkumar-smk/SAP_OPENAI_Project
- **SAP HANA Docs**: https://help.sap.com/docs/hana-cloud
- **OpenAI Docs**: https://platform.openai.com/docs
- **LangChain Docs**: https://python.langchain.com
- **Streamlit Docs**: https://docs.streamlit.io

---

## License

This project is provided as-is for educational and commercial use.

---

**Last Updated**: June 2026  
**Version**: 1.0  
**Status**: Production Ready
