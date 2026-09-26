# SysInfo API

## 1. What this app does

SysInfo API is a small FastAPI application that runs on a Linux server and provides useful information about that server through HTTP API endpoints.

The application can show general system information such as the hostname, operating system, architecture and uptime. It can also return CPU, memory, disk, network and running-process information.

The purpose of this project is not only to build a FastAPI application, but also to learn how to deploy and manage a Python application manually on RHEL/CentOS using Linux, systemd, Nginx, firewall rules, SELinux, Bash and Cron without using Docker or Kubernetes.

---

## 2. API Endpoints

### Health Check

Checks whether the application is responding.

```bash
curl http://localhost:8000/health
```

Example response:

```json
{
  "status": "ok",
  "service": "sysinfo-api"
}
```

### System Information

Returns basic information about the server, including hostname, platform, architecture, uptime and current timestamp.

```bash
curl http://localhost:8000/system/info
```

### CPU Information

Returns CPU-related information such as physical CPU count, logical CPU count, current CPU usage and CPU frequency.

```bash
curl http://localhost:8000/system/cpu
```

### Memory Information

Returns total, used and available memory along with the percentage of memory currently being used.

```bash
curl http://localhost:8000/system/memory
```

### Disk Information

Returns disk usage information for the root filesystem, including total, used and free space.

```bash
curl http://localhost:8000/system/disk
```

### Network Information

Returns network interface information along with sent and received bytes and packets.

```bash
curl http://localhost:8000/system/network
```

### Top Processes

Returns the top five processes using the most CPU.

```bash
curl http://localhost:8000/system/processes
```

---

## 3. Deployment From Scratch

### Phase 1 — Git Setup

First, Git is installed and configured. A project directory named `sysinfo-api` is created and initialized as a Git repository.

The application folders are then created for the FastAPI code, routes, logs and scripts.

A first commit is made so that the initial project structure is stored in Git.

### Phase 2 — Python Environment and Manual Run

Python 3 and pip are installed. A Python virtual environment is created so that the project's dependencies remain isolated from the system Python environment.

The packages listed in `requirements.txt` are installed.

The application is first started manually using Uvicorn:

```bash
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```

The API is then tested using `curl`. FastAPI's Swagger UI can also be opened through `/docs`.

### Phase 3 — systemd Service

After confirming that the application works manually, a systemd service is created.

The service allows Linux to manage the FastAPI application as a background service. It can start the application during boot and restart it when it fails.

The service can be controlled using:

```bash
sudo systemctl start sysinfo-api
sudo systemctl stop sysinfo-api
sudo systemctl restart sysinfo-api
sudo systemctl status sysinfo-api
```

### Phase 4 — Nginx Reverse Proxy

Nginx is installed and configured as a reverse proxy.

Nginx listens on port 80 and forwards incoming requests to the FastAPI application running on `127.0.0.1:8000`.

The firewall is configured to allow HTTP traffic, and the required SELinux setting is applied so that Nginx can connect to the backend application.

### Phase 5 — Log Archiving With Bash and Cron

A Bash script is created to archive the Nginx access log.

The script compresses the log, stores it in an archive directory and removes archives older than 30 days.

Cron is then configured to run this script automatically every Sunday at 2 AM.

The script is tested manually before relying on the scheduled Cron job.

---

## 4. Configuration Files

### `sysinfo-api.service`

This is the systemd service configuration for the FastAPI application.

It tells systemd which Linux user should run the application, which project directory should be used and which Uvicorn command should start the application.

It also defines the restart behavior so that systemd can restart the application if it fails.

Location:

```text
/etc/systemd/system/sysinfo-api.service
```

### `sysinfo-api.conf`

This is the Nginx configuration for the SysInfo API.

It tells Nginx to listen on port 80 and forward requests to the FastAPI application running on port 8000.

Location:

```text
/etc/nginx/conf.d/sysinfo-api.conf
```

---

## 5. Checking Logs

### FastAPI / systemd logs

To view the logs of the application service:

```bash
sudo journalctl -u sysinfo-api
```

To follow the logs live:

```bash
sudo journalctl -u sysinfo-api -f
```

### Nginx access log

```text
/var/log/nginx/sysinfo-api.access.log
```

This contains requests received by Nginx for the application.

### Nginx error log

```text
/var/log/nginx/sysinfo-api.error.log
```

This contains Nginx-related errors for the application.

### Application log

The FastAPI application also writes its own log to:

```text
logs/app.log
```
