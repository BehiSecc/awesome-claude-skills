---
name: polaris-datainsight-doc-extract
description: Extract structured data from Office documents (DOCX, PPTX, XLSX, HWP, HWPX) using the Polaris AI DataInsight Doc Extract API. Use when the user wants to parse, analyze, or extract text, tables, charts, images, or shapes from document files. Invoke this skill whenever the user mentions extracting content from Word, PowerPoint, Excel, HWP, or HWPX files, wants to parse document structure, needs to convert document data for RAG pipelines, or asks about reading tables, charts, or text from Office-format documents — even if they don't explicitly mention "DataInsight" or "Polaris".
license: Apache-2.0 (for skill definition); Service usage subject to Polaris AI DataInsight Terms of Service.
---

# Polaris AI DataInsight — Doc Extract Skill

Use the Polaris AI DataInsight Doc Extract API to extract text, images, tables, charts, shapes, equations, and more from Word, PowerPoint, Excel, HWP, and HWPX files, returning everything as a structured `unifiedSchema` JSON. A single API call gives you the full document structure without any manual parsing.

---

## When to Use This Skill

- The user wants to extract text, tables, charts, or images from DOCX, PPTX, XLSX, HWP, or HWPX files
- The user needs to understand a document's structure (page count, element types, position data, etc.)
- The extracted data will be used in a RAG pipeline, data analysis workflow, or automation task
- Table data needs to be converted to CSV, or chart data needs to be broken down into series and labels
- The user needs to parse special elements like headers, footers, equations, or shapes

---

## What This Skill Does

1. **Authentication** — Authenticates with the Polaris DataInsight API via the `x-po-di-apikey` header.
2. **Upload and extract** — Sends the file as a multipart/form-data POST request and extracts the full document structure.
3. **Parse ZIP response** — The API returns a ZIP file; extract it and load the `unifiedSchema` JSON inside.
4. **Deliver structured data** — Returns a JSON organized by page and element type (text, table, chart, image, shape, equation, etc.).
5. **Support multiple usage patterns** — Handles full text extraction, table-to-CSV conversion, RAG chunk generation, and more.

---

## How to Use

### Prerequisites

**Get an API Key**: Sign up at https://datainsight.polarisoffice.com and generate your API key.

**Authentication**: Include the API key as a header on every request.

```
Header: x-po-di-apikey: $POLARIS_DATAINSIGHT_API_KEY
```

**Set the environment variable**:

```bash
export POLARIS_DATAINSIGHT_API_KEY="your-api-key-here"
```

### Limits

| Item | Limit |
|------|-------|
| Supported formats | HWP, HWPX, DOCX, PPTX, XLSX |
| Max file size | 25 MB |
| Timeout | 10 minutes |
| Rate limit | 10 requests per minute |

### Basic Usage

**Endpoint**:

```
POST https://datainsight-api.polarisoffice.com/api/v1/datainsight/doc-extract
```

**Extract a document with Python:**

```python
import requests
import json
import zipfile
import io

def extract_document(file_path: str, api_key: str) -> dict:
    with open(file_path, "rb") as f:
        response = requests.post(
            "https://datainsight-api.polarisoffice.com/api/v1/datainsight/doc-extract",
            headers={"x-po-di-apikey": api_key},
            files={"file": f}
        )

    if response.status_code != 200:
        raise Exception(f"API error: {response.status_code} - {response.text}")

    # Response is a ZIP file
    zip_buffer = io.BytesIO(response.content)
    with zipfile.ZipFile(zip_buffer) as z:
        json_files = [name for name in z.namelist() if name.endswith('.json')]
        if json_files:
            with z.open(json_files[0]) as jf:
                return json.load(jf)

    raise Exception("No JSON found in ZIP")

# Example usage
import os
api_key = os.environ["POLARIS_DATAINSIGHT_API_KEY"]
schema = extract_document("report.docx", api_key)
print(f"Extracted {schema['totalPages']} pages")
```

**Extract with curl:**

```bash
curl -X POST "https://datainsight-api.polarisoffice.com/api/v1/datainsight/doc-extract" \
  -H "x-po-di-apikey: $POLARIS_DATAINSIGHT_API_KEY" \
  -F "file=@example.docx" \
  --output result.zip

unzip result.zip -d result/
cat result/*.json | python -m json.tool
```

---

## Advanced Usage

### Response Structure (unifiedSchema)

**Root:**

```json
{
  "docName": "sample.docx",
  "totalPages": 3,
  "pages": [ ... ]
}
```

**Page (`pages[]`):**

```json
{
  "pageNum": 1,
  "pageWidth": 595.3,
  "pageHeight": 842.0,
  "extractionSummary": {
    "text": 5, "image": 2, "table": 1, "chart": 1
  },
  "elements": [ ... ]
}
```

**Element types (`elements[].type`):**

| type | Description |
|------|-------------|
| `text` | Text block |
| `image` | Image |
| `table` | Table |
| `chart` | Chart |
| `shape` | Shape |
| `equation` | Equation |
| `header` / `footer` | Header / Footer |

**Common element structure:**

```json
{
  "type": "text",
  "id": "te1",
  "boundaryBox": { "left": 40, "top": 80, "right": 300, "bottom": 120 },
  "content": { "text": "Body content here" }
}
```

**Table content:**

```json
{
  "content": {
    "html": "<table>...</table>",
    "csv": "Header1,Header2\nValue1,Value2",
    "json": [
      {
        "metrics": { "rowaddr": 0, "coladdr": 0, "rowspan": 1, "colspan": 1 },
        "para": [{ "content": [{ "text": "Cell content" }] }]
      }
    ]
  }
}
```

**Chart content:**

```json
{
  "content": {
    "chart_type": "column",
    "title": "Annual Sales Comparison",
    "x_axis_labels": ["Q1", "Q2", "Q3", "Q4"],
    "series_names": ["2023", "2024"],
    "series_values": [[100, 200, 150, 300], [120, 220, 180, 320]],
    "csv": "Quarter,2023,2024\nQ1,100,120\nQ2,200,220"
  }
}
```

### Usage Patterns

**Extract all text:**

```python
def get_all_text(schema: dict) -> str:
    texts = []
    for page in schema.get("pages", []):
        for el in page.get("elements", []):
            if el["type"] == "text" and el.get("content", {}).get("text"):
                texts.append(el["content"]["text"])
    return "\n".join(texts)
```

**Extract tables as CSV:**

```python
def get_tables_as_csv(schema: dict) -> list:
    tables = []
    for page in schema.get("pages", []):
        for el in page.get("elements", []):
            if el["type"] == "table":
                csv_data = el.get("content", {}).get("csv", "")
                if csv_data:
                    tables.append(csv_data)
    return tables
```

**Generate RAG chunks:**

```python
def make_rag_chunks(schema: dict) -> list:
    chunks = []
    doc_name = schema.get("docName", "")
    for page in schema.get("pages", []):
        for el in page.get("elements", []):
            text = el.get("content", {}).get("text") or el.get("content", {}).get("csv") or ""
            if text.strip():
                chunks.append({
                    "source": doc_name,
                    "page": page["pageNum"],
                    "type": el["type"],
                    "text": text.strip()
                })
    return chunks
```

---

## Example

**User:** "Extract all table data from this DOCX report as CSV."

**Output:**

```python
import os
schema = extract_document("report.docx", os.environ["POLARIS_DATAINSIGHT_API_KEY"])
tables = get_tables_as_csv(schema)
for i, csv_data in enumerate(tables):
    print(f"=== Table {i+1} ===")
    print(csv_data)
```

```
=== Table 1 ===
Quarter,Revenue,Cost
Q1,1200,800
Q2,1500,900

=== Table 2 ===
Item,Amount
Labor,500
Operations,300
```

**Inspired by:** Polaris Office DataInsight API documentation and workflow.

---

## Tips

- The response is always a **ZIP file**. Do not try to parse `response.content` directly as JSON — use `zipfile.ZipFile` to extract it first.
- `content.csv` is available for both `table` and `chart` elements, making it the most convenient format for data extraction.
- The rate limit is 10 requests per minute. When processing multiple files, add a delay (e.g., `time.sleep(6)`) between calls.
- Use `boundaryBox` to determine where each element sits on the page — useful for layout analysis.
- Always store the API key in an environment variable (`POLARIS_DATAINSIGHT_API_KEY`) and never hardcode it.

---

## Common Use Cases

- **Document search systems**: Extract full text and store it in a vector database for semantic search
- **Automated report analysis**: Collect table and chart data from PPTX/DOCX reports for analysis
- **HWP digitization**: Convert HWP/HWPX documents into structured, machine-readable data
- **RAG pipeline setup**: Split documents into chunks for use in LLM-based Q&A systems
- **Data migration**: Move table and chart data from legacy Office documents into a database

---

## License & Terms

- **Skill Definition:** This `SKILL.md` file is provided under the [Apache 2.0](https://www.apache.org/licenses/LICENSE-2.0) license.
- **Service Access:** Usage of the DataInsight API requires a valid subscription or license key.
- **Restrictions:** Unauthorized redistribution of the API endpoints or bypassing authentication is strictly prohibited.
- **Support:** For licensing inquiries, visit [https://datainsight.polarisoffice.com](https://datainsight.polarisoffice.com).
