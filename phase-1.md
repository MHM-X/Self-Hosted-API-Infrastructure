# Self-Hosted API Infrastructure Lab — Phase 1

## Overview

This project is the first infrastructure phase of my Cloud & DevOps learning roadmap.

---

## Architecture

![Architecture](images/phase-1-architecture.png)

---

### Network Design

The internal infrastructure uses a dedicated private network:

```text
192.168.56.0/24
```

| Component        |     IP Address | Role                                |
| ---------------- | -------------: | ----------------------------------- |
| OPNsense         | `192.168.56.9` | Firewall / Gateway / DNS            |
| Load Balancer    | `192.168.56.8` | Nginx reverse proxy / load balancer |
| Application VM 1 | `192.168.56.6` | Flask + Gunicorn + Nginx            |
| Application VM 2 | `192.168.56.7` | Flask + Gunicorn + Nginx            |
| PostgreSQL       | `192.168.56.5` | Database server                     |

All internal servers use OPNsense as their default gateway.

Direct NAT access from the internal VMs was removed so that outbound traffic from the lab follows the intended gateway architecture.

---

## Application Layer

The infrastructure hosts the backend of my **White** application.

The backend was migrated from a Backend-as-a-Service architecture toward a manually managed infrastructure stack in order to understand what happens underneath managed platforms.

### Backend

* Python
* Flask
* Gunicorn
* Nginx

### Database

* PostgreSQL

The database is hosted on a dedicated VM rather than running on the application servers.

This separation allows the application and database layers to scale and be managed independently.

---

## Infrastructure Components

### OPNsense

Used as the central:

* Firewall
* Network gateway
* DNS resolver
* Internet egress point

All internal servers route external traffic through OPNsense.

### Nginx Load Balancer

The load balancer distributes incoming application requests between the two Flask application servers.

```text
Client
  ↓
Nginx Load Balancer
  ├── VM1
  └── VM2
```

Backend distribution was verified using response headers from the application servers.

### Flask + Gunicorn

Flask provides the application API while Gunicorn acts as the production WSGI application server.

Nginx sits in front of Gunicorn on each application VM.

```text
Nginx
  ↓
Gunicorn
  ↓
Flask
```

### PostgreSQL

PostgreSQL runs on its own dedicated server.

The application servers communicate with PostgreSQL over the private network rather than through the public internet.

### Cloudflare Tunnel

Cloudflare Tunnel provides external access to the application without exposing the internal servers directly to the internet.

The public API is available through:

```text
https://api.white.ps
```

The external traffic path is:

```text
Client
  ↓
Cloudflare
  ↓
Cloudflare Tunnel
  ↓
OPNsense
  ↓
Nginx Load Balancer
  ↓
Application Servers
```

---

## Technology Stack

| Layer                  | Technology                |
| ---------------------- | ------------------------- |
| Firewall / Gateway     | OPNsense                  |
| Load Balancer          | Nginx                     |
| Reverse Proxy          | Nginx                     |
| Backend                | Flask                     |
| Application Server     | Gunicorn                  |
| Database               | PostgreSQL                |
| Secure External Access | Cloudflare Tunnel         |
| Operating System       | Ubuntu Server             |
| Virtualization         | VirtualBox                |
| Network                | Host-Only Private Network |
| API Testing            | Postman / cURL            |

---

>This phase established the infrastructure foundation for the next stages of the roadmap.

---
