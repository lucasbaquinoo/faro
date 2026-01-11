# Faro Observability Stack

A complete observability stack for frontend applications using Grafana Faro, Loki, Tempo, and Grafana Alloy. This setup enables collection, storage, and visualization of logs and traces from web applications.

## 📋 Overview

This project provides a Docker Compose-based observability stack that allows you to:
- **Collect** frontend telemetry data (logs, traces, measurements) via Grafana Faro
- **Process** the data using Grafana Alloy
- **Store** logs in Loki and traces in Tempo
- **Visualize** everything in Grafana dashboards

## 🏗️ Architecture

```
Frontend Application (with Faro SDK)
          ↓
    Grafana Alloy (Faro Receiver)
          ↓
    ┌─────┴─────┐
    ↓           ↓
  Loki       Tempo
  (Logs)    (Traces)
    ↓           ↓
    └─────┬─────┘
          ↓
       Grafana
    (Visualization)
```

## 🧩 Components

### 1. **Grafana Alloy**
- Acts as the Faro receiver endpoint
- Receives telemetry data from frontend applications via HTTP
- Routes logs to Loki and traces to Tempo
- Listens on port `12347` with CORS enabled for all origins

### 2. **Loki**
- Log aggregation system
- Stores logs in the filesystem (`/tmp/loki`)
- Uses TSDB store with filesystem backend
- Accessible on port `3100`

### 3. **Tempo**
- Distributed tracing backend
- Receives traces via OTLP (OpenTelemetry Protocol)
- Stores traces locally in `/tmp/tempo`
- Accessible on port `3200` (HTTP), `4317` (OTLP gRPC), `4318` (OTLP HTTP)

### 4. **Grafana**
- Visualization and dashboarding platform
- Pre-configured with Loki and Tempo as datasources
- Anonymous access enabled for easy local development
- Accessible on port `3001`

## 📦 Prerequisites

- **Docker** (version 20.10+)
- **Docker Compose** (version 2.0+)

## 🚀 Quick Start

### 1. Clone the Repository

```bash
git clone https://github.com/lucasbaquinoo/faro.git
cd faro
```

### 2. Start the Stack

```bash
docker compose up -d
```

This will start all services in detached mode. The first run will download the necessary Docker images.

### 3. Verify Services

Check that all services are running:

```bash
docker compose ps
```

You should see 4 services running:
- `faro-grafana-1`
- `faro-loki-1`
- `faro-tempo-1`
- `faro-alloy-1`

### 4. Access Grafana

Open your browser and navigate to:
```
http://localhost:3001
```

No login is required - anonymous access is enabled with Admin privileges.

## 🔌 Ports

| Service | Port | Purpose |
|---------|------|---------|
| Grafana | 3001 | Web UI |
| Loki | 3100 | Log ingestion API |
| Tempo | 3200 | Tempo HTTP API |
| Tempo | 4317 | OTLP gRPC receiver |
| Tempo | 4318 | OTLP HTTP receiver |
| Alloy | 12347 | Faro receiver endpoint |

## 📝 Configuration Files

### Alloy Configuration (`alloy/config.alloy`)
Configures the Faro receiver and defines how data flows to Loki and Tempo:
- Faro receiver listens on `0.0.0.0:12347`
- CORS allowed for all origins (`*`)
- Logs are sent to Loki
- Traces are sent to Tempo via OTLP

### Loki Configuration (`loki/loki.yaml`)
- Authentication disabled for simplicity
- Uses filesystem storage
- TSDB schema (v13) for efficient log storage
- Data stored in `/tmp/loki`

### Tempo Configuration (`tempo/tempo.yaml`)
- OTLP receivers enabled (gRPC and HTTP)
- Local filesystem storage
- Data stored in `/tmp/tempo`

### Grafana Datasources (`grafana/provisioning/datasources/ds.yaml`)
Pre-configured datasources:
- **Loki** (default): `http://loki:3100`
- **Tempo**: `http://tempo:3200`

## 🔧 Integrating with Your Frontend

To send data from your frontend application to this stack:

### 1. Install Faro Web SDK

```bash
npm install @grafana/faro-web-sdk
```

### 2. Initialize Faro in Your App

```javascript
import { initializeFaro } from '@grafana/faro-web-sdk';

initializeFaro({
  url: 'http://localhost:12347/collect',
  app: {
    name: 'your-app-name',
    version: '1.0.0',
    environment: 'development',
  },
});
```

### 3. View Data in Grafana

1. Open Grafana at `http://localhost:3001`
2. Go to "Explore"
3. Select "Loki" datasource to view logs
4. Select "Tempo" datasource to view traces

## 🛑 Stopping the Stack

To stop all services:

```bash
docker compose down
```

To stop and remove all data (volumes):

```bash
docker compose down -v
```

## 🔍 Troubleshooting

### Services not starting
```bash
# Check service logs
docker compose logs -f

# Check specific service
docker compose logs -f alloy
```

### Cannot connect to Faro endpoint
- Ensure Alloy is running: `docker compose ps alloy`
- Check if port 12347 is available: `netstat -an | grep 12347`
- Verify CORS settings in `alloy/config.alloy`

### No data in Grafana
- Check if your frontend is sending data to `http://localhost:12347/collect`
- Verify Alloy logs: `docker compose logs alloy`
- Ensure Loki and Tempo are running: `docker compose ps`

### Port conflicts
If any port is already in use, modify the port mappings in `docker-compose.yml`:
```yaml
ports:
  - "NEW_PORT:CONTAINER_PORT"
```

## 📚 Learn More

- [Grafana Faro Documentation](https://grafana.com/docs/grafana-cloud/monitor-applications/frontend-observability/)
- [Grafana Alloy Documentation](https://grafana.com/docs/alloy/latest/)
- [Loki Documentation](https://grafana.com/docs/loki/latest/)
- [Tempo Documentation](https://grafana.com/docs/tempo/latest/)
- [Grafana Documentation](https://grafana.com/docs/grafana/latest/)

## 📄 License

This project is open source and available under the MIT License.

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.
