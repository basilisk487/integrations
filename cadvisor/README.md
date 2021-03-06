
## Exporting cAdvisor Stats to Wavefront

cAdvisor supports exporting stats to Wavefront (https://wavefront.com). Below are the additional command line arguments needed to tell cAdvisor to export stats to your local Wavefront proxy.

### What You'll Need
1. A Wavefront account.
2. A Wavefront proxy installed on your network.

Set the storage driver to Wavefront.

` -storage_driver=wavefront`

### Additional Arguments

#### Required: The *ip:port* of your Wavefront proxy

The proxy should be installed on your network and accessible by the Docker host machine.

 `-storage_driver_wf_proxy_host=ip:port`

#### Required: Source

Source tag value for metrics collected by this cAdvisor instance. We usually recommend setting this to the hostname of the docker host (not the container hostname). On some environments
(AWS ECS for example) it is not possible or inconvenient (sans ugly hacks) to retrieve the hostname of the Docker host when launching cAdvisor. As a solution to this you can use a docker label
as part of the source name. The storage driver will automatically suffix the name with cAdvisor's container ID (hostname). For example, if you are running a service in Docker Compose called "web",
the source name will become `web-xxxxxxx` where `xxxxxxx` is cAdvisor's container ID. We recommend choosing a label that does not have high cardinality across hosts.
This is the best compromise between flexibility in source naming while still maintaining reasonable cardinality of sources.

To include a Docker label in the source name (example):

`-storage_driver_wf_source={com.amazonaws.ecs.task-definition-family}`

In this example, assume the value of the `com.amazon.ecs.task-definition-family` label for a container is "web" and the hostname of the cAdvisor container is "cadvisor01". The source name for metrics emitted from this container would be `web-cadvisor01`

If you just want to use a static value for the source (example):

`-storage_driver_wf_source=mydockerhost`

or, if you're starting containers from the command line using Docker run command, set the source to the hostname of the Docker host machine:

`-storage_driver_wf_source=$(hostname)`

#### Optional: Metric Prefix

It will default to "cadvisor." if not set.

`-storage_driver_wf_prefix=cadvisor.`

#### Optional: Flush Interval

This will default to 10 seconds if not set.

`-storage_driver_wf_interval=10`

#### Optional: Additional Tags

This a string of additional tags that will be applied to every metric.

`-storage_driver_wf_add_tags="az=us-west1 env=dev"`

#### Optional: Taggify Labels

Defaults to true. Determines whether docker labels should be added as point tags to metrics.

`-storage_driver_wf_taggify_labels=true`

#### Optional: Label Filter

`-storage_driver_wf_label_filter=com.docker.compose.service,com.docker.compose.project`

Pass a comma separated list of docker label keys that should be added as tags to metrics.

## Examples

### Docker Run

```shell
sudo docker run \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:rw \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --publish=8080:8080 \
  --detach=true \
  --name=cadvisor \
  wavefronthq/cadvisor:latest \
  -storage_driver=wavefront \
  -storage_driver_wf_source=$(hostname) \
  -storage_driver_wf_proxy_host=YOUR_PROXY_HOST:2878
```

### Docker Compose

```yaml
cadvisor:
  container_name: cadvisor
  image: wavefronthq/cadvisor:latest
  command: -storage_driver=wavefront -storage_driver_wf_source=$(hostname) -storage_driver_wf_proxy_host=YOUR_PROXY_HOST:2878
  restart: always
  ports:
    - "8080:8080"
  volumes:
    - /:/rootfs:ro
    - /var/run:/var/run:rw
    - /sys:/sys:ro
    - /var/lib/docker/:/var/lib/docker:ro
```
