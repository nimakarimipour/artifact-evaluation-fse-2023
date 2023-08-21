# Artifact for "Practical Inference of Nullability Types" (FSE 2023)

This README.md file provides information about the artifact for "Practical Inference of Nullability Types." The artifact includes the implementation of our tool in Java, used benchmarks in the paper and the scripts to collect data used in tables.

To access the artifact, you can find it on Zenodo. To get started, please follow the instructions below: 

# Container Structure

This docker image contains:

* source code (NullAwayAnnotator) of our tool (will be found in /var/NullAwayAnnotator)
* all benchmarks (will be cloned in /var/benchmarks)
* scripts to reproduce our experiments (will be found in /var/AE)

## Setup

  1. Install Docker based on your system configuration: [Get Docker](https://docs.docker.com/get-docker/).
  2. Import the artifact into Docker: `docker load annotator-ae-fse-2023`
  3. Run the Docker image (give container at least 16gigs of ram): `docker run --name annotator-ae annotator-ae-fse-2023 &`
  4. Access docker container shell: `docker exec -it annotator-ae bash`

All required packages have been already installed in the docker image, the docker can be safely executed with no internet connection.

Our paper's goal is to infer nullable annotations for projects to adapt NullAway for JAva code. Our tool is available at [NullAwayAnnotator](https://github.com/ucr-riple/NullAwayAnnotator).

This Docker container offers a means to reproduce the data reported in the paper. Additionally, it includes guidelines (last section) on how to execute our tool on benchmarks that are not part of our evaluation. This enables users to extend the analysis and apply the techniques to other projects.

## Expected time of execution

All experiements are executed on an ubuntu system with 64gigs of ram and 13th Generation Intel Corei7 processor with 16 cores.
The expected time to generate the full table is as below:
1. Table 1: approximately 5.5 hours
2. Tabl2 2: approximately 28 hours
3. Table 3: approximately 4 hours

To generate a row of a table, please pass the benchmark name to the script as written.


## Cached results
Every script found in the `AE` directory gathers the necessary data by processing the tool's final output for each benchmark. To circumvent the time-intensive nature of tool execution, results from each benchmark's tool output are cached. Nevertheless, if a complete experiment is preferred, append `--fresh` to each script. This will prompt the scripts to execute the tool on every benchmark and subsequently compile the data from the execution output.


## Reproducing Table 1

Table 1 in our paper presents benchmarks specifications and error reduction from our tool on depths zero, one and five.
You can reproduce Table 1 by executing the `table1.py` script. This script runs the tool on benchmarks and runs NullAway on the output and collects the number of remaining reported errors.
The output is shown on stdout and is also saved in the file `~/var/results/table1.txt`.

To execute the script using cached results, run:
```shell
cd /var/AE && python3 table1.py [benchmark-name]
```
To execute the script and rerunning the full experiment, run:
```shell
cd /var/AE && python3 table1.py [benchmark-name] --fresh
```
To generate the full table, please run (should take approximately 5.5 hours):
```shell
cd /var/AE && python3 table1.py all [optinal --fresh]
```
## Reproducing Table 2

Table 2 in our paper presents running time and number of builds for unoptimized and optimized configurations.
You can reproduce Table 2 by executing the `table2.py` script. This script runs the tool on benchmarks and measures the spent time to execute the tool and reads the generated log to retrieve the number of builds.
The output is shown on stdout and is also saved in the file `~/var/results/table2.txt`. Please note this table contains performance numbers that vary dependent on the hardware configurations, however, the ratios in table shown as (x X) should be close to the generated numbers by the script.

To execute the script and rerunning the full experiment, run:
```shell
cd /var/AE && python3 table2.py [benchmark-name]
```
To generate the full table, run (should take approximately 28 hours):
```shell
cd /var/AE && python3 table2.py all
```

## Reproducing Table 3 

Table 3 in our paper presents the Number of annotations injected by our tool.
You can reproduce Table 3 by executing the `table3.py` script. This script runs the tool on benchmarks and collects the number of annotations added by the tool. It also measures the number of lines that are enclosed by a suppression annotation and computes the percentage of unchecked
The output is shown on stdout and is also saved in the file `~/var/results/table3.txt`.

To execute the script using cached results, run:
```shell
cd /var/AE && python3 table3.py [benchmark-name]
```
To execute the script and rerunning the full experiment, run:
```shell
cd /var/AE && python3 table3.py [benchmark-name] --fresh
```
To generate the full table, run (should take approximately 4 hours):
```shell
cd /var/AE && python3 table3.py all [optinal --fresh]
```

## Guidelines on running the tool on a new benchmark

These instructions are tailored specifically for Gradle projects and may not be suitable for projects of a more complex structure.
### Installation

1. The projects need to integrate `nullaway` to its build systems. The instruction can be found [here](https://github.com/uber/NullAway)
2. Add `AnnotatorScanner` (one of our tool artifacts) to the annotation processor path:
    ```groovy
    annotationProcessor "edu.ucr.cs.riple.annotator:annotator-scanner:1.3.8"
    ```
3. Add dependencies to use annotations added by Annotator
    ```groovy
    compileOnly "com.uber.nullaway:nullaway-annotations:0.10.10"
    compileOnly "org.jspecify:jspecify:0.3.0"
    ```
4. Add the following flags to errorprone configuration added in step 1 of this section:
    ```groovy
    def scanner_path =  "${rootDir.absolutePath}/out/scanner.xml"
    def nullaway_path =  "${rootDir.absolutePath}/out/checker.xml"

    options.errorprone {
            check("AnnotatorScanner", CheckSeverity.ERROR)
            option("NullAway:SerializeFixMetadata", "true")
            option("NullAway:FixSerializationConfigPath", nullaway_path)
            option("AnnotatorScanner:ConfigPath", scanner_path)
    }
    ```

### Running the tool:

To run the tool, use the command below:
```shell
java -jar ./path/to/annotator-core.jar -d "/path/to/project/out/" -i com.uber.nullaway.annotations.Initializer -bc "cd /path/to/targetProject && ./gradlew build -x test" -fr "org.jspecify.annotations.NullUnmarked"
```