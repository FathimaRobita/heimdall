# Network Monitoring System

A system that monitors authorized computers on a network and shows approved system and network metrics on an admin dashboard.

> Monitoring is limited to authorized computers and approved system/network metrics.

## Architecture

```
Registered Computers (01, 02, 03, 04, others)
        |  Local Monitoring Agent (Rust) -> JSON
        |  LAN / Wi-Fi, HTTPS / TLS
        v
MAIN COMPUTER - CENTRAL SERVER
   Backend REST API (Rust, Axum, Tokio, Serde JSON, Tower HTTP, Tracing)
   Admin Dashboard (HTMX, MiniJinja, Tailwind CSS) — rendered and served by the backend
   Background Processing (Tokio async tasks)
   Database (MySQL Community Edition, SQLx)
        ^
        |  HTTPS / TLS
Administrator Computer (Web browser)
```

The admin dashboard is rendered server-side: the Axum backend serves HTML
fragments powered by **HTMX** and styled with **Tailwind CSS**, so there is no
separate frontend build/runtime (no React, no Node server in production).

## Technology Stack

| Layer | Technology |
|---|---|
| Agent | Rust, Serde JSON |
| Backend | Rust, Axum, Tokio, Tower HTTP, Tracing, MiniJinja |
| Database | MySQL Community Edition, SQLx |
| Background jobs | Tokio async tasks |
| Frontend | HTMX, MiniJinja templates, Tailwind CSS |

## Project Structure

```
.
├── agent/            # Local Monitoring Agent (Rust)            — #1, #6
├── backend/          # REST API + HTMX dashboard server (Rust)  — #2
│   ├── templates/    #   MiniJinja HTML templates (HTMX dashboard)
│   ├── static/       #   Tailwind CSS + static assets
│   ├── migrations/   # MySQL migrations                          — #3
│   └── src/
│       └── tasks/    # Background processing                     — #5
├── docs/             # Documentation                             — #3
└── README.md
```

> The admin dashboard is server-rendered by the backend. The HTML templates and
> Tailwind styling currently live in `backend/templates/` and `backend/static/`
> so the backend is self-contained; the dashboard design (issue #4) is built on
> top of these templates.

## Team and Responsibilities

| Issue | Module | Developer | Folder |
|---|---|---|---|
| #1 | Local Monitoring Agent | Simha | `agent/` |
| #2 | Backend REST API | Robeeta | `backend/` |
| #3 | Database (MySQL + SQLx) | Yaswini | `backend/migrations/`, `docs/` |
| #4 | Admin Dashboard | Farwes | `backend/templates/`, `backend/static/` |
| #5 | Background Processing | Safana | `backend/src/tasks/` |
| #6 | Browser History Monitoring | Kumar | `agent/src/browser_history.rs` |

## Getting Started

### Prerequisites

- Rust (stable) and Cargo
- MySQL Community Edition
- (Optional, for Tailwind rebuild) Node.js 18+ and npm

### Database

```sql
CREATE DATABASE network_monitoring CHARACTER SET utf8mb4;
```

### Backend + Dashboard

```bash
cd backend
cp .env.example .env     # set DATABASE_URL, JWT_SECRET
cargo run
```

The backend serves the API and the HTMX dashboard on the same port
(default `http://localhost:3000`). Open `/login` in a browser.

Create the initial admin user:

```bash
cd backend
cargo run --bin seed_admin -- --username admin --password yourpassword
```

### (Optional) Rebuild Tailwind CSS

A pre-built stylesheet is included so the dashboard works without Node.
To regenerate after editing templates:

```bash
cd backend
npm install
npm run build:css        # one-off minified build
# or, while developing:
npm run watch:css
```

## API

See [`backend/API.md`](backend/API.md) for the full REST contract.

## Contribution Workflow

1. Fork the repository and clone your fork.
2. Pick your assigned issue and create a branch:
   ```bash
   git checkout -b feature/<short-name>
   ```
3. Work only inside your assigned folder to avoid merge conflicts.
4. Commit with the issue number:
   ```bash
   git commit -m "Add CPU metrics collection (#1)"
   ```
5. Push the branch and open a Pull Request to the main repository.
6. Write `Closes #<issue-number>` in the PR description.
7. Do not push directly to `main`.

## Privacy and Authorization

- Only registered and authorized computers send data.
- Only approved metrics are collected.
- Browser history monitoring is opt-in, shows a notice on the monitored computer, is admin-only and has a data retention limit.

## License

To be decided by the project owner.
