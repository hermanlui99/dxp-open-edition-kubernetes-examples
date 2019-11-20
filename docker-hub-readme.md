# About Planet 9
Unify everything!
With Neptune DX Platform’s Planet 9 you get the best of both worlds. A consistent, award-winning UX for all your apps and a drag-and-drop, low-code platform that simplifies your application development, integration, deployment, and management efforts.

With fragmentation on the rise in IT, unifying your UX has never been more critical. But application development the old-fashioned way opens the enterprise up to unacceptable risk. Get back control with Planet 9 and standardize app development and optimize knowledge transfer from one resource to another. Take advantage of Planet 9’s unmatched architectural flexibility to take your apps and infrastructure anywhere with integration into any cloud, any backend and any architecture.

With application development for any backend based on REST APIs, running on state of the art backend and cloud-agnostic NodeJS, you now have the freedom of choice and flexibility to innovate – whether you’re developing new apps and front-end functionality, building applications on existing backends, or merging the best functionality of multiple systems into one powerful app.

Do more. Do it faster and with less risk. Scale effortlessly. Look brilliant. That’s life on Planet 9.

[Visit Official Website](https://www.neptune-software.com)

# Configuration

## General 

The default Planet 9 instance does not run as a `root` therefore the Planet 9 instance cannot listen by default on port `80` or `443`. Take that into consideration when choosing the port that Planet 9 should bind to. By default Planet 9 binds to `8080`. To configure SSL, see the **SSL** section.

A Planet 9 instance always starts up with a local admin user. You can set the initial admin password with `INITIAL_ADMIN_PASSWORD`. This value will be only used if the underlying database has yet to be initialized. Once the underlying database is initialized, the password will be persisted in the underlying database. The value will not be valid anymore once the password has been changed in the Planet 9's cockpit.

Note that you can also modify the majority of these settings in the cockpit, whatever is configured in the cockpit takes precedence over environment variables.

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

You can enable SSL within Planet9 itself. By default, this is disabled, set the `ENABLE_SSL` flag to enable SSL and specify other relevant environment variables.

| Name | Default | Description | 
| ---  | ---     | ---         |
| `ENABLE_SSL`   | `false` | Run Planet9 in HTTPS mode. |
| `SSLCA`   | `Empty String` | The SSL certificate authority. (Required if SSL enabled and a SSL Certificate Authority is issued.) |
| `SSLCERT`   | `Empty String` |  The SSL certificate. (Required if SSL enabled) |
| `SSLKEY`   | `Empty String` | The SSL private key. (Required if SSL enabled)  |
| `SSLPASSPHRASE`   | `Empty String` |  The SSL passphrase (Required if SSL enabled and a passhrase is required) |
| `SSL_PORT`   | `8081` | The port to bind on for SSL. (Required if SSL enabled) |


## Database

Planet 9 persists all its state (Apps, Server Scripts, APIs, ...) and the majority of its configuration in an underlying database. **By default, a Planet 9 instance will initialize (or connect to) an SQLite database**. 

The SQLite database is initialized within the container. This means that the SQLite database is ephemeral and will be deleted on termination of the container. At the time of writing, mounting volume for the SQLite database is not supported. But this support is in the feature pipeline. ***SQLite is only intended for sandbox environments***.

Planet 9 supports PostgreSQL for its underlying database. More support for other database types (e.g. MS SQL) will be added in the near future.

**Important**: *For more production-like environments, we suggested to use any other DB type aside from SQLite.*

### Configure PostegreSQL

You can configure your connection string with a single environment variable, using `DB_URI_POSTGRES` or you can configure each part of a connection with individual variables. 

The target database **must have** a schema named `planet9` before connecting a Planet 9 instance to it. For more information regarding this consult the ["Planet 9 installation guide"](https://www.neptune-software.com/download-your-trial/). If the database is empty, Planet 9 will initialize all required tables and data on startup.

| Name | Default | Description |
| ---  | ---     | ---         |
| `DB_TYPE` | `sqlite` | One of the following values `sqlite`, `postgresql`. |
| `DB_URI_POSTGRES` | `Empty String` | The URI for the database. If this is set, other POSTGRES options are ignored |
| `DB_PSQL_HOST`    | `localhost` |  Database host name/IP |
| `DB_PSQL_PORT`    | `5432` |  Database port |
| `DB_PSQL_USER`    | `Empty String` | DB user |
| `DB_PSQL_PASSWORD`| `Empty String` | DB password |
| `DB_PSQL_SSL`     | `false` | Enable SSL mode for the database |
| `DB_PSQL_DBNAME`  | `Planet9` | The database name |

