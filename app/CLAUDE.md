# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Deeploy is a TUI-first Docker deployment platform - a modern alternative to Coolify/Dokploy that lives in your terminal. Users install a lightweight server on any VPS (Hetzner, DigitalOcean, etc.), connect via a local CLI, and deploy Docker containers through an intuitive terminal interface. Think "Kubernetes but simple" meets "Vercel but self-hosted".

### Core Vision
- **TUI-First**: Full terminal UI using Bubble Tea - no web dashboard needed
- **Simple Setup**: Server install in <5 min, first deploy in <10 min  
- **Docker Native**: Deploy anything with a Dockerfile or docker-compose.yml
- **GitHub Integrated**: Connect repos, deploy on push (via GitHub App)
- **Modern DX**: Vim-style navigation, streaming logs, instant feedback

## Development Commands

### Web Development (Full Stack)
```bash
make dev              # Runs templ, server, and tailwind in parallel
make server           # Run web server only with hot reload (port 8090)
make templ            # Watch and generate templ templates
make tailwind         # Watch and compile Tailwind CSS
```

### TUI Development
```bash
go run cmd/tui/main.go        # Run TUI application
DEBUG=1 go run cmd/tui/main.go # Run with debug logging
make cli-debug                # Debug with Delve (port 43000)
make cli-log                  # Tail debug.log file
```

### Building
```bash
./scripts/build-binaries.sh   # Build for darwin/linux amd64/arm64
```

## Architecture

### Directory Structure
- `/cmd/web/` - Web server entry point
- `/cmd/tui/` - Terminal UI entry point
- `/internal/` - Core business logic:
  - `handlers/api/` - JSON API handlers
  - `handlers/web/` - HTML template handlers
  - `services/` - Business logic layer
  - `data/` - Repository pattern for data access
  - `ui/` - Templ templates for web UI
  - `tui/` - Bubble Tea components for terminal UI

### Tech Stack
- **Backend**: Go 1.23.3
- **Database**: SQLite with golang-migrate
- **Web UI**: Templ templates + Tailwind CSS
- **TUI**: Bubble Tea + Lipgloss
- **Auth**: JWT tokens (Bearer for API, Cookie for web)

### Key Patterns

#### Authentication Flow
- Dual token support: Bearer tokens for CLI/API, cookies for web
- Three middleware types:
  - `Auth()` - Validates token, adds user to context
  - `RequireAuth()` - Redirects to login if not authenticated
  - `RequireGuest()` - Redirects authenticated users away

#### Route Structure
- **API Routes**: `/api/*` prefix, JSON responses
- **Web Routes**: Direct paths, HTML responses
- All routes defined in `/internal/routes/`

#### Data Model
- **Users**: Authentication and ownership
- **Projects**: Top-level organizational unit
- **Pods**: Services within projects (renamed from "services")

### Environment Variables
Create `.env` file with:
```
GO_ENV=dev
JWT_SECRET=your-secret-key
```

## Common Tasks

### Adding a New API Endpoint
1. Create handler in `/internal/handlers/api/`
2. Add route in `/internal/routes/api.go`
3. Use JSON request/response pattern
4. Apply `auth.Auth()` middleware if authentication required

### Adding a New Web Page
1. Create templ template in `/internal/ui/`
2. Create handler in `/internal/handlers/web/`
3. Add route in `/internal/routes/web.go`
4. Run `make dev` to see changes with hot reload

### Working with Templates
- Templ files use `.templ` extension
- Components can be imported and composed
- Run `make templ` to watch for changes

### Database Changes
1. Create migration file in `/internal/db/migrations/`
2. Follow naming convention: `YYYYMMDDHHMMSS_description.up.sql`
3. Migrations run automatically on startup

## MVP Plan

### Phase 1: Foundation (Week 1) - Server ↔️ CLI Connection

#### 1.1 API Key System
- Create `internal/auth/apikey.go` for API key generation & validation
- Add `api_keys` table to database schema
- Implement middleware for API key authentication
- Key format: `dpl_` + 32 random characters (not JWT for simplicity)

#### 1.2 Server Setup
- Generate API key during installation script
- Display API key in terminal after installation
- Simple landing page for web visitors
- No setup UI needed for MVP

#### 1.3 CLI Connection
- Implement `cmd/tui/connect.go` for connection flow
- Save server config to `~/.deeploy/config.yml`
- Add `/api/health` endpoint for connection verification
- Show connection status in TUI

### Phase 2: Project Management (Week 2) - CRUD via TUI

#### 2.1 TUI Views
- Dashboard view showing project list
- Project detail view showing pods
- Create/Edit forms using Bubble Tea components
- Navigation: j/k (up/down), Enter (select), ESC (back), ? (help)

#### 2.2 API Endpoints
- `GET/POST/PUT/DELETE /api/projects`
- `GET/POST/PUT/DELETE /api/pods`
- SSE endpoint for real-time updates

#### 2.3 Database Schema Updates
```sql
pods table additions:
+ dockerfile_path TEXT
+ docker_compose_path TEXT
+ env_vars JSON
+ port_mappings JSON
+ status TEXT (pending/building/running/stopped)
+ container_id TEXT
```

### Phase 3: Docker Integration (Week 3-4) - Deploy Real Containers

#### 3.1 Docker Service
- Create `internal/docker/client.go` wrapper around Docker SDK
- Implement build from Dockerfile/docker-compose.yml
- Container lifecycle management (start/stop/remove)
- Log streaming implementation

#### 3.2 Deployment Flow
1. Clone repository (public repos first, no GitHub auth)
2. Detect Dockerfile or docker-compose.yml
3. Build Docker image
4. Run container with configurations
5. Update pod status in database

#### 3.3 TUI Deployment Features
- Deploy form with repo URL and branch selection
- Streaming build logs display
- Deployment status indicators
- Error handling and display

### Phase 4: GitHub Integration (Week 5) - Private Repo Support

#### 4.1 GitHub App Setup
- Create GitHub App (manual process initially)
- Implement installation webhook handler
- Store installation_id in database
- Token exchange for API calls

#### 4.2 Enhanced Setup Flow
- Add "Connect GitHub" button to setup page
- Implement OAuth flow
- Store encrypted GitHub tokens

#### 4.3 Private Repository Features
- Use GitHub token for authenticated git clone
- Webhook handler for deploy-on-push
- Branch selection from GitHub API

### Phase 5: Networking & Domains (Week 6) - Make Apps Accessible

#### 5.1 Traefik Integration
- Install Traefik as system container
- Dynamic configuration for routes
- Automatic SSL via Let's Encrypt

#### 5.2 Domain Management
- Add domain field to pods
- Generate Traefik labels for containers
- Show SSL status in TUI

## Technical Decisions

### Architecture Choices
- **API Keys over JWT**: Simpler implementation, no refresh complexity
- **SQLite remains**: No PostgreSQL for MVP, keep it simple
- **SSE over WebSockets**: Simpler streaming implementation
- **GitHub App over OAuth**: More professional, better for organizations

### TUI Design Philosophy
- **Navigation**: Vim-style keybindings (j/k/h/l)
- **Help System**: ? key shows contextual help
- **Forms**: Bubble Tea's textarea and textinput components
- **Layout**: List on left, details on right (similar to k9s)

### Security Considerations
- API keys stored as SHA256 hashes
- GitHub tokens encrypted with server secret
- API keys shown only once during installation
- No root containers in production

### DevOps Setup
- Systemd service for server management
- Auto-restart on crashes
- SQLite backup via cron
- Structured JSON logging

## Not in MVP (Future Features)
- Multi-user management (only single admin for MVP)
- Build caching optimization
- Resource limits and monitoring
- Metrics and observability
- Buildpacks support
- Multi-server orchestration
- Web UI (except minimal setup page)

## Success Criteria
1. ✅ Server installed in under 5 minutes
2. ✅ CLI connected in under 1 minute
3. ✅ First app deployed in under 5 minutes
4. ✅ Logs streaming in real-time in TUI
5. ✅ Domain accessible with automatic SSL

## Important Notes

- Always use the repository pattern in `/internal/data/` for database access
- Keep business logic in `/internal/services/`
- Use context for passing user information from middleware
- TUI and web share the same authentication system
- No test files exist yet - consider adding tests when implementing new features
- API keys are stored as hashes, never plaintext
- GitHub tokens are encrypted before storage
- All container operations go through `/internal/docker/` service