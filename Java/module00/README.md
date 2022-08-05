# An Introduction to StormForge Optimize Pro
---
### What problem does StormForge Optimize Pro solve?
Managing application parameters for Kubernetes deployments is difficult at best and near impossible at scale.

Beyond setting basic Kubernetes native deployment parameters like `requests` and `limits`, there are a whole host of *application-specific* environment variables that directly impact both application performance and the total cost to run the application.

![Kubernetes Parameters Can Be Complicated!](/Java/Assets/Images/k8s-param-gauges.png)
<sub>Figure 1: Application parameters can include resource `requests` and `limits` as well as other variables like `HEAP_SIZE`, `PARALLEL_GC_THREADS`, and many more. </sub>

--- 
The default behavior is to overprovision resources, ship features quickly, and worry about efficiency *(read: cost overruns)* later.

As organizations mature, it's common to create initiatives that result in the need for applications to be "tuned" to improve costs, performance, or both - manual application tuning exercises can involve too many people and take too much time.

In the end, exhaustive tuning of these applications may not seem worth the effort.

### How does StormForge Optimize Pro solve this problem?
StormForge Optimize Pro is a tool for experimenting with application configurations in Kubernetes. With the StormForge Optimize Pro Controller installed in your cluster, you can run experiments that rapidly iterate application parameter values and measure their outcome via one or more metrics.

#### What is an Experiment?
An Experiment is the basic unit of organization in StormForge Optimize. The purpose of an experiment is to try different configurations of an application’s parameters and measure their impact.

Experiments are objects written in YAML and when submitted via `kubectl`, autonomously follow an execution timeline until the configured `experimentBudget` is exhausted :

![Experiment Lifecycle](/Java/Assets/Images/experiment-timeline.png)
<sub>Figure 2. StormForge Optimize Pro's Experiment Cycle</sub>

Once the Experiment is completed, the results can be viewed via the [StormForge Optimize User Interface](https://app.stormforge.io) or via the `stormforge` CLI.

#### 





