
# Deploying nopCommerce Using Docker Compose on Linux: Step-by-Step with Data Persistence and Portainer

**By Z | Published: July 3, 2025**

---

## 🧭 Overview

In this guide, I walk through how I deployed **nopCommerce** using **Docker Compose** on a **Linux environment**, solved critical **data persistence** issues, and integrated **Portainer** for container management — without Docker Desktop.

---

## 🎯 Project Goal

- Use **Docker Compose** to set up nopCommerce and SQL Server containers
- Ensure **user data is retained** after container restarts or system reboots
- Bypass the nopCommerce installation screen on every restart
- Manage containers with a UI using **Portainer** (Docker Desktop isn’t supported on Linux)

---

## 🧱 Step-by-Step Setup

### ✅ Step 1: Initial Docker Compose File

```yaml
version: '3.4'

services:
  db:
    image: mcr.microsoft.com/mssql/server:2019-latest
    environment:
      SA_PASSWORD: "YourStrong!Passw0rd"
      ACCEPT_EULA: "Y"
    ports:
      - "1433:1433"

  nopcommerce:
    image: nopcommerceteam/nopcommerce:latest
    depends_on:
      - db
    ports:
      - "8080:80"
```

```bash
docker-compose up
```

![Installation Screen](sandbox:/mnt/data/Installation.jpg)
*Initial Installation Screen: Configure store and database*

---

### ⚠️ Step 2: First Critical Issue – Data Lost on Restart

After adding test users, I restarted the containers.

```bash
docker-compose down
docker-compose up
```

❌ **Issue**: nopCommerce showed the installation page again. All data lost.

---

### 🔍 Step 3: Root Cause – No Volume = No Persistence

By default, containers don't retain changes unless volumes are used.

---

### 💾 Step 4: Solution – Add Docker Volumes

```yaml
version: '3.4'

services:
  db:
    image: mcr.microsoft.com/mssql/server:2019-latest
    environment:
      SA_PASSWORD: "YourStrong!Passw0rd"
      ACCEPT_EULA: "Y"
    ports:
      - "1433:1433"
    volumes:
      - db_data:/var/opt/mssql

  nopcommerce:
    image: nopcommerceteam/nopcommerce:latest
    depends_on:
      - db
    ports:
      - "8080:80"
    volumes:
      - nop_data:/app/App_Data

volumes:
  db_data:
  nop_data:
```

```bash
docker-compose down -v
docker-compose up -d
```

![Docker Volumes](sandbox:/mnt/data/volumes.png)
*Docker Volumes: Showing persisted data volumes*

---

### 🔁 Step 5: Validation – Test Data Persistence

```bash
docker-compose down
docker-compose up -d
```

![nopCommerce Product Admin](sandbox:/mnt/data/adminPanel.png)
*Admin Panel: nopCommerce dashboard with saved product data*

![SQL Server Logs](sandbox:/mnt/data/logs.png)
*SQL Logs: Database service starting and loading nopCommerce DB*

---

## 🖥️ Step 6: Add Portainer UI

Add this to `docker-compose.yml`:

```yaml
  portainer:
    image: portainer/portainer-ce
    container_name: portainer
    restart: always
    ports:
      - "9000:9000"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - portainer_data:/data

volumes:
  db_data:
  nop_data:
  portainer_data:
```

![Portainer UI](sandbox:/mnt/data/portainerUI.png)
*Portainer Dashboard: Showing running containers*

---

## 🧪 Frontend Store Test

![nopCommerce UI](sandbox:/mnt/data/UI.png)
*Frontend Store UI: nopCommerce storefront running on Docker*

---

## 🧠 Key Troubleshooting Steps

| Issue | Fix |
|-------|-----|
| Data lost after restart | Add Docker volumes |
| Installation screen repeats | Persist `/app/App_Data` |
| No UI on Linux | Use Portainer |
| SQL fails to connect | Use strong SA password |
| App can’t reach DB | Wait or use health check |

---

## 🏁 Conclusion

I now have a stable nopCommerce setup that survives reboots and can be managed via UI.

---
