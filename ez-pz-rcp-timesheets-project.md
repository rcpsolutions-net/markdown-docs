# RCP Timesheets v1.0 — Docker Dev Setup

*Prompted and edited by Lawrence Ham*

**RCP Timesheets** is a microservices-based system built with Docker, organized as a monorepo.

## Project Structure

```
rcp-timesheets/
├── docker-compose.yml
├── frontend/          # Angular: Dockerfile, package.json, angular.json, src
├── backend/
│   ├── api/           # each service: Dockerfile, package.json, src
│   ├── ocr-service/
│   └── other-microservices/
└── docker-config/     # mongo-init.js
```

## Prerequisites

- **Docker Desktop** (Docker Engine + Compose).
- Angular and Fastify projects with their `package.json` deps in place (`nodemon` for Fastify, `ng serve` for Angular).

## Prepare Your Projects

**Fastify (backend)** — every Fastify service needs `start` + `dev` scripts:

```json
"scripts": { "start": "node src/index.js", "dev": "nodemon src/index.js" }
```

Listen on **`0.0.0.0`** so other containers and the host can reach it: `fastify.listen({ port: 3000, host: '0.0.0.0' }, ...)`.

**Angular (frontend)** — allow host access:

```json
"scripts": { "start": "ng serve --host 0.0.0.0 --disable-host-check" }
```

Proxy API calls to the Fastify gateway on the host (`proxy.conf.json`, referenced in `angular.json` "serve" options):

```json
{ "/api": { "target": "http://localhost:3000", "secure": false, "changeOrigin": true } }
```

> The `API_BASE_URL` in `docker-compose.yml` is for *internal container* use. Browser-side requests go through the `localhost` proxy.

## Run Everything

From the project root:

```bash
docker compose up --build
```

- `--build`: (re)build images for services with a `build` context — use on first run or after a `Dockerfile` change.
- `up`: create and start all containers defined in `docker-compose.yml`.

## Hot-Reloading

- **Fastify/Node:** source dir mounted as a Docker **volume** + **`nodemon`** inside the container → saving on the host restarts the server.
- **Angular:** frontend volume + **`ng serve`** in the container → browser hot-reload.

## Accessing Services

- Frontend: `http://localhost:4200`
- API Gateway: `http://localhost:3000`
- OCR Service (if exposed): `http://localhost:3001`
- MongoDB: `mongodb://user:password@localhost:27017/mydatabase`
- RabbitMQ Management UI: `http://localhost:15672` (login `user`/`password`)

## Stopping

- **`Ctrl+C`** in the terminal running `docker compose up`.
- `docker compose down` — stop + remove containers, keep volumes.
- `docker compose down --volumes` — also delete named volumes (resets MongoDB data).

## Considerations & Best Practices

- **Env vars** for all config (DB URLs, API keys); Compose's `environment` is fine for dev.
- **Internal comms:** services on the same network (e.g. `app_network`) reach each other **by service name** (e.g. host `mongodb`).
- **`depends_on` ≠ readiness:** it only orders container *start*, not app readiness. Use healthchecks / wait-for-it in prod.
- **Volume trick:** the anonymous volume `/app/node_modules` prevents the host's (possibly incompatible) `node_modules` from clobbering the container's `npm install`.
- **Dev vs prod:** this setup is dev-only. For prod: multi-stage Dockerfiles, no source mounts, a separate `docker-compose.prod.yml` or an orchestrator (Kubernetes), stronger secret management.
- **Debugging:** `docker compose logs -f <service_name>`; interactive debugging may require attaching your IDE (VS Code) to a Node process inside the container.
