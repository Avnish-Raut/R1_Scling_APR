# Master's Thesis Project Outline: Evaluation Strategies for Program Slicing in Automated Program Repair (APR)

## 1. Introduction

- **The Gap:** Extracting minimal, fault-relevant context from large legacy codebases remains a significant challenge. Dynamic program slicing offers a contextual reduction solution, but its viability and theoretical limitations on real-world Java faults require rigorous evaluation.
- **RQ1 (Code Reduction):** How efficiently does dynamic slicing minimize source files compared to the original program?
- **RQ2 (Fault Coverage):** To what extent do the generated slices preserve the fault-relevant root-cause lines?
- **RQ3 (Performance Overhead):** What is the computational cost and scalability limit of instrumentation and dynamic trace generation?

## 2. Background and Related Work

- **2.1 Static vs. Dynamic Slicing:** Trade-offs between conservative static inclusion and execution-based dynamic tracing.
- **2.2 Automated Program Repair (APR):** Contextual requirements and the limitations of providing raw source code to repair engines.

## 3. Methodology and Evaluation Framework

- **3.1 Defects4J Benchmark & Sampling Strategy:** Rationale for targeted project suites (e.g., Commons Cli, Commons Compress, Commons Math). Application of Randomized Stratified Sampling (evaluating 20 random bugs per project) to balance architectural representation against computational constraints.
- **3.2 Pipeline Architecture:** Implementation details of the extraction engine (Slicer4J, Soot Instrumentation, and JUnit trace execution).
- **3.3 Slicing Criteria Variants:**
  - `test_crash`: The top application frame where the test failed.
  - `app_deepest`: The deepest application code frame in the stack trace.
  - `patched`: The ground-truth developer fix line.
- **3.4 Metric Definitions:** Establishing the evaluation formulas (e.g., isolating omission faults using the `UNDEFINED_ADDITIVE` flag and computing reduction via `red_vs_modclass`).

## 4. Theoretical Analysis: Pros and Cons of Tested Slicers

- **4.1 Tool Selection Criteria and Evaluation Scope:** Requirements imposed by the Defects4J benchmark: Java 8 bytecode, library
  subjects without `main` methods, JUnit-driven failure reproduction.The five candidates surveyed (Soot, WALA, Java SDG Slicer, JavaSlicer,Slicer4J) and the criteria that reduced them to three evaluated tools.
- **4.2 Java SDG Slicer: Source-Level Static Slicing:** SDG-based backward slicing,CLI-driven, JDK 11+, no execution required.
- **4.3 Soot: Framework Without a Slicer:** Viable but not affordable within the project timeline.
- **4.4 Slicer4J: Bytecode-Level Dynamic Slicing:** Instrumentation and tracing are
  criterion-independent, making additional criteria per bug near-free; this
  property makes RQ1 tractable
- **4.5 Architectural Limitations:** Weaknesses identified during testing, including JVM heap memory mapping failures, Soot instrumentation overhead, and unhandled legacy bytecode exceptions.
- **4.6 The "Unexecuted Anchor" Limitation:** The theoretical vulnerability of dynamic slicers when dealing with omission bugs (missing code) or unexecuted conditional branches (e.g., the `CRITERION_NOT_EXECUTED` failure state).
- **4.7 4.7 Synthesis: Source-Level versus Bytecode-Level Analysis**

## note : Check with Adam if we need to switch Section 3 and Section 4

## 5. Empirical Results (Core Slicing Metrics)

- **5.1 Code Reduction and Size Efficiency:** Quantitative analysis of slice sizes vs. modified class sizes across the sampled repositories.
- **5.2 Fault Coverage and Preservation:** Hit rates for capturing ground-truth developer patches (FULL vs. PARTIAL vs. NONE).
- **5.3 Precision, F1 Score, and Pipeline Overhead:** Evaluating the noise-to-signal ratio and the computational timings (`t_trace`, `t_slice`) of the dynamic analysis.

## 6. Discussion and Threats to Validity

- **6.1 Interpreting Slicer Engine Crashes:** Analyzing the root causes of pipeline failures (e.g., memory exhaustion on massive trace logs).
- **6.2 Omission Faults and Additive Bugs:** The statistical impact of `UNDEFINED_ADDITIVE` bugs on historical fault localization metrics.
- **6.3 Domain-Specific Noise:** Contrasting trace pollution across architectures (e.g., heavy arithmetic lookup tables in `Math` vs. deep compression algorithms in `Compress`).
- **6.4 Selection Bias and Sampling:** Acknowledging the random 20-bug threshold and how dataset sampling impacts the generalizability of the findings.

## 7. Conclusion and Future Work

- **7.1 Conclusion:** Slicer4J achieves high mathematical code reduction and surgical fault isolation, but is constrained by the necessity of executed anchor lines, trace file scalability, and bytecode compatibility.
- **7.2 Future Work: Downstream LLM & APR Suitability:**
  - _Token Constraints:_ Using slices to fit strict LLM context windows.
  - _AST Fragmentation:_ Addressing Slicer4J's raw output (orphaned variables, missing brackets) and the risk of LLM hallucinations.
  - _Syntax Reconstruction:_ The future need for a post-processing wrapper to convert raw slice logs into syntactically valid code blocks for automated repair tools.
