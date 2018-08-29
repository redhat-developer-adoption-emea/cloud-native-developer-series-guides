## Deploying Base Resources

Before we start creating and deploying our Cool Store Portal services we need to create some base resources, namely:

* 1 x Database for 'Inventory'

> Before you deploy these resources be sure you're in the correct namespace: **{{COOLSTORE_PROJECT}}-{{OPENSHIFT_USER}}**
> 
~~~shell
$ oc project
Using project "{{COOLSTORE_PROJECT}}-{{OPENSHIFT_USER}}" on server "{{OPENSHIFT_CONSOLE_URL}}".
~~~

#### Inventory Database
By default our 'Inventory' service will use PostgreSQL, so please run the following command to deploy an instance of PostgreSQL.

~~~shell
oc new-app postgresql-persistent \
    --param=DATABASE_SERVICE_NAME=inventory-postgresql \
    --param=POSTGRESQL_DATABASE=inventory \
    --param=POSTGRESQL_USER=inventory \
    --param=POSTGRESQL_PASSWORD=inventory \
    --labels=app=inventory
~~~

Well done! You are ready to move on to the next lab.
