# E-commerce Data Copilot 🛒

An intelligent conversational analytics platform that transforms e-commerce data (Olist Brazilian dataset) into interactive insights using Google Gemini. Ask questions in natural language, receive computed answers with generated pandas code, and enjoy a seamless chat experience.

## ✨ Key Features

- **🗣️ Conversational Interface**: Natural language queries with context-aware responses
- **🤖 Smart Intent Detection**: Automatically distinguishes between analytical questions and casual conversation
- **📊 Automated Code Generation**: Gemini generates and executes pandas code to answer your questions
- **🔍 RAG-Enhanced Retrieval**: TF-IDF-based retrieval augments prompts with relevant table context
- **📁 Flexible Data Loading**: Supports CSV and Excel files with automatic schema detection
- **☁️ File Upload**: Drag-and-drop interface for in-memory data analysis
- **✅ Data Quality Validation**: Automatic checks for null values, duplicates, and structural issues
- **🔌 Plugin System**: Extensible architecture for definitions, currency conversion, and utilities
- **🆕 Session Management**: "New Chat" button to start fresh conversations
- **💻 Dual Interface**: Full-featured web UI (Streamlit) and command-line interface
- **🔄 Error Recovery**: Automatic retry logic for transient model failures
- **🧪 Comprehensive Testing**: Pytest suite covering core functionality

## Project Structure

```
.
├── app.py                  # Streamlit web application (main entry point)
├── src/
│   ├── agent/              # Core agent components
│   │   ├── prompt.py       # Schema summary and prompt assembly
│   │   ├── model.py        # Gemini model wrapper
│   │   ├── executor.py     # Sandboxed code execution
│   │   ├── intent.py       # Intent classification (chat vs analysis)
│   │   └── memory.py       # Conversation persistence (optional)
│   ├── data/
│   │   └── loader.py       # CSV and Excel data ingestion
│   ├── retrieval/
│   │   └── tfidf.py        # RAG retrieval with TF-IDF
│   ├── utils/
│   │   ├── logging_config.py  # Centralized logging
│   │   ├── plugins.py      # Plugin system for extensions
│   │   └── validation.py   # Data quality validation
│   └── cli.py              # Command-line interface
├── tests/                  # Pytest test suites
│   ├── test_executor.py
│   ├── test_intent.py
│   ├── test_plugins.py
│   └── test_prompt.py
├── data/                   # Place .csv or .xlsx source files here
├── requirements.txt        # Python dependencies
├── .env.example            # Environment variable template
├── pytest.ini              # Pytest configuration
└── README.md
```

## Quick start (Streamlit UI)

1. **Install Python 3.11+** and create a virtual environment:
   ```powershell
   python -m venv .venv
   .venv\Scripts\Activate.ps1
   ```
2. **Install dependencies**:
   ```powershell
   pip install -r requirements.txt
   ```
3. **Download the dataset** from [Kaggle](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce/) and place the original `.csv` files (or Excel exports) into the `data/` directory. The app supports both CSV and Excel.
4. **Configure Gemini** by setting your API key (never commit this value). Either:
   - Temporary (current shell):
     ```powershell
     $env:GEMINI_API_KEY = "your-key-here"
     ```
   - Create a `.env` file (auto-loaded):
     ```
     GEMINI_API_KEY=your-key-here
     # Optional: override default model (falls back to gemini-2.5-flash)
     MODEL_NAME=gemini-2.5-flash
     ```
   ```powershell
   $env:GEMINI_API_KEY = "your-key-here"
   ```
5. **Launch the app**:
   ```powershell
   streamlit run app.py
   ```
6. Open the provided local URL and start asking questions:

   **Analytical Questions:**

   - `Which product category sold the most units in the past two quarters?`
   - `Show the average order value for the electronics category.`
   - `Compare delivery delays by state.`
   - `What's the revenue trend over time?`

   **Conversational:**

   - `Hello` → Get a friendly greeting and examples
   - `Thanks` → Polite acknowledgment
   - The agent automatically detects conversational vs analytical intent

## 🎯 Conversational Mode

The application intelligently classifies your input:

- **Analytical Mode**: Questions about data trigger code generation and execution
- **Chat Mode**: Greetings, thanks, and general conversation get natural language responses
- **Context-Aware**: Remembers recent analysis and provides relevant suggestions

This makes the agent feel more natural and user-friendly, not just a code generator.

## CLI usage

Query data from the terminal with conversational support:

```powershell
# Analytical query
python -m src.cli "Average order value for electronics category"

# Conversational query
python -m src.cli "Hello"

# Use custom model
python -m src.cli "Top categories last quarter" --model gemini-2.5-flash

# Specify data directory
python -m src.cli "Total revenue" --data-dir ./custom_data
```

The CLI has full feature parity with the web app including intent classification and retrieval augmentation.

## 🔌 Plugin System

The application includes an extensible plugin architecture for adding capabilities:

**Built-in Plugins:**

- `define`: Look up e-commerce term definitions (GMV, AOV, CAC, LTV, etc.)
- `currency_convert`: Convert between currencies with mock exchange rates
- `format_number`: Format numbers as currency, percentages, or compact notation

**Usage Example:**

```python
from src.utils.plugins import get_plugin_registry

registry = get_plugin_registry()
result = registry.invoke("define", term="AOV")
# Returns: "AOV: Average Order Value - average amount spent per order"
```

**Extend with Custom Plugins:**

```python
def my_custom_plugin(arg1: str, arg2: int) -> str:
    # Your logic here
    return f"Processed {arg1} with {arg2}"

registry.register("my_plugin", my_custom_plugin)
```

This enables adding external APIs, knowledge bases, or business-specific utilities.

````

## How it works (architecture)

### Data Flow

1. **Data Ingestion**: Loads CSV/Excel files from `data/` folder into pandas DataFrames
   - CSV files: `<filename>`
   - Excel files: `<workbook>__<sheet>`
   - Supports in-memory uploads via web UI

2. **Intent Classification**: Analyzes user input to determine:
   - **Chat Mode**: Greetings, pleasantries, thanks → Natural language response
   - **Analysis Mode**: Data queries → Code generation + execution

3. **RAG Retrieval**: For analytical queries:
   - Builds TF-IDF index from table schemas (columns + sample rows)
   - Retrieves top-K relevant table contexts
   - Augments prompt with retrieved information

4. **Code Generation**: Gemini generates structured JSON:
   ```json
   {
     "thoughts": "Analysis plan",
     "python_lines": ["code line 1", "line 2", ...],
     "final_answer": "Natural language summary"
   }
   ```

5. **Sandboxed Execution**:
   - Code runs in isolated namespace with only `pd` and `dataframes`
   - No file/network access
   - Produces `analysis_output` dictionary

6. **Result Rendering**:
   - Metrics (numeric KPIs)
   - Tables (query results)
   - Charts (bar/line visualizations)
   - Natural language answers

7. **Error Recovery**: Automatic retry (up to 2 times) for transient model failures

### Data Quality Validation

Uploaded data is automatically validated for:
- Empty tables
- Columns with 100% null values
- High null percentages (>50%)
- Duplicate rows
- Suspiciously wide tables (>100 columns)
- Unnamed/malformed columns

Issues are categorized by severity (error/warning/info) and displayed in an expandable report.

## Design decisions & extensions

- **Deterministic execution**: Gemini is instructed to respond with JSON and to generate pure pandas code. No filesystem or network access is allowed inside the execution sandbox.
- **Transparent reasoning**: The UI exposes the model's high-level plan and the generated code so reviewers can validate each answer.
- **Extensibility hooks**: The `analysis_output` contract already supports metrics and chart data. You can add “smart utilities” such as glossary lookups or geospatial maps by enriching the prompt or post-processing layer.

### Retrieval layer details

The RAG component lives in `src/retrieval/tfidf.py`:

1. Builds a pseudo-document per table summarizing columns, dtypes, and first 5 rows.
2. Uses `scikit-learn` TF-IDF with bigrams and a cosine similarity search.
3. Top-K (default 3) documents are appended to the natural language question inside a `RETRIEVED CONTEXT` section, guiding the model toward the most relevant tables.
4. This keeps latency low and avoids heavy embedding costs.

Regenerate retrieval context automatically by re-running the app/CLI after updating data files (no persistent index yet for simplicity).

### Possible next steps / roadmap

1. Fine-tune schema embeddings and add semantic retrieval for definitions or glossary lookups.
2. Attach external product catalogs or live APIs for enriched product metadata.
3. Persist chat history and analysis artifacts for collaborative sessions.
4. Add authentication and role-based access to protect sensitive operational data.

## Video demo checklist

When preparing your submission video (5–7 minutes), consider covering:

- Problem statement and target users.
- Dataset overview and ingestion pipeline.
- UI walkthrough: asking questions, inspecting code, viewing outputs.
- Architecture diagram (LLM prompt flow, execution sandbox, UI components).
- Potential roadmap and enhancements if granted more time.

## Testing

Run the automated tests:

```powershell
pytest -q
````

## Safety notes

- Never commit `GEMINI_API_KEY` or any other credentials.
- Review generated code before trusting critical business outputs.
- Large Excel files can be memory-intensive; filter or aggregate upstream when possible.
- Review `.env.example` and avoid committing a real `.env` file.
