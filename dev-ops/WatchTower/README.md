## 1. Architecture Diagram
This stack implements a full observability pipeline using Prometheus for scraping/alerting and Grafana for visualization.

mermaid js
graph TD
    subgraph "Application Stack"
        O[order-service :3001]
        T[tracking-service :3002]
        N[notification-service :3003]
    end

    subgraph "Observability Stack"
        P[Prometheus :9090]
        G[Grafana :3000]
        A[Alert Rules]
    end

    P -->|Scrapes /metrics 15s| O
    P -->|Scrapes /metrics 15s| T
    P -->|Scrapes /metrics 15s| N
    P -->|Evaluates| A
    G -->|Queries Data| P

## 2. Setup Instructions
Copy the example environment file: cp .env.example .env

Start the stack: docker compose up --build -d

Verify Services:

Prometheus Targets: http://localhost:9090/targets (All should be UP)

Grafana Dashboard: http://localhost:3000

## 3. Dashboard Walkthrough
image outputs for the graph

![Grafana Dashboard](./images/Grafana Dashbord.png/Prometheus.png/Prometheus alert.png)
 
HTTP Request Rate: Tracks requests per second to visualize load on the logistics services in realtime.  

Error Rate (5xx): Monitors server-side failures to detect system existing and new errors.

Service Health Status: Visualizes the reachability of /health endpoints, if they are UP or DOWN.

## 4. Alert Testing Validation
Testing was performed to ensure the prometheus/alerts.yml rules trigger as expected.

ServiceDown (Critical): Tested by running docker compose stop order-service. The alert fired after the container was unreachable for 1 minute.

HighErrorRate (Warning): Tested by redirecting service traffic to an invalid port to force internal connection failures.and after 5 mins the error rate spiked high

ServiceNotScraping (Warning): Tested by cutting off the network connection (docker compose stop tracking-service <reyla-network> tracking-service) creating a scenario where the container is running but Prometheus cannot reach it for 2 minutes.

## 5. Structured Logging & Commands
All services utilize the json-file logging driver.  

Command 1: View live logs from all services

Bash
docker compose logs -f

JSON Output: 

tracking-service-1  | {"level":"info","service":"tracking-service","msg":"Listening on port 3002"}
notification-service-1  | {"level":"info","service":"notification-service","msg":"Listening on port 3003"}
order-service-1         | {"level":"info","service":"order-service","msg":"Listening on port 3001"}


Command 2: Filter for errors in a specific service

Bash
docker compose logs tracking-service | grep -i "error"

JSON Output:

tracking-service-1 | {"level":"error","service":"tracking-service","msg":"Failed to connect to DB"}

## And right here is my WatchTower!!!
