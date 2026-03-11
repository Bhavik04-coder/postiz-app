# Docker Architecture - Postiz

This document provides a visualization and detailed explanation of the Docker setup for the Postiz project. The project uses a multi-container architecture orchestrated via Docker Compose, separating the application logic, databases, and workflow engine.

## Service Visualization

The following diagram illustrates how the services interact within the Docker environment.

```mermaid
graph TD
    subgraph "External Access"
        User((User)) -->|Port 4007| Postiz[Postiz App Container]
    end

    subgraph "Postiz App Container (Internal Monolith)"
        Postiz --> Nginx[Nginx Reverse Proxy]
        Nginx --> Frontend[Next.js Frontend\nPort 4200]
        Nginx --> Backend[NestJS Backend\nPort 3000]
        PM2[PM2 Process Manager] --> Frontend
        PM2 --> Backend
        PM2 --> Orchestrator[Orchestrator\nTemporal Workers]
    end

    subgraph "Data Layer"
        Backend -->|Postgres Protocol| PG[(Postiz Postgres)]
        Backend -->|Redis Protocol| Redis[(Postiz Redis)]
        Orchestrator -->|Postgres Protocol| PG
    end

    subgraph "Workflow Engine (Temporal Stack)"
        Orchestrator -->|gRPC| Temporal[Temporal Server]
        Temporal -->|Postgres Protocol| TemporalPG[(Temporal Postgres)]
        Temporal -->|Elasticsearch| TemporalES[(Temporal ES)]
        TemporalUI[Temporal UI\nPort 8080] --> Temporal
    end

    subgraph "Monitoring & Dev Tools"
        Postiz -.->|UDP/HTTP| Spotlight[Sentry Spotlight]
        DevUser((Developer)) -->|Port 8081| PGAdmin[pgAdmin]
        DevUser -->|Port 5540| RedisInsight[Redis Insight]
        PGAdmin -.-> PG
        RedisInsight -.-> Redis
    end

    classDef app fill:#f9f,stroke:#333,stroke-width:2px;
    classDef db fill:#9cf,stroke:#333,stroke-width:2px;
    classDef workflow fill:#ffc,stroke:#333,stroke-width:2px;
    class Postiz app;
    class PG,Redis,TemporalPG,TemporalES db;
    class Temporal,TemporalUI,TemporalAdmin workflow;
```

---

## Service Descriptions

### 1. Main Application (`postiz`)
This is the core container that bundles the frontend, backend, and orchestrator.
- **Image**: `ghcr.io/gitroomhq/postiz-app:latest`
- **Internal Components**:
    - **Nginx**: Acts as the entry point, routing requests to the frontend or the `/api` backend.
    - **NestJS Backend**: The main API server.
    - **Next.js Frontend**: The web interface.
    - **Orchestrator**: Temporal workers that handle background jobs (social media posting, scheduling).
- **Process Management**: Uses **PM2** to keep all three Node.js services running concurrently.

### 2. Databases & Caching
- **Postiz Postgres** (`postiz-postgres`): The primary relational database for user data, posts, and configurations.
- **Postiz Redis** (`postiz-redis`): Used for session management, response caching, and internal task queuing.

### 3. Workflow Engine (Temporal Stack)
Postiz leverages **Temporal** for reliable, long-running background tasks (like retrying failed social media posts).
- **Temporal Server**: The core orchestrator for workflows and activities.
- **Temporal Postgres**: dedicated database for Temporal internal state.
- **Temporal Elasticsearch**: Provides advanced search and visibility for workflows.
- **Temporal UI**: A dashboard to monitor and debug workflows (accessible at `http://localhost:8080`).

### 4. Development & Monitoring Tools
- **Spotlight**: A Sentry tool for local debugging of errors and performance.
- **pgAdmin**: Web-based PostgreSQL administration interface.
- **Redis Insight**: Visualization tool for Redis data.

---

## Networking

The architecture uses two primary Docker networks:

1.  **`postiz-network`**: Connects the main application with its primary database (`postiz-postgres`) and cache (`postiz-redis`).
2.  **`temporal-network`**: A dedicated network for the Temporal stack to communicate internally and with the `postiz` app's orchestrator.

## Configuration (Environment Variables)

The services are configured heavily via environment variables in the `docker-compose.yaml`. Key variables include:
- `MAIN_URL`: The public-facing URL.
- `DATABASE_URL`: Connection string for the primary Postgres.
- `REDIS_URL`: Connection string for Redis.
- `TEMPORAL_ADDRESS`: The address where the app can find the Temporal server (`temporal:7233`).

---

## How to Run

### Production-like (bundled)
```bash
docker compose up -d
```

### Development (Infrastructure only)
If you are developing locally and want to run the app via `pnpm run dev` but keep the databases in Docker:
```bash
docker compose -f docker-compose.dev.yaml up -d
```
