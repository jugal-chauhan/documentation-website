---
layout: default
title: Stats 
parent: ML Commons APIs
nav_order: 110
---

# Stats

The Stats API provides basic statistics about ML Commons, such as the number of running tasks. To monitor machine learning workflows using more detailed time-series metrics, see [Monitoring machine learning workflows]({{site.url}}{{site.baseurl}}/monitoring-your-cluster/metrics/getting-started/#monitoring-machine-learning-workflows).
{: .note }

Gets statistics related to the number of tasks. 

## Endpoints

```json
GET /_plugins/_ml/stats
GET /_plugins/_ml/stats/<stat>
GET /_plugins/_ml/<nodeId>/stats/
GET /_plugins/_ml/<nodeId>/stats/<stat>
```

## Example request: Get all stats for all nodes

```json
GET /_plugins/_ml/stats
```
{% include copy-curl.html %}

## Example response

```json
{
  "zbduvgCCSOeu6cfbQhTpnQ" : {
    "ml_executing_task_count" : 0
  },
  "54xOe0w8Qjyze00UuLDfdA" : {
    "ml_executing_task_count" : 0
  },
  "UJiykI7bTKiCpR-rqLYHyw" : {
    "ml_executing_task_count" : 0
  },
  "zj2_NgIbTP-StNlGZJlxdg" : {
    "ml_executing_task_count" : 0
  },
  "jjqFrlW7QWmni1tRnb_7Dg" : {
    "ml_executing_task_count" : 0
  },
  "3pSSjl5PSVqzv5-hBdFqyA" : {
    "ml_executing_task_count" : 0
  },
  "A_IiqoloTDK01uZvCjREaA" : {
    "ml_executing_task_count" : 0
  }
}
```

## Example request: Get all stats for a specific node

```json
GET /_plugins/_ml/<nodeId>/stats/
```
{% include copy-curl.html %}

## Example request: Get a specified stat for a specific node 

```json
GET /_plugins/_ml/<nodeId>/stats/<stat>
```
{% include copy-curl.html %}

## Example request: Get a specified stat for all nodes

```json
GET /_plugins/_ml/stats/<stat>
```
{% include copy-curl.html %}




