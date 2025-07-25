---
layout: default
title: Logs
parent: Configuring OpenSearch
nav_order: 120
redirect_from:
  - /opensearch/logs/
  - /monitoring-your-cluster/logs/
---

# Logs

The OpenSearch logs include valuable information for monitoring cluster operations and troubleshooting issues. The location of the logs differs based on the installation type:

- On Docker, OpenSearch writes most logs to the console and stores the remainder in `opensearch/logs/`. The tarball installation also uses `opensearch/logs/`.
- On most Linux installations, OpenSearch writes logs to `/var/log/opensearch/`.

Logs are available as `.log` (plain text) and `.json` files. Permissions for the OpenSearch logs are `-rw-r--r--` by default, meaning that any user account on the node can read them. You can change this behavior _for each log type_ in `log4j2.properties` using the `filePermissions` option. For example, you might add `appender.rolling.filePermissions = rw-r-----` to change permissions for the JSON server log. For details, see the [Log4j 2 documentation](https://logging.apache.org/log4j/2.x/manual/appenders.html#RollingFileAppender).


## Application logs

For its application logs, OpenSearch uses [Apache Log4j 2](https://logging.apache.org/log4j/2.x/) and its built-in log levels (from least to most severe). The following table describes the logging settings.

| Setting | Data type | Description |
| :--- | :--- | :--- |
| `logger.org.opensearch.discovery` | String | Loggers accept Log4j2’s built-in log levels: `OFF`, `FATAL`, `ERROR`, `WARN`, `INFO`, `DEBUG`, and `TRACE`. Default is `INFO`. |

Rather than changing the default log level (`logger.level`), you change the log level for individual OpenSearch modules:

```json
PUT /_cluster/settings
{
  "persistent" : {
    "logger.org.opensearch.index.reindex" : "DEBUG"
  }
}
```
{% include copy-curl.html %}

The easiest way to identify modules is not from the logs, which abbreviate the path (for example, `o.o.i.r`), but from the [OpenSearch source code](https://github.com/opensearch-project/opensearch/tree/master/server/src/main/java/org/opensearch).
{: .tip }

After this sample change, OpenSearch emits much more detailed logs during reindex operations:

```plaintext
[2019-10-18T16:52:51,184][DEBUG][o.o.i.r.TransportReindexAction] [node1] [1626]: starting
[2019-10-18T16:52:51,186][DEBUG][o.o.i.r.TransportReindexAction] [node1] executing initial scroll against [some-index]
[2019-10-18T16:52:51,291][DEBUG][o.o.i.r.TransportReindexAction] [node1] scroll returned [3] documents with a scroll id of [DXF1Z==]
[2019-10-18T16:52:51,292][DEBUG][o.o.i.r.TransportReindexAction] [node1] [1626]: got scroll response with [3] hits
[2019-10-18T16:52:51,294][DEBUG][o.o.i.r.WorkerBulkByScrollTaskState] [node1] [1626]: preparing bulk request for [0s]
[2019-10-18T16:52:51,297][DEBUG][o.o.i.r.TransportReindexAction] [node1] [1626]: preparing bulk request
[2019-10-18T16:52:51,299][DEBUG][o.o.i.r.TransportReindexAction] [node1] [1626]: sending [3] entry, [222b] bulk request
[2019-10-18T16:52:51,310][INFO ][o.e.c.m.MetaDataMappingService] [node1] [some-new-index/R-j3adc6QTmEAEb-eAie9g] create_mapping [_doc]
[2019-10-18T16:52:51,383][DEBUG][o.o.i.r.TransportReindexAction] [node1] [1626]: got scroll response with [0] hits
[2019-10-18T16:52:51,384][DEBUG][o.o.i.r.WorkerBulkByScrollTaskState] [node1] [1626]: preparing bulk request for [0s]
[2019-10-18T16:52:51,385][DEBUG][o.o.i.r.TransportReindexAction] [node1] [1626]: preparing bulk request
[2019-10-18T16:52:51,386][DEBUG][o.o.i.r.TransportReindexAction] [node1] [1626]: finishing without any catastrophic failures
[2019-10-18T16:52:51,395][DEBUG][o.o.i.r.TransportReindexAction] [node1] Freed [1] contexts
```

The DEBUG and TRACE levels are extremely verbose. If you enable either one to troubleshoot a problem, disable it after you finish.

There are other ways to change log levels:

1. Add lines to `opensearch.yml`:

   ```yml
   logger.org.opensearch.index.reindex: debug
   ```
   {% include copy.html %}

   Modifying `opensearch.yml` makes the most sense if you want to reuse your logging configuration across multiple clusters or debug startup issues with a single node.

2. Modify `log4j2.properties`:

   ```properties
   # Define a new logger with unique ID of reindex
   logger.reindex.name = org.opensearch.index.reindex
   # Set the log level for that ID
   logger.reindex.level = debug
   ```
   {% include copy.html %}

   This approach is extremely flexible but requires familiarity with the [Log4j 2 property file syntax](https://logging.apache.org/log4j/2.x/manual/configuration.html#Properties). In general, the other options offer a simpler configuration experience.

   If you examine the default `log4j2.properties` file in the configuration directory, you can see a few OpenSearch-specific variables:

   ```properties
   appender.console.layout.pattern = [%d{ISO8601}][%-5p][%-25c{1.}] [%node_name]%marker %m%n
   appender.rolling_old.fileName = ${sys:opensearch.logs.base_path}${sys:file.separator}${sys:opensearch.logs.cluster_name}.log
   ```

   - `${sys:opensearch.logs.base_path}` is the directory for logs (for example, `/var/log/opensearch/`).
   - `${sys:opensearch.logs.cluster_name}` is the name of the cluster.
   - `${sys:opensearch.logs.node_name}` is the name of the node.
   - `[%node_name]` is the name of the node.

## Search request slow logs

New in version 2.12, OpenSearch offers request-level slow logs for search. These logs rely on thresholds to define what qualifies as "slow." All requests which exceed the threshold are logged.

Search request slow logs are enabled dynamically through the [Cluster Settings API]({{site.url}}{{site.baseurl}}/api-reference/cluster-api/cluster-settings/). Unlike shard slow logs, search request slow log thresholds are configured for total request took time. By default, logs are disabled (all thresholds are set to `-1`).

```json
PUT /_cluster/settings
{
"persistent" : {
      "cluster.search.request.slowlog.level" : "TRACE",
      "cluster.search.request.slowlog.threshold.warn": "10s",
      "cluster.search.request.slowlog.threshold.info": "5s",
      "cluster.search.request.slowlog.threshold.debug": "2s",
      "cluster.search.request.slowlog.threshold.trace": "10ms"
}
}
```
{% include copy-curl.html %}

A line from `opensearch_index_search_slowlog.log` might look like this:

```plaintext
[2023-10-30T15:47:42,630][TRACE][c.s.r.slowlog] [runTask-0] took[80.8ms], took_millis[80], phase_took_millis[{expand=0, query=39, fetch=22}], total_hits[4 hits], search_type[QUERY_THEN_FETCH], shards[{total: 10, successful: 10, skipped: 0, failed: 0}], source[{"query":{"match_all":{"boost":1.0}}}], id[]
```

Search request slow logs can consume considerable disk space and affect performance if you set low threshold values. Consider enabling them temporarily for troubleshooting or performance tuning. To disable search request slow logs, return all thresholds to `-1`.
{: .important}

## Shard slow logs

OpenSearch has two *shard slow logs*, logs that help you identify performance issues: the search slow log and the indexing slow log.

These logs rely on thresholds to define what qualifies as a "slow" search or "slow" indexing operation. For example, you might decide that a query is slow if it takes more than 15 seconds to complete. Unlike application logs, which you configure for modules, you configure slow logs for indexes. By default, both logs are disabled (all thresholds are set to `-1`).

Unlike search request slow logs, shard slow log thresholds are configured for individual shard took time.

```json
GET <some-index>/_settings?include_defaults=true
{
  "indexing": {
    "slowlog": {
      "reformat": "true",
      "threshold": {
        "index": {
          "warn": "-1",
          "trace": "-1",
          "debug": "-1",
          "info": "-1"
        }
      },
      "source": "1000",
      "level": "TRACE"
    }
  },
  "search": {
    "slowlog": {
      "level": "TRACE",
      "threshold": {
        "fetch": {
          "warn": "-1",
          "trace": "-1",
          "debug": "-1",
          "info": "-1"
        },
        "query": {
          "warn": "-1",
          "trace": "-1",
          "debug": "-1",
          "info": "-1"
        }
      }
    }
  }
}
```
{% include copy-curl.html %}

To enable these logs, increase one or more thresholds:

```json
PUT <some-index>/_settings
{
  "indexing": {
    "slowlog": {
      "threshold": {
        "index": {
          "warn": "15s",
          "trace": "750ms",
          "debug": "3s",
          "info": "10s"
        }
      },
      "source": "500",
      "level": "INFO"
    }
  }
}
```
{% include copy-curl.html %}

In this example, OpenSearch logs indexing operations that take 15 seconds or longer at the WARN level and operations that take between 10 and 14.*x* seconds at the INFO level. If you set a threshold to 0 seconds, OpenSearch logs all operations, which can be useful for testing whether slow logs are indeed enabled.

- `reformat` specifies whether to log the document `_source` field as a single line (`true`) or let it span multiple lines (`false`).
- `source` is the number of characters of the document `_source` field to log.
- `level` is the minimum log level to include.

A line from `opensearch_index_indexing_slowlog.log` might look like this:

```plaintext
node1 | [2019-10-24T19:48:51,012][WARN][i.i.s.index] [node1] [some-index/i86iF5kyTyy-PS8zrdDeAA] took[3.4ms], took_millis[3], type[_doc], id[1], routing[], source[{"title":"Your Name", "Director":"Makoto Shinkai"}]
```

Shard slow logs can consume considerable disk space and affect performance if you set low threshold values. Consider enabling them temporarily for troubleshooting or performance tuning. To disable shard slow logs, return all thresholds to `-1`.

## Task logs

OpenSearch can log CPU time and memory utilization for the top N memory-expensive search tasks when task resource consumers are enabled. By default, task resource consumers will log the top 10 search tasks at 60 second intervals. These values can be configured in `opensearch.yml`.

Task logging is enabled dynamically through the [Cluster Settings API]({{site.url}}{{site.baseurl}}/api-reference/cluster-api/cluster-settings/):

```json
PUT _cluster/settings
{
  "persistent" : {
    "task_resource_consumers.enabled" : "true"
  }
}
```
{% include copy-curl.html %}

Enabling task resource consumers can have an impact on search latency.
{:.tip}

Once enabled, logs will be written to `logs/opensearch_task_detailslog.json` and `logs/opensearch_task_detailslog.log`.

To configure the logging interval and the number of search tasks logged, add the following lines to `opensearch.yml`:

```yaml
# Number of expensive search tasks to log
cluster.task.consumers.top_n.size:100

# Logging interval
cluster.task.consumers.top_n.frequency:30s
```
{% include copy.html %}

## Deprecation logs

Deprecation logs record when clients make deprecated API calls to your cluster. These logs can help you identify and fix issues prior to upgrading to a new major version. By default, OpenSearch logs deprecated API calls at the WARN level, which works well for almost all use cases. If desired, configure `logger.deprecation.level` using `_cluster/settings`, `opensearch.yml`, or `log4j2.properties`.
