
# 🐳 From Chaos to Control: My Journey with Docker, SQL Server, and Secure Development

Deploying a full-stack application with Docker sounds straightforward in theory — spin up containers, connect the database, and you're off to the races. But in reality, it’s a series of small battles. Here's my story of how I went from frustration to a fully functioning, secure, and persistent environment using Docker, SQL Server, volumes, and Ngrok.

---

## 🏗️ Setting the Stage: Why Docker?

I was working on a **nopCommerce** project and decided to containerize the environment to make development and deployment easier. Docker Compose felt like the right tool to orchestrate services — a web app and a SQL Server backend.

The goal:
- Run SQL Server in a container
- Connect it with nopCommerce
- Ensure data is **persistent**
- Enable **HTTPS** access via **Ngrok**

Simple enough? Not quite.

---

## 😫 The First Hurdle: Ephemeral Databases

I started with a basic `docker-compose.yml` setup:

```yaml
services:
  sqlserver:
    image: mcr.microsoft.com/mssql/server:2022-preview-ubuntu-22.04
    environment:
      ACCEPT_EULA: "Y"
      MSSQL_SA_PASSWORD: "yourStrong()Password"
      MSSQL_PID: "Evaluation"
    ports:
      - "1433:1433"
```

Everything ran fine — until I stopped the container. When I restarted it, **all the data was gone**. Every table I created, every record I inserted... wiped out. That's when I learned about Docker's default **ephemeral storage**.

---

## 💾 Lesson Learned: Persisting SQL Server Data

To prevent data loss, I created a **named volume** and mounted it to SQL Server’s data directory:

```yaml
volumes:
  - sql_data:/var/opt/mssql
```

Then I added a `volumes` block at the end of the file:

```yaml
volumes:
  sql_data:
```

This change ensured that even if the container was stopped or recreated, the database files remained intact. Finally, persistent data. ✅

---

## 🔗 Integrating nopCommerce: More Than Just a Connection String

Connecting nopCommerce to SQL Server wasn't plug-and-play. I had to:
- Match the **host and port** used in Docker
- Ensure the database was reachable using the container name (`sqlserver`)
- Adjust `appsettings.json` to use the correct connection string
- Wait for SQL Server to fully boot up before nopCommerce tried to connect

After some tweaking and container restarts, the application and database were talking.

---

## 🔐 Next Challenge: HTTPS in Development

Most modern APIs and third-party services demand **HTTPS**, even in development. Exposing my local site over HTTPS wasn't as easy as enabling a flag — that’s where **Ngrok** came in.

Ngrok allows secure tunnels to your local environment. I installed Ngrok and started a tunnel like this:

```bash
ngrok http 5000
```

Now I had a public HTTPS URL like `https://randomsubdomain.ngrok.io`, which forwarded to `localhost:5000`. This solved several problems:
- Secure frontend/backend communication
- Easy webhook testing from third-party services
- SSL termination handled externally

---

## 🎯 What I Learned

This journey taught me far more than Docker commands:
- Persistence matters. Always configure volumes early.
- Services might depend on startup order. Use `depends_on` or wait-for-it scripts.
- HTTPS in development is essential for many real-world scenarios.
- Containerization doesn't eliminate problems — it just makes them easier to isolate and solve.

---

## ✅ Final Setup Overview

```yaml
version: '3.8'

services:
  sqlserver:
    image: mcr.microsoft.com/mssql/server:2022-preview-ubuntu-22.04
    container_name: sqlpreview
    hostname: sqlpreview
    environment:
      ACCEPT_EULA: "Y"
      MSSQL_SA_PASSWORD: "yourStrong()Password"
      MSSQL_PID: "Evaluation"
    ports:
      - "1433:1433"
    volumes:
      - sql_data:/var/opt/mssql

volumes:
  sql_data:
```

And in a separate terminal:

```bash
ngrok http 5000
```

---

## 💬 Conclusion

What started as a simple Docker experiment turned into a crash course in container persistence, service orchestration, and secure tunneling. If you're just starting out with Docker and databases, don't be discouraged by the bumps — they’re part of the process. Solve them one at a time, and you’ll build something far more stable and scalable in the end.

---

## 📌 P.S. Coming Soon…

In a future post, I’ll show how I set up **Jenkins** for CI/CD to automate builds and deployments using this exact stack. Stay tuned!
