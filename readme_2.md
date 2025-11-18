
# LLM-Based API Documentation to Structured JSON Conversion

This stage describes how to transform each API entry from the previously generated
`testable_apis.csv` or `testable_apis.xlsx` into a standardized JSON schema using an LLM.
The JSON output represents a **machine-actionable structured API specification**, which will
be used for downstream automated test case generation.

---

## 1. Input Source

The input file must be one of the following formats:

- `testable_apis.csv`
- `testable_apis.xlsx`

Both formats must contain the following **two columns**:

| Column Name     | Description |
|-----------------|-------------|
| `api_full_name` | Fully qualified API name (`module.path.function`) |
| `api_doc_text`  | The documentation content (extracted from `__doc__` or official documentation page) |

Example input:

| api_full_name  | api_doc_text |
|----------------|--------------|
| tf.abs         | Applies the abs function element-wise... |
| tf.math.log    | Computes natural logarithm element-wise... |

---

## 2. Processing Goal

For each row in the input file, call an LLM with the provided documentation text and produce
a **standardized JSON file** following the agreed schema.

Each API will generate **one standalone `.json` file**, named using:

```

{api_full_name}.json

```

Example (or directly input into one json file):

```

tf.abs.json
tf.math.log.json

````

---

## 3. JSON Output Schema

The generated JSON file **must** follow this strict structure:

```json
{
  "api_name": "",
  "module_path": "",
  "params": {
    "<param_name>": {
      "type": "",
      "size": "",
      "default": "",
      "flag": "", 
      "description": ""
    }
  },
  "output": {
    "type": "",
    "numbers": "",
    "description": ""
  },
  "constraints": [
    "<optional rule 1, if applicable>",
    "<optional rule 2, if applicable>"
  ]
}
````

Notes:

* `flag` ∈ { "Required", "Optional" }
* Empty fields must be explicitly represented as empty strings `""`

---

## 4. LLM Prompt Template

For **each** API entry, call the LLM using the following prompt.
Do not modify variable names or output structure.

```
You are an expert API reverse-engineering assistant. 
Your task is to convert API documentation text into a strictly formatted JSON specification
that supports automated test input generation.

### API Information
API Full Name: {{api_full_name}}
API Documentation Text:
"""
{{api_doc_text}}
"""

### Instructions
1. Infer parameter names, types, sizes/shapes, default values, and required/optional flags.
2. If information cannot be determined, leave the field as an empty string ("").
3. All parameters must appear under the "params" dictionary.
4. Output only valid JSON following the schema below with no explanation and no comments.

### JSON Schema
{
  "api_name": "",
  "module_path": "",
  "params": {
    "<param_name>": {
      "type": "",
      "size": "",
      "default": "",
      "flag": "", 
      "description": ""
    }
  },
  "output": {
    "type": "",
    "numbers": "",
    "description": ""
  },
  "constraints": [
    ""
  ]
}
```

---

## 5. Expected Output Example (Simplified)

For `tf.abs`:

```json
{
  "api_name": "abs",
  "module_path": "tf",
  "params": {
    "x": {
      "type": "Tensor or numeric",
      "size": "",
      "default": "",
      "flag": "Required",
      "description": "Input tensor or numeric argument"
    }
  },
  "output": {
    "type": "Tensor",
    "numbers": "1",
    "description": "Element-wise absolute value"
  },
  "constraints": []
}
```

---

## 6. Output Storage Structure

All JSON files must be saved into:

```
doc2info/json_specs/
    ├── tf.abs.json
    ├── tf.math.log.json
    ├── ...
```

---

## 7. Completion Criteria

✓ All APIs in the CSV/Excel file are processed
✓ Each API has exactly one valid JSON file
✓ JSON schema is filled with no missing keys
✓ No natural language or explanatory text is present in output files
✓ JSON parses successfully without reformatting

---
