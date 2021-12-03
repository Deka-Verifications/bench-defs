# SV-COMP Reproducibility
This repository describes the configuration of the competition machines (below)
and the benchmark definition for each verifier (folder [benchmark-defs/](benchmark-defs/)),
in order to make results of the competition reproducible.


## Components for Reproducing Competition Results

The competition uses several components to execute the benchmarks.
The components are described in the following table.

| Component              | Repository                                                      | Participants             |
| ---                    | ---                                                             | ---                      |
| Verification Tasks     | https://github.com/sosy-lab/sv-benchmarks                       | add, fix, review tasks   |
| Benchmark Definitions  | https://gitlab.com/sosy-lab/sv-comp/bench-defs                  | define their parameters  |
| Tool-Info Modules      | https://github.com/sosy-lab/benchexec/tree/main/benchexec/tools | define inferface         |
| Verifier Archives      | https://gitlab.com/sosy-lab/sv-comp/archives-2022               | submit to participate    |
| Benchmarking Framework | https://github.com/sosy-lab/benchexec                           | (use to test their tool) |
| Competition Scripts    | https://gitlab.com/sosy-lab/benchmarking/competition-scripts    | (use to reproduce)       |
| Witness Format         | https://github.com/sosy-lab/sv-witnesses                        | (know)                   |
| Task-Definition Format | https://gitlab.com/sosy-lab/benchmarking/task-definition-format | (know)                   |
| Remote Execution       | https://gitlab.com/sosy-lab/software/coveriteam                 | (use to test their tool) |


## Setup
The following steps set up the benchmarking environment:
- `git clone https://gitlab.com/sosy-lab/sv-comp/bench-defs.git ./`
- `make init` (takes a while: downloads several GB of data from repositories)
- `make update`

For reproducing results of a specific edition of the competition, please checkout the tag for that edition.

The following sections assume that the working directory is the same as used in the above commands.

## Executing a Benchmark for a Particular Tool

Assume that we would like to reproduce results for the tool `CPAchecker`,
including results validation.
This can be achieved using the following command:

`scripts/execute-runs/mkRunVerify.sh cpachecker`

The above command executes the verification runs with tool `CPAchecker`, and
afterwards all result validators that are declared in `benchmark-defs/category-structure.yml`.

If we would like to execute only verification runs, then we can use the following command:

`scripts/execute-runs/execute-runcollection.sh "../../benchexec/bin/benchexec" cpachecker cpachecker.xml witness.graphml .graphml ../../results-verified/`

The parameters specify the:
- benchmarking utility (BenchExec) to be used to run the benchmark,
- tool archive, i.e., we use the archive file `archives/*/cpachecker.zip`,
- benchmark definition, i.e., we use `benchmark-defs/cpachecker.xml`,
- name of the witness files, to which the unification script links the witness produced by the tool,
- pattern using which the unification script searches for produced witnesses, and
- the directory in which the results shall be stored.

For quick tests and sanity checks, BenchExec can be told to restrict the execution to a certain test-set.
For example, to restrict the execution to the sub-category `ReachSafety-ControlFlow`,
you can replace `"../../benchexec/bin/benchexec"` by `"../../benchexec/bin/benchexec -t ReachSafety-ControlFlow"`.

Furthermore, BenchExec can be told to overwrite limit from the benchmark definitions (which should be used only for test executions).
To see if a tool generally works and produces outputs, you could use (assuming we use a machine with 8 cores and 32 GB of RAM)
the additional parameters `--timelimit 60 --memorylimit 3GB --limitCores 1 --numOfThreads 8` to
- limit the CPU time to `60 s`,
- limit the memory to `3GB`,
- limit the number of cores to `1`, and
- set the number of runs executed in parallel to `8`.

It is important to execute the tools (when running experiments) in a container.
Since we use BenchExec, this is done automatically.
In order to protect our file system and to give proper write access to the tool inside the container,
we add the setup of the overlay filesystem using the parameters
- `--read-only-dir /` to make sure the tool we execute does not write at unexpected places,
- `--overlay-dir /home` to let BenchExec setup a directory to the tool inside the container, and
- `--overlay-dir .` to give permission to write to the working directory.

A complete command line would look as follows:

`scripts/execute-runs/execute-runcollection.sh "../../benchexec/bin/benchexec --timelimit 60 --memorylimit 3GB --numOfThreads 8 --limitCores 1 -t ReachSafety-ControlFlow --read-only-dir / --overlay-dir /home --overlay-dir ." cpachecker cpachecker.xml witness.graphml .graphml ../../results-verified/`


In the following we explain some of the steps that the above script does for us.

### Unpack a Tool

The following command unpacks the tool `CPAchecker`:
- `mkdir bin/cpachecker-32KkXQ0CzM`
- `scripts/execute-runs/mkInstall.sh cpachecker bin/cpachecker-32KkXQ0CzM`

### Assemble Provenance Information for a Tool

The following command prints information about the repositories and their versions:
- `scripts/execute-runs/mkProvenanceInfo.sh cpachecker`

### Execute a Benchmark for a Tool

- `cd bin/cpachecker-32KkXQ0CzM`
- `../../benchexec/bin/benchexec -t ReachSafety-ControlFlow ../../benchmark-defs/cpachecker.xml -o ../../results-verified/`


## Computing Environment on Competition Machines

### Docker Image

SV-COMP provides a Docker image that tries to provide an environment
that has mostly the same packages installed as the competition machines.
The Docker image is described here:
https://gitlab.com/sosy-lab/benchmarking/competition-scripts/#docker-image

### Parameters of RunExec

The parameters that are passed to the [BenchExec](https://github.com/sosy-lab/benchexec) [1]
executor [runexec](https://github.com/sosy-lab/benchexec/blob/main/doc/runexec.md) on the competition machines
are described here:
https://gitlab.com/sosy-lab/benchmarking/competition-scripts/#parameters-of-runexec


## References

[1]: Dirk Beyer, Stefan Löwe, and Philipp Wendler.
     Reliable Benchmarking: Requirements and Solutions.
     International Journal on Software Tools for Technology Transfer (STTT), 21(1):1-29, 2019.
     https://doi.org/10.1007/s10009-017-0469-y


