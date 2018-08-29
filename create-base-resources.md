## Deploying Base Resources

Before we start creating and deploying our Cool Store Portal services we need to create some base resources, namely:

* 1 x database for 'Inventory'
* 1 x Database for 'Catalog'

> Before you deploy these database be sure you're in the correct namespace **{{COOLSTORE_PROJECT}}-{{OPENSHIFT_USER}}**

#### Inventory Database

~~~shell
oc new-app postgresql-persistent \
    --param=DATABASE_SERVICE_NAME=inventory-postgresql \
    --param=POSTGRESQL_DATABASE=inventory \
    --param=POSTGRESQL_USER=inventory \
    --param=POSTGRESQL_PASSWORD=inventory \
    --labels=app=inventory
~~~

#### Catalog Database

~~~shell
oc new-app postgresql-persistent \
    --param=DATABASE_SERVICE_NAME=catalog-postgresql \
    --param=POSTGRESQL_DATABASE=catalog \
    --param=POSTGRESQL_USER=catalog \
    --param=POSTGRESQL_PASSWORD=catalog \
    --labels=app=catalog
~~~ 

Well done! You are ready to move on to the next lab.
