# MOD04 - P95 Latency Experiments

## Overview

As previously mentioned, the Pet Clinic startup time experiment used in module 3 was chosen to fit the timeframe of this single day workshop. Now that that experiment has been successfully created and running, let’s shift gears a bit and examine the results of a previously run, performance-based experiment for Pet Clinic.

This experiment was driven by a *StormForge Performance Testing* test case trial job that ran for 1 minute and 30 seconds each invocation and used an imported HAR file (generated using the instructions from module 2). The total experiment run time for 150 trials came in at around 10 ½ hrs.

It is important to keep in mind that when crafting an experiment, as with any type of scientific experiment, there is usually a question in mind for which you currently do not yet have enough data to answer. The goal of the experiment is then to gather enough data as accurately and quickly as possible to develop your answer. The key is conducting the experiment in a systematic manner so that accurate conclusions can be drawn from the different combinations of input variables to the experiment. The catch is that, once the number of input variables exceeds four, human beings have great difficulty reasoning about the experiment at hand. The experiment that we will be reviewing in this module has six input parameters with some parameter value possibilities ranging from 5 to 4500. This makes for a total input variable combination of over 585 trillion possibilities. This would be impossible to explore without some sort of ML assistance!


Lastly, as we’ll see after examining this experiment, the data can often reveal more insights than anticipated. This is attributed to the fact that, once you have a few ways to quickly compare your data points, new knowledge can be quickly gained. 

Let’s explore this in the next section!
## Lab
### Details
* URL: https://app.stormforge.io/
* Login Information: provided by workshop host(s)
* Application: `pet-clinic`
* Scenario: `pet-clinic-p95-latency`
* Experiments: `pet-clinic-latency-gc-jit`
* Load Test Case: `420 users viewing, adding, and modifying owner, pet, and visit record data`
  *  Users arrived over three separate phases:
      * duration: 30s, rate: 5 user/s
      * duration: 30s, rate: 7 user/s
      * duration: 30s, rate: 2 user/s


The Pet Clinic experiment details are outlined below for review.

#### pet-clinic-latency-gc

Number of trials: `150`

**Input parameters:**
* `cpu` - same setting for both requests and limits
* `memory` - same setting for both requests and limits
* `gc_newRatio` - ratio between young and old generation heap space for objects. 
* `gc_survivorRatio` - ratio between new and survivor heap space for objects in memory
* `threadStackSize` - setting for how much memory to allocate each thread’s stack.
* `initialCodeCacheSize` - initial amount of memory used by the JIT to cache compiled code. The JIT will determine if and when it makes sense to compile Java byte code down to native machine code for speeding up execution.

**Output metrics (primary):**
* `P95 Latency` - as measured by *StormForge Performance Testing*.
* `Cost` - measured according to the following formula
  * `( (cpu * 17) * (memory * 3) ) / 1000`

**Application Patches section:**

As with the experiment module 3, we only patched the Deployment object for Pet Clinic in this experiment. In it we are setting the cpu and memory values for both requests and limits to the same input parameter values respectively. In addition we are exploring the use of the `NewRatio`, `SurvivorRatio`, `ThreadStackSize`, and `InitialCodeCacheSize` JVM options along with ensuring that the +PreferContainerQuotaForCPUCount JVM option is enabled at runtime by passing along dynamically constructed strings (with each trial) as the value of the JAVA_TOOL_OPTIONS environment variable. When the JVM sees this environment variable on startup, it automatically applies the settings.

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
                    value: '-XX:+PreferContainerQuotaForCPUCount -XX:NewRatio={{ .Values.gc_newRatio }} -XX:SurvivorRatio={{ .Values.gc_survivorRatio }} -Xss{{ .Values.threadStackSize }}k -XX:InitialCodeCacheSize={{ .Values.initialCodeCacheSize }}k'

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

### Additional Info:

This experiment required specific RBAC setup during the experiment along with a `Secret` resource used for authentication.

## Hands-on
#### Login to app.stormforge.io

Visit https://app.stormforge.io (you will be redirect to an authentication page) and login using the provided shared workshop login credentials:

![browser-address](/Java/Assets/Images/browser-address.png)

![browser-login](/Java/Assets/Images/browser-login.png)

#### Explore `pet-clinic-latency-gc-jit` experiment

From the main page, choose the `pet-clinic` application…

![mod04-walkthrough-1](/Java/Assets/Images/mod04-walkthrough-1.png)

…and then the pet-clinic-p95-latency scenario:

![mod04-walkthrough-2](/Java/Assets/Images/mod04-walkthrough-2.png)

Click the  pet-clinic-latency-gc-jit experiment to view the results:

![mod04-walkthrough-3](/Java/Assets/Images/mod04-walkthrough-3.png)

Notice the apparent trendline of peach and orange colored dots in the scatter plot shown. This is known as a Pareto front. Essentially this is the frontier of optimal data points for the given output metrics plotted on the X and Y axis and indicates a true tradeoff relationship between these metrics.

![mod04-walkthrough-4](/Java/Assets/Images/mod04-walkthrough-4.png)

By using ML to drive experimentation and establish correlations between various combinations of input parameter values and associated measured output metric values, *StormForge Optimize Pro* facilitates the fast discovery of this Pareto front enabling us to see a clear visual pattern for what the data is telling us. In this case, we can see that though the app team may have done some initial optimization to establish the current baseline configuration, it looks like there are better performing, and cheaper options (we can have our cake and eat it too in this case)!

Click the baseline square dot shown in blue. A large tooltip appears providing quick details of the baseline experiment trial:

![mod04-walkthrough-5](/Java/Assets/Images/mod04-walkthrough-5.png)

This baseline trial is usually the first trial to run, is configured to run the application as it currently exists, and serves as the basis for comparison between any other trial. 

If you scroll down toward the bottom page, you can see the full details of this selected baseline trial including the current input parameter values used for the trial, the range of values for the input parameters that is configured for ML to explore, and finally the resulting metrics measurements.  As you click other trial data points, you can scroll back down here to see how those particular trial details compare against the baseline and recommended results.

![mod04-walkthrough-6](/Java/Assets/Images/mod04-walkthrough-6.png)

Scroll back up to the graph and click the recommended square dot shown in orange and scroll down toward the bottom of the page. Notice the input values and resulting output metric values in comparison to the baseline values.

![mod04-walkthrough-7](/Java/Assets/Images/mod04-walkthrough-7.png)

*StormForge Optimize Pro* will typically recommend a trial that exists on the Pareto front that represents a good balance between the two trade-off metrics displayed. Having said this, keep in mind that only you or someone from your application product team really understands which metric takes highest priority. The thing to keep in mind is that you know which direction to move on the Pareto front to choose the configuration setting that best fits your specific needs.


#### Secondary Insights: CPU-bound or Memory-bound?

As mentioned in this module’s overview section, once we have data in hand we can often make secondary discoveries. Let’s see if we have enough information to determine if this application is a CPU bound or Memory bound application.

Scroll back up to the main graph at the top of the page. To the right of the graph there are three dropdown boxes. The first box labeled *Filter By* allows us to choose which trials we would like to have plotted on the graph. The next two allow us to choose which data we would like to have plotted on the *X-Axis* and *Y-Axis* respectively. Click the *X-Axis* dropdown and choose *Memory* from the list:

![mod04-walkthrough-8](/Java/Assets/Images/mod04-walkthrough-8.png)

The graph should be updated and show something similar to the following:

![mod04-walkthrough-9](/Java/Assets/Images/mod04-walkthrough-9.png)

Notice that, though there are a few optimal data points between 900MB and 1650MB or Memory, all the rest of the optimal data points show memory configurations using 600MB - 650MB. This indicates that ML recognized that there was no correlation between memory values and performance. In addition, since ML was asked to minimize the resource cost, ML determined it was ok to use the lowest memory settings from the range and still achieve the performance gains.


If Pet Clinic isn’t memory bound, let’s see what a CPU bound application looks like. Choose *CPU* from the *X-Axis* dropdown. The graph should be updated and show something similar to the following:

![mod04-walkthrough-10](/Java/Assets/Images/mod04-walkthrough-10.png)

Notice that the *CPU* vs *P95* graph strongly resembles the Pareto front curve from the *Cost vs P95* graph we first examined. This indicates that ML found a very strong correlation between cpu values and P95 latency results. This indicates that this application is indeed a CPU bound application. The more CPU we throw at Pet Clinic, the better it will perform for this specific load scenario. The graph also shows us where the point of diminishing returns exists (right at the recommended datapoint) where larger jumps in CPU values give us less and less latency improvements.

#### Secondary Insights: Additional Tuning Info

Finally, let’s examine two of the other JVM tuneable input parameters that ML found to have some impact on overall performance as well.

Choose *GC Survivor Ratio* from the *X-Axis* dropdown.

![mod04-walkthrough-11](/Java/Assets/Images/mod04-walkthrough-11.png)

Notice that the majority of the optimal data points use values of 8 or 9. ML favored these values most for the optimal points. Given that 8 is the default value for this setting in the JVM, it could be left unmodified based on these results.

Next choose *Initial Code Cache Size* from the *X-Axis* dropdown.

![mod04-walkthrough-12](/Java/Assets/Images/mod04-walkthrough-12.png)

For this setting, ML favored values between 900k and 1500k (most heavily favored). This is a drastic difference from the default JVM value of 500. As a result, we can conclude that this setting has a moderately strong impact on the overall results. 

Feel free to click around and see if you can draw your own conclusions from the different data plots. This concludes our review of this experiment.

## Additional information
* Graeme S. Halford, Rosemary Baker, Julie E. McCredden, & John D. Bain. (2005). [How many variables can humans process?](https://pubmed.ncbi.nlm.nih.gov/15660854/) Psychological Science, Volume 16 (Issue 1), pg 70-76
doi: 10.1111/j.0956-7976.2005.00782.x
* Percentile Definitions and Calculations [Wikipedia Page](https://en.wikipedia.org/wiki/Percentile)
* Pareto Front [Wikipedia Page](https://en.wikipedia.org/wiki/Pareto_front)
* *StormForge Optimize Pro* [Experiment and Trial reference](https://docs.stormforge.io/optimize-pro/reference/)
* Pet Clinic official [Github repository](https://github.com/spring-projects/spring-petclinic)
* Oracle Java 8 [CLI Documentation](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html#BGBCIEFC)


<p align="center">
  <b>You have completed MOD04. You may now proceed to MOD05.</b>
</p>
