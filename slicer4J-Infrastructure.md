# Infrastructure Setup & Technical Resolutions: Slicer4J and Defects4J Integration

## 1. Executive Summary
Integrating the Defects4J benchmark suite with Slicer4J requires bridging legacy academic program analysis frameworks with a modern execution environment. Several architectural mismatches exist during the initial setup, primarily revolving around the underlying Soot analysis engine, classpath structures, and memory management. This document outlines the core technical blockers and the applied engineering resolutions used to establish a stable, automated slicing pipeline.

## 2. Environment Configuration
* **Target Directory:** `/home/research_slicing/parallel_execution/`
* **Permissions:** Shared access via `studadmin` group (`chmod -R 775`).
* **Frameworks:** Defects4J, Slicer4J, DynamicSlicingCore.
* **Runtime Context:** Automated batch execution mapping isolated bug IDs to dedicated `node_auto` workspaces.

## 3. Technical Roadblocks and Resolutions

### Issue 1: Soot Engine Concurrency Crashes
* **The Problem:** During the bytecode instrumentation phase (`-m i`), the Slicer4J pipeline sporadically fails with core dumps or thread-related exceptions.
* **Root Cause:** Slicer4J relies on an older version of the Soot program analysis framework. This legacy version is not thread-safe when running on modern multi-core host machines. The JVM attempts to parallelize bytecode analysis, leading to fatal race conditions within Soot's internal state.
* **The Resolution:** Enforce strict sequential execution for the instrumentation phase by applying operating system and JVM-level resource constraints. Use `taskset -c 0` to pin the process to a single physical CPU core, and pass `-XX:ActiveProcessorCount=1` to the JVM to prevent internal thread spawning.
* **Execution Target:**
  ```bash
  taskset -c 0 java -XX:ActiveProcessorCount=1 -Xmx4g -jar slicer4j-jar-with-dependencies.jar -m i -j app.jar -o slicer_workspace -lc DynamicSlicingLogger.jar
  ```

### Issue 2: Classpath Fragmentation (The "Fat JAR" Assembly)
* **The Problem:** Slicer4J fails to generate execution traces because it cannot dynamically map the compiled test files executing against the isolated production codebase.
* **Root Cause:** Defects4J compiles projects into a split directory structure (e.g., `target/classes` for production code and `target/test-classes` for unit tests). Slicer4J requires a unified target file to properly instrument and trace the bytecode.
* **The Resolution:** Engineer an intermediate build step in the pipeline. Before passing the codebase to Slicer4J, a bash routine aggregates all compiled `.class` files from both production and test directories into a unified `combined_classes` directory. This is subsequently packaged into a single "Fat JAR" (`app.jar`), serving as the singular source of truth for the slicer.

### Issue 3: Memory Exhaustion via Standard Libraries
* **The Problem:** Running a backward slice on standard Apache Commons bugs results in out-of-memory (OOM) errors, severely extending processing time without yielding a valid `slice.log`.
* **Root Cause:** By default, dynamic slicing algorithms attempt to trace execution down to the lowest level. Without boundaries, Slicer4J attempts to analyze the entirety of the Java Standard Library (e.g., `java.lang.String` or `java.util.List`), exponentially exploding the execution graph.
* **The Resolution:** Integrate Slicer4J's pre-computed stub models to define hard analysis boundaries. Map the `-sd` flag to the `models/summariesManual` directory and the `-tw` flag to the `EasyTaintWrapperSource.txt` file. This instructs the underlying engine to skip deep tracing of standard JDK libraries and strictly isolate the analysis to the target application logic.