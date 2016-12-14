# Gradle Profiler

A tool to automate the gathering of profiling and benchmarking information for Gradle builds. 

Profiling information can be captured using one of several different tools:

- Java flight recorder built into the Oracle JVM
- Using [Honest Profiler](https://github.com/RichardWarburton/honest-profiler)
- Using a [Gradle build scan](https://gradle.com)
- Using YourKit profiler
- Produce chrome trace output

## Installing

First, build and install the `gradle-profiler` app using:

    > ./gradlew installDist
 
This will install the application into `./build/install/gradle-profiler`.

## Profiling a build using JFR

To run the `gradle-profiler` app use:

    > ./build/install/gradle-profiler/bin/gradle-profiler --profile jfr --project-dir <root-dir-of-build> <task>...
    
Where `<root-dir-of-build>` is the directory containing the build to be profiled, and `<task>` is the name of the task to run,
exactly as you would use for the `gradle` command.

The profiler will run the build several times to warm up a daemon, then enable the flight recorder and run the build.
Once complete, the results will be written to a file called `profile-out/profile.jfr`.

When the profiler runs the build, it will use the tasks you specified. The profiler will use the default
Gradle version, Java installation and JVM args that have been specified for your build, if any.
This generally works the same way as if you were using the Gradle wrapper. For example, the profiler will use the values 
from your Gradle wrapper properties file, if present, to determine which Gradle version to run.

You can use the `--gradle-version` option to specify a Gradle version or installation to use to run the build, overriding the version specified in 
the Gradle wrapper properties file.

## Benchmarking a build

Run the app using:

    > ./build/install/gradle-profiler/bin/gradle-profiler --benchmark --project-dir <root-dir-of-build> <task>...

Results will be written to a file called `profile-out/benchmark.csv`.

You can use the `--gradle-version` option to specify a Gradle version or installation to use to benchmark the build. You can specify multiple versions
and each of these is used to benchmark the build, allowing you to compare the behaviour of several different Gradle versions.

## Command line options

- `--project-dir`: Directory containing the build to run (required).
- `--benchmark`: Benchmark the build. Runs the builds more times and writes the results to a CSV file.
- `--profile <profiler>`: Profile the build using the specified profiler. Can be used with or without `--benchmark`.
    - `--profile jfr`: Profile using JFR
    - `--profile hp`: Profile using [Honest Profiler](https://github.com/RichardWarburton/honest-profiler)
    - `--profile buildscan`: Profile by collecting a build scan
    - `--profile chrome-trace`: Profile by generating chrome trace output
    - `--profile yourkit`: Profile using YourKit. Uses CPU tracing and captures a snapshot into the result directory 
- `--gradle-version <version>`: specifies a Gradle version or installation to use to run the builds, overriding the default for the build. Can specify multiple versions.
- `--output-dir <dir>`: Directory to write results to.
- `--no-daemon`: Uses `gradle --no-daemon` to run the builds. The default is to use the Gradle tooling API and Gradle daemon.
- `-D<key>=<value>`: Defines a system property when running the build, overriding the default for the build.

## Applying changes between build executions

A scenario can define a Java source file that should be changes and reverted in alternating builds. You can use this to benchmark or profile an incremental build.

## Scenario file

A scenario file can be provided to define scenarios to benchmark or profile. Use the `--scenario-file` option to provide this. The scenario file is defined in [Typesafe config](https://github.com/typesafehub/config) format. Below is an example:

    # Scenarios are run in alphabetical order
    assemble {
        versions = ["3.0", "3.1"]
        tasks = ["assemble"]
    }
    clean_build {
        versions = ["/Users/me/gradle"]
        tasks = ["clean", "build"]
        gradle-args = ["--parallel"]
        system-properties {
            key = "value"
        }
        run-using = no-daemon // value can be "no-daemon" or "tooling-api"
        apply-abi-change-to = "src/main/java/MyThing.java"
    }

Values are optional and default to the values provided on the command-line or defined in the build.
