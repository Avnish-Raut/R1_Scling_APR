program-slicing/
│
├── README.md
│
├── env/
│ ├── setup.sh
│ ├── versions.txt
│ └── notes.md
│
├── docs/
│ ├── literature/
│ ├── proposal/
│ ├── meetings/
│ ├── evaluation/
│ └── paper/
│
├── defects4j/
│ ├── framework/
│ ├── projects/
│ └── bug_lists/
│
├── scripts/
│ ├── checkout_bug.sh
│ ├── compile_bug.sh
│ ├── run_failing_test.sh
│ ├── collect_bug_info.sh
│ ├── run_static_slice.sh
│ ├── run_dynamic_slice.sh
│ ├── run_hybrid_slice.sh
│ ├── evaluate_bug.sh
│ └── batch_runner.sh
│
├── configs/
│ ├── projects.yaml
│ ├── experiments.yaml
│ └── slicers.yaml
│
├── workspace/
│ ├── buggy/
│ └── fixed/
│
├── data/
│ ├── traces/
│ ├── slices/
│ ├── metrics/
│ ├── patches/
│ └── processed/
│
├── results/
│ ├── static/
│ ├── dynamic/
│ ├── hybrid/
│ ├── csv/
│ ├── plots/
│ └── reports/
│
├── logs/
│ ├── checkout/
│ ├── compile/
│ ├── tests/
│ ├── slicing/
│ └── evaluation/
│
└── tools/
├── soot/
├── slicer4j/
└── custom/
