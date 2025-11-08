# Quick Start Guide

## 🚀 Getting Started in 3 Minutes

### 1. Setup (One-time)

```powershell
# Create virtual environment
python -m venv .venv
.venv\Scripts\Activate.ps1

# Install dependencies
pip install -r requirements.txt

# Set API key (create .env file)
echo "GEMINI_API_KEY=your-api-key-here" > .env
```

### 2. Add Your Data

Place CSV or Excel files in the `data/` folder:

```
data/
  ├── orders.csv
  ├── customers.xlsx
  └── products.xlsx
```

### 3. Launch

```powershell
streamlit run app.py
```

## 💬 How to Use

### Analytical Questions

Ask about your data in natural language:

- "What are the top 5 products by revenue?"
- "Show average order value by category"
- "Compare sales between Q1 and Q2"
- "Which customers spent the most?"

### Conversational

Just chat naturally:

- "Hello" → Get welcomed with examples
- "Thanks" → Polite acknowledgment
- "Help" → Usage guidance

### Features You'll Love

#### 🆕 New Chat Button

Click the "🆕 New Chat" button in the sidebar to start fresh. This clears:

- Current conversation
- Uploaded files
- All session data

#### 📤 Upload Files

Drag and drop CSV/Excel files directly in the sidebar. They're processed in-memory without writing to disk.

#### 📊 Data Quality Report

After loading data, check the "Data Quality Report" to see:

- Empty tables
- Null value warnings
- Duplicate row counts
- Column quality issues

#### 🔍 Inspect Generated Code

Every answer includes expandable sections:

- **Model plan**: The agent's reasoning
- **Generated code**: The pandas code executed
- **Result**: Tables, metrics, and charts

## 🎯 Tips & Tricks

### Get Better Results

1. **Be specific**: "Revenue for electronics in 2023" vs "Show me money"
2. **Use table/column names**: Reference actual column names when known
3. **Ask follow-ups**: "Show more details" or "Break that down by month"

### Troubleshooting

**No data showing up?**

- Ensure files are in `data/` folder
- Check file format (CSV or .xlsx/.xls)
- Look for Data Quality warnings

**Model errors?**

- Verify GEMINI_API_KEY in .env file
- Check internet connection
- The app auto-retries 2 times for transient failures

**Wrong results?**

- Inspect the generated code (click "Generated code" expander)
- Rephrase your question to be more specific
- Check if column names match your query

## 🔌 Advanced: Using Plugins

Plugins extend the agent's capabilities:

```python
from src.utils.plugins import get_plugin_registry

registry = get_plugin_registry()

# Get business term definition
result = registry.invoke("define", term="AOV")
print(result.data)  # "AOV: Average Order Value..."

# Convert currency
result = registry.invoke("currency_convert", amount=100, from_currency="USD", to_currency="EUR")
print(result.data["converted_amount"])  # ~92.0

# Format numbers
result = registry.invoke("format_number", value=1234567.89, format_type="compact")
print(result.data)  # "1.2M"
```

### Add Your Own Plugin

```python
def custom_plugin(param1: str) -> str:
    # Your logic here
    return f"Processed: {param1}"

registry.register("my_plugin", custom_plugin)
result = registry.invoke("my_plugin", param1="test")
```

## 💻 Command Line Interface

Run queries from the terminal:

```powershell
# Basic query
python -m src.cli "Total revenue for 2023"

# Conversational
python -m src.cli "Hello"

# Custom model
python -m src.cli "Top products" --model gemini-2.5-flash

# Custom data directory
python -m src.cli "Average sales" --data-dir ./my_data
```

## 📝 Example Questions

### Sales Analysis

- "What's our total revenue?"
- "Top 10 products by sales volume"
- "Monthly revenue trend for last year"
- "Which category has highest average order value?"

### Customer Insights

- "How many unique customers do we have?"
- "Customer lifetime value distribution"
- "Top 5 customers by total spend"
- "Average orders per customer"

### Performance Metrics

- "Delivery time by shipping carrier"
- "Order fulfillment rate by region"
- "Products with highest return rate"
- "Compare Q1 vs Q2 performance"

### Conversational

- "Hello" / "Hi" / "Good morning"
- "Thanks" / "Thank you"
- "What can you help me with?"

## 🧪 Testing Your Setup

Run the test suite to ensure everything works:

```powershell
# Run all tests
pytest -v

# Run specific test file
pytest tests/test_plugins.py -v

# Quick smoke test
pytest tests/test_intent.py
```

You should see: **14 tests passed** ✅

## 🆘 Getting Help

1. Check the **README.md** for architecture details
2. Review **IMPROVEMENTS.md** for latest features
3. Look at the generated code when results are unexpected
4. Use the Data Quality Report for data issues
5. Try the CLI for debugging: more verbose error messages

## 🎓 Learning Path

1. **Day 1**: Basic queries ("Show me revenue")
2. **Day 2**: Specific analyses ("Top products in electronics")
3. **Day 3**: Comparisons ("Q1 vs Q2 sales")
4. **Day 4**: Follow-up questions ("Break that down by region")
5. **Day 5**: Upload custom data and explore plugins

---

**You're all set!** Start with "Hello" and let the agent guide you. 🚀
