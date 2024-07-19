Trace validation agent
======================

This agent validates JFR traces for the CPU time sampler and
the normal sampler by comparing it with each other
and the results of calling `Thread.getAllStackStraces()` 
(essentially my [tiny-profiler](https://github.com/parttimenerd/tiny-profiler).

It is used to validate the upcoming CPU time sampler
(see [parttimenerd_jfr_cpu_time_sampler3](https://github.com/openjdk/jdk-sandbox/tree/parttimenerd_jfr_cpu_time_sampler3).

Idea
----

We assume that all three samplers should produce the same
result when walking the stack at exactly the same time.

The minimal sampling interval for JFR is 1ms, so the events are somewhat spaced.
But this is not a problem: 

To check if a sample `s` is valid, we get the sample of the
same thread from the other sampler that lies directly before (`a`) and the one
directly after our sample (`b`).
There should than be a significant overlap between the common prefix of `a` and `b` and 
our sample `s` (many of the top frames should be the same).
This is especially true if our other sampler has a small sampling interval.

The idea is now to compare all three samplers with each other
and report the cases where no overlap is found, which should be mostly errors.
We also report the cases where the overlap is very small compared to the total stack depth.

*Truncated traces are ignored for simplicity.*

Implementation
--------------

We implemented this as a Java agent, that runs the sampling alongside the application.

Misc Ideas
----------

We can use this to compute the effective sampling intervals for each sampler and thread,
giving the median relative difference, to hopefully show that the CPU time sampler
closer adheres to the sampling interval.

Instrument N percent of all methods with `entry` and `exit` and use this for validation.

Build
-----
Required JDK 17 or newer.

```bash
mvn package
```

Usage
-----
Run
```bash
  java -javaagent:target/trace-validation ...
```
to start the validation.
See `java -javaagent:target/trace-validation=help` for options.

Example: For running it with renaissance, use
```bash
    test -e renaissance.jar || wget https://github.com/renaissance-benchmarks/renaissance/releases/download/v0.15.0/renaissance-gpl-0.15.0.jar -O renaissance.jar
    java -javaagent:target/validation-agent.jar -jar renaissance.jar all
```

License
-------
MIT, Copyright 2024 SAP SE or an SAP affiliate company, Johannes Bechberger and factor analyzer contributors

This project is a prototype of the SapMachine team at SAP SE.