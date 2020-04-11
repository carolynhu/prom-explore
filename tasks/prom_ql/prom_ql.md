# PromQL

## Time series data

every data point is a called a "sample"
- metrics
- timestamp
- value

```bash
<--------------- metric ---------------------><- timestamp -><-value->
http_request_total{status="200", method="GET"}@1434417560938 => 94355
```

metrics saved in tsdb as the form of `__name__=<metric name>`

## Metrics Types

- Counter
    - only inc()
    - naming convention: _total
    - examples:
      ```bash
      rate(http_requests_total[5m])
      topk(10, http_requests_total)
      ```
- Gauge
    - can both increase() and decrease()
    - system current status
    
- Histogram
- Summary

