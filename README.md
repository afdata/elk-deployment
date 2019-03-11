# Kubernetes ELK deployment

## Important notes

ES port have opened to realize direct connetions without `redis-logstash-indexer`

For example we can use Fluentd. A lightweight replacement for logstash. All advantages you can find here: <https://www.fluentd.org/why>

Also a lot of examples here: <https://github.com/fluent/fluentd/tree/master/example>
The number of settings for elasticsearch output: <https://github.com/uken/fluent-plugin-elasticsearch>

```bash
On host machine (compute node) has to start: "sysctl -w vm.max_map_count=262144"
```

  If kubernetes has version older than v1.8, for CronJob need to use:
  `batch/v2alpha1` If newer, have be `apiVersion: batch/v1beta1` .

  To check API version which have been installed:
  `kubectl api-versions`
  If `batch/v2alpha1` API is absent:
    `vi /etc/kubernetes/apiserver` and add `--runtime-config=batch/v2alpha1=true`.

To start filebeat need to add to `rbac.authorization.k8s.io/v1alpha1` in `KUBE_API_ARGS` part:

```bash
--authorization-mode=RBAC,ABAC
```

Restart `kube-apiserver` and check versions: `kubectl api-versions`

## Description for yaml files

- `namespace/elasticsearch-ns.yaml` - namespace deployment;
- `ingress/kibana.yaml` - deployment for kibana ingress, providing traefik configuration to backend `kibana-loadbalancer`;
- `ingress/logstash-api.yaml` - Ingress to have ability to connect to logstash API;
- `ingress/redis-indexer.yaml` - Ingress to open redis connectivity on port 6379 for external connections;
- `kibana.yaml` - Kibana deployment with service;
- `elasticsearch.yaml` - StatefulSet for elasticsearch pods and Services for port 9300 (Nodes connecting each other), port 9200 (Input connections);
- `logstash.yaml` - Logstash deployment with Services for lumberjack and API ports;
- `redis.yaml` - Redis pod deployment with service for port 6379;
- `curator.yaml` - Cronjob deployment for curator starts;
- `curator-job` - Job for one time starts of curator clean jobs;
- `congigs/es-config.yaml` - Config map for elasticsearch;
- `configs/es-role-mapping.yaml` - Config map for impementing auth roles in ES;
- `configs/es-users.yaml` - Config map for users on ES;
- `configs/curator-config.yaml` - Config map for curator;
- `configs/logstash-config.yaml` - Config map for indexer.conf;
- `configs/redis-config.yaml` - Config map for redis server;
- `configs/curator-config.yaml` - Config map for curator.

## Elasticsearch

### The main DB to store data and settings. If Users or roles have changed via config files, need to wait for deploying changes to all ES nodes

Or might be used `curl` query directly to ES.
Here described how to do it: <https://www.elastic.co/guide/en/elasticsearch/reference/6.6/security-api-put-role.html>

## Kibana

### Working as a WebUI. On last versions at first time after start can be notification about "Kibana server is not ready yet". After ES starts the Kibana will work as usual

## Logstash

### Collecting logs on input, filtrating them, pushing into DB

The work schema described here:
<https://www.bogotobogo.com/Hadoop/ELK/ELK_Logstash_Shipper_Logstash_Indexer_with_Redis_Broker.php>

With redis+logstash-indexer we unloading logstash shippers on servers where we collecting logging data.

To have an access for system logs need to start:

```bash
setfacl -m u:logstash:r /var/log/{syslog,auth.log}
```

## Curator

### Providing cronjobs for cleaning old indexes

We have possibility to set CronJob for scheduled starts and Job for one time cleaning of indexes.
All settings and filters for cleaning you can find in `configs/curator-config.yaml`
