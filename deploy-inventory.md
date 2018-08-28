## Deploying the Inventory Service

As explained in the introduction we need an Inventory service as part of the Cool Store Portal architecture to provide the quantity (hence the availability too) of a given product by ID.

We're going to save time by deploying the 'inventory' service instead of coding it. But before we actually deploy the service we're going to explain briefly some concepts.

#### Technology

Our **'Inventory'** service has been implemented as a Java EE Microprofile application mainly using JAX-RS (REST) and JPA (persistency). Red Hat Openshift provides several runtimes to build services using Node.js, Java, C#, etc. Given that the service is implemented using Java EE Microprofile technology we're going to use  WildFly Swarm runtime. 

#### What is WildFly Swarm?

Java EE applications are traditionally created as an `ear` or `war` archive including all dependencies and deployed in an application server. Multiple Java EE applications can and were typically deployed in the same application server. This model is well understood in the development teams and has been used over the past several years.

WildFly Swarm offers an innovative approach to packaging and running Java EE applications by packaging them with just enough of the Java EE server runtime to be able to run them directly on the JVM using `java -jar`. For more details on various approaches to packaging Java 
applications, read [this blog post](https://developers.redhat.com/blog/2017/08/24/the-skinny-on-fat-thin-hollow-and-uber).

WildFly Swarm is based on WildFly and it's compatible with MicroProfile, which is a community effort to standardize the subset of Java EE standards such as JAX-RS, CDI and JSON-P that are useful for building microservices applications.

Since WildFly Swarm is based on Java EE standards, it significantly simplifies refactoring existing Java EE applications to microservices and allows much of the existing code-base to be reused in the new services.

#### WildFly Swarm Maven Project 

The `inventory-wildfly-swarm` project has the following structure which shows the components of the WildFly Swarm project laid out in different subdirectories according to Maven best practices:

![Inventory Project]({% image_path wfswarm-inventory-project.png %}){:width="340px"}

This is a minimal Java EE project with support for JAX-RS for building RESTful services and JPA for connecting to a database. [JAX-RS](https://docs.oracle.com/javaee/7/tutorial/jaxrs.htm) is one of Java EE standards that uses Java annotations to simplify the development of RESTful web services. [Java Persistence API (JPA)](https://docs.oracle.com/javaee/7/tutorial/partpersist.htm) is another Java EE standard that provides Java developers with an object/relational mapping facility for managing relational data in Java applications.

#### Deploy Inventory on OpenShift

It’s time to build and deploy our service on OpenShift. 

OpenShift [Source-to-Image (S2I)]({{OPENSHIFT_DOCS_BASE}}/architecture/core_concepts/builds_and_image_streams.html#source-build) 
feature can be used to build a container image from a git repository. OpenShift S2I uses the [supported OpenJDK container image](https://access.redhat.com/documentation/en-us/red_hat_jboss_middleware_for_openshift/3/html/red_hat_java_s2i_for_openshift) to build the final container image of the 
Inventory service by building the WildFly Swam uber-jar from source code (build strategy **'Source'**), using Maven, to the OpenShift platform.

Next commands are going to deploy our Inventory service. Please be sure you're at {{COOLSTORE_PROJECT}} before executing them.

* **Name:** inventory
* **S2I runtime:** redhat-openjdk18-openshift
* **Image tag:** 1.4
* **Repository:** {{LABS_GIT_REPO}}
* **Context Directory:** inventory-wildfly-swarm

~~~shell
$ oc new-app redhat-openjdk18-openshift:1.4~{{LABS_GIT_REPO}} \
        --context-dir=inventory-wildfly-swarm \
        --name=inventory

$ oc expose svc/inventory
~~~

Once this completes, your project should be up and running. OpenShift runs the different components of the project in one or more pods which are the unit of runtime deployment and consists of the running 
containers for the project. 

Let's take a moment and review the OpenShift resources that are created for the Inventory REST API:

* **Build Config**: `inventory` build config is the configuration for building the Inventory container image from the inventory source code
* **Image Stream**: `inventory` image stream is the virtual view of all inventory container images built and pushed to the OpenShift integrated registry.
* **Deployment Config**: `inventory` deployment config deploys and redeploys the Inventory container image whenever a new Inventory container image becomes available
* **Service**: `inventory` service is an internal load balancer which identifies a set of pods (containers) in order to proxy the connections it receives to them. Backing pods can be added to or removed from a service arbitrarily while the service remains consistently available, 
enabling anything that depends on the service to refer to it at a consistent address (service name or IP).
* **Route**: `inventory` route registers the service on the built-in external load-balancer and assigns a public DNS name to it so that it can be reached from outside OpenShift cluster.

You can review the above resources in the OpenShift Web Console or using `oc describe` command:

> `bc` is the short-form of `buildconfig` and can be interchangeably used 
> instead of it with the OpenShift CLI. The same goes for `is` instead 
> of `imagestream`, `dc` instead of `deploymentconfig` and `svc` instead of `service`.

~~~shell
$ oc describe bc inventory
$ oc describe is inventory
$ oc describe dc inventory
$ oc describe svc inventory
$ oc describe route inventory
~~~

You can see the exposed DNS url for the Inventory service in the OpenShift Web Console or using OpenShift CLI:

~~~shell
$ oc get routes

NAME        HOST/PORT                                        PATH       SERVICES  PORT  TERMINATION   
inventory   inventory-{{COOLSTORE_PROJECT}}.roadshow.openshiftapps.com   inventory  8080            None
~~~

Copy the route url for the Inventory service and verify the API Gateway service works using `curl`:

> The route urls in your project would be different from the ones in this lab guide! Use the one from yor project.

~~~shell
$ curl http://{{INVENTORY_ROUTE_HOST}}/api/inventory/329299

{"itemId":"329299","quantity":35}
~~~

> TODO: RE	VIEW WHY IS THIS SO IMPORTANT! IF THIS TEST IS NOT RUN BEFORE  curl http://gateway-coolstore-user1.apps.madrid.openshiftworkshop.com/api/products {"error":"Connection was closed"}cvicensa-mbp:cloud-native-reduced-labs cvicensa$ curl http://gateway-coolstore user1.apps.madrid.openshiftworkshop.com/api/products {"error":"Connection refused: /172.30.196.227:8080"}c

Well done! You are ready to move on to the next lab.
