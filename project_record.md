# Program Slicing on Defects4J — Project Record

Chronological record of decisions, failures, fixes and findings, from the
original proposal through to the current state.

This covers till the last meeting on 5th October

---

## Phase 0 — The original plan

**Goal.** Evaluate program slicing on Defects4J: how much code reduction is
achievable while preserving fault-relevant information.

**Slicers surveyed.**

| Tool                      | Type    | Assessment at the time                                                    |
| ------------------------- | ------- | ------------------------------------------------------------------------- |
| Soot                      | Static  | Mature framework, no built-in slicer; Java ≤7 documented                  |
| WALA                      | Static  | Has a slicer; "targets Java 17+" (this judgement was wrong — see Phase 2) |
| Java SDG Slicer (mistupv) | Static  | "Best ready-to-use static slicer", JDK 11+                                |
| JavaSlicer (backes)       | Dynamic | JDK 1.6/1.7 only, obsolete                                                |
| Slicer4J                  | Dynamic | "Very high" research value, JRE 8                                         |

**Planned Projects** Lang, Chart, Time, Math (~200 bugs).
**Current Plan** Cli,Compress,Math -- Fully , and around 20 random bugs from every project if possible

**Planned split.** One person on Slicer4J, one on Java SDG Slicer.
**Current** Only Slicer4J

**Original RQ sets.** Set A compared static vs dynamic; Set B covered
criterion sensitivity, fault coverage, call-graph precision and reduction.

---

## Phase 1 — Java SDG Slicer: four failure modes

### 1.1 Setup

`prepare_project.sh` (checkout, compile, test, export metadata) worked
without trouble. The slicer itself ran correctly on the README's toy example
in under a second.

### 1.2 First failure: OOM on Math 1

Running with `-i src/main/java` (whole project) exhausted a 24 GB heap after
17 minutes.

Thread dumps and GC logs eventually identified **four distinct failure
modes**, each with a different cause:

| #   | Failure                                    | Mechanism                                                                                                                                            | Scope sensitivity           |
| --- | ------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------- |
| 1   | `UnsupportedOperationException ... from T` | `VariableVisitor` cannot resolve a type variable in a local declaration; thrown during `buildCFGs`, which builds a CFG for **every method in scope** | Worse with **wide** scope   |
| 2   | `ObjectTree.clone` OOM                     | Field-sensitive object model expands recursive field types without bound                                                                             | Worse with **wide** scope   |
| 3   | Classloader resolution spin                | `ClassLoaderTypeSolver` retries failed type lookups with no negative cache; 100% CPU, heap saturated by transient garbage                            | Worse with **narrow** scope |
| 4   | `UnsolvedSymbolException`                  | A type is missing from `-i`                                                                                                                          | Worse with **narrow** scope |

### 1.3 The structural problem

Failures 1 and 2 argue for a narrow scope; 3 and 4 for a wide one. Because
the tool builds CFGs for all classes in scope rather than only those
reachable from the criterion, **no scope satisfied all four** on either Lang
or Math.

Two further constraints were established:

- `-i` is **recursive** which passing a package directory includes all
  subpackages. Expressing "this package but not its children" requires a
  staging tree of flat copies.
- There is **no classpath flag**. `-i` takes source directories only; JDK
  types are handled internally, third-party library types cannot be supplied
  at all. Adding `target/classes` to the JVM classpath has no effect.

### 1.4 Screening result

A feasibility screen over Lang (5 bugs, whole-project scope):

```
3 GENERICS  (failed in around 5sec)
2 TIMEOUT   (300 sec, 2.8 GB RSS — resolution spin)
0 OK
```

**Zero of five feasible.** Generic local variable declarations are ordinary
modern Java and pervasive in commons-lang.

### 1.5 Conclusion

Java SDG Slicer cannot analyse Defects4J subjects. This is a reportable
negative result: the tool's published evaluation covers small programs, and
none of these four failure modes appear there.

---

## Phase 2 — Alternatives

### 2.1 The bytecode insight

All four failures come from parsing Java **source**. Bytecode-level tools
avoid three of them structurally — generics are erased by javac, there is no
symbol solver, and types are fully resolved before analysis.

The existing `prepare_project.sh` already produces compiled classes and
exports `cp.compile` and `dir.bin.classes`, so a pivot was cheap.

### 2.2 JOANA — abandoned

WALA-based, bytecode input, has SDG slicing. Build failed:

- `contrib/parameter-setter` submodule hosted at `git.scc.kit.edu`, which
  does not resolve
- Referenced by `api/joana.api/build.xml`, so not trivially optional

A dead upstream host on a low-activity project was judged too risky against
the deadline.

### 2.3 Soot — built and working

Soot is a framework, not a slicer: it provides Jimple IR, call graphs,
def/use chains. Slicing has to be implemented.

Four programs were written and run successfully on Lang 1:

| Step                              | Result                                                                                                                      |
| --------------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| 1. Load bytecode, print Jimple    | Works. **76.1%** of Jimple statements carry source line numbers (the rest are compiler temporaries and identity statements) |
| 2. Call graph from trigger test   | Works                                                                                                                       |
| 3. Intraprocedural backward slice | Works after a bug fix (below)                                                                                               |
| 4. Interprocedural slice          | Runs; over-approximates                                                                                                     |

**CHA vs SPARK on Lang 1** — SPARK was both faster _and_ more precise, which
inverts the usual expectation:

|                       | CHA       | SPARK     |
| --------------------- | --------- | --------- |
| Call graph time       | 681 s     | **185 s** |
| Edges                 | 2,360,044 | 836,129   |
| Reachable app methods | 1,317     | **690**   |

CHA generates so many spurious edges that traversing them costs more than
computing points-to information.

**Bug found in Step 3: control dependence.** The implementation had the
post-dominance arguments reversed and omitted one of the two conditions.
Correct form (Ferrante, Ottenstein & Warren): X is control dependent on
branch B iff some successor of B is post-dominated by X **and** X does not
post-dominate B. The buggy version pulled in everything _after_ the
criterion. Fixing it took the slice for `NumberUtils:458` from 5 lines to the
correct 3 (lines 451, 454, 458 — the null check, the blank check, and the
criterion).

**Known limitation of Step 4.** Context-insensitive: entering a method from
one call site and exiting through another. Slices are conservative. A
descent-flag guard was designed but not applied before the pivot.

**Cost profile.** Call graph 185 s; slicing 0.03 sec. A factor of around 6000 — so
additional criteria per bug are effectively free once the graph exists.

### 2.4 Decision

The static side was dropped; the project continued with Slicer4J alone.

Consequence: **Set A (static vs dynamic) became unanswerable.** Set B plus
gold-standard comparison was adopted instead.

---

## Phase 3 — Slicer4J pipeline

### 3.1 Build chain

Three levels, each installing into the next:

1. FlowDroid fork → `soot-infoflow:2.10.0-mandoline`
2. DynamicSlicingCore `core/` → `slicer.dynamic:1.0.0`
3. Slicer4J → the fat jar

Problems encountered: root-owned `target/` directories from an earlier
`sudo` run blocking `mvn clean`; StubDroid failing because
`-Dmaven.test.skip=true` prevents the `tests` classifier jar being built
(harmless — `soot-infoflow` is the only artefact needed downstream).

Java was switched to 8 by default, matching Slicer4J's requirement.
Defects4J **2.0.0** is used (the `-latest` install requires Java 11).

### 3.2 Pipeline design

The key structural decision: **instrumentation and tracing are identical for
every criterion**, so they run once per bug and criteria are swept cheaply.

```
checkout → compile → test → jar → instrument → trace     (once per bug)
                                                  |
                    +-----------------+-----------+
                 slice C1          slice C2    slice C3   (cheap)
```

**Three criterion variants:**

| Variant       | Definition                                   | Rationale                                                                  |
| ------------- | -------------------------------------------- | -------------------------------------------------------------------------- |
| `test_crash`  | Line in the test where the failure surfaced  | Mirrors real debugging: symptom known, cause unknown                       |
| `app_deepest` | Deepest application frame in the stack trace | Closer to the fault; assumes knowledge a developer lacks at the start      |
| `patched`     | A line the developer changed                 | **Circular for coverage by construction** — included to quantify that bias |

**Gold standard:** buggy and fixed checkouts are diffed; buggy-side line
numbers of removed/changed lines are the fault locations.

### 3.3 Bugs found and fixed, in order

Every one of these silently produced wrong results before it was found.

| #   | Symptom                                                    | Cause                                                                                                                                                                                                                     | Effect if unfixed                                                                                        |
| --- | ---------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------- |
| 1   | `Trace size: 0`, empty slice                               | `-sl` resolved relative to `-o`; per-variant output dirs meant `static-log.log` was not found                                                                                                                             | All slices empty                                                                                         |
| 2   | Second and third variants always empty                     | `rm -rf "$SLICE_OUT"` deleted the instrumentation output after the first variant                                                                                                                                          | Only one variant ever worked                                                                             |
| 3   | `[: 0\n0: integer expression expected`, malformed CSV rows | `grep -c ... \|\| echo 0` prints `0` **and** exits 1, so both fire; embedded newline split CSV rows                                                                                                                       | Corrupted CSV; garbage statuses                                                                          |
| 4   | `comm: file is not in sorted order`                        | `comm` compares lexically; inputs were numerically sorted                                                                                                                                                                 | Undercounted `patch_in_slice`; inflated `NONE`                                                           |
| 5   | All bugs `UNDEFINED_ADDITIVE`                              | `defects4j checkout` will not create the parent directory; every fixed checkout failed                                                                                                                                    | **All fault coverage undefined**                                                                         |
| 6   | `URI is not hierarchical`                                  | Test resources (`.zip`, `.tar`) packed into `app.jar`; `getResource()` returned a `jar:` URI and `new File(uri)` threw                                                                                                    | 19 of 47 Compress bugs never reproduced; 13 previously recorded as `OK` were slicing the wrong execution |
| 7   | Valid bugs flagged `NOT_REPRODUCED`                        | Reproduction check matched on exception class, then on message text — both differ between Ant's JUnit 3 adapter and JUnitCore (`AssertionFailedError` vs `ComparisonFailure`; `"expected:<[]bar>"` vs `"expected:<...>"`) | Valid subjects discarded                                                                                 |
| 8   | `patch_total` inflated; criteria on unsliceable lines      | Diff extraction counted imports, comments, blanks. A fix swapping `Map`→`Set` changes import lines, which never execute                                                                                                   | `FULL` coverage unreachable; `patched` criterion on line 19 (an import)                                  |
| 9   | Negative `red_vs_modclass` (−9%, −27%)                     | Multi-class slice divided by single-class denominator                                                                                                                                                                     | Nonsensical reduction figures                                                                            |
| 10  | Rows for bugs never run; `CHECKOUT_FAILED` cascade         | Two concurrent runs sharing the workspace                                                                                                                                                                                 | Entire run corrupted                                                                                     |
| 11  | `trace_size` identical across unrelated bugs               | Variable not reset per bug, and extracted _after_ the failure checks                                                                                                                                                      | Scalability data unusable                                                                                |

**Resolution of #7:** matching on the **test method name**, which is stable
across runners. Three check designs were tried; two were silently wrong in
opposite directions.

**Effect of #8 on results** (Cli):

|                   | Before filtering | After |
| ----------------- | ---------------- | ----- |
| `patched` OK      | 26               | 30    |
| `patched` FULL    | 14               | 18    |
| `test_crash` FULL | 6                | 9     |

Patch-line filtering materially changes fault coverage. Most work using
Defects4J patches as ground truth does not state whether it filters.

### 3.4 Discoveries about the tool

**`trace.log` is not the trace.** It holds JUnit console output plus
base64/ZLIB logger records (~1 MB). The expanded trace is
`trace.log_icdg.log` — **7.2 GB** for Compress 23.

**Trace size varies by seven orders of magnitude**, from 21 entries (Cli 5)
to 105,181,891 (Compress 36). Slicer4J builds roughly one graph node per
trace entry; Compress 36 built ~49M nodes before `OutOfMemoryError: GC
overhead limit exceeded` at 24 GB.

**Degenerate slices.** When the criterion is a bare `fail("...")` call, it
has no incoming data dependences and the backward slice contains only
itself. Three Cli bugs produced 1–2 line slices for this reason. The bug in
those cases is that an exception was _not_ thrown — an omission in the
execution, mirroring omission faults in patches.

**Additive fixes are common.** Roughly a quarter of bugs are fixed purely by
adding lines, leaving no buggy-side fault location. Coverage is _undefined_
for these, not zero — scoring them as misses would measure the developer's
fixing style rather than the slicer. Example: Compress 1's fix adds a single
`this.finish();` call.

---

## Phase 4 — Current results

From the most recent clean runs (all fixes applied except #11).

### Feasibility (`test_crash`)

| Project  | Bugs | OK  | Rate |
| -------- | ---- | --- | ---- |
| Cli      | 39   | 36  | 92%  |
| Compress | 47   | 42  | 89%  |
| Math     | 106  | 58  | 54%  |

Failures: 3 Cli `NO_CRITERION` (trigger test defined in a parent class, so
the stack shows the parent name); 2 Compress `SLICE_EMPTY`, 1
`SLICE_TIMEOUT`. 13 Math `SLICE_EMPTY` ,12 `SLICE_TIMEOUT`, 12 `NO_CRITERION`, 8 `TRACE_TIMEOUT`

### Fault coverage (additive bugs excluded from the denominator)

| Variant       | Cli n | Cli hit   | Cli FULL | Compress n | Compress hit | Compress FULL | Math n | Math hit  | Math FULL |
| ------------- | ----- | --------- | -------- | ---------- | ------------ | ------------- | ------ | --------- | --------- |
| `test_crash`  | 30    | 13 (43%)  | 9 (30%)  | 32         | 11 (34%)     | 6 (19%)       | 44     | 21 (48%)  | 14 (32%)  |
| `app_deepest` | 7     | 5 (71%)   | 1 (14%)  | 13         | 10 (77%)     | 3 (23%)       | 18     | 12 (67%)  | 7 (39%)   |
| `patched`     | 30    | 30 (100%) | 18 (60%) | 26         | 26 (100%)    | 11 (42%)      | 46     | 46 (100%) | 24 (52%)  |

**Coverage rises monotonically as the criterion approaches the fault, in both
projects.** `patched` at 100% confirms the comparison pipeline is sound (a
slice always contains its own criterion) and quantifies the circularity of
patch-anchored evaluation.

### Criterion availability

`app_deepest` applies to only 11/39 Cli , 15/47 Compress and 25/106 Math bugs, because most
failures are assertions with no application frame in the stack. When it does
apply, coverage is roughly double that of `test_crash`. A better criterion
that is available less often.

### Cost

Setup and instrumentation dominate; slicing itself is usually seconds.
Slicing time and feasibility both degrade sharply with trace size.

---

## Appendix — Configuration of record

|               |                                               |
| ------------- | --------------------------------------------- |
| Defects4J     | 2.0.0                                         |
| Java          | 8 (Slicer4J requirement)                      |
| Slicer4J heap | 24 GB                                         |
| Timeouts      | instrument 1200 s, trace 1800 s, slice 1200 s |
| Machine       | 31 GB RAM, 16 cores                           |
| Subjects      | Cli (39 bugs), Compress (47 bugs)             |

Results, logs and the script version are snapshotted under
`/home/research_slicing/slicer4j/results`
`/home/research_slicing/slicer4j/logs/`

Script location `/home/research_slicing/tools/run_slicer.sh`
