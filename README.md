<div align="center">

# Self-Hosted API Infrastructure

### A production-style backend environment, built from the network up in three phases

</div>

---

## 📖 About

This project hosts the backend of my **White** application on infrastructure that I designed, built, and automated myself.

The backend was migrated from a Backend-as-a-Service architecture to a manually managed stack, with one goal: **to understand what happens underneath managed platforms.**

The project is built in three phases, and each phase builds on the previous one:

| Phase | Focus |
| ----- | ----- |
| **Phase 1** | Design and deploy the infrastructure: private network, firewall, load balancer, application servers, and database |
| **Phase 2** | Containerize every service with Docker and manage them with Docker Compose |
| **Phase 3** | Automate testing and image publishing with a GitHub Actions CI/CD pipeline |

The public API is live at **`https://api.white.ps`**, served through Cloudflare Tunnel without exposing the internal servers directly to the internet.

---

## 🗺️ Phases

<table>
  <tr>
    <td width="55%" align="center">
      <a href="./phase-1/README.md">
        <img src="images/phase-1-architecture.png" alt="Phase 1 Architecture" width="100%"/>
      </a>
    </td>
    <td width="45%" valign="top">
      <h3>Phase 1 · Infrastructure</h3>
      <p>A private network where <b>OPNsense</b> acts as the firewall, gateway, and DNS resolver. An <b>Nginx load balancer</b> distributes traffic between two <b>Flask + Gunicorn</b> application servers, backed by a dedicated <b>PostgreSQL</b> server. External access is provided through <b>Cloudflare Tunnel</b>.</p>
      <p>
        <code>OPNsense</code> · <code>Nginx</code> · <code>Flask</code> · <code>Gunicorn</code> · <code>PostgreSQL</code> · <code>Cloudflare Tunnel</code>
      </p>
      <a href="./phase-1/README.md"><b>Read Phase 1 →</b></a>
    </td>
  </tr>

  <tr>
    <td width="45%" valign="top">
      <h3>Phase 2 · Docker</h3>
      <p>Every service was containerized. First with plain <code>docker run</code>, then with <b>Docker Compose</b>. Each Flask server now runs an <b>internal Nginx reverse proxy</b> in front of the backend, connected through a dedicated Docker bridge network, with persistent volumes for the database.</p>
      <p>
        <code>Docker</code> · <code>Docker Compose</code> · <code>Docker Networks</code> · <code>Volumes</code> · <code>Docker Hub</code>
      </p>
      <a href="./phase-2/README.md"><b>Read Phase 2 →</b></a>
    </td>
    <td width="55%" align="center">
      <a href="./phase-2/README.md">
        <img src="images/phase-2.0-architecture.png" alt="Phase 2.0 Architecture" width="100%"/>
      </a>
      <br><br>
      <a href="./phase-2/README.md">
        <img src="images/phase-2.1-architecture.png" alt="Phase 2.1 Architecture" width="100%"/>
      </a>
    </td>
  </tr>

  <tr>
    <td width="55%" align="center">
      <a href="./phase-3/README.md">
        <img src="images/phase-3-architecture.png" alt="Phase 3 Architecture" width="100%"/>
      </a>
    </td>
    <td width="45%" valign="top">
      <h3>Phase 3 · CI/CD Pipeline</h3>
      <p>A <b>GitHub Actions</b> pipeline that runs the test suite against a real PostgreSQL service container, then builds the Docker image and pushes it to <b>Docker Hub</b>, tagged with the Git commit SHA for full traceability.</p>
      <p>
        <code>GitHub Actions</code> · <code>pytest</code> · <code>Docker Hub</code> · <code>Secrets</code>
      </p>
      <a href="./phase-3/README.md"><b>Read Phase 3 →</b></a>
    </td>
  </tr>
</table>

---

## 🧱 Network Design

The internal infrastructure runs on a dedicated private network:

```text
192.168.56.0/24
```

| Component        | IP Address     | Role                                |
| ---------------- | -------------- | ----------------------------------- |
| OPNsense         | `192.168.56.9` | Firewall / Gateway / DNS            |
| Load Balancer    | `192.168.56.8` | Nginx reverse proxy / load balancer |
| Application VM 1 | `192.168.56.6` | Nginx + Gunicorn + Flask            |
| Application VM 2 | `192.168.56.7` | Nginx + Gunicorn + Flask            |
| PostgreSQL       | `192.168.56.5` | Database server                     |

---

## 🔀 Request Flow

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
  ├── Flask Server 1 (Nginx → Gunicorn → Flask)
  └── Flask Server 2 (Nginx → Gunicorn → Flask)
              ↓
         PostgreSQL
```

---

## 🛠️ Technology Stack

| Layer                  | Technology                |
| ---------------------- | ------------------------- |
| Firewall / Gateway     | OPNsense                  |
| Load Balancer          | Nginx                     |
| Backend                | Flask + Gunicorn          |
| Database               | PostgreSQL                |
| Containerization       | Docker, Docker Compose    |
| CI/CD                  | GitHub Actions            |
| Image Registry         | Docker Hub                |
| Secure External Access | Cloudflare Tunnel         |
| Operating System       | Ubuntu Server             |
| Virtualization         | VirtualBox                |

<div align="center">

Part of my Cloud &amp; DevOps learning roadmap.

</div>
