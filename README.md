# Liberty Framework
This public repository contains all the files needed for the deployment

Liberty Framework is a no-code development platform built with cutting-edge technologies such as React, Node.js, and PostgreSQL. This framework includes essential management tools like Traefik, Rundeck, and PgAdmin, allowing you to create web applications in no time, without the need for coding skills.

Liberty Framework also features a native connector for JD Edwards, enabling efficient management of access rights, task segregation, and Oracle license compliance. With its modern architecture, it provides a comprehensive solution to design robust applications in record time.

- Go to [Liberty Framework](https://github.com/fblettner/liberty-public) for details about our framework
- Go to [Documentation](https://docs.nomana-it.fr/) for all products documentation, knowledge base...
- Go to [Repository](https://docs.nomana-it.fr/) for all others products available as public resources
- Go to [Demo](https://liberty.nomana-it.fr/) for testing our framework


# Liberty Framework Services Documentation 🚀

This document provides an overview of the functionality and configuration of the services within the **Liberty Framework**, including **Node.js**, **PostgreSQL**, **pgAdmin**, **Rundeck**, **OIDC**, and **Filebrowser**. These services are integrated with **Traefik** as a reverse proxy, enabling both HTTP and HTTPS access with automated routing. 

---

## 1. **Node.js Service (`liberty-node`)** 🟢

- **Image**: `ghcr.io/fblettner/liberty-node:latest`
- **Command**: Runs the Node.js app (`app.js`) on port `3002`.
- **Security Options**: 
  - 🔒 `label:disable`: Disables SELinux labels.
  - ⚙️ `cap_drop`: Removes unnecessary Linux capabilities like `MKNOD` and `AUDIT_WRITE`.
- **Networks**: Connected to the `liberty-network`.
- **Working Directory**: `/opt/liberty`
- **Depends on**: PostgreSQL (`pg`) service.
- **Traefik Configuration**:
  - 🌐 **API Routing**: HTTP and HTTPS routing for `/api` using `PathPrefix`.
  - 📡 **Socket Routing**: HTTP and HTTPS routing for `/socket` and `/socket.io`.
  - ⚛️ **React Application**: Handles HTTP and HTTPS routing for the React app with a middleware for error pages.
  - 🚀 **Compression**: `compress-middleware` applied to several routes for better performance.
  - 🔌 **Port Configuration**: Node.js runs on port `3002`.

---

## 2. **PostgreSQL Service (`liberty-pg`)** 🐘

- **Image**: `ghcr.io/fblettner/liberty-pg:latest`
- **Command**: Runs the PostgreSQL server with optimized settings for performance:
  - `shared_buffers=2GB`
  - `track_activity_query_size=1MB`
  - `work_mem=256MB`
  - `maintenance_work_mem=128MB`
  - Other configurations to optimize WAL size, checkpoint timing, and costs.
- **Volumes**: Data stored in the `pg-data` volume.
- **Networks**: Connected to `liberty-network`.
- **Traefik Configuration**:
  - 🛠️ **TCP Router**: Routes PostgreSQL traffic via `db` entry point.
  - 🔌 **Port**: Exposed on port `5432`.

---

## 3. **pgAdmin Service (`liberty-pgadmin`)** 🖥️

- **Image**: `ghcr.io/fblettner/liberty-pgadmin:latest`
- **User**: Root privileges enabled.
- **Volumes**: pgAdmin data stored in the `pgadmin-data` volume.
- **Environment**: Sets the `SCRIPT_NAME=/pgadmin` for pgAdmin web access.
- **Depends on**: PostgreSQL (`pg`).
- **Networks**: Connected to `liberty-network`.
- **Traefik Configuration**:
  - 🌐 **HTTP Router**: Routes requests for `/pgadmin`.
  - 🔌 **Port**: Exposed on port `3003`.

---

## 4. **Rundeck Service (`liberty-rundeck`)** 🛠️

- **Image**: `ghcr.io/fblettner/liberty-rundeck:latest`
- **Security Options**:
  - 🔒 Disables SELinux labels.
  - ⚙️ Drops capabilities `MKNOD` and `AUDIT_WRITE`.
- **Volumes**: 
  - Data stored in the `rundeck-data` volume.
  - Configurations in `rundeck-config` and `talend-config`.
- **Depends on**: PostgreSQL (`pg`).
- **Networks**: Connected to `liberty-network`.
- **Traefik Configuration**:
  - 🌐 **Routing**: Handles HTTP and HTTPS requests for `/rundeck`.
  - ⚠️ **Error Pages Middleware**: Applied to both HTTP and HTTPS routes.
  - 🔌 **Port**: Exposed on port `4440`.

---

## 5. **OIDC Service (`liberty-keycloak`)** 🔐

- **Image**: `ghcr.io/fblettner/liberty-keycloak:latest`
- **Command**: Starts the Keycloak OIDC server with proxy headers and hostname settings.
- **Environment Variables**:
  - 🔄 `PROXY_ADDRESS_FORWARDING`: Enables proxy address forwarding.
  - 🌍 `KC_HOSTNAME_PATH` and `KC_HTTP_RELATIVE_PATH`: Configured to `/oidc`.
- **Depends on**: PostgreSQL (`pg`).
- **Networks**: Connected to `liberty-network`.
- **Traefik Configuration**:
  - 🌐 **HTTP and HTTPS Routing**: Routes `/oidc` requests.
  - 🔌 **Port**: OIDC runs on port `9000` (Keycloak internally uses port `8080`).
  - 🌍 **CORS Middleware**: Configures Cross-Origin Resource Sharing (CORS) for all origins and credentials.

---

## 6. **Filebrowser Service (`liberty-filebrowser`)** 📂

- **Image**: `ghcr.io/fblettner/liberty-filebrowser:latest`
- **Healthcheck**: Ensures service health by checking `/health` endpoint every 30 seconds.
- **Volumes**: 
  - Configuration in `fb-config` and data in `fb-data`.
  - Shares Rundeck, Talend, Traefik certificates, and configuration via other volumes.
- **Restart Policy**: Set to `unless-stopped`.
- **Networks**: Connected to `liberty-network`.
- **Traefik Configuration**:
  - 🌐 **Routing**: Routes HTTP requests to `/filebrowser`.
  - 🛠️ **Middleware**: Uses `stripprefix` to remove `/filebrowser` from the path for internal routing.
  - 🔌 **Port**: Exposed on port `80`.

---

## Volumes 🗃️

- **fb-config**: Stores Filebrowser configuration.
- **fb-data**: Stores Filebrowser data.
- **pg-data**: Stores PostgreSQL data.
- **pgadmin-data**: Stores pgAdmin data.
- **rundeck-data**: Stores Rundeck data.
- **rundeck-config**: Stores Rundeck configuration.
- **talend-config**: Stores Talend configuration.
- **nginx-config**: Stores Nginx configuration.
- **traefik-certs**: Stores Traefik certificates (external).
- **traefik-config**: Stores Traefik configuration (external).
- **shared-data**: Stores shared data (external).

---

## Networks 🌐

- **liberty-network**: External network for inter-service communication.

---

This configuration enables a scalable, containerized microservice architecture with **Node.js** for application logic, **PostgreSQL** for database management, **pgAdmin** for database administration, **Rundeck** for automation, **Keycloak OIDC** for authentication, and **Filebrowser** for file management. **Traefik** serves as the reverse proxy, handling routing and applying security middleware for all services.