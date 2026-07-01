# Literature Review Summaries: Program Slicing Research

---

## Paper 1

**"Program Slicing: A Brief Retrospective"**
Keith B. Gallagher & Suzanne J. Kozaitis
_IEEE Transactions on Software Engineering_, Vol. 51, pp. 720–724, March 2025
DOI: 10.1109/TSE.2025.3538279

### Overview

This paper is a concise retrospective article published in IEEE Transactions on Software Engineering, written by one of the field's early pioneers. Gallagher — who collaborated with Mark Weiser in the formative years of program slicing — reflects on the trajectory of the technique from its inception in the late 1970s and early 1980s through to the present day. The paper serves as both a historical account and a commentary on the state and direction of the field.

### Key Definitions and Conceptual Framing

The authors define program slicing as a software analysis technique originally developed by the late Mark Weiser, designed to automatically remove code from a program that does not affect a given computation. A **program slice** is any selection of program statements that preserves the same behaviour as the original program at a specific point for a given set of variables — referred to as the **slicing criterion**. The core utility of the technique is that it allows a software engineer to focus on an immediate computation while safely ignoring unrelated statements and variables, reducing cognitive load during tasks such as debugging, maintenance, and comprehension.

### Historical Trajectory

The paper traces the development of slicing from Weiser's foundational ICSE 1981 paper and his subsequent 1984 IEEE TSE article, through the decades of refinement that followed. Gallagher positions this retrospective in the context of a field that has matured across multiple dimensions: from purely static backward slicing to dynamic variants, forward slicing, inter-procedural slicing, observation-based slicing (ORBS), and, more recently, learning-based and LLM-assisted approaches. The paper highlights the role of the Program Dependence Graph (PDG), introduced by Ferrante, Ottenstein, and Warren in 1987, as a pivotal data structure that formalised and accelerated progress in slicing algorithms.

### Scope and Applications

Gallagher and Kozaitis enumerate the breadth of applications that have been explored for program slicing over several decades, including:

- **Software maintenance and evolution**: identifying code that can be safely changed without side effects
- **Debugging and fault localisation**: narrowing the search space for bugs to the smallest relevant subset of statements
- **Program comprehension**: helping developers understand what a piece of code does by focusing on a variable or output
- **Testing and regression analysis**: determining which tests need to be re-run after a change
- **Security and taint analysis**: tracing data flows related to potentially vulnerable variables
- **Refactoring and modularisation**: decomposing programs into cohesive functional units

### Commentary on Modern Directions

The retrospective gestures toward the challenges and opportunities that remain. The authors observe that despite decades of research, practical adoption of slicing tools in industrial settings has been limited, partly due to scalability issues with classical graph-based approaches and the difficulty of handling modern language features (e.g., dynamic dispatch, concurrency, native code, lambda expressions). The paper notes the growing role of machine learning and pre-trained language models in enabling new forms of slicing — particularly for incomplete or partial code — as an exciting frontier that may overcome some of the limitations of traditional techniques.

### Significance for the Literature Review

This paper is valuable as a **foundational and contextualising reference** for any literature review on program slicing. It situates the field historically, provides authoritative definitions, and identifies the major milestones and open problems. For a review covering both classical and modern (LLM-based) slicing approaches, this retrospective establishes the conceptual baseline against which newer contributions (such as Slicer4J, DynAbs, NS-Slicer, and LLM-based slicing) can be measured.

---

## Paper 2

**"Program Slicing in the Era of Large Language Models"**
Kimya Khakzad Shahandashti, Mohammad Mahdi Mohajer, Alvine Boaye Belle, Song Wang, Hadi Hemmati
_arXiv preprint_, arXiv:2409.12369v1, 2024

### Overview

This paper presents the first comprehensive empirical study evaluating large language models (LLMs) on the task of program slicing — covering both static and dynamic slicing of Java programs. The study benchmarks four state-of-the-art LLMs (GPT-4o, GPT-3.5 Turbo, Llama-2-7B-Chat, and Gemma-7B) using advanced prompting strategies, develops a novel taxonomy of LLM slicing failures, and evaluates two improvement strategies guided by that taxonomy. The work is conducted at York University, Canada, and positions itself as bridging a gap in the literature: while LLMs have been applied to many software engineering tasks, their capacity to perform formal dependency-based analysis like program slicing had not previously been studied.

### Research Questions

The paper is structured around four research questions:

- **RQ1**: How do LLMs perform on static program slicing?
- **RQ2**: How do LLMs perform on dynamic program slicing?
- **RQ3**: What are the characteristics of unsuccessful static program slices (root causes and locations)?
- **RQ4**: How do prompt enhancement strategies improve LLM-based static slicing accuracy?

### Methodology

**Dataset**: 100 Java programs were randomly selected from a dataset of LeetCode solutions. These programs were preprocessed for compatibility with traditional slicing tools: JavaSlicer was used for static slicing ground truth and Slicer4J for dynamic slicing ground truth. Slicing criteria (the target variable and line) were manually selected to focus on the program's core logic.

**Ground Truth Construction**: A two-step process was used — traditional tools generated candidate slices, which were then manually verified by two experienced reviewers to correct any inaccuracies.

**LLMs Evaluated**:

- GPT-4o (OpenAI, 8K context, released March 2024)
- GPT-3.5 Turbo (OpenAI, 4K context)
- Llama-2-7B-Chat (Meta AI, 4K context, 7B parameters)
- Gemma-7B (Google, 8K context, 7B parameters)

**Prompting Strategies**: Three setups were evaluated for static slicing — zero-shot, one-shot, and one-shot with Chain-of-Thought (CoT). Dynamic slicing used the same three strategies. Each experiment was run three times and averaged to mitigate non-determinism.

**Evaluation Metrics**:

- **Accuracy-D (Dependence Accuracy)**: ratio of correctly predicted inter-statement dependencies
- **Accuracy-EM (Exact-Match Accuracy)**: fraction of cases where predicted slice exactly matches ground truth

### Results: RQ1 — Static Slicing

GPT-4o achieved the best performance with one-shot + CoT prompting: **60.84% Accuracy-D** and **7.33% Accuracy-EM**. For comparison, the NS-Slicer baseline (CodeBERT variant) achieved 60.52% Accuracy-D and 0% Accuracy-EM. This means GPT-4o slightly outperformed NS-Slicer on dependency accuracy and produced more exact matches, though the overall performance remains insufficient for practical deployment. The ordering of models from best to worst was GPT-4o > Gemma-7B > GPT-3.5 Turbo > Llama-2. Zero-shot prompting consistently underperformed relative to one-shot and CoT variants, with GPT-4o dropping to 30.91% under zero-shot.

### Results: RQ2 — Dynamic Slicing

Dynamic slicing proved even more challenging. GPT-4o achieved its best performance of **59.69% Accuracy-D** under the zero-shot strategy — counter-intuitively, additional in-context examples did not help and sometimes hurt performance on dynamic slicing, likely because the additional context introduced confusion around runtime execution semantics. No model achieved any exact matches (0% Accuracy-EM) across any dynamic slicing experiment, highlighting a fundamental gap in LLM capability for execution-aware analysis. A Mann-Whitney U test found no statistically significant difference between static and dynamic slicing performance (p = 0.62), though exact matches favoured static slicing for all models.

### Results: RQ3 — Taxonomy of Failures

Manual analysis of 92 unsuccessful slicing cases (from GPT-4o under one-shot + CoT) produced a novel taxonomy organised along two dimensions:

**Root Causes (3 categories)**:

1. **Lack of Logic Understanding** — failures related to conditional branches (the LLM omits else branches), loops (failure to propagate slice criteria across iterations), and method invocations (failure to track data flow through built-in data structure methods like `add()`, `remove()`).
2. **Code Complexity** — failures caused by ambiguous multi-line expressions and, most critically, **Complex Control Flow** (nested loops and conditionals). Complex Control Flow was the **most frequent root cause**, appearing in 39 of 92 cases.
3. **Model-Specific Constraints** — context window limits causing truncation, confusion between code and embedded text (comments, string literals), and JSON output parsing failures.

**Fault Locations (6 categories)**: Conditional Statements, Loop Constructs, Method Invocations and Returns, Variable Declarations and Assignments, Class Declarations, and Imports. **Variable Declarations and Assignments** was the most common fault location, appearing in 78 of 92 cases — indicating that LLMs frequently fail to correctly track variable dependencies at the point of declaration or modification.

### Results: RQ4 — Prompt Enhancement Strategies

Two improvement strategies, both guided by the taxonomy, were evaluated:

1. **Enhanced Prompt Crafting**: The one-shot example was replaced with an example specifically targeting the most frequent failure patterns (Complex Control Flow and Variable Declarations). GPT-4o improved by approximately 4%; GPT-3.5 Turbo improved by 6.4%; Gemma-7B by 5.1%; Llama-2 by 2.5%.

2. **Iterative Prompting** (applied to GPT-4o only): After identifying a failed slice, the model was given explicit feedback identifying the root cause and failure location, then asked to regenerate. A single iteration improved GPT-4o's Accuracy-D by 3.9%. The authors note that while effective, this technique requires a human-in-the-loop for verification.

### Limitations and Threats to Validity

The study is limited to Java programs from LeetCode, which may not generalise to other languages or larger industrial codebases. The same dataset was used to build the taxonomy and evaluate improvements, which introduces some circularity. The study only covers four LLMs and does not include newer or fine-tuned code models.

### Significance for the Literature Review

This paper is a **central empirical contribution** for any review on LLM-based program analysis. It provides systematic evidence of where and why LLMs fall short in a formal dependency analysis task, and the taxonomy of failures offers a conceptual framework for future work. Its comparison with NS-Slicer also bridges learning-based and prompt-based approaches to slicing.

---

## Paper 3

**"Focused Dynamic Slicing for Large Applications using an Abstract Memory-Model"**
Alexis Soifer, Diego Garbervetsky, Victor Braberman, Sebastian Uchitel
_arXiv preprint_, arXiv:2211.04560v1, November 2022

### Overview

This paper proposes a novel approach to dynamic program slicing that addresses one of the most persistent limitations in the field: **scalability to large real-world applications**. The key insight is that classical dynamic slicers are bottlenecked by the use of concrete memory references to identify data dependencies — a mechanism that scales poorly when libraries, frameworks, and application code are all instrumented uniformly. The authors introduce **abstract dynamic slicing**, a technique that replaces memory reference tracking with a symbolic memory analysis model that operates on program terms (symbols) rather than runtime addresses. This enables a coarser-grained treatment of code outside the focus of analysis (e.g., libraries and frameworks), making it possible to omit instrumentation for that code entirely. The paper presents **DynAbs**, an implementation of this approach for C#, and evaluates it on two large, modern open-source C# projects: the Roslyn C# compiler and the PowerShell console.

### Motivation and Core Problem

Classical dynamic slicing requires an execution trace — a record of all statements executed and the memory addresses involved in each operation. For large applications (hundreds of thousands of lines of code), this trace can be enormous and the memory reference processing prohibitively expensive. A specific pain point is the inability to distinguish the code a developer cares about (the application code under analysis) from code they do not (third-party libraries and framework internals). Since both kinds of code must be fully instrumented in classical approaches, there is no opportunity to trade precision for efficiency.

### Technical Approach: Abstract Dynamic Slicing

The core innovation is replacing concrete memory references with a **memory analysis model** that works with program symbols (i.e., named variables and fields as they appear in source code). The key architectural consequence is:

- For **code within the analysis focus** (the application code), standard dynamic dependency tracking is performed using symbolic identifiers rather than runtime memory addresses.
- For **code outside the focus** (libraries, frameworks), instrumentation can be entirely omitted. A coarser-grained abstraction is applied that over-approximates the data dependencies, sacrificing some precision but making the analysis tractable.

This "focused" slicing model means the analyst can declare which parts of the codebase they want to slice precisely, and everything else is handled approximately. The approach is proven sound — it never misses true dependencies, it may only over-include.

### Tool: DynAbs for C#

DynAbs implements abstract dynamic slicing for C# programs. The tool:

- Instruments only the designated focus code (application classes of interest), generating symbolic execution traces rather than memory-address-based traces
- Processes these traces to identify data and control dependencies using the symbolic model
- Applies pre-built or inferred summaries for uninstrumented framework code
- Computes backward slices from a given slicing criterion (e.g., a test assertion)

### Evaluation

The authors evaluated DynAbs on slicing criteria derived from test assertions in two of the largest C# projects on GitHub:

- **Roslyn** (the C# compiler): a large, complex project with extensive use of C# framework classes
- **PowerShell** (Microsoft's scripting engine): another major, modern C# application

Results demonstrated that DynAbs can slice relevant portions of both applications **within a few minutes**, something that would be computationally infeasible with a standard trace-based dynamic slicer. The evaluation also showed that **reducing the code-to-be-sliced focus** (i.e., narrowing what is precisely analysed) produces significant speedups with only marginal relative precision loss. This validates the core design trade-off: by being selective about what is deeply analysed, large-scale practical slicing becomes achievable.

A comparison with a state-of-the-art dynamic slicing tool on a common benchmark confirmed that DynAbs produces comparable precision on smaller programs while dramatically outperforming classical tools on large-scale targets.

### Significance for the Literature Review

This paper makes an important contribution to the **scalability frontier** of dynamic slicing. It demonstrates that the binary instrumentation model — where either everything is traced or nothing is — is not the only option, and that symbolic abstraction offers a viable path to industrial-scale applicability. For a literature review, this work is particularly relevant in the context of discussions around the practical limitations of dynamic slicing tools like Slicer4J (which is constrained to smaller programs) and the need for more scalable architectures. DynAbs also introduces a methodological shift (focus-based analysis) that has broader relevance for program analysis research.

---

## Paper 4

**"Slicer4J: A Dynamic Slicer for Java"**
Khaled Ahmed, Mieszko Lis, Julia Rubin
_Proceedings of the 29th ACM Joint Meeting on ESEC/FSE 2021_, pp. 1570–1574
DOI: 10.1145/3468264.3473123

### Overview

This paper presents **Slicer4J**, a new open-source dynamic slicing tool for Java programs, contributed by researchers at the University of British Columbia. The work directly addresses a significant gap in the Java tooling ecosystem: at the time of publication, the only publicly available dynamic slicing tool for Java was **JavaSlicer**, which had substantial limitations — it only supported Java 6 and below, did not handle multithreading, and could not process native Java framework methods. Slicer4J was designed to overcome all three limitations. The paper describes the tool's architecture, key design decisions, and an evaluation on benchmark programs as well as real-world buggy projects from the Defects4J dataset.

### Motivation

Dynamic program slicing is widely useful for tasks such as debugging, fault localisation, regression testing, and security analysis. However, the practical utility of any dynamic slicer depends on its ability to handle modern language features and realistic program structures. JavaSlicer's inability to handle Java versions above 6 (excluding features like lambda expressions), its lack of multithreading support, and its failure on framework/native methods meant that it was effectively unable to analyse most contemporary Java applications. Slicer4J was created to fill this void.

### Architecture and Design

Slicer4J operates on Java Archive (JAR) files and produces backward dynamic slices. Its pipeline comprises three phases:

**1. Instrumentation**: Slicer4J uses Soot (a Java bytecode analysis and transformation framework) to statically instrument the JAR prior to execution. It records a unique execution thread ID and a unique statement identifier for each executed statement. An option for basic-block-level instrumentation (recording block IDs rather than individual statement IDs) is available for less multithreaded code, reducing trace overhead. For shared variables (object fields), Slicer4J uses `System.identityHashCode()` to assign unique runtime identifiers to objects, enabling accurate tracking of field-level data dependencies across multiple threads.

**2. Framework Method Modeling**: Since Java framework classes (e.g., `java.lang.System`, `java.util.ArrayList`) are not part of the input JAR, they cannot be instrumented. Slicer4J handles this by importing **taint-flow summaries** from FlowDroid and StubDroid — static analyses that describe how data flows through framework methods. These summaries are reinterpreted as assignment statements for slicing purposes (e.g., `System.arraycopy(src, ..., dst, ...)` is modelled as `dst = src`). Users can also supply custom summaries for their own native or user-defined methods.

**3. Trace Parsing and TDCG Construction**: After the instrumented program is executed (via command line or test script), Slicer4J parses the trace and constructs a **Thread-aware, Inter-procedural Dynamic Control-flow Graph (TDCG)**. This graph has nodes for each executed statement instance and three types of edges: control-flow edges (within a thread), thread-control-flow edges (across threads), and control-dependence edges (derived from Soot's static control dependency graph). Dynamic slicing then applies Agrawal et al.'s backward slicing algorithm on this TDCG.

**4. Output**: Slicer4J outputs the slice as a list of source code line numbers (when debug information is available in the JAR), plus a raw Jimple-level representation of the slice with thread and instance annotations.

### Evaluation

**Benchmark Programs**: Seven custom benchmark programs were used to verify Slicer4J's capabilities beyond those of JavaSlicer. These included tests for intra-procedural and inter-procedural data flows, exception-based control dependence, multithreaded data flows, Java 9 constructs (lambda expressions), native method calls, and framework methods used internally by JavaSlicer's own instrumentation classes. Slicer4J produced the correct expected slice in all seven cases. JavaSlicer produced the expected slice for the three classic (Java 6, single-threaded) benchmarks but failed on all four cases exercising Slicer4J's extended capabilities.

**Defects4J Programs**: Slicer4J was also evaluated on three real-world Java projects from the Defects4J bug dataset — JacksonDatabind, Gson, and JacksonCore — slicing from failing test assertions to identify whether the actual buggy lines fell within the slice. Both Slicer4J and JavaSlicer successfully identified the buggy lines in all three cases. However, Slicer4J produced substantially smaller slices for two of the three projects (58 lines vs 367 for JacksonDatabind; 13 lines vs 311 for Gson), due to more accurate modeling of bytecode field-access dependencies. JavaSlicer over-includes statements because its bytecode-level analysis adds erroneous data-flow dependencies (e.g., when processing `y.f = x`, it incorrectly adds `y` to the working set alongside `x`). Slicer4J avoids this.

**Performance**: Slicer4J's instrumentation was static (performed once, ahead of execution), compared to JavaSlicer's JVM-level instrumentation performed at class load time. For programs with small traces relative to code size, JavaSlicer's on-demand instrumentation was faster. For programs with very large traces (Gson: 580K-line trace; JacksonCore: 809K-line trace), Slicer4J's approach was more efficient overall. Slicing times were under five minutes in all evaluated cases, which the authors deem acceptable.

### Limitations

Slicer4J cannot slice dynamically generated classes (those produced at runtime), as these are not present in the JAR at instrumentation time. The TDCG is constructed from the full trace before slicing begins, which increases initial overhead; on-demand TDCG construction is left as future work. Multi-threaded trace writing introduces synchronisation overhead that may slow down heavily threaded applications. Finally, Slicer4J's support for Java constructs is bounded by what Soot supports (Java 9 at time of writing).

### Significance for the Literature Review

Slicer4J is a **practically significant tool contribution** that directly enables downstream research. It is used as the dynamic slicing ground truth generator in the LLM-based slicing empirical study (Paper 2 above), and it fills the gap that JavaSlicer left for modern Java. For a literature review on program slicing, Slicer4J is important both as a tooling milestone and as an illustration of how design choices in instrumentation, framework modeling, and graph construction affect precision, recall, and scalability. Its handling of multithreading is particularly novel among open-source Java slicers and opens the door for slicing concurrent programs in realistic Java applications.

---

## Cross-Cutting Themes and Connections

Reading these four papers together, several themes emerge that are directly relevant for a literature review on program slicing:

**Scalability vs. Precision Trade-offs**: DynAbs (Paper 3) explicitly trades precision for scalability via abstract memory modelling. Slicer4J (Paper 4) improves precision over JavaSlicer by avoiding spurious bytecode dependencies, but at some additional slicing time cost. Classical tools consistently struggle to scale to large codebases — a gap that both abstraction-based (DynAbs) and learning-based (LLM/NS-Slicer) approaches attempt to bridge.

**Static vs. Dynamic Slicing**: The Gallagher retrospective (Paper 1) frames these two paradigms historically. Slicer4J is a dynamic tool; the LLM study (Paper 2) evaluates both. LLMs appear to handle static slicing slightly better than dynamic, likely because dynamic slicing requires execution-context reasoning that is difficult to capture in a prompt.

**LLM Limitations in Formal Analysis**: Paper 2 provides compelling evidence that general-purpose LLMs — even the most capable at the time (GPT-4o) — fall well short of tool-level reliability in dependency tracking, particularly for complex control flow and nested structures. The failure taxonomy provides a roadmap for targeted improvements.

**Tool Availability and Reproducibility**: Slicer4J's open-source release (GitHub) directly enabled the LLM study to use it as a ground truth generator, illustrating how tool contributions catalyse empirical research. DynAbs similarly contributes implementation artifacts. The LLM study also releases its dataset and code, promoting reproducibility.

**Modern Language Features**: A consistent theme across Papers 3 and 4 is the challenge of handling modern language features (lambdas, native methods, threads, framework internals). Classical static analysis tools were not designed for these, and DynAbs and Slicer4J each represent targeted engineering efforts to address specific subsets of these challenges.

Part 2

1. A Survey of LLM-based Software Repair: Taxonomies, Design Paradigms, and Applications
   Overview: This paper presents a comprehensive taxonomy classifying 62 recent LLM-based software repair systems into four distinct design paradigms (fine-tuning, prompting, procedural pipelines, and agentic frameworks). It overlays these paradigms with two enhancement layers: retrieval-augmented generation (RAG) and analysis-augmented generation (AAG).

Key Definitions and Conceptual Framing: The study defines its taxonomy based on two axes: control authority over the repair loop and whether the base model's parameters are adapted or frozen. RAG is defined as augmenting input with external knowledge (e.g., code snippets, documentation), while AAG incorporates static or dynamic program-analysis results.

Historical Trajectory: The adoption of LLM-based software repair has rapidly shifted from single-turn, zero-shot prompting (dominant in 2022) to more complex procedural and agentic workflows by 2024 and 2025. Fine-tuning has evolved from full-parameter updates to parameter-efficient fine-tuning (PEFT) and reinforcement learning fine-tuning (RLFT).

Scope and Applications: The surveyed systems cover a wide defect scope, including educational assignments, single-function bugs, multi-hunk repository-level issues, and security vulnerabilities across C/C++ and Java.

Commentary on Modern Directions: The authors stress the need to separate generation from evaluation via "LLMs-as-Judges" and highlight critical bottlenecks like the high computational cost of agentic loops and data contamination in benchmarks. They suggest cost-aware curricula and hybrid repair policies that default to simple heuristics before escalating to full autonomous agents.

Significance for the Literature Review: It provides the most up-to-date and unified framework for understanding the LLM program repair landscape. By consolidating benchmark scopes, evaluation practices, and pass@k definitions into a single comparison grid, it exposes how metric design heavily skews reported success rates.

2. On the Role of Context Granularity in LLM-Driven Program Repair
   Overview: This paper investigates how different sizes of surrounding code (context granularity) affect the success of LLM-based Automated Program Repair (APR) using GPT-4. It proposes a novel context based on backward static slicing and compares it to five standard granularities.

Key Definitions and Conceptual Framing: Context granularity is defined as the amount of code surrounding a buggy line provided to the repair system. The proposed Sliced context uses backward static slicing to isolate only the lines that are data- and control-dependent on the bug, discarding irrelevant lines.

Historical Trajectory: Traditional APR relied heavily on rule-based heuristics and genetic programming. With the rise of Neural Machine Translation models for code, context became an issue; providing entire files often degraded performance due to the "lost in the middle" phenomenon.

Scope and Applications: Evaluated against 109 single-line bugs from the Defects4J dataset using zero-shot learning in GPT-4. Metrics tracked were syntactically compilable, plausible (passes tests), and semantically correct patches.

Commentary on Modern Directions: The authors found that while a full-file context produced the most correct patches overall, the Sliced context achieved the highest Correct/Plausible ratio (79%). This means highly focused contexts generate semantically accurate patches and reduce overfitting to weak test cases.

Significance for the Literature Review: It challenges the "bigger context is better" assumption in LLM prompts. By proving that static slicing significantly reduces noise and yields a better Correct/Plausible patch ratio than full-file context or state-of-the-art systems like AlphaRepair, it tightly bridges program analysis with prompt engineering.

3. Program Decomposition and Translation with Static Analysis
   Overview: This work tackles the issue of translating massive, industry-scale software files using LLMs restricted by context window limits. It evaluates how method-level decomposition using static analysis enables the processing and translation of massive source files that otherwise trigger out-of-context errors.

Key Definitions and Conceptual Framing: Program decomposition involves breaking down large programs into smaller, independent fragments (methods) that retain their original behavior. A Call Graph (CG) is used to map dependencies and orchestrate a bottom-up translation strategy.

Historical Trajectory: As LLMs revolutionized Programming Language Processing (PLP) and code translation, their hard token limits (e.g., 2K tokens for standard models) became a severe bottleneck when exposed to heavily interdependent, enterprise-scale legacy codebases.

Scope and Applications: The empirical study analyzed roughly 60,000 methods across 20 well-known Apache Commons projects in Java using the StarCoder LLM. Translation feasibility was qualitatively analyzed on the Apache Commons CLI project.

Commentary on Modern Directions: The study demonstrates that roughly 30% of standard industry files fail to fit within a 2K token window. Method-level decomposition eliminates 99.5% of these out-of-context errors, reducing input sizes to consume only about 5% of the window, leaving crucial space for the prompt and generated output.

Significance for the Literature Review: It provides quantitative evidence on the necessity of static pre-processing for LLMs. The paper validates that merging traditional Call Graph analysis with LLM pipelines is a scalable solution for whole-repository translation tasks.
