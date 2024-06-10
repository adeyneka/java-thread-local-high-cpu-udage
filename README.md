# java-thread-local-high-cpu-usage

## Our System/Runtime Configuration
- two 32 cores, 64GB server Ubuntu 22.04.2 LTS
- OpenJDK 64-Bit Server VM (20+36) for linux-amd64 JRE (20+36), built on 2023-03-21T00:00:00Z by "temurin" with gcc 11.2.0
- -XX:+UnlockExperimentalVMOptions -XX:+UnlockDiagnosticVMOptions -XX:+UseShenandoahGC -XX:ShenandoahGCHeuristics=static -XX:ShenandoahMinFreeThreshold=40 -XX:ShenandoahPacingMaxDelay=20 -XX:ConcGCThreads=12 -XX:CICompilerCount=4 -XX:+UseLargePages -XX:+UseTransparentHugePages -XX:+AlwaysPreTouch
- The load between 2 instances is distributed by a load balancer

## Problem
Recently, we observed that the CPU usage on `instance_1` started to gradually increase, diverging from the expected balanced load.
instance_1             |  instance_2
:-------------------------:|:-------------------------:
![image](https://github.com/adeyneka/java-thread-local-high-cpu-udage/assets/1788378/5767d1b8-b937-4b8e-ae97-dc94d9f8e634) | ![image](https://github.com/adeyneka/java-thread-local-high-cpu-udage/assets/1788378/e0b5e6fd-c532-4078-82ea-b21dc3a68880)

## Diagnostic
We checked business and hardware metrics and they were the same on two instances(except CPU).

We captured jfr and cpu/memory flamegraph and it showed CPU was consumed by access to threadlocal.
We put in MDC ~3-5 parameters for every request. 
This configuration has been running smoothly for over a year, and this is the first occurrence of the issue.

Also we observed load was dropped when we tried to profile application(see stages on the graph)
```
java.lang.ThreadLocal$ThreadLocalMap.expungeStaleEntry(int)
java.lang.ThreadLocal$ThreadLocalMap.remove(ThreadLocal)
java.lang.ThreadLocal.remove(Thread)
java.lang.ThreadLocal.remove()
ch.qos.logback.classic.util.LogbackMDCAdapter.clear()
org.slf4j.MDC.clear()
```

![image](https://github.com/adeyneka/java-thread-local-high-cpu-udage/assets/1788378/743d8e13-26d6-471e-bfc5-96c16c387c4e)
