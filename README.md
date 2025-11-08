# E-commerce Data Copilot 🛒

An advanced conversational analytics platform designed to transform e-commerce data into actionable insights using Google Gemini. This tool allows users to interact with their data through natural language queries, generating pandas code and visualizations seamlessly.

---

## ✨ Key Features

- **🗣️ Conversational Interface**: Engage with your data using natural language queries.
- **🤖 Smart Intent Detection**: Automatically identifies whether the input is analytical or conversational.
- **📊 Automated Code Execution**: Generates and executes pandas code to answer analytical queries.
- **🔍 Contextual Retrieval**: Augments queries with relevant table context using TF-IDF.
- **📁 Data Compatibility**: Supports CSV and Excel files with schema detection.
- **☁️ File Upload**: Drag-and-drop interface for quick data uploads.
- **✅ Data Validation**: Detects null values, duplicates, and structural anomalies.
- **🔌 Extensible Plugins**: Add custom utilities like term definitions or currency conversion.
- **🆕 Session Management**: Start fresh conversations with a "New Chat" button.
- **💻 Dual Interface**: Use the Streamlit web app or the command-line interface (CLI).
- **🔄 Robust Error Handling**: Retries transient model failures automatically.
- **🧪 Comprehensive Testing**: Includes a Pytest suite for core functionality.

---

## 📂 Project Structure

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
│   │   └── tfidf.py        # Contextual retrieval with TF-IDF
│   └── utils/
│       ├── logging_config.py  # Centralized logging
│       ├── plugins.py      # Plugin system for extensions
│       └── validation.py   # Data quality validation
│
|
├── data/                   # Place .csv or .xlsx source files here
├── requirements.txt        # Python dependencies
├── .env.example            # Environment variable template
├── pytest.ini              # Pytest configuration
└── README.md
```

---

## 🚀 Quick Start (Streamlit UI)

1. **Install Python 3.11+** and create a virtual environment:
   ```powershell
   python -m venv .venv
   .venv\Scripts\Activate.ps1
   ```
2. **Install dependencies**:
   ```powershell
   pip install -r requirements.txt
   ```
3. **Prepare Your Data**: Place your `.csv` or `.xlsx` files in the `data/` directory. The app supports both formats.
4. **Configure Gemini**: Set your API key in the environment. Either:
   - Temporary (current shell):
     ```powershell
     $env:GEMINI_API_KEY = "your-key-here"
     ```
   - Persistent (via `.env` file):
     ```
     GEMINI_API_KEY=your-key-here
     MODEL_NAME=gemini-2.5-flash
     ```
5. **Launch the App**:
   ```powershell
   streamlit run app.py
   ```
6. Open the provided local URL and start interacting with your data.

---

## 🎯 Conversational Mode

The application intelligently classifies your input:

- **Analytical Mode**: Generates and executes pandas code for data queries.
- **Chat Mode**: Handles greetings, thanks, and general conversation.
- **Context-Aware**: Remembers recent queries to provide relevant suggestions.

---

## 🖥️ CLI Usage

Query data directly from the terminal:

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

The CLI supports all features available in the web app.

---

## 🔌 Plugin System

Extend the application with custom plugins:

**Built-in Plugins:**

- `define`: Look up e-commerce term definitions (e.g., GMV, AOV, CAC).
- `currency_convert`: Convert between currencies with mock exchange rates.
- `format_number`: Format numbers as currency, percentages, or compact notation.

**Custom Plugin Example:**

```python
def my_custom_plugin(arg1: str, arg2: int) -> str:
    return f"Processed {arg1} with {arg2}"

from src.utils.plugins import get_plugin_registry
registry = get_plugin_registry()
registry.register("my_plugin", my_custom_plugin)
```

---

## 🛠️ How It Works

### Data Flow

1. **Data Ingestion**: Loads CSV/Excel files into pandas DataFrames.
2. **Intent Classification**: Determines if input is conversational or analytical.
3. **Contextual Retrieval**: Uses TF-IDF to retrieve relevant table context.
4. **Code Generation**: Gemini generates structured JSON with pandas code.
5. **Sandboxed Execution**: Runs code in an isolated environment.
6. **Result Rendering**: Displays metrics, tables, charts, and natural language summaries.

---

### Data Validation

Automatically checks for:

- Empty tables
- Columns with 100% null values
- High null percentages (>50%)
- Duplicate rows
- Unnamed or malformed columns

---

## 🛣️ Roadmap

1. Add semantic retrieval for glossary lookups.
2. Integrate external APIs for enriched metadata.
3. Enable collaborative sessions with persistent chat history.
4. Implement authentication for secure data access.

---

## 🎥 Demo Checklist

When creating a demo video, include:

- Problem statement and target users.
- Dataset overview and ingestion pipeline.
- UI walkthrough: queries, code inspection, and outputs.
- Architecture overview.
- Future roadmap.

---

## 🧪 Testing

Run tests with:

```powershell
pytest -q
```

---

## ⚠️ Safety Notes

- Never commit sensitive credentials like `GEMINI_API_KEY`.
- Review generated code before executing critical operations.
- Optimize large datasets upstream to avoid memory issues.
- Use `.env.example` as a template for environment variables.
