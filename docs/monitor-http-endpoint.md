---
title: Monitor HTTP Endpoints
id: monitor-http-endpoints
---

:::note
This feature is available [0.11.3](https://github.com/SigNoz/signoz/releases/tag/v0.11.3) onwards. If you are on a lower version, please follow these instructions to upgrade

[Docker](/docs/operate/docker-standalone/#upgrade)

[Kubernetes](/docs/operate/kubernetes/#upgrade)
:::

<p>&nbsp;</p>

With SigNoz, you can monitor the health of the HTTP endpoints and set up an alert in case of HTTP endpoints failure status codes.

## Steps

* Add the httpcheck receiver to `otel-collector-config.yaml` 
  ```yaml {2-10}
  receivers:
    httpcheck:
      endpoint: http://example.com
      method: GET
      collection_interval: 10s
  ```
  The HTTP Check Receiver can be used for synthetic checks against HTTP endpoints. This receiver will make a request to the specified endpoint using the configured method. This scraper generates a metric labelled for each HTTP response status class with a value of 1 if the status code matches the class.

* Next, we will modify the pipeline to include the receiver we have enabled above.
    ```yaml {4}
    service:
        ....
        metrics:
          receivers: [otlp, httpcheck]
          processors: [batch]
          exporters: [clickhousemetricswrite]
    ```

* We can restart the otel collector container so that new changes are applied and see the metrics generated for synthetic checks.

* This receiver creates a metric name `httpcheck_status` with value 1 if the check resulted in status_code matching the status_class, otherwise 0. For more info on the additional metrics and attributes available, please read the documentation [here](https://github.com/open-telemetry/opentelemetry-collector-contrib/blob/main/receiver/httpcheckreceiver/documentation.md).

### Monitoring Health of Multiple Endpoints

If you want to monitor the health of multiple endpoints with this receiver, then you need to add one instance of receiver for each endpoint since it currently supports only one endpoint. Following is the sample config that monitors two endpoints.

  ```yaml {2-10}
  receivers:
    httpcheck/example:
      endpoint: http://example.com
      method: GET
      collection_interval: 10s
    httpcheck/my-app:
      endpoint: http://my-app.com
      method: GET
      collection_interval: 1s
  ...
  ...

    service:
        ....
        metrics:
          receivers: [otlp, httpcheck/example, httpcheck/my-app]
          processors: [batch]
          exporters: [clickhousemetricswrite]
  ```
