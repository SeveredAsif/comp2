# Login Node + React with Observability (Dockerized)

This repository contains a login application with full observability stack:
- **backend**: Node.js + Express + PostgreSQL (JWT auth) with Prometheus metrics
- **frontend**: React (Vite) built and served by nginx
- **observability**: Prometheus + Grafana + Node Exporter + cAdvisor

## Quick Start (requires Docker)

```bash
# Start all services
docker compose up --build
```

## Services & Ports

| Service | Port | Description |
|---------|------|-------------|
| App (nginx) | 80 | Main application (frontend + API proxy) |
| Grafana | 3000 | Metrics dashboard |
| Prometheus | 9090 | Metrics storage & query |
| cAdvisor | 8080 | Container metrics |
| Backend /metrics | 4000 | Application metrics endpoint |

## Grafana Access

- URL: http://localhost:3000
- **Anonymous access enabled** (no login required)
- Optional login: `admin` / `grafana`
- Pre-configured dashboard: "Login App Observability Dashboard"

## What's Monitored

 **Container Metrics** (via cAdvisor):
- CPU usage per container
- Memory usage per container
- Network I/O

 **System Metrics** (via Node Exporter):
- Host CPU usage
- Host memory usage
- Disk I/O

 **Application Metrics** (via backend /metrics):
- HTTP request rate
- HTTP response time (p95)
- Request count by route and status code
- Node.js process metrics (event loop, heap, etc.)

 **Alerts** (configured in Prometheus):
- High CPU usage (> 70% for 1 min)
- High memory usage (> 80% for 2 min)
- Backend down (unhealthy for 30s)

## Viewing Metrics

1. **Grafana Dashboard**: http://localhost:3000
   - Auto-loaded: "Login App Observability Dashboard"
   - Shows CPU, memory, response time, request rate

2. **Prometheus**: http://localhost:9090
   - Query metrics directly
   - View alert status: http://localhost:9090/alerts

3. **Backend Metrics**: http://localhost:4000/metrics
   - Raw Prometheus format metrics (Can't access directly, prometheus collects it, because nginx exposes port 80 only)

## Alert Dispatcher (Bonus Script)

A Bash script that fetches active alerts from Prometheus API and logs them locally:

```bash
# Run once to check current alerts
bash alert_dispatcher.sh --once

# Run in continuous mode (checks every 30s)
bash alert_dispatcher.sh

# View alert history
cat alerts.log
```

**Features:**
- Fetches alerts from Prometheus `/api/v1/alerts` endpoint
- Categorizes alerts as FIRING or PENDING
- Timestamps all checks
- Logs to `alerts.log` file
- Supports continuous monitoring mode
- Environment variables: `PROMETHEUS_URL`, `LOG_FILE`, `CHECK_INTERVAL`

**Sample output:**
```
[2025-11-13 23:18:07] Alert Status: 0 firing, 2 pending
   PENDING: HighCPUUsage - Container login_backend using high CPU
   No firing alerts
```

---

## Project Explanation

This project demonstrates a **complete observability stack** for monitoring containerized applications locally. Built as a lightweight, production-ready setup using industry-standard tools.

### **What Was Built:**

1. **Full-Stack Application** 
   - Node.js/Express backend with PostgreSQL database
   - React (Vite) frontend
   - JWT-based authentication
   - Nginx reverse proxy for unified access

2. **Metrics Collection Infrastructure**
   - **Backend /metrics endpoint**: Exposes Prometheus-formatted metrics including HTTP request duration, request counts, and Node.js process metrics
   - **cAdvisor**: Collects container-level CPU, memory, network metrics
   - **Node Exporter**: Provides host system metrics (CPU, memory, disk)
   - **Prometheus**: Scrapes all metrics every 15s and stores time-series data

3. **Visualization & Alerting**
   - **Grafana dashboard**: Pre-configured with 7+ panels showing CPU, memory, response times, and request rates
   - **Alert rules**: CPU > 70%, Memory > 80%, Backend down
   - **Auto-provisioning**: Dashboard and datasource load automatically on startup

4. **Alert Notification System (Bonus)**
   - Bash script that polls Prometheus alerts API
   - Logs firing/pending alerts with timestamps
   - Simulates alert dispatch without requiring external services

### **Why This Matters:**

- **Early problem detection**: Catch CPU spikes, memory leaks, or service outages before users notice
- **Performance insights**: Track response times and optimize slow endpoints
- **Resource planning**: Monitor container resource usage to right-size deployments
- **Production-ready**: All components run in Docker, portable to any environment
- **Zero external dependencies**: Entire stack runs locally, no cloud services needed

### **Use Cases:**

- Local development with production-like monitoring
- Testing application behavior under load
- Learning observability best practices
- Foundation for production monitoring (add AlertManager for real notifications)

---

## CI/CD

The GitHub Actions workflow deploys to Azure VM on push to main.
Set repository secrets:
- `AZURE_VM_IP`
- `AZURE_VM_KEY` (SSH private key)
