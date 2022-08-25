# MOD03 - Pet Clinic Startup Experiment

## Overview

In this module we will be creating and starting a Pet Clinic experiment to explore the JVM startup times resulting from various CPU and memory pod resource settings.

![pet-clinic-experiment-over](/Java/Assets/Images/pet-clinic-experiment-over.png)

Why is this information useful to know you might ask? The reason is that once resulting startup times are known for specific input values of CPU and memory, you can now make informed decisions on how many resources to allocate to your app so that you can ensure the app will start in a reasonable amount of time and balance that against ensuring the app can be quickly scheduled by Kubernetes. In addition, once you know startup times for your app, you can also:

* Tune/set ideal Startup/Readiness probe time settings, without guesswork, ensuring the quickest time for a pod to be ready for service
* Know how long additional capacity deployed during an HPA scaling event will take before being available. This is a combination of the app startup time and how quickly it passes its startup/readiness probes. In some cases, this info can also be used to determine what utilization threshold to set and when the HPA should scale to compensate for pod spin-up lead times.
* If capacity is usually pre-deployed ahead of known usage spikes (Wallstreet opening bell, sales events, etc), this information can be used to determine when the pre-deploying needs to happen (minimize too-early deployment costs)
* Know how long Deployment rollouts and/or canary deployment strategies will take to complete

**<table><tr><td>In addition to all the above mentioned reasons, we wanted to demonstrate a useful experiment that can be completed and ready for analysis within 2 - 3 hours (within a single workshop session).</td></tr></table>**

### Anatomy of an experiment

With StormForge Optimize Pro, we define an experiment as a custom resource containing the following components at a minimum:
* Parameters - specified inputs
* Metrics - measured outputs
* Patch(es) - change template(s) applied to the application under experiment in between trials
* Trial - specific instance of the experiment executed after the application has been appropriately patched with current parameter values.
    * Trial Job - kubernetes Job definition that executes the appropriate trial workload or action against the application to properly conduct the experiment scenario

We will explore the specifics of an experiment further as we create the Pet Clinic startup time experiment in the next section.

## Lab
### Details
Namespace: `pet-clinic`<br>
Deployment: `pet-clinic`<br>
Service: `pet-clinic`<br>
Ingress: `pet-clinic`<br>

The Pet Clinic experiment details are outlined below for review. They are also already included in the example `experiment` YAML file referenced in the lab steps further below.

**Input parameters (same setting for both requests and limits):**<br>
`cpu`<br>
`memory`<br>
`gc_newRatio`<br>
`gc_survivorRatio`<br>
`initialCodeCacheSize`<br>
`threadStackSize`<br>

**Output metrics:**<br>
* `Startup Time` - measured as a scraped value from the Pet Clinic startup log file using `'Started PetClinicApplication in (\d+\.*\d*) seconds'` as a search regular expression. If the application didn’t print out a startup time in the logs, it could be calculated as *ContainersReady* status timestamp minus container’s *startedAt* timestamp. The reasoning behind using these data points is that we do not want to include any possible image fetching time and other scheduling delays when calculating app startup time.
    * *ContainersReady* value: `kubectl -n ${NAMESPACE} get pod ${APP_POD} -o json | jq -r '.status.conditions[] | select(.status == "True" and .type == "ContainersReady") | .lastTransitionTime'`
    * *startedAt* value: `kubectl -n ${NAMESPACE} get pod ${APP_POD} -o json | jq -r '.status.containerStatuses[0].state.running.startedAt'`
* `Cost` - measured according to the following formula
    * `( (cpu * 17) * (memory * 3) ) / 1000`

**Application Patches section:**

We are only patching the Deployment object for Pet Clinic. In it we are setting the `cpu` and `memory` values for both requests and limits to the same input parameter values respectively. In addition we are ensuring that the `+PreferContainerQuotaForCPUCount` JVM option is enabled at runtime by passing it along as the value of the `JAVA_TOOL_OPTIONS` environment variable. When the JVM sees this environment variable on startup, it automatically applies the setting.

    patches:
      - targetRef:
      kind: Deployment
      apiVersion: apps/v1
      name: pet-clinic
      namespace: pet-clinic
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
                value: '-XX:+PreferContainerQuotaForCPUCount'

**Trial section:**

    trialTemplate:
    metadata:
      labels:
        stormforge.io/application: 'pet-clinic'
        stormforge.io/objective: 'cost-vs-startup-time'
        stormforge.io/scenario: 'pet-clinic-startup-time'
    spec:
      readinessGates:
      - selector:
          matchLabels:
            stormforge.io/application: 'pet-clinic'
            stormforge.io/scenario: 'pet-clinic-startup-time'
        failureThreshold: 60
        conditionTypes:
        - ContainersReady
      jobTemplate:
        metadata:
          labels:
            stormforge.io/application: 'pet-clinic'
            stormforge.io/objective: 'cost-vs-startup-time'
            stormforge.io/scenario: 'pet-clinic-startup-time'
        spec:
          template:
            metadata:
              labels:
                stormforge.io/application: 'pet-clinic'
                stormforge.io/objective: 'cost-vs-startup-time'
                stormforge.io/scenario: 'pet-clinic-startup-time'
            spec:
              containers:
              - name: startup-checker
                image: public.ecr.aws/stormforge/examples/startup-checker:latest
                imagePullPolicy: Always
                env:
                - name: NAMESPACE
                  value: pet-clinic
                - name: APP_NAME
                  value: pet-clinic
              serviceAccountName: optimize-setup-sa
      setupServiceAccountName: optimize-setup-sa
      setupTasks:
      - name: monitoring
        args:
        - prometheus
        - $(MODE)

**Additional Info:**

If the trial job or trial setup tasks require specific RBAC permissions, appropriate `ServiceAccounts`, `Roles/ClusterRoles`, and `RoleBindings/ClusterRoleBindings` must be created and configured for use by the trial job and/or setup tasks. These kubernetes objects were required for this experiment and were included in the `experiment` YAML file referenced below.



## Hands-on
### Clone examples repo

If you have't completed this step from **MOD01**, change directory to a good working directory and then run<br>

    git clone https://github.com/thestormforge/examples.git

This repo contains several example experiments. For this module we will be using the files located in `examples/java/pet-clinic/experiments/`

### Explore `experiment-startup.yaml`

Using your favorite text editor, open up `examples/java/pet-clinic/experiments/experiment-startup.yaml` and review its contents to familiarize yourself with experiment details.


### Confirm current cluster authentication

If you have not done so already, authenticate with the StormForge cloud services using the stormforge CLI tool using the command below. Note that using the --force flag will simply force a re-login if you were already logged in.

    stormforge login --force

Opening your default browser to visit:

	https://auth.stormforge.io/authorize?audience=https%3A%2F%2Fapi.stormforge.io%2F&client_id=pE3kMKdrMTdW4DOxQHesyAuFGNOWaEke&code_challenge=_U-E-BKpaHKKFHKIr1Dx3pgzsh1X8ORPplFuqw3jUzE&code_challenge_method=S256&redirect_uri=http%3A%2F%2F127.0.0.1%3A8085%2F&response_type=code&scope=offline_access+manage%3Aclients&state=nPX8X25As6fZn2OUzIE12w

    You are now logged in.

**Note:** The default login action is to open a browser window automatically and attempt an automated, browser-based authentication flow with api.stormforge.io. If this fails you may need to manually login via `stormforge login --force --qr` or `stormforge login --force --url` and follow the instructions provided along with the supplied verification code.

Validate logged-in status with the CLI tool as well.

    stormforge ping                                                                                                                                                                                                                                 7m 4s
    PING api.stormforge.io (54.85.38.4, 34.231.143.103, 23.22.11.158): HTTP/1.1
    PONG time=315.235ms


Finally, validate that `kubectl` is correctly authenticated:

    kubectl get nodes
    NAME                             STATUS   ROLES    AGE    VERSION
    ip-192-168-56-110.ec2.internal   Ready    <none>   43m    v1.21.12-eks-5308cf7
    ip-192-168-8-109.ec2.internal    Ready    <none>   134m   v1.21.12-eks-5308cf7

### Create experiment

Create the experiment:

    cd examples/java/pet-clinic
    kubectl apply -f experiments/experiment-startup.yaml

Output:

    experiment.optimize.stormforge.io/pet-clinic-startup created
    serviceaccount/optimize-setup-sa created
    clusterrole.rbac.authorization.k8s.io/optimize-setup-cr created
    clusterrolebinding.rbac.authorization.k8s.io/optimize-setup-crb created

To monitor the status of the trials, run the following command and look for similar output below:

    kubectl -n pet-clinic get trials -w
<br>
Output:

    NAME                     STATUS       ASSIGNMENTS            VALUES
    pet-clinic-startup-000   Setting up   cpu=300, memory=1000
    pet-clinic-startup-000   Setting up   cpu=300, memory=1000
    pet-clinic-startup-000   Setting up   cpu=300, memory=1000
    pet-clinic-startup-000   Setup Created   cpu=300, memory=1000
    pet-clinic-startup-000   Setup Created   cpu=300, memory=1000
    pet-clinic-startup-000   Patching        cpu=300, memory=1000
    pet-clinic-startup-000   Patching        cpu=300, memory=1000
    pet-clinic-startup-000   Patching        cpu=300, memory=1000
    pet-clinic-startup-000   Patched         cpu=300, memory=1000
    pet-clinic-startup-000   Patched         cpu=300, memory=1000
    pet-clinic-startup-000   Waiting         cpu=300, memory=1000

From here we can watch the experiment trials progress through their various stages or see any issues with the experiment that may need to be addressed.


### Experiment troubleshooting

Though the experiment has been well tested, you may want to try modifying the experiment or even crafting your own. In any case, if the experiment fails to create or during trial runs, here are some helpful places to investigate.

Optimize Pro Controller Logs:

    kubectl -n stormforge-system logs `kubectl -n stormforge-system get pods | grep optimize-controller | cut -d " " -f1`
<br>
Output:

    …
    {"level":"info","ts":1660765923.0315225,"logger":"controllers.Server","msg":"Created new trial","experiment":"examples/pet-clinic-startup","reportTrialURL":"https://api.stormforge.io/v1/experiments/pet-clinic-startup/trials/67","assignments":[{"name":"cpu","value":1106},{"name":"memory","value":688}]}
    {"level":"error","ts":1660765991.0884452,"logger":"controller-runtime.controller","msg":"Reconciler error","controller":"trial-job","request":"examples/pet-clinic-startup-067","error":"jobs.batch \"pet-clinic-startup-067\" already exists"}
    {"level":"info","ts":1660766007.5368335,"logger":"controllers.Metric","msg":"Retrying Prometheus query to include final scrape","trial":"examples/pet-clinic-startup-067","startTime":1660765991,"completionTime":1660765992,"lastScrapeEndTime":1660765999.88382}
    {"level":"info","ts":1660766013.1584847,"logger":"controllers.Server","msg":"Reported trial","experiment":"examples/pet-clinic-startup","trial":"examples/pet-clinic-startup-067","values":{"values":[{"metricName":"startup-time","value":39},{"metricName":"cost","value":20.866},{"metricName":"cost-cpu-requests","value":18.802},{"metricName":"cost-memory-requests","value":2.064}],"startTime":"2022-08-17T19:53:11Z","completionTime":"2022-08-17T19:53:12Z"},"reportTrialURL":"https://api.stormforge.io/v1/experiments/pet-clinic-startup/trials/67"}
    {"level":"info","ts":1660766013.5412104,"logger":"controllers.Server","msg":"Created new trial","experiment":"examples/pet-clinic-startup","reportTrialURL":"https://api.stormforge.io/v1/experiments/pet-clinic-startup/trials/68","assignments":[{"name":"cpu","value":255},{"name":"memory","value":653}]}
    {"level":"info","ts":1660766199.0956974,"logger":"controllers.Metric","msg":"Retrying Prometheus query to include final scrape","trial":"examples/pet-clinic-startup-068","startTime":1660766182,"completionTime":1660766183,"lastScrapeEndTime":1660766197.072664}
    {"level":"info","ts":1660766203.6465013,"logger":"controllers.Server","msg":"Reported trial","experiment":"examples/pet-clinic-startup","trial":"examples/pet-clinic-startup-068","values":{"values":[{"metricName":"startup-time","value":129},{"metricName":"cost","value":6.294},{"metricName":"cost-cpu-requests","value":4.335},{"metricName":"cost-memory-requests","value":1.959}],"startTime":"2022-08-17T19:56:22Z","completionTime":"2022-08-17T19:56:23Z"},"reportTrialURL":"https://api.stormforge.io/v1/experiments/pet-clinic-startup/trials/68"}
    {"level":"info","ts":1660766203.711725,"logger":"controllers.Server","msg":"Created new trial","experiment":"examples/pet-clinic-startup","reportTrialURL":"https://api.stormforge.io/v1/experiments/pet-clinic-startup/trials/69","assignments":[{"name":"cpu","value":683},{"name":"memory","value":652}]}
    …

Experiment resource description (look toward the end of the output for `Status` and `Events` info:

    kubectl -n pet-clinic describe experiment pet-clinic-startup
    …
    Status:
    Active Trials:  1
    Phase:          Running
    Events:           <none>

Pet Clinic pod description and/or app logs:

    kubectl -n pet-clinic describe pod pet-clinic-6b6cb7c68f-w226w

    kubectl -n pet-clinic describe pod pet-clinic-6b6cb7c68f-w226w


Finally, description and/or logs from the specific trial job pod (note that the number in the pod name below corresponds to the trial number of the experiment:

    kubectl -n pet-clinic describe pod pet-clinic-startup-001-hllrd

    kubectl -n pet-clinic logs pod pet-clinic-startup-001-hllrd


## Additional information
* StormForge Optimize Pro [Experiment and Trial reference](https://docs.stormforge.io/optimize-pro/reference/)
* StormForge [examples repository](https://github.com/thestormforge/examples)
* Pet Clinic official [Github repository](https://github.com/spring-projects/spring-petclinic)
* Oracle Java 8 [CLI Documentation](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html#BGBCIEFC)

**You have completed MOD03. You may now proceed to MOD04**
