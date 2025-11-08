# System Architecture

## High-Level Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                         USER INTERFACES                          │
├────────────────────────────┬────────────────────────────────────┤
│   Streamlit Web App        │      Command Line Interface        │
│   - File upload UI         │      - Terminal queries            │
│   - Chat interface         │      - Batch processing            │
│   - Data preview           │      - Scripting support           │
│   - New Chat button        │                                    │
└────────────────────────────┴────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────┐
│                      INTENT CLASSIFICATION                       │
│  • Analyzes user input                                          │
│  • Routes to Chat Mode or Analysis Mode                         │
│  • Checks for greetings, keywords, table/column references     │
└─────────────────────────────────────────────────────────────────┘
                     │                           │
        ┌────────────┘                           └──────────────┐
        ▼                                                        ▼
┌──────────────────┐                          ┌──────────────────────────┐
│   CHAT MODE      │                          │    ANALYSIS MODE         │
│  Natural Lang.   │                          │  Code Generation         │
│  Response        │                          └──────────────────────────┘
└──────────────────┘                                    │
                                                        ▼
                                          ┌──────────────────────────┐
                                          │   DATA INGESTION         │
                                          │  • Load CSV/Excel        │
                                          │  • Schema extraction     │
                                          │  • Quality validation    │
                                          └──────────────────────────┘
                                                        │
                                                        ▼
                                          ┌──────────────────────────┐
                                          │   RAG RETRIEVAL          │
                                          │  • TF-IDF indexing       │
                                          │  • Top-K document search │
                                          │  • Context augmentation  │
                                          └──────────────────────────┘
                                                        │
                                                        ▼
                                          ┌──────────────────────────┐
                                          │   GEMINI MODEL           │
                                          │  • Structured prompt     │
                                          │  • JSON response         │
                                          │  • Retry logic (×2)      │
                                          └──────────────────────────┘
                                                        │
                                                        ▼
                                          ┌──────────────────────────┐
                                          │   CODE EXECUTION         │
                                          │  • Sandboxed environment │
                                          │  • Pandas operations     │
                                          │  • Error handling        │
                                          └──────────────────────────┘
                                                        │
                                                        ▼
                                          ┌──────────────────────────┐
                                          │   RESULT RENDERING       │
                                          │  • Tables                │
                                          │  • Metrics               │
                                          │  • Charts                │
                                          │  • Natural language      │
                                          └──────────────────────────┘
```

## Component Details

### 1. Data Layer (`src/data/`)

```
loader.py
├── load_excel_data()  → Loads .xlsx/.xls files
└── Supports multi-sheet workbooks with naming: <workbook>__<sheet>
```

**New: CSV Support**

- Automatically detects and loads .csv files
- Naming: `<filename_stem>`

### 2. Intent Classification (`src/agent/intent.py`)

```
classify_intent(question, dataframes)
├── Check greeting patterns (regex)
├── Check for table/column name references
├── Check for analytical keywords (sum, average, trend, etc.)
└── Return: "chat" | "analysis"
```

**Keywords Detected:**

- Analytical: sum, average, trend, compare, correlation, distribution, etc.
- Conversational: hi, hello, thanks, thank you, etc.

### 3. Retrieval System (`src/retrieval/tfidf.py`)

```
TF-IDF Retriever
├── build_docs() → Synthesize pseudo-documents from tables
├── build_retriever() → Create scikit-learn TF-IDF vectorizer
└── retrieve(query, k) → Top-K relevant documents by cosine similarity
```

**Document Structure:**

```
Table: orders__main
Rows: 1000, Columns: 5
Columns: order_id (int64), customer_id (object), amount (float64)...
Sample: [{"order_id": 1, "customer_id": "ABC", "amount": 99.99}, ...]
```

### 4. Agent Core (`src/agent/`)

#### Prompt Assembly (`prompt.py`)

```
build_schema_summary(dataframes)
├── Iterate through tables (max 8 for summary)
├── Extract: rows, columns, dtypes, sample data
└── Return formatted text summary

assemble_prompt(question, history, schema)
├── Combine system prompt
├── Add dataset profile
├── Include conversation history (last 6 messages)
└── Append current question
```

#### Model Interaction (`model.py`)

```
configure(api_key)         → Set up Gemini SDK
get_model(model_name)      → Initialize model instance
generate_json(model, prompt) → Call model and parse JSON response
```

#### Code Execution (`executor.py`)

```
execute_code(code, dataframes)
├── Create sandboxed namespace
│   ├── global: {pd: pandas}
│   └── local: {dataframes: {...}, analysis_output: None}
├── compile() and exec() code
├── Extract analysis_output
└── Return structured result
```

**Safety:**

- No file I/O
- No network access
- No imports allowed
- Only pandas operations

### 5. Validation System (`src/utils/validation.py`)

```
DataValidator
├── validate_dataframe(name, df)
│   ├── Check empty tables
│   ├── Check null columns (100% null)
│   ├── Check high null percentage (>50%)
│   ├── Check duplicate rows
│   ├── Check wide tables (>100 cols)
│   └── Check unnamed columns
└── validate_all(dataframes)
    ├── Run checks on all tables
    ├── Aggregate statistics
    └── Return issues by severity
```

**Issue Severities:**

- **Error**: Critical problems blocking analysis
- **Warning**: Potential issues affecting quality
- **Info**: Informational notices

### 6. Plugin System (`src/utils/plugins.py`)

```
PluginRegistry
├── register(name, func)
├── invoke(name, **kwargs)
├── list_plugins()
└── Built-in plugins:
    ├── define(term) → Business term definitions
    ├── currency_convert(amount, from, to) → Currency conversion
    └── format_number(value, format_type) → Number formatting
```

**Extensibility:**

```python
def my_plugin(**kwargs):
    # Custom logic
    return result

registry.register("my_plugin", my_plugin)
```

## Data Flow Diagram

### Analytical Query Flow

```
User: "What's the average order value?"
  │
  ├─→ Intent Classifier → "analysis"
  │
  ├─→ Load dataframes from data/
  │     ├─→ orders.csv
  │     ├─→ customers.xlsx (Sheet1, Sheet2)
  │     └─→ Validate quality
  │
  ├─→ Build retrieval index
  │     ├─→ Synthesize documents
  │     └─→ TF-IDF vectorization
  │
  ├─→ Retrieve relevant context
  │     └─→ Top 3 documents mentioning "order" and "value"
  │
  ├─→ Assemble prompt
  │     ├─→ System instructions
  │     ├─→ Schema summary
  │     ├─→ Retrieved context
  │     ├─→ Conversation history
  │     └─→ User question
  │
  ├─→ Call Gemini (with retry)
  │     └─→ Generate JSON:
  │         {
  │           "thoughts": "Calculate mean of order amounts",
  │           "python_lines": [
  │             "total = dataframes['orders']['amount'].mean()",
  │             "analysis_output = {'answer_text': f'${total:.2f}'}"
  │           ],
  │           "final_answer": "The average order value is $127.45"
  │         }
  │
  ├─→ Execute code in sandbox
  │     ├─→ Run pandas operations
  │     └─→ Extract analysis_output
  │
  └─→ Render results
        ├─→ Display answer text
        ├─→ Show metrics
        ├─→ Render tables
        └─→ Generate charts
```

### Conversational Query Flow

```
User: "Hello"
  │
  ├─→ Intent Classifier → "chat"
  │
  └─→ Generate natural response
        └─→ "Hello! I'm your data copilot. You can ask me..."
```

## Session Management

```
Streamlit Session State
├── messages: List[Dict]         → Chat history
├── uploaded_dfs: Dict[str, DataFrame]  → In-memory uploads
└── [cleared by "New Chat" button]

Memory (Optional)
├── memory.json (not currently used)
└── Removed for simplicity - fresh start on each session
```

## Error Handling Strategy

### Model Call Failures

```
try:
    response = model.generate_content(...)
except Exception as exc:
    if attempt < max_retries:
        retry  # Automatic retry up to 2 times
    else:
        raise ValueError("Failed after retries")
```

### Code Execution Errors

```
try:
    exec(code, global_env, local_env)
except Exception as exc:
    traceback_text = traceback.format_exc()
    raise RuntimeError(f"Code failed: {exc}\n{traceback}")
```

### Data Loading Errors

```
try:
    df = pd.read_csv(file)
except Exception as exc:
    st.toast(f"Failed to load: {exc}", icon="⚠️")
    continue  # Skip file, don't crash
```

## Performance Considerations

### Retrieval Speed

- TF-IDF: ~50-100ms for small-medium datasets
- In-memory indexing (no disk persistence)
- Scales to ~50 tables, 1M total rows

### Model Latency

- Gemini API call: ~1-3 seconds
- JSON parsing: <10ms
- Code execution: <100ms (typical queries)

### Memory Usage

- DataFrames held in memory
- Streamlit caching for loaded data
- Typical: 50-500MB for e-commerce datasets

## Security Model

### Sandboxed Execution

```python
global_env = {"pd": pd}  # ONLY pandas allowed
local_env = {
    "dataframes": {...},
    "analysis_output": None
}
# No __import__, open(), requests, etc.
```

### API Key Handling

- Loaded from `.env` file (auto-loaded by dotenv)
- Never logged or displayed
- Not included in version control (.gitignore)

### Input Validation

- Data quality checks before analysis
- Schema verification
- Type checking in plugins

## Scalability Path

### Current Limits

- 50-100 tables (TF-IDF memory)
- 1M total rows (pandas memory)
- Single user per session

### Future Scaling

1. **Vector DB**: Replace TF-IDF with Pinecone/Weaviate
2. **Caching**: Redis for query results
3. **Async**: Background data loading
4. **Distributed**: Dask for large DataFrames
5. **Multi-tenant**: User authentication + isolation

---

**Architecture Principles:**

- ✅ Separation of concerns (data, agent, retrieval, UI)
- ✅ Fail-safe defaults (retry, validation)
- ✅ Extensibility (plugins, modular design)
- ✅ Observability (code inspection, quality reports)
- ✅ Security (sandboxing, no credentials in code)
