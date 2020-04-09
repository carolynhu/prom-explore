# Setup and Configure Prometheus
## Installation

https://prometheus.io/docs/introduction/first_steps/

For Mac user, download darwin-amd64.tar.gz, untar and cd to prom repo, run `./prometheus`, which will initialize a prometheus server in your local
and create a /data folder, all the scraped data will be stored here by default.

## Configure Prometheus

Prometheus Monitoring requires a system configuration usually in the form a `.yaml` file. For example, here is
a sample `prometheus.yaml` file to scrape from our servers running at `localhost:9888`, `localhost:9988` and `localhost:9989`

```
global:
  scrape_interval: 10s

  external_labels:
    monitor: 'media_search'

scrape_configs:
  - job_name: 'media_search'

    scrape_interval: 10s

    static_configs:
      - targets: ['localhost:9888', 'localhost:9988', 'localhost:9989']
```

## Starting Prometheus
Having successfully downloaded Prometheus and setup your config.yaml file, you should now be able to run

```
prometheus --config.file=config.yaml
```

## Viewing Prometheus output
You should now be able to navigate to http://localhost:9090/

