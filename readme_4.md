# Stage 4 — Test Generation, Execution, and Coverage Collection

This stage consumes the runtime API configuration objects produced in Stage 3 and performs actual test generation, execution, validation, and coverage collection.  
While Stage 3 prepares *how an API can be tested*, Stage 4 performs the testing process itself in an iterative fuzz-style loop.

---

## 1. Input Source

This stage consumes the initialized runtime API objects:

runtime_api_objects/*.init

Each file represents one logical API blueprint and includes:

- Base valid input template  
- Mutation capability configuration  
- Constraint validators  

These objects define how test inputs can be generated and mutated.

---

## 2. Purpose of This Stage

1. **Generate test inputs** from base templates using mutation strategies  
2. **Execute APIs** using generated candidate inputs  
3. **Validate execution results** with constraints and runtime checks  
4. **Collect coverage signals** during execution  
5. **Provide feedback** for iterative fuzz-style exploration  

The output will be used for:

- coverage-guided exploration  
- bug detection and abnormal behavior logging  
- seed scheduling and adaptive mutation  

---

## 3. Test Generation Strategy

For each runtime API object, Stage 4 generates candidate inputs starting from the base valid template.

Generation follows:

- Apply one or more mutation operators
- Ensure mutated inputs pass validators
- Produce multiple candidate inputs per iteration

Typical mutation examples:

| Category | Example |
|----------|--------|
| Numeric mutation | small perturbation to scalar values |
| Shape mutation | expand or shrink tensor dimensions within bounds |
| Structure mutation | modify list/tuple length |
| Value mutation | noise injection, scaling |
| Combined mutation | shape + value change |

The goal is to explore valid input space rather than produce random invalid inputs.

---

## 4. API Execution

Each generated input is executed against the target API.

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
| Execution metadata | runtime info (dtype, shapes, timing, etc.) |

Execution logic must rely on the **internal API abstraction**, not hard-coded library calls.

---

## 5. Validation and Oracles

After execution, results are validated using multiple checks.

### 5.1 Constraint Validation

- Input constraints (from Stage 3)
- Output structure checks
- dtype and shape consistency

### 5.2 Basic Runtime Checks

- NaN / Inf detection  
- abnormal shape results  
- invalid dtype transitions  

### 5.3 Optional Advanced Oracles

- differential testing across libraries  
- invariant checks  
- semantic consistency checks  

Invalid executions must be logged but should not stop the testing loop.

---

## 6. Coverage Collection

Coverage is collected during execution and serves as the primary feedback signal.

Coverage must be handled depending on library implementation.

### 6.1 Native / Backend Coverage (C/C++ Libraries)

For deep learning libraries with native backends (e.g., TensorFlow, PyTorch, OpenCV):

Steps:

1. Download the official source repository.
2. Build the library from source (do not use precompiled wheels).
3. Enable coverage instrumentation (e.g., gcov / compiler flags).
4. Run Stage 4 tests against the instrumented build.
5. Collect native coverage results.

Native coverage is the primary signal for fuzzing deep learning libraries.

---

### 6.2 Python Layer Coverage

For Python wrapper layers:

- Use `coverage.py` to measure execution coverage.

This complements native coverage but does not replace it.

---

### 6.3 Pure Python Libraries

If a library is fully implemented in Python:

- `coverage.py` alone is sufficient.

---

## 7. Feedback-Driven Exploration

Coverage signals may guide future testing:

- prioritize inputs that increase coverage  
- retain interesting seeds  
- adapt mutation scheduling  

This enables iterative fuzz-style exploration.

---

## 8. Library-Agnostic Execution Principle (Critical)

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

## 9. Output of Stage 4

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

## 10. Completion Criteria

✓ Test inputs successfully generated  
✓ APIs executed without breaking the loop  
✓ Coverage collected correctly  
✓ Execution logs recorded  
✓ Library-agnostic execution preserved  
✓ Supports deep learning libraries broadly (native + Python)
