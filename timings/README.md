# Timing Records

This directory records local timing observations for Weave performance work. These files are for development evidence, not portable product output.

Use `weave-performance.csv` for comparable command runs. The first pass can be filled manually from terminal output. If timing instrumentation grows a CSV sink later, aim it here with something like `WEAVE_TIMING_CSV=/home/djradon/hub/semantic-flow/weave/dependencies/github.com/semantic-flow/weave-dev-archive/timings/weave-performance.csv`.

Suggested practice:

- record the Weave commit, workload, command, mesh root shape, and candidate/path counts
- keep timings phase-oriented rather than only total wall time
- keep SFLO all-terms runs local-only unless we separately decide to add a checked-in generated fixture
- do not record host-local secrets or machine-specific paths beyond broad workload labels
