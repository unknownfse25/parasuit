# ParaSuit

Fully Automated and Program-Adaptive Parameter Tuning for Symbolic Execution

<img src="https://github.com/unknownfse25/parasuit/assets/150991397/d405595c-1eb7-4cd9-a100-f30450bf3bb4" width=30%, height=30%/>



### Build ParaSuit
First, you have to clone our source code. 
```bash
$ git clone https://github.com/unknownfse25/parasuit.git
```

Second, build ParaSuit with Dockerfile. If you run the command below, ParaSuit and KLEE-2.1 will be built, and a benchmark (grep-3.4) will be installed.
```bash
$ cd parasuit
/parasuit $ docker build -t parasuit .
```

Third, connect to Docker using the command below. The command will take you to a directory named parasuit.
```bash
/parasuit $ docker run -it --ulimit='stack=-1:-1' parasuit
```

### Run ParaSuit
Finally, you can run ParaSuit with the following code. (e.g. grep-3.4) We provide the optimal parameter set provided by [KLEE](https://klee-se.org/docs/coreutils-experiments/) as initial parameter values.
```bash
/parasuit/benchmarks $ parasuit -t 36000 -p initial_values.json -d ParaSuit grep-3.4/obj-llvm/src/grep.bc grep-3.4/obj-gcov/src/grep
```
Format : parasuit -t <time_budget> -p <json_file> -d <output_dir> <path_to_bc_file(llvm)> <path_to_exec_file(gcov)>
+ -t : Time Budget (seconds)
+ -p : Initial Parameter Values (.json file)
+ -d : Output Directory


Then, you will see logs as follows.
```bash
[INFO] ParaSuit : Coverage will be recorded at "ParaSuit/coverage.csv" at every iteration.
[INFO] ParaSuit : Selected ParaSuit. Parameters will be tuned.
[INFO] ParaSuit : All configuration loaded. Start testing.
[INFO] ParaSuit : Iteration: 1 Iteration budget: 30 Total budget: 36000 Time Elapsed: 41 Coverage: 1346
```

When the time budget expires without error, you can see the following output.
```bash
[INFO] ParaSuit : Iteration: 264 Iteration budget: 120 Total budget: 36000 Time Elapsed: 35344 Coverage: 3389 
[INFO] ParaSuit : Iteration: 265 Iteration budget: 120 Total budget: 36000 Time Elapsed: 35510 Coverage: 3389 
[INFO] ParaSuit : Iteration: 266 Iteration budget: 120 Total budget: 36000 Time Elapsed: 35717 Coverage: 3389 
[INFO] ParaSuit : Iteration: 267 Iteration budget: 120 Total budget: 36000 Time Elapsed: 35886 Coverage: 3389 
[INFO] ParaSuit : Iteration: 268 Iteration budget: 114 Total budget: 36000 Time Elapsed: 36060 Coverage: 3389 
[INFO] ParaSuit : ParaSuit done. Achieve 3389 coverage.
```

### Run KLEE Default
You can also run KLEE without any tuning by following the command below.
```bash
/parasuit/benchmarks $ parasuit -t 36000 -d KLEEdefault --tool klee grep-3.4/obj-llvm/src/grep.bc grep-3.4/obj-gcov/src/grep
```


## Reporting Results
### Branch Coverage
If you want to get results about how many branches ParaSuit has covered, run the following command.
```bash
/parasuit/benchmarks $ python3 report_coverage.py --benchmark grep-3.4 ParaSuit KLEEdefault 
```

If the command is executed successfully, you will get a graph like the following in a file named "coverage_result.png".
<img src="https://github.com/unknownfse25/parasuit/assets/150991397/525cf702-e7e5-4152-84fb-6c19661db64b" width=30%, height=30%/>

### Bug Finding
If you want to check information about what bugs ParaSuit has found, run the following command.
```bash
/parasuit/benchmarks $ python3 report_bugs.py --benchmark grep-3.4 ParaSuit
```

If the command is executed successfully, you will get a bug report in a file named "bug_result.txt".
```bash
/parasuit/benchmarks $ cat bug_result.txt
# Example from gcal-4.1
TestCase : /ParaSuit/iteration-11/test000007.ktest
Arguments : "./gcal" "   " "   " "@/" "- " "@-" 
CRASHED signal 11
File: ../../src/file-io.c
Line: 740
```


## Usage
```
$ parasuit --help
usage: parasuit [-h] [--klee KLEE] [--klee-replay KLEE_REPLAY] [--gcov GCOV] [-p JSON] [--iteration-time-budget INT] [--threshold FLOAT] [--n-tune INT] [--clust-range INT] [-d OUTPUT_DIR] [--gcov-depth GCOV_DEPTH] [--tool TOOL] [-t INT] [llvm_bc] [gcov_obj]
```


### Optional Arguments
| Option | Description |
|:------:|:------------|
| `-h, --help` | show help message and exit |
| `-d, --output-dir` | Directory where experiment results are saved |
| `--gcov-depth` | Depth from the obj-gcov directory to the directory where the gcov file was created |
| `--tool TOOL` | Select a tool to run symbolic execution (e.g. parasuit, klee)|


### Executable Settings
| Option | Description |
|:------:|:------------|
| `--klee` | Path to "klee" executable |
| `--klee-replay` | Path to "klee-replay" executable |
| `--gcov GCOV` | Path to "gcov" executable |


### Hyperparameters
| Option | Description |
|:------:|:------------|
| `-p, --parameter-values` | Path to JSON file that defines parameter spaces |
| `--iteration-time-budget` | Time budget for each iteration |
| `--threshold` | Threshold to group parameters into a group in the parameter selection step |
| `--n-tune` | Number of times each parameter is compared in the parameter selection step |
| `--clust-range` | Clustering execution cycle at the parameter value sampling stage |

In the JSON file used as the value of --parameter-values, there are two entries: `space` and `defaults`. For `space`, it includes parameters that should be tuned. For `defaults`, it contains parameters whose value must be fixed to a specific value. If you want to use new parameter values in the experiment, edit the defaults section of the JSON file.


### Required Arguments
| Option | Description |
|:------:|:------------|
| `-t, --budget` | Total time budget of ParaSuit |
| `llvm_bc` | LLVM bitecode file for klee |
| `gcov_obj` | Executable with gcov support |

## Usage of Other Programs
### /benchmarks/report_bugs.py
```
/parasuit/benchmarks$ python3 report_bugs.py --help
usage: report_bugs.py [-h] [--benchmark STR] [--table PATH] [DIRS ...]
```
| Option | Description |
|:------:|:------------|
| `-h, --help`  | Show this help message and exit |
| `DIRS`        | Name of directory to detect bugs |
| `--benchmark` | Name of benchmark & verison |
| `--table`     | Path to save bug table graph |


### /benchmarks/report_coverage.py
```
/parasuit/benchmarks$ python3 report_coverage.py --help
usage: report_coverage.py [-h] [--benchmark STR] [--graph PATH] [--budget TIME] [DIRS ...]
```
| Option | Description |
|:------:|:------------|
| `-h, --help`  | Show help message and exit |
| `DIRS`        | Names of directories to draw figure |
| `--benchmark` | Name of benchmark & verison |
| `--graph`     | Path to save coverage graph |
| `--budget`    | Time budget of the coverage graph |


## Source Code Structure
Here are brief descriptions of the files. Some less-important files may be omitted.
```
.
├── benchmarks                  Testing & reporting directory
    ├── report_bugs.py          Reporting bug-finding results
    └── report_coverage.py      Reporting branch coverage results
└── parasuit                    Main source code directory
    ├── bin.py                  Entry point of ParaSuit
    ├── boolean.py              Tuning boolean parameters
    ├── collect.py              Value sampling method when collecting data
    ├── construct_sample.py     Value sampling using clustering
    ├── errcheck.py             Handling parameter values that cause errors
    ├── exechandler.py          Converting parameter values to parameter command format of symbolic executor
    ├── grouping.py             Group parameters and extract representative parameters from each group
    ├── keyword_filter.py       Removing default or useless parameters from the parameter set
    ├── klee.py                 Running KLEE 
    ├── parameters.py           Including parameters and description of symbolic executor
    ├── paramselect.py          Algorithm of Parameter Selection
    ├── parasuit.py             Core algorithm of ParaSuit
    ├── setminimum.py           Extract the minimum default parameters from the parameter value given as input
    ├── string.py               Tuning string parameters
    ├── symbolic_executor.py    Interface for all symbolic executors (e.g., KLEE)
    └── symparams.py            Tuning symbolic parameters (--sym-arg, --sym-files)
```


