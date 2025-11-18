
---

# Stage1: API Auto-Discovery & Testability Filtering Guidelines 

This document describes how to automatically identify *testable APIs* from any software library **without relying on a predefined API list (txt)**.
The purpose is to discover APIs that are suitable for automated input generation and execution-based testing.

---

## **1. Objective**

Automatically extract and filter APIs from a target library, and produce a list of **testable APIs** based on predefined criteria regarding **callability, input constructability, safety, determinism, and executability**.

---

## **2. Entry Condition**

The process begins with **a reference to a target library module**, e.g.:

* A dynamic import (e.g., `import torch`)
* A loaded namespace 
* A root object (e.g., `cv2`, `np`, `tf`, `jax.numpy`)

No API list, documentation file, or index text is required.

---

## **3. High-Level Procedure**

1. **Discover APIs automatically** by recursively exploring attributes in the target module.
2. **Identify callable symbols** that represent executable functional behavior.
3. **Extract minimal behavioral and parameter information** (e.g., name, signature, docs if available).
4. **Evaluate testability** according to standardized rules.
5. **Record testable APIs** and log reasons for exclusions.
6. **Export two final lists** for downstream test input generation.

---

## **4. Inclusion Criteria (Must Satisfy All)**

An API is **testable** only if it meets **all** of the following conditions:

| Category                 | Requirement                                                                                   |
| ------------------------ | --------------------------------------------------------------------------------------------- |
| Callable                 | Must be a callable function, method, or operator                                              |
| Has Inputs               | Must require **one or more user-provided input parameters**                                   |
| Constructible Parameters | Inputs must be programmatically constructible (e.g., numbers, arrays, tensors, tuples, bools) |
| Self-contained           | Executable without I/O, external resources, user interaction, or environment configuration    |
| Deterministic            | Does not depend on global states, external devices, runtime sessions, or unstable randomness  |
| Safe to execute          | Expected to run within bounded time and resource consumption                                  |

---

## **5. Exclusion Criteria (Any Match → Disqualified)**

| Type                                | Description                                                | Examples                                   |
| ----------------------------------- | ---------------------------------------------------------- | ------------------------------------------ |
| Not callable                        | Pure objects, constants, modules                           | `np.pi`, `tf.float32`                      |
| No input required                   | Zero-argument APIs                                         | `tf.random.set_seed()`, deprecated helpers |
| I/O or resource binding             | File, network, device, camera, database, GPU/TPU           | `cv2.VideoCapture`, `tf.io.read_file`      |
| Training / Optimization / Lifecycle | Stateful behavior, long loops, model fitting               | `model.fit`, `optimizer.step`              |
| Context / Scope management          | Session, tracing, compiling, gradient recording            | `tf.GradientTape`, `jax.jit`               |
| Uncontrollable randomness           | Non-seedable stochastic execution                          | remote sampling                            |
| Unbounded complexity                | Compilation, graph building, full training, big transforms | lazy eval, meta-ops                        |
| Runtime side effects                | Mutates global states or environment                       | env setters                                |
| Overloaded constructors             | Classes, layers, estimators                                | `torch.nn.Conv2d`, `tf.keras.Model`        |
| Operator wrappers                   | Raw / low-level / IR bound                                 | `tf.raw_ops.*`                             |

---

## **6. Output Format**

Refer to the txt file under doc2info/ .



