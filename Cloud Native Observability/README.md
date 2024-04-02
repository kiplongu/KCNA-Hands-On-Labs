# Cloud Native Observability - Study Tips
For the KCNA exam you will need an understanding of the following -

The 3 pillars of Cloud Native Observability

Where and why you would use each Pillar, i.e. Logs, Metrics, Traces

The different types of Metrics, i.e. Gauges, Counters, Meters, Histograms

The relation between Alerting and the 3 pillars of Cloud Native Observability

Be aware of kubectl top

Have an understanding of options such as OpenTracing/OpenTelemetry and that these operate at the Application layer


Cloud Native Observability - Further Study
Kubernetes kubectl provides a simple means of gathering resource utilisation with the command kubectl top

Try this out in the lab!

Also, for your awareness and for your KCNA study, you should also have an understanding of OpenTracing/OpenTelemetry and in particular, that these operate at an Application layer. Take some time to familiarise yourself with these.

OpenTracing/OpenTelemetry is an open-source project under the Cloud Native Computing Foundation (CNCF) that provides APIs and instrumentation for distributed tracing. It offers a consistent, expressive, vendor-neutral API for distributed tracing, which is critical for understanding and monitoring the performance of microservices-based applications.

Introduction
Purpose: Aims to provide a standardised and easy way to trace requests across distributed systems, helping developers to monitor and troubleshoot complex microservices architectures.

Vendor-Neutral API: It offers a way to collect trace data without being tied to any specific tracing backend (like Jaeger or Zipkin). This flexibility allows for easy integration and migration between different tracing systems.

How OpenTracing/OpenTelemetry Operates at the Application Layer
Instrumentation: Developers add code to their applications to create and manage spans. These spans represent individual operations or requests, capturing essential data such as start and end times, tags, and logs.

Span Context Propagation: As requests move through different services and processes, the span context is propagated, ensuring a continuous trace across service boundaries. This is often handled via HTTP headers or messaging metadata.

Data Reporting: The instrumented applications report span data to a tracing backend. This data collection is designed to be low-overhead to minimize the impact on application performance.

Analysis and Visualisation: The backend (like Jaeger or Zipkin) aggregates this data, providing a visual representation of the traces. This visualisation aids in performance analysis, debugging, and optimisation.

Resources for Further Learning
Official Documentation: OpenTracing Website (the project is recently archived but still has valuable information) provides guides, API documentation, and a conceptual overview of distributed tracing: https://opentracing.io/

OpenTelemetry Documentation: OpenTelemetry provides guides on moving from OpenTracing to OpenTelemetry: https://opentelemetry.io/docs/migration/opentracing/

By integrating OpenTracing/OpenTelemetry into applications, developers gain critical insights into the performance and behaviour of their systems, making it easier to identify and solve issues in a distributed environment.

