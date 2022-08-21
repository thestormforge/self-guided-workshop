# MOD002 - Exploring the Pet Clinic App

## Overview

Throughout this workshop we will be examining the Pet Clinic application. Pet Clinic is a reference Spring Boot application that uses the traditional MVC architecture that was common across enterprises before the advent of microservices. This monolithic application is a great example of how a “legacy” Java application can not only be successfully containerized and migrated to Kubernetes, but also, as we’ll see throughout the course of this workshop, optimized to run well without requiring a re-write.
>image placeholder


Pet Clinic tracks a list of pet owners, their various pets, and the associated pet visits to the clinic for various veterinarian services with standard RESTful CRUD (Create, Read, Update, and Delete) operations.

The instance of Pet Clinic built for this workshop is using Java 8 (OpenJDK 1.8.0_333).


## Lab
### Details 
Namespace: `examples`<br>
Deployment: `pet-clinic`<br>
Service: `pet-clinic`<br>
Ingress: `pet-clinic`

### Hands-on

#### Explore environment using the CLI

Use `kubectl` to retrieve all the key kubernetes objects associated with Pet Clinic

    ❯ kubectl -n examples get all,ingress -l app=pet-clinic
    NAME                              READY   STATUS    RESTARTS   AGE
    pod/pet-clinic-76495cfb55-g6bbk   1/1     Running   0          74s

    NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
    service/pet-clinic   ClusterIP   10.100.83.156   <none>        8080/TCP   35d

    NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/pet-clinic   1/1     1            1           35d

    NAME                                    DESIRED   CURRENT   READY   AGE
    replicaset.apps/pet-clinic-76495cfb55   1         1         1       77s

    NAME                                   CLASS    HOSTS                        ADDRESS                                                                         PORTS   AGE
    ingress.networking.k8s.io/pet-clinic   <none>   pet-clinic.stormforge.show   a45d762ff9fe44059b7f76ea55edd243-1165a9e8428ca9e8.elb.us-east-1.amazonaws.com   80      35d

In this workshop we are only running Pet Clinic with a single pod. While Pet Clinic can be configured to run against a persistent database such as Postgres or MariaDB, we are using an embedded, in-memory database for the workshop.


#### Find publicly accessible hostname

In order to publicly access Pet Clinic, we need to know either a resolvable FQDN (fully-qualified domain name, either an A or CNAME) or a publicly accessible IP address, along with the IP port. We can obtain this by getting details for the Pet Clinic ingress object:

    ❯ kubectl -n examples get ingress/pet-clinic
    NAME         CLASS    HOSTS                        ADDRESS                                                                         PORTS   AGE
    pet-clinic   <none>   pet-clinic.stormforge.show   a45d762ff9fe44059b7f76ea55edd243-1165a9e8428ca9e8.elb.us-east-1.amazonaws.com   80      35d

The workshop host should be able to provide guidance on which should be used (FQDN or Address) for browser access.


#### Explore application from web browser

Using the FQDN we discovered above, let’s access Pet Clinic in a web browser:
>image placeholder

Let’s find some pet owners from the home page:
>image placeholder

Leave the Last name field empty and click Find Owner:
>image placeholder
From the results page, click on Betty Davis
>image placeholder
We can see that Betty has a single pet hamster named Basil but has not yet visited the clinic
>image placeholder

Feel free to click around to explore Pet Clinic more if needed.
#### A note about load tests

In module 3 we’ll learn a bit more about what comprises an experiment. One of the key components of an experiment is an appropriate trial job that is applied to the application during the experiment. For many performance-based experiments the appropriate trial job is a load test. Modern load testing tools like StormForge Performance Testing or K6 can be used to quickly setup test cases by importing HTTP ARchive (HAR) files and performing some minor modifications. HAR files can be recorded by using the developer tools included with most browsers. 

For Chrome, developer tools can be opened from any page by right-clicking anywhere on the page and choosing Inspect from the context menu:
>image placeholder


From the resulting window that opens click the Network tab and ensure that the Recording button is depressed:
>image placeholder



Click around the application. As you do, you should notice each new request recorded in the developer tools pane. When you are finished recording all the application operations you need, you can generate the HAR file by right-clicking any entry in the Name pane, choose Copy, then chose Copy all as HAR from the sub menu. This will save the contents of the recording as text into the clipboard.
>image placeholder


You may have noticed another option: Save all as HAR with content. The reason we do not use this is is that ALL content (images, stylesheets, fonts, etc) will be saved to the file instead of just the HTTP requests.

Once you have your HAR file contents in memory, either paste it to a text file manually (this could take some time) or use a handy tool like pbpaste in Mac OS:

`❯ pbpaste > har.txt`

Once you have your HAR file you can import into your choice of load testing tool for further customization.


## Additional information
* Pet Clinic official [Github repository](https://github.com/spring-projects/spring-petclinic)
* Oracle Java 8 [CLI Documentation](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html#BGBCIEFC)
