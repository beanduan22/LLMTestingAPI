
# Doc-Driven API Testability Filtering Protocol

This document defines a strict rule-based decision workflow for determining whether an API is **testable** based **solely on documentation content** (Python `__doc__`).  
No runtime execution, signature inspection, or external metadata is required.

---

## Step 1 — Documentation Availability Check

**Rule 1.1 — Reject immediately if:**
- The doc is missing, empty, placeholder-like, or contains no usable semantics  
  (e.g., only `...`, `pass`, a single vague sentence, or copy-pasted template text).

**If doc is readable and meaningful → Continue.**

---

## Step 2 — Input Parameter Validity Check

Documentation must clearly describe function parameters.

**Reject if:**
- No parameter description is present.
- All parameters are optional and no required input is documented.
- Parameter descriptions are too vague to infer type or usage  
  (e.g., "some object", "anything", "data structure", "stuff").

**Proceed only if:**  
At least one **required** and **documented** input parameter exists.

---

## Step 3 — Constructible Input Type Check

Each required parameter must have a type that can be **programmatically constructed** without external resources.

**Allowed (constructible) input types include:**
```

int, float, bool, string, tuple, list, array, tensor, ndarray, numeric types

```

**Reject if any required parameter type falls into non-constructible categories, including:**
```

file path, URL, network resource, hardware device, session object,
model instance, graph handle, context manager, stream, iterator,
callback, opaque handle, environment object

```

If all required parameters are constructible → Continue.

---

## Step 4 — Side Effects and External Dependency Check

Scan documentation for any indication of side effects or external dependency.

**Reject immediately if documentation mentions or implies any of the following:**

| Category | Disqualifying keywords or patterns |
|----------|------------------------------------|
| I/O | file, save, load, write, read, open, download |
| Hardware / devices | camera, microphone, GPU, TPU, device handle |
| Execution context | session, scope, global, attach, register |
| Training / lifecycle | train, fit, learn, optimizer, backward, checkpoint |
| Distributed / remote | cluster, server, worker, RPC, remote execution |
| Asynchronous / event-driven | callback, hook, listener, subscribe |

If no such signals are found → Continue.

---

## Step 5 — Explicit Return Value Check

Documentation must identify **a meaningful and capturable return value**.

**Reject if:**
- No return description is provided.
- Return value is described only as a side effect (e.g., in-place modification).
- Return behavior is unclear or unspecified.
- Output depends on volatile global state.

**Proceed only if documentation states or implies a return value that is data-form, e.g.:**
```

returns a tensor
returns an ndarray
returns a numeric value
returns (value, index)

```

---

## Step 6 — Final Decision

| Outcome | Condition |
|---------|-----------|
| **TESTABLE** | All steps (1–5) passed |
| **NOT TESTABLE** | Any step fails |
| **UNCERTAIN** | Information insufficient for a confident decision |

---

## Output Format

Only APIs classified as **TESTABLE** will be saved into a structured tabular file (CSV or Excel).

| File | Description |
|-------|-------------|
| `testable_apis.csv` or `testable_apis.xlsx` | Contains TESTABLE APIs and their associated documentation content |

### Table Format

| Column | Description |
|--------|-------------|
| `api_full_name` | The full qualified name of the API (e.g., `tf.math.abs`) |
| `api_doc_text` | The extracted documentation content (`__doc__` or official doc text) |

### Example content (`testable_apis.csv`):

| api_full_name | api_doc_text |
|----------------|--------------|
| tf.abs | Applies the abs function element-wise on a tensor... |
| tf.math.log | Computes natural logarithm of x element-wise... |
| ... | ... |

---





