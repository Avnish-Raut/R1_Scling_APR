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
|      Research of approaches       |   27    |    30     |   20    |
|    Proper setup for Defects4j     |    8    |    15     |    6    |
|       Research on Defects4j       |   25    |    20     |   24    |
|      Deciding on approaches       |   16    |    10     |   14    |
| Deciding on evaluation strategies |   15    |    10     |   15    |
|         Perform Analysis          |   80    |    90     |   102   |
|        Evaluate the result        |   38    |    40     |   32    |
|       Write research paper        |   20    |    15     |   20    |
|             **Total**             | **241** |  **240**  | **243** |

Note : in hrs
Exam week :20th July to 27th July
Buffer time included for exams,other project and vacation -- If buffer time used , then some tasks will run parallel
