# Planet 9 & Kubernetes

The Planet 9 docker image is hosted on [Docker Hub](https://hub.docker.com/r/neptunesoftware/planet9).
You can find the name and tags on Docker Hub.

# Examples

Here are a few examples on deploy Planet 9 within kubernetes. Configuring security (e,g, RBAC, Policies, etc...) and hosting of databases is out of scope in this document.

All examples use the same `requests` and `limits` settings. They are just arbitrary set values. Each Planet 9 application and API is unique in it's own way, translating in different needs for required compute resources. Based on monitoring, we advice to finetine the compute resources you need to allocate to your pods.

## Example 1: Basic

See `samples/sample-1/deployment.yml`

The most basic Planet 9 deployment fronted with a Load Balancer.

**Example Characteristics**
* 1x Planet 9 instance in one ReplicaSet.
* Database used is SQLite, and non persistent.
* Custom NPM Modules are non persistent.
* Planet 9 is available on port `8080` via the Public IP of the LoadBalancer. 
    * You can find this IP address via with the `kubectl get svc` command.

## Example 2: Configure Database

See `samples/sample-2/deployment.yml`

A Planet 9 deployment with multiple pods, all using the same database fronted with a Load Balancer. Example shows how to configure environment variables.

**Example Characteristics**
* 3x Planet 9 instances in one ReplicaSet.
* Database used is PostgreSQL, and peristing.
* Custom NPM Modules are not persist.
* No `secrets` are used in this.
* Planet 9 is available on port `8080` via the Public IP of the LoadBalancer. 

## Example 3: Shared Volume For Custom NPM Modules (AKS)

See `samples/sample-3/deployment.yml`

A Planet 9 deployment with multiple pods, all using the same database fronted with a Load Balancer. Example shows how to configure the shared PersistentVolume for NPM modules.

his example is with AKS for the Kubernetes Cluster and uses Azure Files as PersistentVolume provider.

Once the deployment is running, make sure to set the custom NPM path in the Planet 9 cockpit (Settings > Customizing > General > Custom NPM Installation Path). Based on the example `deployment.yml`file, the value should be `/home/planet9/customnpm` .

**Example Characteristics**
* 3x Planet 9 instances in one ReplicaSet.
* Database used is PostgreSQL, and peristing.
* Custom NPM Modules are persisted.
* No `secrets` are used in this.
* Planet 9 is available on port `8080` via the Public IP of the LoadBalancer. 

## Example 4: Using Secrets 

See `samples/sample-4/deployment.yml`

A Planet 9 deployment with multiple pods, all using the same database fronted with a Load Balancer. Example shows how to configure secrets.

Before applying/creating the deployment, create a secret for the DB user and Password:
```bash
kubectl create secret generic db-user-pass \ 
    --from-literal=dbuser=MyDbUser \
    --from-literal=dbpass='MyDbUserPassword'
```

**Example Characteristics**
* 3x Planet 9 instances in one ReplicaSet.
* Database used is PostgreSQL, and peristing.
* Custom NPM Modules are not persist.
* `secrets` are used to secure sensitive credentials.
* Planet 9 is available on port `8080` via the Public IP of the LoadBalancer. 