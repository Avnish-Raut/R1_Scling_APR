## Evaluation Strategies for Program Slicing in Automated Program Repair (APR)

This document outlines the evaluation metrics used to assess the effectiveness of the implemented program slicer. The evaluation framework is divided into two phases: core slicing metrics (baseline performance) and downstream LLM-based APR metrics (practical application for patch generation).

### 1. Core Slicing Metrics (Baseline)

* **Code Reduction (Slice Size):** Measures the extent to which the original program is minimized. It is calculated using the ratio of the sliced code to the original program: `(1 - slice_ratio) * 100`. A higher reduction percentage indicates a more efficient slice.
* **Fault Coverage:** Evaluates whether the generated slice retains the fault-relevant code. This is verified by cross-referencing the slice against the corresponding Defects4J developer patch to ensure all required modified lines are preserved.
* **Slice Precision & F1 Score:** Quantifies the presence of irrelevant code within the slice. Precision is calculated as the ratio of fault-relevant statements to the total statements in the slice. This is combined with recall to compute an F1 score, providing a balanced measure of filtration accuracy.
* **Overhead (Execution Time):** Tracks the computational cost of generating the slice. While not the primary objective, recording execution time is necessary to evaluate the scalability and practical viability of the approach, particularly for dynamic slicing techniques.

### 2. Downstream LLM & APR Metrics

* **Token Reduction:** Assesses the decrease in token count compared to the raw file. Because Large Language Models (LLMs) operate within strict context window constraints, token reduction serves as a more accurate indicator of LLM processing efficiency than standard line-count reduction.
* **Syntactic Validity (Parsability):** A pass/fail metric verifying that the generated slice can be successfully parsed into an Abstract Syntax Tree (AST). Slices that produce syntactically invalid code (e.g., missing brackets, orphaned variables) significantly increase the risk of LLM hallucinations during patch generation.
* **Downstream APR Plausibility:** Evaluates the functional utility of the slice. The APR tool is executed on both the original source file and the sliced file to determine if the reduced context successfully enables the model to generate a plausible patch that passes the associated test suite.