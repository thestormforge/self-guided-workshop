# MOD005 - Pet Clinic Startup Results

## Overview

In this module we will review the experiment results for our Pet Clinic startup time experiment and explore possible follow-up experiments that could be conducted or useful decisions made based on experiment findings.
## Lab
### Details
* URL: [https://app.stormforge.io/](https://app.stormforge.io)
* Application: `pet-clinic`
* Scenario: `pet-clinic-p95-latency`
* Experiments: `pet-clinic-startup`

### Hands-on

Login to [app.stormforge.io](https://app.stormforge.io)

Visit [https://app.stormforge.io](https://app.stormforge.io) (you will be redirect to an authentication page) and login using the provided shared workshop login credentials:

<p align="center">
  <img src="/Java/Assets/Images/browser-address.png" />
</p>
<p align="center">
  <img src="/Java/Assets/Images/browser-login.png" />
</p>

Explore `pet-clinic-startup` experiment

From the main page, choose the `pet-clinic` application…
 
<p align="center">
  <img src="/Java/Assets/Images/pet-clinic-walkthrough-1.png" />
</p>

…and then the pet-clinic-startup-time scenario:

<p align="center">
  <img src="/Java/Assets/Images/pet-clinic-walkthrough-2.png" />
</p>

Let’s examine the `pet-clinic-latency-gc` results first:

<p align="center">
  <img src="/Java/Assets/Images/pet-clinic-walkthrough-3.png" />
</p>


Notice that a Pareto front exists for this experiment also (if it didn’t, that would indicate that there is no direct trade-off relationship between the two metrics we chose: Startup Time and Resource Cost):

<p align="center">
  <img src="/Java/Assets/Images/pet-clinic-walkthrough-4.png" />
</p>

Notice that our baseline configuration, from a startup time standpoint, also has plenty of room for improvement (as we also saw from the experiment results in Module 004). There are at least a few configurations that are both faster and cheaper than our baseline.

Also, notice that for each measured startup time, there seems to be a range “plateau” where, for several values of resource `Cost` there was no change in `Startup Time`. This information is good to know, especially in conjunction with other performance optimization experiments as this would allow us to tune for both a balanced ideal Pet Clinic startup time and performance. In fact, this experiment could effectively be combined with a P95 latency vs Cost experiment to see how ML reconciles this for us (an experiment to try on your own with your trial subscription perhaps).









## Additional information
* Percentile Definitions and Calculations [Wikipedia Page](https://en.wikipedia.org/wiki/Percentile)
* Pareto Front [Wikipedia Page](https://en.wikipedia.org/wiki/Pareto_front)
* StormForge Optimize Pro [Experiment and Trial reference](https://docs.stormforge.io/optimize-pro/reference/)
* Pet Clinic official [Github repository](https://github.com/spring-projects/spring-petclinic)
* Oracle Java 8 [CLI Documentation](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html#BGBCIEFC)


<p align="center">
  <b>You have completed MOD05. You may now proceed to MOD06.</b>
</p>
