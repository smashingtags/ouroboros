Command line arguments can be viewed by running:

```
docker run --rm pyouroboros/ouroboros --help
```
> All command line arguments can be substituted with an environment variable. command line options are kebab-case and environment variables are SCREAMING_CASE. E.g. `--docker-sockets vs. DOCKER_SOCKETS`. All examples will be given as environment variables for a `docker run` as --help will show command line examples

***

* Usage
  * [Help](#help)
  * [Core](#core)
    * [Version](#version)
    * [Docker Sockets](#docker-sockets)
    * [Docker TLS](#docker-tls)
    * [Docker TLS Verify](#docker-tls-verify)
    * [Interval](#interval)
    * [Cron](#cron)
    * [Log Level](#log-level)
    * [Self Update](#self-update)
    * [Run Once](#run-once)
    * [Dry Run](#dry-run)
  * [Docker Specifics](#docker-specifics)
    * [Monitor](#monitor)
    * [Ignore](#ignore)
    * [Label Enable](#label-enable)
    * [Labels Only](#labels-only)
    * [Cleanup](#cleanup)
    * [Latest](#latest)
    * [Repository User](#repository-user)
    * [Repository Password](#repository-password)
  * [Data Export Options](#data-export-options)
    * [Data Export](#data-export)
    * [Prometheus](#prometheus)
      * [Prometheus Address](#prometheus-address)
      * [Prometheus Port](#prometheus-port)
    * [Influxdb](#influxdb)
      * [Influx URL](#influx-url)
      * [Influx Port](#influx-port)
      * [Influx Username](#influx-username)
      * [Influx Password](#influx-password)
      * [Influx Database](#influx-database)
      * [Influx SSL](#influx-ssl)
      * [Influx Verify SSL](#influx-verify-ssl)
  * [Notifications](#notifications)
  * [Labels](#labels)
* [Examples](#examples)
  * [Configuration](#configuration)
    * [.env File](#env-file)

***

## Help
**Type:** Boolean - Interrupting  
**Command Line:**  `-h, --help`  

Shows the help message then exits

## Core
### Version
**Type:** Boolean - Interrupting  
**Command Line:**  `-v, --version`  

Shows the current version number then exits

### Docker Sockets
**Type:** List - Space separated  
**Command Line:**  `-d, --docker-sockets`  
**Environment Variable:** `DOCKER_SOCKETS`  
**Default:** `unix://var/run/docker.sock`  
**Example:** `-e DOCKER_SOCKETS="unix://var/run/docker.sock tcp://192.168.1.100:2376"`  

Allows you to define a list of docker sockets. If defined, it does not include the local socket by default.

### Docker TLS
**Type:** Boolean  
**Command Line:**  `-t, --docker-tls`  
**Environment Variable:** `DOCKER_TLS`  
**Default:** `False`  
**Example:** `-e DOCKER_TLS=true`  

Enables docker TLS secure client connections by certificate

Example cert structure:
```
   /etc/docker/certs.d/         <-- Certificate directory
    └── localhost:5000          <-- socket as defined in config after //
       ├── client.cert          <-- Client certificate
       ├── client.key           <-- Client key
       └── ca.crt               <-- CA certificate
```
and would be mounted something like
```
-v /etc/docker/certs.d:/etc/docker/certs.d
```

### Docker TLS Verify
**Type:** Boolean  
**Command Line:**  `-T, --docker-tls-verify`  
**Environment Variable:** `DOCKER_TLS_VERIFY`  
**Default:** `False`  
**Example:** `-e DOCKER_TLS_VERIFY=true`  

Enables verifying docker TLS secure client connections by certificate  

### Interval
**Type:** Integer  
**Command Line:**  `-i, --interval`  
**Environment Variable:** `INTERVAL`
**Default**: `300`  
**Example:** `-e INTERVAL=300`  

The interval in seconds between checking for updates. There is a hard-coded 30 second minimum. Anything lower than that will set to 30. 

### Cron
**Type:** String  
**Command Line:**  `-C, --cron`  
**Environment Variable:** `CRON`  
**Default**: `None`  
**Example:** `-e CRON="*/5 * * * *"`   

The schedule defined when to check for updates. If not defined, runs at interval.

### Log Level
**Type:** String - Choice  
**Command Line:**  `-l, --log-level`  
**Environment Variable:** `LOG_LEVEL`  
**Choices:**
* debug
* info
* warn
* error
* critical

**Default:** `info`  
**Example:** `-e LOG_LEVEL=info`  

Sets your logging verbosity level.

### Self Update
**Type:** Boolean  
**Command Line:**  `-u, --self-update`  
**Environment Variable:** `SELF_UPDATE`  
**Default:** `False`  
**Example:** `-e SELF_UPDATE=true`  

Let ouroboros update itself in addition to your other containers. Self updates require the the container to be named either ouroboros or ouroboros-updated and will alternate the former names for updates. Self updates will wipe update counters for notifications.

### Run Once
**Type:** Boolean - Interrupting  
**Command Line:**  `-o, --run-once`  
**Environment Variable:** `RUN_ONCE`  
**Default:** `False`  

Ouroboros will only do a single pass of all container checks, and then exit. This is a great way to granularly control scheduling with an outside scheduler like cron. If during the single pass ouroboros has to self-update, it will do another full pass after updating itself to ensure that all containers were checked.

### Dry Run
**Type:** Boolean  
**Command Line:**  `-A, --dry-run` 
**Environment Variable:** `DRY_RUN`  
**Default:** `False`  

Ouroboros will only simulate container updates, but will not perform the actions. Best for testing configurations, seeing what updates **would** be applied. Best used with `run-once`.

## Docker Specifics

### Monitor
**Type:** List - Space separated  
**Command Line:**  `-m, --monitor`  
**Environment Variable:** `MONITOR`  
**Default:** `All`  
**Example:** `-e MONITOR="nginx telegraf portainer"`  

Define a list of containers you would like to monitor instead of all containers. If defined, labels then ignore take precedence. If a container is listed that does not match the name of currently running containers, it will be ignored.

### Ignore
**Type:** List - Space separated  
**Command Line:**  `-n, --ignore`  
**Environment Variable:** `IGNORE`  
**Default:** `None`  
**Example:** `-e IGNORE="mariadb influxdb mongo postgres"`  

Define a list of containers you would like to ignore updates for. If a container is listed that does not match the name of currently running containers, it will be ignored.


### Label Enable
**Type:** Boolean  
**Command Line:**  `-k, --label-enable`  
**Environment Variable:** `LABEL_ENABLE`  
**Default:** `False`  
**Example:** `-e LABEL_ENABLE=true`  

If a container has a `com.ouroboros.enable` label, only watch it if it is set to `true`. Supersedes monitor/ignore in precedence. This can be achieved by setting `LABEL com.ouroboros.enable="false"` in your Dockerfile or passing the label during creation of the container with `docker run -d --label=com.ouroboros.enable="false" person/image:tag`. See [Labels](#labels) for a list of all available labels.

### Labels Only
**Type:** Boolean  
**Command Line:**  `-m, --labels-only`  
**Environment Variable:** `LABELS_ONLY`  
**Default:** `False`  
**Example:** `-m LABELS_ONLY=true`  

This flag allows a more strict control over ouroboros's updates. If the container does not have a `com.ouroboros.enable` label, it will be ignored completely. If label_enable is not true, this option is ignored. See [Labels](#labels) for a list of all available labels.

### Cleanup
**Type:** Boolean  
**Command Line:**  `-c, --cleanup`  
**Environment Variable:** `CLEANUP`  
**Default:** `False`  
**Example:** `-e CLEANUP=true`  

Remove the old images after updating. If you have multiple containers using the same image, it will ensure all containers are updated before successfully removing the image.

### Latest
**Type:** Boolean  
**Command Line:**  `-L, --latest`  
**Environment Variable:** `LATEST`  
**Default:** `False`  
**Example:** `-e LATEST=true`  

Pull the `:latest` tags and update all containers to it, regardless of the current tag the container is running as.

### Repository User
**Type:** String  
**Command Line:**  `-r, --repo-user`  
**Environment Variable:** `REPO_USER`  
**Default:** `None`  
**Example:** `-e REPO_USER=johndoe1970`  

Define a username for repository authentication. Will be ignored without defining a repository password.

### Repository Password
**Type:** String  
**Command Line:**  `-R, --repo-pass`  
**Environment Variable:** `REPO_PASS`  
**Default:** `None`  
**Example:** `-e REPO_PASS=0791eodnhoj`  

Define a password for repository authentication. Will be ignored without defining a repository username.

## Data Export Options

### Data Export
**Type:** String - Choice  
**Command Line:**  `-D, --data-export`  
**Environment Variable:** `DATA_EXPORT`  
**Choices:**
* prometheus
* influxdb  

**Example:** 
`-e DATA_EXPORT=influxdb`  

Enables exporting metric data to your chosen application. If you choose Prometheus, and self update, **host** port bindings will not persist. It is recommended to reverse proxy your exporter through nginx. If you choose influxdb, you must set influx database at minimum.

### Prometheus
**Full Example:**
```
-e PROMETHEUS_ADDR=0.0.0.0 \
-e PROMETHEUS_PORT=3579  
```
See [Prometheus Metrics](#prometheus-metrics) for more insight

### Prometheus Address
**Type:** String  
**Command Line:**  `-a, --prometheus-addr`  
**Environment Variable:** `PROMETHEUS_ADDR`  
**Default:** `127.0.0.1`  
**Example:** `-e PROMETHEUS_ADDR=0.0.0.0`  

Bind address for prometheus exporter to attach to.

### Prometheus Port
**Type:** Integer  
**Command Line:**  `-p, --prometheus-port`  
**Environment Variable:** `PROMETHEUS_PORT`  
**Default:** `8000`  
**Example:** `-e PROMETHEUS_PORT=3579`  

Bind port for prometheus exporter to attach to.

### Influxdb
**Full Example:**
```
-e INFLUX_URL=influx.domain.tld \
-e INFLUX_PORT=443 \
-e INFLUX_USERNAME=ouroboros \
-e INFLUX_PASSWORD=soroboruo \
-e INFLUX_DATABASE=ouroboros \
-e INFLUX_SSL=true \
-e INFLUX_VERIFY_SSL=true
```

### Influx URL
**Type:** String  
**Command Line:**  `-I, --influx-url`  
**Environment Variable:** `INFLUX_URL`  
**Default:** `127.0.0.1`  
**Example:** `-e INFLUX_URL=192.168.1.100`  

URL influxdb is listening on

### Influx Port
**Type:** Integer  
**Command Line:**  `-P, --influx-port`  
**Environment Variable:** `INFLUX_PORT`  
**Default:** `8086`  
**Example:** `-e INFLUX_PORT=8123`  

Port influxdb is listening on

### Influx Username
**Type:** String  
**Command Line:**  `-U, --influx-username`  
**Environment Variable:** `INFLUX_USERNAME`  
**Default:** `root`  
**Example:** `-e INFLUX_USERNAME=ouroboros`  

Username to authenticate to influx with.

### Influx Password
**Type:** String  
**Command Line:**  `-x, --influx-password`  
**Environment Variable:** `INFLUX_PASSWORD`  
**Default:** `root`  
**Example:** `-e INFLUX_PASSWORD=soroboruo`  

Password to authenticate to influx with.

### Influx Database
**Type:** String  
**Command Line:**  `-X, --influx-database`  
**Environment Variable:** `INFLUX_DATABASE`  
**Default:** `None`  
**Example:** `-e INFLUX_DATABASE=ouroboros`  

Database to save data points to. Not set and required if using influxdb.

### Influx SSL
**Type:** Boolean  
**Command Line:**  `-s, --influx-ssl`  
**Environment Variable:** `INFLUX_SSL`  
**Default:** `None`  
**Example:** `-e INFLUX_SSL=true`  

Use SSL when connecting to influxdb. (mainly used when influxdb is behind an https reverse proxy)

### Influx Verify SSL
**Type:** Boolean  
**Command Line:**  `-V, --influx-verify-ssl`  
**Environment Variable:** `INFLUX_VERIFY_SSL`  
**Default:** `False`  
**Example:** `-e INFLUX_VERIFY_SSL=true`  

Verify the ssl certificate used when connecting to influxdb. only used in conjunction with influx ssl

## Notifications

**Type:** List - Space separated  
**Command Line:**  `-N, --notifiers`  
**Environment Variable:** `NOTIFIERS`  
**Default:** `None`  
**Example:** `-n NOTIFIERS="mailtos://myUsername:myPassword@gmail.com?to=receivingAddress@gmail.com jsons://webhook.site/something"`  

Ouroboros uses [apprise](https://github.com/caronc/apprise) to support a large variety of notification platforms.

Notifications are sent per socket, per run. The notification contains the socket, the containers updated since last start, and a list of containers updated this pass with from/to SHA.

More information can be found in the [notifications wiki](https://github.com/pyouroboros/ouroboros/wiki/Notifications).

### Skip Startup Notifications

**Type:** Boolean  
**Command Line:**  `--skip-startup-notifications`  
**Environment Variable:** `SKIP_STARTUP_NOTIFICATIONS`  
**Default:** `False`  
**Example:** `--skip-startup-notifications`  

Doesn't send startup notifications if arg is passed.

## Labels
* `com.ouroboros.enable`

  Example: `com.ouroboros.enable=true`  
  Used for [Label Enable](#label-enable)
* `com.ouroboros.depends_on`

  Example: `com.ouroboros.depends_on=postgres,redis`
  To simulate docker-compose depends_on functionality. If defined on any containers (by container _name_), all referenced containers will be turned off before the update process and then turned back on after the update process.
* `com.ouroboros.hard_depends_on`

    Example: `com.ouroboros.hard_depends_on=postgres,redis`  
  Same as above with the exception that it will recreate the container instead of just turning it off/ton
* `com.ouroboros.stop_signal`

  Example: `com.ouroboros.stop_signal=SIGKILL`  
  Define a stop signal to send to the container instead of SIGKILL. Can be string or int.

## Examples

### Configuration

### .env File

You can provide a docker env file to supplement a config file with all the above listed arguments by utilizing the supported environment variables.

```bash
docker run -d --name ouroboros \
  --env-file ouroboros.env \
  -v /var/run/docker.sock:/var/run/docker.sock \
  pyouroboros/ouroboros
```

Sample `ouroboros.env`:

```
DOCKER_SOCKETS=tcp://localhost:2375
INTERVAL=60
MONITOR="container_1 container_2"
```

### Scenarios

### Prometheus metrics

Ouroboros keeps track of containers being updated and how many are being monitored. Said metrics are exported using [prometheus](https://prometheus.io/). You can also bind the http server to a different interface for systems using multiple networks. `--prometheus-port` and `--prometheus-addr` can run independently of each other without issue.

> Prometheus exporter will not be reachable by default inside of a container. You will need to intentionally bind to `0.0.0.0` for docker network interfaces to be able to reach the exporter the host network. This was done intentionally for security reasons.

#### Bind Address & Port

> Bind Address default is `127.0.0.1`

> Port Default is `8000`

```bash
docker run -d --name ouroboros \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -p 8000:8000 pyouroboros/ouroboros --data-export prometheus --prometheus-addr 0.0.0.0
 ```

You should then be able to see the metrics at http://localhost:8000/

**Example text from endpoint:**

```
# HELP containers_updated_total Count of containers updated
# TYPE containers_updated_total counter
containers_updated_total{container="all"} 2.0
containers_updated_total{container="alpine"} 1.0
containers_updated_total{container="busybox"} 1.0
# TYPE containers_updated_created gauge
containers_updated_created{container="all"} 1542152615.625264
containers_updated_created{container="alpine"} 1542152615.6252713
containers_updated_created{container="busybox"} 1542152627.7476819
# HELP containers_being_monitored Count of containers being monitored
# TYPE containers_being_monitored gauge
containers_being_monitored 2.0
```