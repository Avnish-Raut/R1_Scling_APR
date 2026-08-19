# 🚀 How to Run the Slicer4J Evaluation Pipeline (From 0)

This guide walks you through the exact commands needed to evaluate an entire Defects4J repository, package the resulting logs, and download them to your local machine.

## Step 1: Find the Number of Bugs
Before running a batch, you need to know exactly how many bugs exist in the target project. Defects4J provides an `info` command that lists project details.

# Replace 'Math' with your target project (e.g., Cli, Closure, Time)
defects4j info -p Math

# Syntax: nohup ./run_eval_batch.sh <Project> <MaxBugs> > <OutputLog.log> 2>&1 &
nohup ./run_eval_batch.sh Math 106 > math_batch_output.log 2>&1 &

tail -f math_batch_output.log

# This searches for all slice passes, timing logs, and validation reports, packaging them instantly.
find . -type f \( -name "slice_*.log" -o -name "static-log.log" -o -name "timing_*.log" -o -name "validation_report.txt" -o -name "*_error.log" \) | tar -czvf target_repo_results.tar.gz -T -

# Replace <YOUR_VM_IP> with the actual IP address of your server
scp rodan@<YOUR_VM_IP>:/home/rodan/r1_project/parallel_execution/target_repo_results.tar.gz ~/Downloads/