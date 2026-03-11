# Stage 4 — Test Generation, Execution, and Coverage Collection

This stage consumes the runtime API configuration objects produced in Stage 3 and performs actual test generation, execution, validation, and coverage collection.  
While Stage 3 prepares *how an API can be tested*, Stage 4 performs the testing process itself in an iterative fuzz-style loop.

---

### 1 Seed Instantiation from Parameter Constraints

The input to Stage 4 is not a concrete executable test case yet.  
Instead, each runtime API object provides a parameter-level specification
produced in Stage 3, where every parameter includes metadata such as:

- type
- size
- default
- flag
- description

First instantiates these parameter constraints into concrete valid seed inputs.

Example parameter specification (simplified):

{
  "api": "torch.nn.functional.conv2d",
  "parameters": {
    "input": {
      "type": "tensor",
      "size": [1, 3, 32, 32],
      "default": null,
    },
    "weight": {
      "type": "tensor",
      "size": [16, 3, 3, 3],
      "default": null,
    },
    "stride": {
      "type": "int",
      "default": 1,
    }
  }
}

Example instantiated seed:

{
  "input": "tensor(1,3,32,32)",
  "weight": "tensor(16,3,3,3)",
  "stride": 1
}

These instantiated seeds are then used as the starting points for mutation-based test generation.

And you need to input this real data~(seed) to API and make sure this is right and no bugs here.


## 2. Mutation Strategy

If you the seed pass our validation after privious step, and in this step, we are going to mutation.

| Strategy | Description | Example |
|--------|-------------|--------|
| Type Mutation | Change the data type of a parameter while maintaining valid API constraints. | `float32 → float64`, or passing a string instead of a number |
| Size Mutation | Modify the dimensionality or size of structured inputs such as tensors, lists, or tuples while respecting dependency constraints. | `(H,W,3) → (H,W,1)` |
| Value Mutation-Adding Noise | see VistaFuzz. | - |
| Value Mutation-Random Masking | see VistaFuzz. | - |
| Value Mutation-Division | see VistaFuzz. | - |


In this step, just follow the VistaFuzz, the input is seed and API info (Constrains) and apply mutation stratguis. Just totally following Vistafuzz.

And the output is similar like seed, but may have different value, size or some else.

When we get the mutated seed (test cases), We are going to test APIs.

---

## 3. API Execution

Each generated test cases is executed against the target API.

Execution must:

- Import the API using the internal representation
- Invoke the API with the generated inputs
- Capture outputs and metadata

During execution, the system must record:

| Signal | Description |
|--------|-------------|
| Output | Returned values |
| Exception | Raised runtime errors |
| Crash | Abnormal termination |
| NaN value | - |
| Inconsistency (new) | different between cpu and gpu |
| Execution metadata | runtime info (dtype, shapes, timing, etc.) |

This is also totally followed Vistafuzz (expect inconsistency).

Invalid executions must be logged but should not stop the testing loop.

If there were no bugs, go to the next mutation, which is same with Vistafuzz.

---

## 4. Coverage Collection

Coverage is collected during execution and serves as the primary feedback signal.

Coverage must be handled depending on library implementation.

### 4.1 Native / Backend Coverage (C/C++ Libraries)

For deep learning libraries with native backends (e.g., TensorFlow, PyTorch, OpenCV):

Steps:

1. Download the official source repository.
2. Build the library from source (do not use precompiled wheels).
3. Enable coverage instrumentation (e.g., gcov / compiler flags).
4. Run Stage 4 tests against the instrumented build.
5. Collect native coverage results.

Native coverage is the primary signal for fuzzing deep learning libraries.

---

### 4.2 Python Layer Coverage

For Python wrapper layers:

- Use `coverage.py` to measure execution coverage.

This complements native coverage but does not replace it.

---

### 4.3 Pure Python Libraries

If a library is fully implemented in Python:

- `coverage.py` alone is sufficient.

---

## 5. Feedback-Driven Exploration

Coverage signals may guide future testing:

- prioritize inputs that increase coverage  
- retain interesting seeds  
- adapt mutation scheduling  

This enables iterative fuzz-style exploration.

---

## 6. Library-Agnostic Execution Principle (Critical)

Stage 4 must remain **library-agnostic**.

Implementation must:

- Avoid hard-coding library names  
- Avoid binding logic to TensorFlow-only or PyTorch-only APIs  
- Use the internal API representation from Stage 3  
- Separate logical API identity from library implementation  

This ensures the framework supports:

- multiple deep learning libraries  
- scalable experimentation  
- consistent coverage methodology  

In short:

**API identity must be decoupled from library identity.**

---

## 7. Output of Stage 4

This stage produces execution artifacts such as:

execution_results/
coverage_reports/
crash_logs/
seed_corpus/

Each may include:

| Component |
|-----------|
| Executed test inputs |
| Coverage metrics |
| Exception / crash logs |
| Interesting seeds |
| Execution metadata |

These outputs are used for analysis, debugging, and further exploration.

---

## 8. Completion Criteria

✓ Test inputs successfully generated  
✓ APIs executed without breaking the loop  
✓ Coverage collected correctly  
✓ Execution logs recorded  
✓ Library-agnostic execution preserved  
✓ Supports deep learning libraries broadly (native + Python)
