# MOD004 - P95 Latency Experiments

## Overview

As previously mentioned, the Pet Clinic startup time experiment used in module 3 was chosen to fit the timeframe of this single day workshop. Now that that experiment has been successfully created and running, let’s shift gears a bit and examine the results of a couple of previously run, performance-based experiments for Pet Clinic.

These two experiments were driven by a *StormForge Performance Testing* test case trial job that ran for 1 minute and 30 seconds each invocation and used an imported HAR file (generated using the instructions from module 2). The total experiment run time for 100 trials each came in around 6 hrs per experiment.

It is important to keep in mind that when crafting an experiment, as with any type of scientific experiment, there is usually a question in mind for which you currently do not yet have enough data to answer. The goal of the experiment is then to gather enough data as accurately and quickly as possible to develop your answer. As we’ll see after examining these two experiments, it can often take more than a single experiment to arrive at meaningful answers. Oftentimes the data resulting from the first experiment provides clarity and refinement for a second, more accurate experiment (ie before the first experiment, we didn’t know what we didn’t know… now we do). Let’s explore this in the next section!
## Lab
### Details
URL: `https://app.stormforge.io/`<br>
Application: `pet-clinic`<br>
Scenario: `pet-clinic-p95-latency`<br>
Experiments: `pet-clinic-latency-gc`, `pet-clinic-latency-jit`<br>
Load Test Case: `420 users viewing, adding, and modifying owner, pet, and visit record data`<br>
`Users arrived over three separate phases:`
* `duration: 30s, rate: 5 user/s`
* `duration: 30s, rate: 7 user/s`
* `duration: 30s, rate: 2 user/s`


The Pet Clinic experiment details are outlined below for review.

**pet-clinic-latency-gc**

**Number of trials:** `100`

**Input parameters:**<br>
* `cpu` - same setting for both requests and limits
* `memory` - same setting for both requests and limits
* `gc_newRatio` - ratio between young and old generation heap space for objects. 
* `gc_survivorRatio` - ratio between new and survivor heap space for objects in memory

**Output metrics (primary):**<br>
* `P95 Latency` - as measured by StormForge Performance Testing.
* `Cost` - measured according to the following formula
    * `( (cpu * 17) * (memory * 3) ) / 1000`

**Application Patches section:**

As with the experiment module 3, we only patched the Deployment object for Pet Clinic in this experiment. In it we are setting the `cpu` and `memory` values for both requests and limits to the same input parameter values respectively. In addition we are exploring the use of the `NewRatio` and `SurvivorRatio` JVM options along with ensuring that the `+PreferContainerQuotaForCPUCount` JVM option is enabled at runtime by passing along dynamically constructed strings (with each trial) as the value of the `JAVA_TOOL_OPTIONS` environment variable. When the JVM sees this environment variable on startup, it automatically applies the settings.

    patches:
    - targetRef:
      kind: Deployment
      apiVersion: apps/v1
      name: pet-clinic
      namespace: examples
    patch: |
      spec:
        template:
          spec:
            containers:
            - name: spring-petclinic
              resources:
                limits:
                  cpu: '{{ index .Values.cpu}}m'
                  memory: '{{ index .Values.memory}}Mi'
                requests:
                  cpu: '{{ index .Values.cpu}}m'
                  memory: '{{ index .Values.memory}}Mi'
              env:
              - name: JAVA_TOOL_OPTIONS
                value: '-XX:+PreferContainerQuotaForCPUCount -XX:NewRatio={{ .Values.gc_newRatio }} -XX:SurvivorRatio={{ .Values.gc_survivorRatio }}'

**Trial section:**

Notice in this trial section that a different image was used to serve as the trial job pod. This time a custom-built *StormForm Performance Testing* image was used to manage authentication and test case calling from the *StormForge Performance Testing* cloud service.

    trialTemplate:
    metadata:
      labels:
        stormforge.io/application: 'pet-clinic'
        stormforge.io/objective: 'cost-vs-p95-latency'
        stormforge.io/scenario: 'pet-clinic-p95-latency'
    spec:
      readinessGates:
      - selector:
          matchLabels:
            stormforge.io/application: 'pet-clinic'
            stormforge.io/scenario: 'pet-clinic-p95-latency'
        failureThreshold: 60
        conditionTypes:
        - ContainersReady
      jobTemplate:
        metadata:
          labels:
            stormforge.io/application: 'pet-clinic'
            stormforge.io/objective: 'cost-vs-p95-latency'
            stormforge.io/scenario: 'pet-clinic-p95-latency'
        spec:
          template:
            metadata:
              labels:
                stormforge.io/application: 'pet-clinic'
                stormforge.io/objective: 'cost-vs-p95-latency'
                stormforge.io/scenario: 'pet-clinic-p95-latency'
            spec:
              containers:
              - name: stormforger
                image: thestormforge/optimize-trials:v0.0.3-stormforge-perf
                env:
                - name: TITLE
                  valueFrom:
                    fieldRef:
                      fieldPath: metadata.name
                - name: TEST_CASE
                  value: <redacted test case name>
                - name: STORMFORGER_JWT
                  valueFrom:
                    secretKeyRef:
                      name: stormforge-perf-access-token
                      key: STORMFORGER_JWT
      setupServiceAccountName: optimize-setup-sa
      setupTasks:
      - name: monitoring
        args:
        - prometheus
        - $(MODE)

**Additional Info:**

This experiment required specific RBAC setup during the experiment along with a `Secret` resource used for authentication.


### pet-clinic-latency-jit

**Number of trials:** `100`

**Input parameters:**
* `cpu` - same setting for both requests and limits
* `memory` - same setting for both requests and limits
* `threadStackSize` - 
* `biasLocking` - 

**Output metrics (primary):**
* `P95 Latency` - as measured by StormForge Performance Testing.
* `Cost` - measured according to the following formula
    * `( (cpu * 17) * (memory * 3) ) / 1000` 

**Application Patches section:**

As with the experiment module 3, we only patched the Deployment object for Pet Clinic in this experiment. In it we are setting the `cpu` and `memory` values for both requests and limits to the same input parameter values respectively. In addition we are exploring the use of the `NewRatio` and `SurvivorRatio` JVM options along with ensuring that the `+PreferContainerQuotaForCPUCount` JVM option is enabled at runtime by passing along dynamically constructed strings (with each trial) as the value of the `JAVA_TOOL_OPTIONS` environment variable. When the JVM sees this environment variable on startup, it automatically applies the settings.

    patches:
        - targetRef:
            kind: Deployment
            apiVersion: apps/v1
            name: pet-clinic
            namespace: examples
        patch: |
        spec:
            template:
            spec:
                containers:
                - name: spring-petclinic
                resources:
                    limits:
                    cpu: '{{ index .Values.cpu}}m'
                    memory: '{{ index .Values.memory}}Mi'
                    requests:
                    cpu: '{{ index .Values.cpu}}m'
                    memory: '{{ index .Values.memory}}Mi'
                env:
                - name: JAVA_TOOL_OPTIONS
                    value: '-XX:+PreferContainerQuotaForCPUCount -Xss{{ .Values.threadStackSize }}k -XX:{{ .Values.biasLocking }}'

**Trial section:**

Notice in this trial section that a different image was used to serve as the trial job pod. This time a custom-built StormForm Performance Testing image was used to manage authentication and test case calling from the StormForge Performance Testing cloud service.


    
    trialTemplate:
        metadata:
            labels:
                stormforge.io/application: 'pet-clinic'
                stormforge.io/objective: 'cost-vs-p95-latency'
                stormforge.io/scenario: 'pet-clinic-p95-latency'
            spec:
            readinessGates:
            - selector:
                matchLabels:
                    stormforge.io/application: 'pet-clinic'
                    stormforge.io/scenario: 'pet-clinic-p95-latency'
                failureThreshold: 60
                conditionTypes:
                - ContainersReady
            jobTemplate:
                metadata:
                labels:
                    stormforge.io/application: 'pet-clinic'
                    stormforge.io/objective: 'cost-vs-p95-latency'
                    stormforge.io/scenario: 'pet-clinic-p95-latency'
                spec:
                template:
                    metadata:
                    labels:
                        stormforge.io/application: 'pet-clinic'
                        stormforge.io/objective: 'cost-vs-p95-latency'
                        stormforge.io/scenario: 'pet-clinic-p95-latency'
                    spec:
                    containers:
                    - name: stormforger
                        image: thestormforge/optimize-trials:v0.0.3-stormforge-perf
                        env:
                        - name: TITLE
                        valueFrom:
                            fieldRef:
                            fieldPath: metadata.name
                        - name: TEST_CASE
                        value: <redacted test case name>
                        - name: STORMFORGER_JWT
                        valueFrom:
                            secretKeyRef:
                            name: stormforge-perf-access-token
                            key: STORMFORGER_JWT
            setupServiceAccountName: optimize-setup-sa
            setupTasks:
            - name: monitoring
                args:
                - prometheus
                - $(MODE)

**Additional Info:**

This experiment required specific RBAC setup during the experiment along with a Secret resource used for authentication.

## Hands-on
Login to app.stormforge.io

Visit [https://app.stormforge.io](https://app.stormforge.io) (you will be redirect to an authentication page) and login using the provided shared workshop login credentials:
>image placeholder
>image placeholder


### Explore `pet-clinic-latency-gc` experiment

From the main page, choose the `pet-clinic` application and then the `pet-clinic-p95-latency` scenario:

<insert clean screenshot for application screen here from clean workshop tenant>
<insert clean screenshot for scenario screen here from clean workshop tenant>


Let’s examine the `pet-clinic-latency-gc` results first:

<insert clean screenshot for pet-clinic-latency-gc screen here from clean workshop tenant>


Notice the apparent trendline of peach and orange colored dots in the scatter plot shown. This is known as a Pareto front. Essentially this is the frontier of optimal data points for the given output metrics plotted on the X and Y axis and indicates a true tradeoff relationship between these metrics.

<insert clean screenshot for pet-clinic-latency-gc pareto front>

By using ML to drive experimentation and establish correlations between various combinations of input parameter values and associated measured output metric values, *StormForge Optimize Pro* facilitated the extremely fast discovery of this Pareto front enabling us to see a clear visual pattern for what the data is telling us.


Click the baseline square dot shown in blue and scroll down toward the bottom page

Click the recommended square dot shown in orange and scroll down toward the bottom of the page. Notice the input values and resulting output metric values in comparison to the baseline values.

Secondary Insights: CPU-bound or Memory-bound?

Plot Memory vs P95
Plot NewRatio and SurvivorRatio vs P95
Plot CPU vs P95.
Notice the curve closely resembles the original Pareto front curve for P95 vs Cost. This indicates a strong correlation between CPU and P95 latency and, as such, indicates that this is a CPU-bound application. As a result, memory and memory-related (heap, GC settings) parameter exploration will have little to no impact on P95, at least at given load test levels.

### Explore `pet-clinic-latency-jit` experiment

In looking at the first experiment, though it focused on exploring various garbage collection settings, we learned that Pet Clinic, under the current load test, is CPU-bound and that exploring CPU-related settings (thread stack size, etc) is likely to have more impact . Therefore, let’s examine the `pet-clinic-latency-jit` results next as this experiment focuses more on these CPU input parameters. From the  `pet-clinic-latency-gc` results page, click the Experiment breadcrumb link:

Then click the pet-clinic-latency-jit experiment:

<insert clean screenshot for pet-clinic-latency-jit screen here from clean workshop tenant>




## Additional information
* Percentile Definitions and Calculations [Wikipedia Page](https://en.wikipedia.org/wiki/Percentile)
* Pareto Front [Wikipedia Page](https://en.wikipedia.org/wiki/Pareto_front)
* StormForge Optimize Pro [Experiment and Trial reference](https://docs.stormforge.io/optimize-pro/reference/)
* Pet Clinic official [Github repository](https://github.com/spring-projects/spring-petclinic)
* Oracle Java 8 [CLI Documentation](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html#BGBCIEFC)


