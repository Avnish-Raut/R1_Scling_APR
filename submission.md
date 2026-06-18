## Context

Modern software systems consist of thousands of lines of source code distributed across numerous files and modules. When developers debug software defects, only a small fraction of the codebase is typically relevant to the observed failure. However, identifying these relevant portions manually is difficult and time-consuming. Program analysis techniques such as program slicing have been proposed to automatically extract the subset of code that influences a particular program behavior or execution outcome.

Program slicing has been studied extensively in software engineering for debugging, maintenance, testing, and program comprehension. A slice contains only the statements that are directly or indirectly related to a slicing criterion, such as a variable value, program output, or failing test case. This project focuses on evaluating the effectiveness of program slicing techniques using the Defects4J benchmark, a widely used collection of real-world Java bugs.

## Problem

### Practical Problem

Large software projects contain substantial amounts of code that are unrelated to a specific defect. Developers often spend significant effort navigating through source files, dependencies, and execution paths before locating the root cause of a failure. This challenge becomes more severe as project size and complexity increase. Existing debugging processes frequently require developers to analyze large portions of a codebase even though only a small subset contributes to the observed failure.

An effective slicing technique could significantly reduce the amount of code that must be examined during debugging. By isolating only the statements relevant to a failing test case, developers may be able to identify faults more quickly and with less manual effort.

### Scientific Problem

Although program slicing has been studied for decades, there is still limited understanding of how effectively different slicing approaches reduce code size while preserving fault-relevant information in modern software benchmarks. Static slicing often produces large slices because it conservatively includes all possible dependencies, whereas dynamic slicing may generate smaller slices by considering actual execution traces.

The scientific challenge is therefore to determine how much reduction can be achieved through slicing while maintaining sufficient information to identify the fault. Furthermore, it is necessary to investigate whether dynamic slicing, static slicing, or hybrid approaches provide the best trade-off between slice size and fault coverage within real-world defects from Defects4J.

## Time plan :

Literature review [2 weeks - 22nd June to 4th July]
Research of approaches[2 weeks - 29th June to 11th July]
Research on Defects4j suitable projects[2 weeks - 6th July to 18th July]
Deciding on approaches[ 2 weeks - 20th July to 7th August]
Select or come up with evaluation stategies[2 weeks - 20th July to 7th August]
Note : Buffer time 2 weeks
Perform Parallel Analysis[9 weeks - 10th August to 10th October]

- Run tests on Defects4j
- provide them to slicers
- compare with standard benchmark
- repeat

Note : Buffer time 3 weeks
Evaluate the result [3 weeks - 12th October to 31st October]
Write research paper[2 weeks - 1st November to 15th November]
Submission on artifacts [20th November ]

|               Task                |  Akash  | Son Thien | Avnish  |
| :-------------------------------: | :-----: | :-------: | :-----: |
|         Literature Review         |   12    |    10     |   10    |
|      Research of approaches       |   26    |    30     |   20    |
|       Research on Defects4j       |   13    |    20     |   11    |
|    Proper setup for Defects4j     |   25    |    20     |   23    |
|      Deciding on approaches       |   16    |    10     |   14    |
| Deciding on evaluation strategies |   15    |    10     |   15    |
|         Perform Analysis          |   80    |    90     |   102   |
|        Evaluate the result        |   38    |    40     |   32    |
|       Write research paper        |   20    |    15     |   20    |
|             **Total**             | **245** |  **245**  | **245** |

Note : in hrs
Exam week :20th July to 27th July
Buffer time included for exams,other project and vacation -- If buffer time used , then some tasks will run parallel

## Slicer Summary :

### Static slicers

1. Soot

Name: Soot
Category: Static
Java Compatibility: Official docs say Soot processes Java bytecode/source up to Java 7; it is mainly a legacy baseline for Java 8 work.
Research Value: High
Pro: Mature analysis framework; supports call-graph construction, points-to analysis, def/use chains, and interprocedural data-flow analysis.
Cons: The docs explicitly say the old Soot line is succeeded by SootUp, and the documented Java input range stops at Java 7, so Java 8 compatibility may need extra adaptation.
Build own slicing approach
[Repo](https://github.com/soot-oss/soot)

2. WALA

Name: WALA
Category: Static
Java Compatibility: The current official release is built for Java 17 or newer, so it is not a clean fit for a Defects4j.
Research Value: High
Pro: Strong static-analysis suite; the README explicitly lists a context-sensitive tabulation-based slicer, pointer analysis, and call-graph construction.
Cons: Current release targets Java 17+, which makes setup less convenient for a Defects4J study.
[Repo](https://github.com/wala/WALA)

3. Java SDG Slicer

Name: Java SDG Slicer
Category: Static
Java Compatibility: The repo says it requires JDK 11 or newer.
Research Value: High
Pro: Clean SDG-based slicing; it has a CLI that takes a slicing criterion and outputs the corresponding slice.
Best ready-to-use static slicer
Cons: All method calls must resolve, libraries must be available via -i, unresolved calls cause runtime errors, and the repo notes missing support for parallel features such as threads and synchronized methods.
[Repo](https://github.com/mistupv/JavaSlicer)

### Dynamic Slicers

4. JavaSlicer

Name: JavaSlicer
Category: Dynamic
Java Compatibility: The README says it requires JDK 1.6 or 1.7 and explicitly warns that JDK 1.8 introduces new language features it cannot handle.
Research Value: Medium as a historical baseline
Pro: True dynamic slicing tool; it computes dynamic backward slices of Java programs using a Java agent.
Cons: It is old and explicitly not compatible with Java 8 language features
[Repo](https://github.com/backes/javaslicer)

5. Slicer4J

Name: Slicer4J
Category: Dynamic
Java Compatibility: The README says it requires Java Runtime Environment 8, and it relies on Soot instrumentation that currently supports programs compiled up to Java 9.
Research Value: Very high
Pro: Accurate, low-overhead dynamic slicer; automatically generates backward dynamic slices from an executed statement and the variables used there. It also provides a runnable toolchain and benchmarks.
Cons: Depends on DynamicSlicingCore and Soot, and the repo warns that trace size can hit memory limits because it may use up to 8 GB RAM.
[Repo](https://github.com/resess/Slicer4J)

Best Slicer to work it.
