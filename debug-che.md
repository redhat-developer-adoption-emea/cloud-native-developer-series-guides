## Debugging Applications

In this lab you will debug the coolstore application using Java remote debugging and look into line-by-line code execution as the code runs inside a container on OpenShift.

#### Investigating Bugs

There are some tools at hand in order to allow us to track down and solve problems, let's see some of them.

##### Reading logs

~~~shell
$ oc get dc
NAME                   REVISION   DESIRED   CURRENT   TRIGGERED BY
catalog                50         1         1         config,image(catalog:latest)
catalog-postgresql     1          1         1         config,image(postgresql:9.6)
gateway                2          1         1         config,image(gateway:latest)
inventory              4          1         1         config,image(inventory:latest)
inventory-postgresql   1          1         1         config,image(postgresql:9.6)
web                    1          1         1         config,image(web:latest)
$ oc logs dc/gateway | grep -i error
...
WARNING: Inventory error for 444436: status code 204
...
~~~

##### Enable Remote Debugging 

Remote debugging is a useful debugging technique for application development which allows looking into the code that is being executed somewhere else on a different machine and execute the code line-by-line to help investigate bugs and issues. Remote debugging is part of  Java SE standard debugging architecture which you can learn more about it in [Java SE docs](https://docs.oracle.com/javase/8/docs/technotes/guides/jpda/architecture.html).

The Java image on OpenShift has built-in support for remote debugging and it can be enabled by setting the `JAVA_DEBUG=true` environment variables on the deployment config for the pod that you want to remotely debug.

An easier approach would be to use the fabric8 maven plugin to enable remote debugging on the Inventory pod. It also forwards the default remote debugging port, 5005, from the Inventory pod to your workstation so simplify connectivity.

Let's imagine you want to add a trace to the `CatalogController` and see the value of a variable, as in the next example. 

~~~java
long s = products.getExactSizeIfKnown();
System.out.println("s ===> " + s);
~~~

Please open `com.redhat.cloudnative.catalog.CatalogController` and change your code accordingly.

~~~java
package com.redhat.cloudnative.catalog;

import java.util.List;
import java.util.Spliterator;
import java.util.stream.Collectors;
import java.util.stream.StreamSupport;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.MediaType;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
@RequestMapping(value = "/api/catalog")
public class CatalogController {

	  @Autowired
    private ProductRepository repository;

    @ResponseBody
    @GetMapping(produces = MediaType.APPLICATION_JSON_VALUE)
    public List<Product> getAll() {
        Spliterator<Product> products = repository.findAll().spliterator();
        long s = products.getExactSizeIfKnown();
        System.out.println("s ===> " + s);
        return StreamSupport.stream(products, false).collect(Collectors.toList());
    }
}
~~~

Enable remote debugging on 'Catalog' by running the following inside the `labs/catalog-spring-boot` directory in the Eclipse Che **Terminal** window:

~~~shell
$ mvn fabric8:debug
~~~~

> The default port for remoting debugging is `5005` but you can change the default port via environment variables. Read more in the [Java S2I Image docs](https://access.redhat.com/documentation/en-us/red_hat_jboss_middleware_for_openshift/3/html/red_hat_java_s2i_for_openshift/reference#configuration_environment_variables).

You are all set now to start debugging using the tools of you choice. 

Do not wait for the command to return! The fabric8 maven plugin keeps the forwarded port open so that you can start debugging remotely.

![Fabric8 Debug]({% image_path remote-debug-che-fabric8.png %}){:width="900px"}

#### Remote Debug with Eclipse Che

Eclipse Che provides a convenience way to remotely connect to Java applications running inside containers and debug while following the code execution in the IDE.

From the **Run** menu, click on **Edit Debug Configurations...**.

![Remote Debug]({% image_path remote-debug-che-config-1.png %}){:width="600px"}

The window shows the debuggers available in Eclipse Che. Click on the plus sign near the Java debugger.

![Remote Debug]({% image_path remote-debug-che-config-2.png %}){:width="700px"}

Configure the remote debugger and click on the **Save** button:

* Check **Connect to process on workspace machine**
* Port: `5005`

![Remote Debug]({% image_path remote-debug-che-config-3.png %}){:width="700px"}

You can now click on the **Debug** button to make Eclipse Che connect to the Inventory service running on OpenShift.

You should see a confirmation that the remote debugger is successfully connected.

![Remote Debug]({% image_path remote-debug-che-config-4.png %}){:width="360px"}

Open `com.redhat.cloudnative.catalog.CatalogController` and click on the editor sidebar on the line number of the first line of the `getAll()` method to add a breakpoint to that line.

![Add Breakpoint]({% image_path remote-debug-che-breakpoint.png %}){:width="600px"}

Open a new **Terminal** window and use `curl` to invoke the Calatog API  in order to pause the code execution at the defined breakpoint.

Note that you can use the the following icons to switch between debug and terminal windows.

![Icons]({% image_path remote-debug-che-window-guide.png %}){:width="700px"}

>  You can find out the Inventory route url using `oc get routes`. Replace 
> `{{INVENTORY_ROUTE_HOST}}` with the Inventory route url from your project.

~~~
$ curl -v http://{{INVENTORY_ROUTE_HOST}}/api/inventory/444436
~~~

Switch back to the debug panel and notice that the code execution is paused at the 
breakpoint on `InventoryResource` class.

![Icons]({% image_path remote-debug-che-window-breakpoint-stop.png %}){:width="900px"}

Click on the _Step Over_ icon to execute one line and retrieve the inventory object for the 
given product id from the database.

![Step Over]({% image_path remote-debug-che-step-over.png %}){:width="340px"}

Click on the the plus icon in the **Variables** panel to add the `products` variable to the list of watch variables. This would allow you to see the value of `products` variable during execution.

![Watch Variables]({% image_path remote-debug-che-variables.png %}){:width="500px"}

![Debug]({% image_path remote-debug-che-values.png %}){:width="900px"}

Look at the **Variables** window 's' value is 8 so the number of retrieved  objects is `8`.

Click on the _Resume_ icon to continue the code execution and then on the stop icon to end the debug session.

Go back to the **Terminal** window where `fabric8:debug` was running. Press `Ctrl+C` to stop the debug and port-forward.

Well done and congratulations for completing all the labs.