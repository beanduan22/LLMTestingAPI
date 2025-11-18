
# Automated API Test Generation Pipeline (DL Library Oriented)

This repository describes a multi-stage automated pipeline designed to:
1. **Discover** deep-learning related APIs from major frameworks
2. **Filter** testable APIs using documentation-driven rules
3. **Convert** API docs into structured machine-readable specifications
4. **Initialize** parameter templates and mutation rules
5. **Enable fully automated test case generation and execution (future stage)**

All steps are **framework-agnostic**, **doc-driven**, and require **no manual annotation**.

---

## Supported Deep Learning Libraries

The pipeline currently targets the following major libraries:

| Category | Libraries |
|----------|------------------------------------------------|
| Deep Learning Frameworks | TensorFlow (`tf`), PyTorch (`torch`), JAX (`jax`), MindSpore (`mindspore`), PaddlePaddle (`paddle`) |

> Additional frameworks can be added as long as a Python import-level namespace is available.

---

## Pipeline Stages Overview

| readme | Name | Purpose | Output |
|--------|----------------------------|--------------------------------------------------------------|-----------------------------|
| **1** | API Auto-Discovery & Testability Filtering | Automatically locate and filter APIs using documentation rules | `testable_apis.csv/.xlsx` |
| **2** | LLM-Based Doc → JSON Conversion | Convert API docs into structured JSON specs | `json_specs/*.json` |
| **3** | Specification Parsing & Mutation Init | Build executable input templates & mutation constraints | `runtime_api_objects/*` |
| **4** | Test Case Generation & Execution | Execute API test suites (mutation or search-based) | coverage & anomaly results |

---

## How to Use This Repository

Follow the pipeline **in order**.

### ✔ Step 1 — Run readme 1 (MUST DO FIRST)

You will automatically:

- import selected libraries
- recursively discover API symbols
- retrieve and analyze doc strings
- filter testable APIs using strict rules
- export results to CSV / Excel

Output File Example:

```

doc2info/testable_apis.csv

```

Columns:

| api_full_name | api_doc_text |
|----------------|--------------|

---

### ✔ Step 2 — Run readme 2

- Read `testable_apis.csv` / `.xlsx`
- Call LLM per API using the predefined prompt
- Produce structured JSON API specification files

Output location:

```

doc2info/json_specs/*.json

```

---

### ✔ Step 3 — Run readme 3

- Read all `.json` specs
- Parse into executable internal structures
- Generate base input templates
- Configure mutation rules & constraints

Output location:

```

runtime_api_objects/*

```

---

## Execution Order Reminder

```

readme_1 → readme_2 → readme_3 → 

```

You **must not skip** readme_1 because:
- all downstream artifacts depend on the filtered set
- duplicate, dangerous, or non-testable APIs must not enter Stage 2

---

## Notes

- The pipeline is **documentation-driven**, not execution-driven
- No manual API curation is required
- No external datasets, internet, or annotation is needed
- The system is expandable to any Python-based library with valid docs

---

## Next Tasks for the User

1. Confirm the set of *initial libraries* to analyze  
   (default recommended: `tf`, `torch`, `onnx`, `jax`, `cv2`, `numpy`)

2. Run **Stage 1** first to generate `testable_apis.csv`

3. Proceed with Stage 2 only after Stage 1 completes successfully

---
