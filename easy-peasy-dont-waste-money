# ✍️ RCP Timesheets v1.0

This document outlines the project structure for the **RCP Timesheets** application, a microservices-based system built with Docker.

---

## 📂 RCP-Timesheets Project Structure

The project is organized into a monorepo, with separate directories for the frontend, backend, and Docker configuration files.

```mermaid
graph TD
    A[rcp-timesheets]
    A --> B{docker-compose.yml}
    A --> C[frontend]
    A --> D[backend]
    A --> E[docker-config]
    
    subgraph frontend
        C --> F{Dockerfile}
        C --> G{package.json}
        C --> H{angular.json}
        C --> I[src]
    end
    
    subgraph backend
        D --> J[api]
        D --> K[ocr-service]
        D --> L[other-microservices]
        
        subgraph api
            J --> M{Dockerfile}
            J --> N{package.json}
            J --> O[src/]
        end
        
        subgraph ocr-service
            K --> P{Dockerfile}
            K --> Q{package.json}
            K --> R[src/]
        end
        
        subgraph other-microservices
            L --> S{Dockerfile}
            L --> T{package.json}
            L --> U[src/]
        end
    end
    
    subgraph docker-config
        E --> V{mongo-init.js}
    end
```


# Initial Setup & Development Workflow

This document outlines the steps for setting up and using the **RCP Timesheets** project for local development with Docker.

-----

## 💻 1.1. Prerequisites

Before you begin, ensure you have the following installed and configured:

  * **Docker Desktop**: This includes Docker Engine and Docker Compose.
  * **Project Dependencies**: Your Angular and Fastify projects should have their `package.json` files with all necessary `dependencies` and `devDependencies` (like `nodemon` for Fastify and `ng serve` for Angular).

-----

## 🛠️ 1.2. Prepare Your Projects

Configure your project's `package.json` files to run within the Docker environment.

### **Fastify (Backend)**

Make sure your `package.json` files for all Fastify services (e.g., `backend/api-gateway/package.json` and `backend/ocrmypdf-service/package.json`) have `start` and `dev` scripts.

```json
"scripts": {
  "start": "node src/index.js",
  "dev": "nodemon src/index.js"
}
```

Ensure your Fastify apps listen on **`0.0.0.0`** so they can be accessed from other containers and your host machine:

```javascript
fastify.listen({ port: 3000, host: '0.0.0.0' }, ...);
```

### **Angular (Frontend)**

In your `frontend/package.json`, set the `start` script to allow host access:

```json
"scripts": {
  "start": "ng serve --host 0.0.0.0 --disable-host-check"
}
```

For API proxying, configure `proxy.conf.json` to redirect API calls to the Fastify gateway running on your host:

```json
// proxy.conf.json (referenced in angular.json "serve" options)
{
  "/api": {
    "target": "http://localhost:3000",
    "secure": false,
    "changeOrigin": true
  }
}
```

> 💡 **Note**: The `API_BASE_URL` in `docker-compose.yml` is for internal container use. Client-side requests from your browser will use the `localhost` proxy.

-----

## 🚀 1.2. Run Everything

Navigate to your project's root directory (`your-project/`) and run the following command to build and start all services:

```bash
docker compose up --build
```

  * **`--build`**: This command instructs Docker Compose to build or rebuild images for services with a `build` context before starting them. Use it for the first run or when you've changed a `Dockerfile`.
  * **`up`**: This command creates and starts all containers defined in your `docker-compose.yml` file.

-----

## 🔄 1.4. Development with Hot-Reloading

This setup is optimized for a fluid development workflow.

  * **Fastify/Node.js**: By mapping your source code directory as a Docker **`volume`** and running **`nodemon`** inside the container, any changes you save on your host machine will automatically trigger the Fastify server to restart.
  * **Angular**: Similarly, the `volume` mapping for the frontend and the use of **`ng serve`** within the container enables automatic hot-reloading in your browser as you make changes.

-----

## 🌐 1.5. Accessing Your Services

Once everything is running, you can access the various services via your host machine:

  * **Frontend**: `http://localhost:4200`
  * **API Gateway**: `http://localhost:3000`
  * **OCR Service (if exposed)**: `http://localhost:3001`
  * **MongoDB**: `mongodb://user:password@localhost:27017/mydatabase`
  * **RabbitMQ Management UI**: `http://localhost:15672` (Login with `user`/`password`)

-----

## 🛑 1.6. Stopping Services

To stop your services, use the following commands in your terminal.

1.  To stop the running containers, press **`Ctrl+C`** in the terminal where `docker compose up` is running.

2.  To stop and remove containers (but keep volumes for data persistence):

    ```bash
    docker compose down
    ```

3.  To stop and remove containers and **delete named volumes** (useful for resetting your MongoDB data):

    ```bash
    docker compose down --volumes
    ```

-----

## ⚙️ 1.7. Important Considerations and Best Practices

### **Environment Variables**

Use environment variables for all configuration, such as database URLs and API keys. Docker Compose's `environment` section is ideal for this in a development environment.

### **Internal Communication**

Services within the same network (e.g., `app_network`) can communicate using their **service names as hostnames**. For example, your Fastify service can connect to the database using `mongodb` as the host.

### **`depends_on` vs. Application Readiness**

The `depends_on` setting in `docker-compose.yml` only ensures that a container **starts** before another, not that the application inside it is fully ready. For production, consider using "health checks" or "wait-for-it" scripts for a more robust startup process.

### **Volume Trick**

The anonymous volume `- /app/node_modules` is a crucial technique for Node.js development. It prevents your host machine's potentially incompatible `node_modules` from overwriting the dependencies installed inside the container, preserving the result of `npm install`.

### **Production vs. Development**

This setup is optimized for **development**. For a production environment, you should:

  * Use multi-stage Dockerfiles to create smaller images.
  * Avoid mounting source code volumes.
  * Use a separate `docker-compose.prod.yml` or an orchestrator like Kubernetes.
  * Implement stronger password and secret management.

### **Debugging**

To view logs, use `docker compose logs -f <service_name>`. For interactive debugging, you may need to configure your IDE (like VS Code) to attach to a Node.js process running inside a Docker container.
