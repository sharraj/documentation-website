---
layout: default
title: Pipelines
has_children: true
nav_order: 10
redirect_from:
  - /data-prepper/pipelines/
  - /clients/data-prepper/pipelines/
---

# Pipelines

The following image illustrates how a pipeline works. 

<img src="{{site.url}}{{site.baseurl}}/images/data-prepper-pipeline.png" alt="Data Prepper pipeline">{: .img-fluid} (FIXME : Rename prepper to Processor).

Pipeline configuration is defined in a YAML file. Each pipeline is a combination of a source, a buffer (optional), a processor chain (optional), and one or more sinks. For example:

```yml
simple-sample-pipeline:
  workers: 2 # the number of workers
  delay: 5000 # in milliseconds, how long workers wait between read attempts
  source:
    random:
  buffer:
    bounded_blocking:
      buffer_size: 1024 # max number of records the buffer accepts
      batch_size: 256 # max number of records the buffer drains after each read
  processor:
    - string_converter:
        upper_case: true
  sink:
    - stdout:
```

- Sources define where your data comes from. In this case, the source is a random UUID generator (`random`).

- Buffers store data as it passes through the pipeline.

  By default, Data Prepper uses its one and only buffer, the `bounded_blocking` buffer, so you can omit this section unless you developed a custom buffer or need to tune the buffer settings.

- Processors perform some action on your data: filter, transform, enrich, etc.

  You can have multiple processors, which run sequentially from top to bottom, not in parallel. The `string_converter` processor transform the strings by making them uppercase.

- Sinks define where your data goes. In this case, the sink is stdout.

Starting from Data Prepper 2.0, you can define pipelines across multiple configuration YAML files, where each file contains the configuration for one or more pipelines. This gives you more freedom to organize and chain complex pipeline configurations. For Data Prepper to load your pipeline configuration properly, place your configuration YAML files in the `pipelines` folder under your application's home directory (e.g. `/usr/share/data-prepper`).
{: .note } [FIXME: Need Examples here ][FIXME: This capability is not available in OpenSearch Ingestion Service)]

FIXME : Workers, buffers, delay are not configurable for OpenSearch Ingestion Service.


## Examples

This section provides some pipeline examples that you can use to start creating your own pipelines. For more pipeline configurations, select from the following options for each component:

- [Buffers]({{site.url}}{{site.baseurl}}/data-prepper/pipelines/configuration/buffers/buffers/)
- [Processors]({{site.url}}{{site.baseurl}}/data-prepper/pipelines/configuration/processors/processors/)
- [Sinks]({{site.url}}{{site.baseurl}}/data-prepper/pipelines/configuration/sinks/sinks/)
- [Sources]({{site.url}}{{site.baseurl}}/data-prepper/pipelines/configuration/sources/sources/)

The Data Prepper repository has several [sample applications](https://github.com/opensearch-project/data-prepper/tree/main/examples) to help you get started. FIXME Similarily OpenSearch Ingestion service has many inbuilt blueprints to help you get started.

### Log ingestion pipeline

The following example `pipeline.yaml` file with SSL and basic authentication enabled for the `http-source` demonstrates how to use the HTTP Source and Grok Prepper (FIXME Processor --> Remove prepper everywhere and make them processor) plugins to process unstructured log data:

```yaml
log-pipeline:
  source:
    http:
      ssl_certificate_file: "/full/path/to/certfile.crt"
      ssl_key_file: "/full/path/to/keyfile.key"
      authentication:
        http_basic:
          username: "myuser"
          password: "mys3cret"
  processor:
    - grok:
        match:
          # This will match logs with a "log" key against the COMMONAPACHELOG pattern (ex: { "log": "actual apache log..." } )
          # You should change this to match what your logs look like. See the grok documenation to get started.
          log: [ "%{COMMONAPACHELOG}" ]
  sink:
    - opensearch:
        hosts: [ "https://localhost:9200" ]
        # Change to your credentials
        username: "admin"
        password: "admin"
        # Add a certificate file if you are accessing an OpenSearch cluster with a self-signed certificate  
        #cert: /path/to/cert
        # If you are connecting to an Amazon OpenSearch Service domain without
        # Fine-Grained Access Control, enable these settings. Comment out the
        # username and password above.
        #aws :
         #sigv4: true
         #region: us-east-1
        # Since we are grok matching for apache logs, it makes sense to send them to an OpenSearch index named apache_logs.
        # You should change this to correspond with how your OpenSearch indices are set up.
        index: apache_logs
```

This example uses weak security. We strongly recommend securing all plugins which open external ports in production environments.
{: .note}


### Metrics pipeline

Data Prepper supports metrics ingestion using Open Telemetry (OTEL). It currently supports the following metric types:

* Gauge
* Sum
* Summary
* Histogram

Other types are not supported. Data Prepper drops all other types, including Exponential Histogram and Summary (FIXME). Additionally, Data Prepper does not support Scope instrumentation.

To set up a metrics pipeline:

```yml
metrics-pipeline:
  source:
    otel_metrics_source:
  processor:
    - otel_metrics_processor: FIXME (otel_metrics)
  sink:
    - opensearch:
      hosts: ["https://localhost:9200"]
      username: admin
      password: admin
      #aws :
       #sigv4: true
       #region: us-east-1
```

### S3 log ingestion pipeline

The following example demonstrates how to use the S3Source and Grok Processor plugins to process unstructured log data from [Amazon Simple Storage Service](https://aws.amazon.com/s3/) (Amazon S3). This example uses application load balancer logs. As the application load balancer writes logs to S3, S3 creates notifications in Amazon SQS. Data Prepper monitors those notifications and reads the S3 objects to get the log data and process it.

```yml
log-pipeline:
  source:
    s3:
      notification_type: "sqs"
      compression: "gzip"
      codec:
        newline:
      sqs:
        queue_url: "https://sqs.us-east-1.amazonaws.com/12345678910/ApplicationLoadBalancer"
      aws:
        sigv4: true
        region: "us-east-1"
        sts_role_arn: "arn:aws:iam::12345678910:role/OpenSearch Ingestion"

  processor:
    - grok:
        match:
          message: ["%{DATA:type} %{TIMESTAMP_ISO8601:time} %{DATA:elb} %{DATA:client} %{DATA:target} %{BASE10NUM:request_processing_time} %{DATA:target_processing_time} %{BASE10NUM:response_processing_time} %{BASE10NUM:elb_status_code} %{DATA:target_status_code} %{BASE10NUM:received_bytes} %{BASE10NUM:sent_bytes} \"%{DATA:request}\" \"%{DATA:user_agent}\" %{DATA:ssl_cipher} %{DATA:ssl_protocol} %{DATA:target_group_arn} \"%{DATA:trace_id}\" \"%{DATA:domain_name}\" \"%{DATA:chosen_cert_arn}\" %{DATA:matched_rule_priority} %{TIMESTAMP_ISO8601:request_creation_time} \"%{DATA:actions_executed}\" \"%{DATA:redirect_url}\" \"%{DATA:error_reason}\" \"%{DATA:target_list}\" \"%{DATA:target_status_code_list}\" \"%{DATA:classification}\" \"%{DATA:classification_reason}"]
    - grok:
        match:
          request: ["(%{NOTSPACE:http_method})? (%{NOTSPACE:http_uri})? (%{NOTSPACE:http_version})?"]
    - grok:
        match:
          http_uri: ["(%{WORD:protocol})?(://)?(%{IPORHOST:domain})?(:)?(%{INT:http_port})?(%{GREEDYDATA:request_uri})?"]
    - date:
        from_time_received: true
        destination: "@timestamp"


  sink:
    - opensearch:
        hosts: [ "https://localhost:9200" ]
        username: "admin"
        password: "admin"
        index: alb_logs
```

FIXME

# Pipeline Splitting

User can replicate incoming events and send them to sub-pipelines. This will help for use cases where same event needs to be processed by different processor chain. After processing in sub-pipelines, events can be written to same index or different index based in sink configuration of sub-pipelines. A maximum of 10 such sub-pipelines are supported.

Example : 

FIXME

# Pipeline Chaining



