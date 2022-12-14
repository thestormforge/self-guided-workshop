# MOD00 - An Introduction to Machine Learning-Based Optimization

## Solving for Kubernetes Complexity at Scale
Managing application parameters for Kubernetes deployments is difficult at best and near impossible at scale.

Beyond setting basic Kubernetes native deployment parameters like `requests` and `limits`, there are a whole host of *application-specific* environment variables that directly impact both application performance and the total cost to run the application.

<p align="center">
  <img src="/Java/Assets/Images/k8s-param-gauges.png" />
</p>

<sub>Figure 1: Application parameters can include resource `requests` and `limits` as well as other variables like `HEAP_SIZE`, `PARALLEL_GC_THREADS`, and many more. </sub>

The default behavior is to overprovision resources, ship features quickly, and worry about efficiency *(read: cost overruns)* later.

As organizations mature, it's common to create initiatives that result in the need for applications to be "tuned" to improve costs, performance, or both - manual application tuning exercises can involve too many people and take too much time.

In the end, exhaustive tuning of these applications may not seem worth the effort.

## The Role of Machine Learning

<p align="center">
  <img src="/Java/Assets/Images/multi-regression.png" />
</p>

Machine learning has become a powerful tool for analyzing large datasets to discover patterns and insights for easier human consumption. Today, these ML-based insights serve to ease decision making for everything from setting your home thermostat to driving your car safely.

<p align="center">
  <img src="/Java/Assets/Images/generic-experiment-results-2.png" />
</p>

When properly applied to resource management in Kubernetes, machine learning will prove essential to ensuring the myriad of deployment parameters are properly set to achieve optimal goals like performance and resource efficiency at scale.

## What is *StormForge Optimize Pro*?
*StormForge Optimize Pro* is a ML-powered tool for experimenting with application configurations in Kubernetes. With the StormForge Optimize Pro Controller installed in your cluster, you can run experiments that rapidly iterate application parameter values and measure their outcome via one or more metrics.

<p align="center">
  <img src="/Java/Assets/Images/trial-flow-controller-updated.png" />
</p>

By continuously testing application configurations in a rapid experimentation loop powered by machine learning, *StormForge Optimize Pro* is able to hone in on the optimal configurations that meet or exceed your goals, reducing overall time and human effort by thousands of hours per year.

## Additional Information
* Learn more about *StormForge* products at [https://www.stormforge.io](https://www.stormforge.io)
* Visit the [StormForge YouTube Channel](https://www.youtube.com/channel/UCW05S9esT9PKb9tkLrnbUoA)


<p align="center">
  <b>You have completed MOD00. You may now proceed to <a href="/Java/module01/README.md">MOD01</a>.</b>
</p>
