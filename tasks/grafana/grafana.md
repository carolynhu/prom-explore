# Grafana

## Setup

- Docker run grafana

```bash
docker run -d -p 3000:3000 grafana/grafana
```

- Go to http://localhost:3000/login

By default, login username and password are: admin.

- Add Prometheus as Data Source

Click `Add data source` -> Select `Prometheus`

- Dashboard

Grafana dashboard backed by JSON file. You can use an existing dashboard by downloading and importing those JSON file. 