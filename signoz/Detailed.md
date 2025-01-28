# SigNoz Setup and Configuration Guide

## Table of Contents
1. [Introduction](#introduction)
2. [Architecture Overview](#architecture-overview)
   - [Diagram](#diagram)
3. [Components](#components)
   - [SigNoz Components](#signoz-components)
   - [Dual OpenTelemetry Collector Setup](#dual-opentelemetry-collector-setup)
   - [Why Two Collectors](#why-two-collectors)
4. [Configuration](#configuration)
   - [Docker OTel Collector](#docker-otel-collector)
   - [Host OTel Collector](#host-otel-collector)
5. [Data Flow](#data-flow)
6. [Port Configurations](#port-configurations)
7. [Common Issues and Solutions](#common-issues-and-solutions)
8. [Best Practices](#best-practices)

## Introduction
SigNoz is an open-source observability platform that helps developers monitor their applications with features like traces, logs, and metrics. This guide covers the setup and configuration of SigNoz with a dual OpenTelemetry collector architecture for optimal data collection and processing.

## Architecture Overview
The setup consists of two OpenTelemetry collectors:
1. **Host OTel Collector**: Runs directly on the server to collect local telemetry data
2. **Docker OTel Collector**: Runs within the SigNoz Docker environment to process and store data

This dual-collector architecture provides:
- Better resource management
- Separation of concerns
- Improved reliability
- Flexible data routing options

### Diagram
![Signoz Diagram](./signoz_diagram.png)

## Components

### SigNoz Components
- **Frontend**: Web interface for visualization (Port 3301)
- **Query Service**: Handles queries and alert management
- **ClickHouse**: Main database for storing telemetry data
- **Otel Collecter**: Collects otel data from otel running inside server.
- **AlertManager**: Manages alert rules and notifications

### Dual OpenTelemetry Collector Setup
1. **Host OTel Collector**:
   - Runs as a system service
   - Collects data from local applications
   - Handles initial data processing
   - Forwards to Docker OTel Collector

2. **Docker OTel Collector**:
   - Runs in Docker environment
   - Processes data from Host Collector
   - Writes to ClickHouse
   - Handles data transformation

### Why Two Collectors?

The dual collector architecture was chosen due to several key challenges encountered with a single collector setup:

1. **Direct ClickHouse Connection Issues**:
   - When using a single collector on the server, direct connections to ClickHouse in Docker resulted in networking complications
   - Database connection errors.

2. **Resource Management**:
   - Single collector trying to both collect and process data for ClickHouse led to high resource usage
   - Server became unresponsive due to memory and CPU spikes
   - Data backlogs and processing delays occurred

3. **Data Pipeline Stability**:
   - Host OTel collector focuses on efficient data collection
   - Docker OTel collector handles proper connection between ClickHouse
   - Separation provides better stability and reliability

4. **Networking Simplicity**:
   - Host collector can efficiently collect local telemetry
   - Docker collector has native access to other SigNoz components
   - Avoids complex network configurations and cross-container communication issues

## Configuration

### Docker OTel Collector
[Refer to](./docker/clickhouse-setup/otel-collector-config.yaml)

### Host OTel Collector
[Refer to](./docker/clickhouse-setup/server-otel-config.yaml)

## Data Flow
1. Application sends telemetry data to Host OTel (4317/4318)
2. Host OTel forwards to Docker OTel (4327/4328)
3. Docker OTel writes to ClickHouse
4. SigNoz Query Service reads from ClickHouse
5. Frontend displays data

## Port Configurations

| Component | Port | Protocol | Purpose |
|-----------|------|----------|----------|
| Frontend | 3301 | HTTP | Web UI |
| ClickHouse | 9000 | TCP | Native Interface |
| ClickHouse | 8123 | HTTP | HTTP Interface |
| Host OTel | 4317 | gRPC | OTLP Receiver |
| Host OTel | 4318 | HTTP | OTLP Receiver |
| Docker OTel | 4327 | gRPC | OTLP Receiver |
| Docker OTel | 4328 | HTTP | OTLP Receiver |


### Common Issues and Solutions

1. **No Data in SigNoz**
   - Verify both collectors are running
   - Check port configurations
   - Ensure ClickHouse is accepting connections

2. **High Resource Usage**
   - Check batch processor settings
   - Verify no loops in data flow
   - Monitor ClickHouse performance

3. **Connection Failures**
   - Verify network connectivity
   - Check TLS settings
   - Ensure correct port mappings

## Best Practices

1. **Resource Management**
   - Configure appropriate batch sizes
   - Set memory limits for collectors
   - Monitor system resources

2. **Security**
   - Use TLS in production
   - Set up authentication
   - Restrict network access

3. **Maintenance**
   - Regular log rotation
   - Monitor disk usage
   - Backup ClickHouse data

4. **Scaling**
   - Monitor collector performance
   - Adjust batch settings as needed
   - Scale ClickHouse if required