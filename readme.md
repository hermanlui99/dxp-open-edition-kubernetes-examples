# Planet9 & Kubernetes

If still any questions remain after reading this document, or linked documents, contact me via `ian@neptune-software.com` regarding any questions.

## Summary

This repo contains different samples to showcase how you can deploy a Planet9 instance with Kuberenetes. 

In addition to this documentation, we strongly suggest to read the ["Planet 9 installation guide"](https://www.neptune-software.com/download-your-trial/) first.

## Docker Image

The Planet 9 docker image is hosted on [Docker Hub](https://hub.docker.com/r/neptunesoftware/planet9).
You can find the name and tags on Docker Hub.

# Pod/Container Configuration

Each Planet 9 container can dynamically configured with environmental variables on startup. All environment variables are optional, and will fallback to a default value if not specified.

## General 

The default Planet 9 instance does not run as a `root` therefore the Planet 9 instance cannot listen by default on port `80` or `443`. Take that in consideration when choosing the port that Planet 9 should bind to. By default Planet 9 binds to `8080`. To configure SSL, see the **SSL** section.

A Planet 9 instance always starts up with a local admin user. You can set the initial admin password with `INITIAL_ADMIN_PASSWORD`. This value will be only used if the underlying database has yet to be initialized. Once the underlying database is initialized, the password will be persisted in the underlying database. The value will not be valid anymore once the password has been changed in the Planet 9's cockpit.

| Name | Default | Description |
| ---  | ---     | ---         |
| `NODE_ENV`   | `production` | The application environment |
| `NAME`   | `Planet 9` | The application/instance name |
| `DESCRIPTION`   | `Installation` | The application description  |
| `INITIAL_ADMIN_PASSWORD`   | `admin` |  The initial default password for the `admin` user. |
| `SESSION_TIMEOUT`   | `180` | User session timeout in minutes           |
| `INSTANCES`   | `2` | The number of child processess that P9 should start up with            |
| `PORT`   | `8080` |             |    |
| `BACKGROUNDJOB_INTERVAL`   | `60` | How often the engine should check to see if there are any jobs to run (in seconds)            |

## SSL

There are several ways to support SSL/HTTPS on a Pod level. It's up to you to choose if you want to have SSL/HTTPS on a Pod level, or terminating the HTTPS at the load balancer and internally continue over HTTP. Two suggestions are:

1. Use a `NGINX` container in the Pod which handles the SSL part and forwards traffic to the Planet 9 container instance which itself will operate on HTTP. This will allow you to listen on port `443` for SSL, unlike the next option.
2. Enable SSL on the Planet 9 container itself. By default this is disabled, set the `ENABLE_SSL` flag to enable SSL and specify other relevant environment variables.

| Name | Default | Description | 
| ---  | ---     | ---         |
| `ENABLE_SSL`   | `false` | Run Planet9 in HTTPS mode. |
| `SSLCA`   | `Empty String` | The SSL certificate authority. (Required if SSL enabled and a SSL Certificate Authority is issued.) |
| `SSLCERT`   | `Empty String` |  The SSL certificate. (Required if SSL enabled) |
| `SSLKEY`   | `Empty String` | The SSL private key. (Required if SSL enabled)  |
| `SSLPASSPHRASE`   | `Empty String` |  The SSL passphrase (Required if SSL enabled and a passhrase is required) |
| `SSL_PORT`   | `8081` | The port to bind on for SSL. (Required if SSL enabled) |


## Database

Planet 9 persists all its state (Apps, Server Scripts, APIs, ...) and the majority of it's configuration in an underlying database. **By default, a Planet 9 instance will initialize (or connect to) a SQLite database**. 

The SQLite database is initialized within the container. This means that the SQLite database is ephemeral and will be deleted on termination of the Pod. At the time of writting, mounting a persistent volume for the SQLite database is not supported. But this support is in the feature pipeline. ***SQLite is only intended for sandbox environments***.

Planet 9 supports PostgreSQL for it's underlying database. More support for other database types (e.g. MS SQL) will be added in the near future.

**Important**: *For more production-like environments, we suggested to use any other DB type aside from SQLite.*

| Name | Default | Description |
| ---  | ---     | ---         |
| `DB_TYPE` | `sqlite` | One of the following values `sqlite`, `postgresql`. |

### Configure PostegreSQL

You can configure your connection string with a single environment variable, using `DB_URI_POSTGRES` or you can configure each part of a connection with individual variables. 

The target database **must have** a schema named `planet9` before connecting a Planet 9 instance to it. For more information regarding this consult the ["Planet 9 installation guide"](https://www.neptune-software.com/download-your-trial/). If the database is empty, Planet 9 will initialize all required tables and data on startup.

| Name | Default | Description |
| ---  | ---     | ---         |
| `DB_URI_POSTGRES` | `Empty String` | The URI for the database. If this is set, other POSTGRES options are ignored |
| `DB_PSQL_HOST`    | `localhost` |  Database host name/IP |
| `DB_PSQL_PORT`    | `5432` |  Database port |
| `DB_PSQL_USER`    | `Empty String` | DB user |
| `DB_PSQL_PASSWORD`| `Empty String` | DB password |
| `DB_PSQL_SSL`     | `false` | Enable SSL mode for the database |
| `DB_PSQL_DBNAME`  | `Planet9` | The database name |

# Architecture

Everything (aside of NPM Modules) in Planet 9 is persisted in the underlying database. Therefore you can use a `ReplicaSet` for deploying multiple Planet 9 pods, as long all Pods connect to the same database.

![Load Balancer Graph With ReplicaSet](images/deployment_load_balancer.png)

You have the liberty to hosts your database within Kubernetes. If you have limited skills available regarding databases, we advise to consider using cloud managed databases as hosting databases with High Availability and Disaster Recovery in mind is an expertise on its own.

Planet 9 is a Node application that forks child processses to locally load balance within the Planet 9 instance. The amount of child processes can be configured (see configuration). The more processes you run, the more vCPU's you should consider to allocate to your pods.

## NPM Modules

A feature of Planet 9 is that you can use NPM modules for server scripts. The path to these NPM modules can be configured in the cockpit. When using Kubernetes, we prefer stateless apps. Theferefor we advice to persist the NPM modules in one of the following ways:

* Create your own custom Planet 9 Docker Image, based on the Planet 9 Docker image and include the NPM modules that you use. Use your customized Docker Image for the Pods.
![](images/deployment_npm_custom_image.png)
* Use a shared persisted volume that you mount to the Planet 9 container. Make sure to use a shared storage type (e.g. AWS EFS, Azure Files). Mount the PersistentVolume/PersistentVolumeClaim to the path that is configured in the Planet 9 cockpit. This NPM Path setting is stored in the database. So all Planet 9 instances that use the same database will have the same NPM path setting.
![](images/deployment_npm_shared_volume.png)

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