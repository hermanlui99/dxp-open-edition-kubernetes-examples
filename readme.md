# DXP Open Edition On Kubernetes

The DXP Open Edition (previously called Planet 9) docker image is hosted on [Docker Hub](https://hub.docker.com/r/neptunesoftware/planet9).
You can find the name and tags on Docker Hub. If you'd like to run locally using Docker Compose, check our [examples](https://github.com/neptune-software/dxp-open-edition-docker-compose-examples).

## System Design

DXP Open Edition requires a database to function. By default, `sqlite` is shipped within the Docker image. If you start a single Docker container, with default configuration, an sqlite database is initialized within the container's ephemeral file storage. If another database is configured (using environment variables), the sqlite database is not created.

It is strongly discourages to use `sqlite` in production and any high available setup. The `sqlite` database does not support concurrent access. This means, that you can't use a persistent volume for the `sqlite` database with `spec.accessModes` set to `ReadWriteMany`. Resulting in a limitation that only a single pod can be mounted to given persisten volume.

For production and high available configurations, an external database is strongly advised. This database can be either hosted in the cluster and outside the cluster. **It is important to verify that your Kubernetes PODs are allowed to connect to your database (verify firewall, vnet, vpc, routing tables, network, ...)**.

### Custom NPM Modules

DXP Open Edition allows you to use custom NPM modules. These NPM modules are installed on the filesystem. When working with an high available configuration (replica set > 1 PODs), you will have to make these custom NPM modules available for all of your DXP Open Edition pods.

**The following options are available:**

1. Create an **custom Docker image** that has the desired NPM Modules pre-installed. This Image can be an extended image of Neptune's [Official Docker Image](https://hub.docker.com/r/neptunesoftware/planet9) or alternatively, you can start from any arbitrary base image and copy the DXP Open Edition binairies in that image, along with the custom NPM Modules.
    * **PRO**
        * Faster
        * There is no state in your deployment as there are no persistent volumes to manage.
        * Scales much better.
    * **CON**
        * A change managment (DevOps) pipeline should be in place to build new Docker images, for when custom NPM Modules should be added or removed of your image.
        * You are unable to ad-hoc install custom NPM Modules via the DXP Open Editio cockpit.

2. Create a shared **Kubernetes persistent volume** with `spec.accessModes` set to `ReadWriteMany` so all DXP Open Edition PODs can share the same persistent volume.
    * **PRO**
        * You are able to ad-hoc install custom NPM Modules via the DXP Open Editio cockpit
        * No need for a change managment (DevOps) pipeline to add/remove custom NPM Modules.
    * **CON**
        * Slower
        * There is state in your deployment as there are persistent volumes to manage. You'll have to use Kubernetes Statefulset.

## Examples

Here are a few examples on deploy DXP Open Edition within kubernetes. Configuring security (e,g, RBAC, Policies, etc...) and hosting of databases is out of scope in this document.

All examples use the same `requests` and `limits` settings. These are just arbitrary set values. Each DXP Open Edition application and API is unique in it's own way, translating in different needs for required compute resources. Based on monitoring, we advice to finetune the compute resources you need to allocate to your pods.

### Example 1: Basic

See `samples/sample-1/deployment.yml`

The most basic DXP Open Edition deployment fronted with a Load Balancer.

**Example Characteristics**
* 1x DXP Open Edition instance in one ReplicaSet.
* Database used is SQLite, and non persistent.
* Custom NPM Modules are non persistent.
* DXP Open Edition is available on port `8080` via the Public IP of the LoadBalancer. 
    * You can find this IP address via with the `kubectl get svc` command.

### Example 2: Configure Database

See `samples/sample-2/deployment.yml`

A DXP Open Edition deployment with multiple pods, all using the same database fronted with a Load Balancer. Example shows how to configure environment variables.

**Example Characteristics**
* 3x DXP Open Edition instances in one ReplicaSet.
* Database used is PostgreSQL, and peristing.
* Custom NPM Modules are not persist.
* No `secrets` are used in this.
* DXP Open Edition is available on port `8080` via the Public IP of the LoadBalancer. 

### Example 3: Shared Volume For Custom NPM Modules (AKS)

See `samples/sample-3/deployment.yml`

A DXP Open Edition deployment with multiple pods, all using the same database fronted with a Load Balancer. Example shows how to configure the shared PersistentVolume for NPM modules.

his example is with AKS for the Kubernetes Cluster and uses Azure Files as PersistentVolume provider.

Once the deployment is running, make sure to set the custom NPM path in the DXP Open Edition cockpit (Settings > Customizing > General > Custom NPM Installation Path). Based on the example `deployment.yml`file, the value should be `/home/planet9/customnpm` .

**Example Characteristics**
* 3x DXP Open Edition instances in one ReplicaSet.
* Database used is PostgreSQL, and peristing.
* Custom NPM Modules are persisted.
* No `secrets` are used in this.
* DXP Open Edition is available on port `8080` via the Public IP of the LoadBalancer. 

### Example 4: Using Secrets 

See `samples/sample-4/deployment.yml`

A DXP Open Edition deployment with multiple pods, all using the same database fronted with a Load Balancer. Example shows how to configure secrets.

Before applying/creating the deployment, create a secret for the DB user and Password:
```bash
kubectl create secret generic db-user-pass \ 
    --from-literal=dbuser=MyDbUser \
    --from-literal=dbpass='MyDbUserPassword'
```

**Example Characteristics**
* 3x DXP Open Edition instances in one ReplicaSet.
* Database used is PostgreSQL, and peristing.
* Custom NPM Modules are not persist.
* `secrets` are used to secure sensitive credentials.
* DXP Open Edition is available on port `8080` via the Public IP of the LoadBalancer. 
