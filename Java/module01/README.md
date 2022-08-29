# MOD01 - Verifying Tools and Access

## Overview
In this module, we'll focus on installing the requisite CLI tools, cloning the workshop repo, and signing up for a *StormForge Optimize* trial account that we'll use to run our experiments.

## Lab

### Prerequisites
Verify that the following tools have been installed on the machine you intend to participate in the workshop with.
1. [kubectl](https://kubernetes.io/docs/tasks/tools/#kubectl)
2. [stormforge CLI](https://docs.stormforge.io/optimize-pro/getting-started/install/#installing-the-stormforge-command-line-interface)
3. [Git](https://github.com/git-guides/install-git)

### Hands On

___

#### Setting the KUBECONFIG environment variable (Optional)

For those attending a guided workshop, a kubeconfig file will be provided by the StormForge team.

While there are many methods for adding and managing contexts in an existing kubeconfig, the simplest approach is to temporarily use the kubeconfig file provided by the StormForge team by setting the KUBECONFIG environment variable.

##### **Step 1** Set your KUBECONFIG environment variable

###### Mac/Linux<br>
    export KUBECONFIG="<path to kubeconfig>"
###### Windows PowerShell<br>
    $Env:KUBECONFIG=("<path to kubeconfig>")

##### **Step 2** Verify that you can access your workshop cluster

    kubectl get svc -n default

**Output**

    NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
    kubernetes   ClusterIP   10.0.0.1     <none>        443/TCP   3d15h

___ 


#### Clone the Workshop Github Repository
Once your prerequisite CLI tools have been installed on your machine, the first step will be to clone the workshop repo to a local directory.

    git clone https://github.com/thestormforge/examples.git

___

#### Sign up for a StormForge Optimize Trial Account
1. visit [https://app.stormforge.io](https://app.stormforge.io)<br>
![signup-with-labels](/Java/Assets/Images/signup-labeled.png)

<sub>
1. Click "Sign Up"<br>
2. Enter your corporate email and choose a password<br>
3. Check the box to agree to the StormForge Terms of Service and Privacy Policy<br>
4. Click "Sign Up >"</sub>

___

#### Confirm your account via email
After you've signed up, you'll receive an email message to confirm your account.

![confirmation email](/Java/Assets/Images/confirmation-email.png)

Click "Confirm my account"

Your default web browser will open and you should see an "Email Verified" message to complete your account confirmation.

![confirmation web](/Java/Assets/Images/confirmation-web.png)

#### Deploy the *StormForge Optimize Pro* Controller

If you have not installed the `stormforge` CLI, please follow the instructions listed in the *Prerequisites* section of this MOD.

**Using the `stormforge` CLI, install the *StormForge Optimize Pro* controller in your cluster**

    stormforge init

**Confirm that the controller has been deployed**

    kubectl get namespaces

**Output**

        NAME                STATUS   AGE
    default             Active   3d15h
    kube-node-lease     Active   3d15h
    kube-public         Active   3d15h
    kube-system         Active   3d15h
    monitoring          Active   3d15h
    pet-clinic          Active   3d15h
    stormforge-system   Active   88s

You'll notice a new `stormforge-system` namespace

You can now check to see that the pod has started and is `Running`

    kubectl get pods -n stormforge-system

**Output**

    NAME                                           READY   STATUS    RESTARTS   AGE
    optimize-controller-manager-576fc5fd59-jcshg   1/1     Running   0          4m6s

<p align="center">
  <b>You have completed MOD01. You may now proceed to <a href="/Java/module02/README.md">MOD02</a>.</b>
</p>
