This document explains what the `run_eval_pipeline.sh` script actually does under the hood. It is a fully automated, idempotent data pipeline designed to extract ground truth from Defects4J, compute dynamic slices, and automatically grade the results.

## The 6-Stage Execution Pipeline

### Stage 1: Setup & Workspace Isolation
The script prevents "Filesystem Sprawl" by dynamically generating an isolated workspace for every bug (e.g., `node_auto_Math_1`). It checks out the buggy version of the project using Defects4J and compiles the source code and tests.

### Stage 2: Criterion A Extraction (Test Crash Line)
To perform dynamic slicing, Slicer4J needs a starting point. The script automatically runs the failing JUnit test, parses the stack trace (`failing_tests`), and isolates the exact Java class and line number where the test crashed. 

### Stage 3: Ground Truth Extraction (Criteria B & C)
The script queries Defects4J's internal framework (`classes.modified`) to find out which file the human developer actually fixed. 
* It navigates deep into the Defects4J installation to read the raw `.src.patch` diff file.
* It utilizes an advanced `awk` parser to dynamically count code lines down from the diff header, pinpointing the exact executable line the developer modified, bypassing formatting inconsistencies between SVN and Git diffs.
* This line becomes the **Ground Truth** for our evaluation.

### Stage 4: Instrumentation & Trace Generation
* The script packages all compiled classes into a Fat JAR (`app.jar`).
* It executes Slicer4J's `JavaInstrumenter` to inject tracking statements into the bytecode. *(Note: Slicer4J is deliberately restricted to a single CPU core using `taskset -c 0` to prevent server CPU exhaustion during parallel batching).*
* It runs the JUnit test again against the instrumented JAR to generate a massive bytecode execution log (`trace.log`).

### Stage 5: Slicing Passes
Slicer4J reads the `trace.log` and constructs a backward Dynamic Control Flow Graph (DCFG). It traces data dependencies backward from the Test Crash Line (Criterion A) to generate `slice_A.log`.

### Stage 6: Automated Validation (Fault Inclusion Check)
The pipeline automatically acts as a grader. 
* It searches inside `slice_A.log` for the exact Ground Truth line extracted in Stage 3.
* **[PASS]:** If the developer's buggy line is in the slice, it means the slicer's data-flow net successfully caught the root cause of the crash.
* **[FAIL]:** If the line is missing, the bug slipped through the slicer (often due to implicit control flow or unhandled exceptions).
* **[ERROR]:** If the Slicer4J engine crashes during heap mapping (common with older bytecode), it is gracefully caught, logged, and the batch moves to the next bug.