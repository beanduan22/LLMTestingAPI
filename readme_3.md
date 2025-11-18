
# JSON Specification Parsing, Initialization, and Mutation Rule Configuration

This stage describes how to load and interpret the structured API JSON files generated in Stage 2, 
create a unified internal representation for test generation, and define mutation constraints 
for controllable and valid input perturbation.

---

## 1. Input Source

This stage consumes the JSON files generated in the previous step:

```

doc2info/json_specs/*.json

```

Each file represents a single or a list API and follows the predefined schema.

---

## 2. Purpose of This Stage

1. **Parse** each JSON file into a standardized internal API object
2. **Initialize** default test-ready input templates for each parameter
3. **Apply mutation rule constraints** to ensure future generated inputs remain valid and meaningful

The output will be used for:
- valid input construction
- adaptive mutation search
- fuzz-style multi-case generation

---

## 3. JSON Parsing and Internal Representation

Each JSON specification must be parsed into an internal structure with the following attributes:

| Field | Purpose |
|--------|--------------------------------|
| api_name      | The function/method name |
| module_path   | The module root for importing |
| params        | Parameter dictionary for initialization |
| output        | Expected return structure |
| constraints   | Extra rules for validation and mutation |

Internal representation example (conceptual):

```

APIObject {
name: "abs",
module: "tf",
params: {
"x": { type, size, default, flag, description }
},
output: { type, numbers, description },
constraints: [...]
}

```

---

## 4. Parameter Initialization Strategy

Each parameter must be initialized with a **base valid input** before mutation.

Initialization follows rules:

| Parameter Type | Initialization Strategy |
|----------------|--------------------------|
| int            | small positive value (e.g., 1) |
| float          | small floating value (e.g., 1.0) |
| bool           | `True` |
| string         | simple placeholder (e.g., `"test"`) |
| list/tuple     | minimal structure with primitive elements |
| ndarray/tensor | minimum valid shape & random numeric content |
| mixed type     | choose the simplest allowed type |

Notes:
- Required parameters must always be initialized
- Optional parameters may be omitted if default is specified
- If a parameter has size or shape information, use the smallest valid dimension

---

## 5. Mutation Rules

After initialization, mutation rules define **how values can be changed** while staying valid.

### 5.1 Allowed Mutation Types

| Mutation Type | Description |
|----------------|-------------|
| Scalar mutation | Add/sub small delta to int/float |
| Range mutation | Expand or shrink numeric range |
| Structure mutation | Change list/tuple length within limits |
| Shape mutation | Adjust tensor/array dimensions semantically |
| Type mutation | Switch between compatible types (e.g. int → float) |
| Random sampling | Inject controlled randomness within safe bounds |

---

## 6. Mutation Constraints

Mutation must comply with **schema + constraints + documentation semantics**.

Rules must prevent invalid test cases:

| Constraint to Respect | Examples |
|------------------------|----------|
| Required shape/dim size | `(H, W)` must remain ≥ minimal |
| Input must remain numeric if required | No string replacements |
| Maintain dtype compatibility | Tensor float → Tensor float |
| Parameter relationships | e.g. `max > min`, `axis < ndim` |

If constraints are violated, mutation must:
1. **Rollback**, or
2. **Auto-correct**, or
3. **Soft-skip & regenerate**

---

## 7. Output of Stage 3

This stage does **not** generate test cases yet.  
It produces **ready-to-use, valid test configuration objects**:

```

runtime_api_objects/
├── tf.abs.init
├── tf.math.log.init
├── ...

```

Each includes:

| Component |
|-----------|
| Base valid input template |
| Mutation capability settings |
| Constraint enforcement rules |

These will be consumed by Stage 4 for real test generation.

---

## 8. Completion Criteria

✓ All JSON specs successfully parsed  
✓ All APIs have initialized valid input templates  
✓ Mutation techniques recorded and constraints applied  
✓ No invalid or unconstrained parameter left  

---

