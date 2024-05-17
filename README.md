# Grafana-Prometheus Monitoring Stack

## Overview

The **Grafana-Prometheus Monitoring Stack** project is designed to create a robust and scalable monitoring system for Docker environments using Docker Swarm. This project leverages Grafana, Prometheus, and Node Exporter to provide comprehensive monitoring and visualization of metrics.

## Components and Functionality

1. **Grafana**
   - **Image:** `grafana/grafana:8.5.3-ubuntu`
   - **Ports:** Exposes port `3000` to allow access to the Grafana dashboard.
   - **Volumes:**
     - `grafana-data:/var/lib/grafana`: Persistent storage for Grafana data.
     - `grafana-configs:/etc/grafana`: Configuration files for Grafana.

   Grafana is a powerful open-source platform for monitoring and observability. It provides a customizable dashboard for visualizing metrics and data collected from various sources.

2. **Prometheus**
   - **Image:** `prom/prometheus:v2.36.0`
   - **Ports:** Exposes port `9090` to access the Prometheus server.
   - **Volumes:**
     - `prom-data:/prometheus`: Persistent storage for Prometheus data.
     - `prom-configs:/etc/prometheus`: Configuration files for Prometheus.

   Prometheus is an open-source monitoring and alerting toolkit designed for reliability and scalability. It collects and stores metrics as time series data, providing a powerful query language (PromQL) for analyzing these metrics.

3. **Node Exporter**
   - **Image:** `prom/node-exporter:v1.3.1`
   - **Ports:** Exposes port `9100` to allow Prometheus to scrape metrics.
   - **Volumes:**
     - `/proc:/host/proc:ro`: Mounts the host's `/proc` directory in read-only mode.
     - `/sys:/host/sys:ro`: Mounts the host's `/sys` directory in read-only mode.
     - `/:/rootfs:ro`: Mounts the root filesystem in read-only mode.
   - **Command:** Customized to exclude specific mount points for accurate filesystem metrics collection.

   Node Exporter is a Prometheus exporter for hardware and OS metrics exposed by *nix kernels, providing detailed insights into system performance and health.

## Deployment

The project consists of two key configuration files:

1. **docker-compose.yml**
   - Defines services for Grafana, Prometheus, and Node Exporter.
   - Sets up volumes for persistent data and configuration storage.

2. **node-exporter.yml**
   - Focuses specifically on configuring the Node Exporter service.
   - Uses the same image and configuration parameters as the Node Exporter service defined in `docker-compose.yml`.

## Deployment Steps

To deploy Grafana, Prometheus, and Node Exporter, follow these steps:

1. **Deploy the stack:**
   ```bash
   docker stack deploy -c docker-compose.yml grafana-prometheus-stack
   ```
2. **Add Prometheus as a data source in Grafana:**
   - Open Grafana at `http://<your-docker-host>:3000`
   - Navigate to Configuration > Data Sources
   - Add a new data source with the following details:
     - **Name:** Prometheus
     - **URL:** `http://prometheus:9090`

3. **Configure Prometheus to scrape Node Exporter:**
   - Edit the Prometheus configuration file:
     ```bash
     nano /var/lib/docker/volumes/monitoring_prom-configs/_data/prometheus.yml
     ```
   - Add the following lines to the `scrape_configs:` section:
     ```yaml
     - job_name: 'node-exporter'
       static_configs:
         - targets: ['node-exporter:9100']
     ```

4. **Reload Prometheus configuration:**
   ```bash
   docker ps | grep prometheus | awk '{print $1}' | xargs docker kill -s SIGHUP
   ```
5. **Import a Grafana dashboard:**
   - Navigate to the Grafana UI
   - Go to Dashboards > Manage > Import
   - Use the following dashboard ID: [1860](https://grafana.com/grafana/dashboards/1860)

## Adding More Servers to Prometheus

To add more servers to Prometheus, follow these steps:

1. **Install Node Exporter on each additional server:**
   ```bash
   docker stack deploy -c node-exporter.yml node-exporter
    ```
2. **Update Prometheus configuration:**
   - Edit the Prometheus configuration file:
     ```bash
     nano /var/lib/docker/volumes/monitoring_prom-configs/_data/prometheus.yml
     ```
   - Add the new server addresses to the `- targets: ['node-exporter:9100']` list under the `- job_name: 'node-exporter'` section, for example:
     ```yaml
     - job_name: 'node-exporter'
       static_configs:
         - targets: ['node-exporter:9100', 'new-server:9100']
     ```

3. **Reload Prometheus configuration:**
   ```bash
   docker ps | grep prometheus | awk '{print $1}' | xargs docker kill -s SIGHUP
   ```
   
