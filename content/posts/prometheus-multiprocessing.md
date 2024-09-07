+++
title = 'Monitoring multi-process Python apps with Prometheus'
date = 2021-04-27T00:00:00+03:00
+++
## Introduction

Recently I run into a problem with collecting metrics from python app running in muliple processes (using uvicorn). When you have these multi-process applications, any of the multiple workers-processes can respond to prometheus's scraping request. Each worker then responds with a value for a metric that it knows of.

Official Prometheus Python client has [multiprocess mode](https://github.com/prometheus/client_python/blob/master/README.md#multiprocess-mode-gunicorn). In FastAPI we are using package `prometheus-fastapi-instrumentator` built on top of official client. Here is the part of the source code of `instrumentator`:

```python
if "prometheus_multiproc_dir" in os.environ:
    pmd = os.environ["prometheus_multiproc_dir"]
    if os.path.isdir(pmd):
        registry = CollectorRegistry()
        multiprocess.MultiProcessCollector(registry)
    else:
        raise ValueError(
            f"Env var prometheus_multiproc_dir='{pmd}' not a directory."
        )
else:
    registry = REGISTRY
```

From the link above:

> The `prometheus_multiproc_dir` environment variable must be set to a directory that the client library can use for metrics. This directory must be wiped between Gunicorn runs (before startup is recommended).

So to solve the problem we set `prometheus_multiproc_dir` environment variable.
