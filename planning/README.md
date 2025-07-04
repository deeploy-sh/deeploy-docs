# Deeploy Planning & MVP Documentation 📚

This folder contains all planning documents and implementation details for the Deeploy MVP.

## Documents

### 1. [SETUP_FLOW.md](./SETUP_FLOW.md)

The complete setup flow from server installation to first deploy. Shows the optimal user flow with API keys, GitHub integration, and TUI navigation.

### 2. [API_KEY_SYSTEM.md](./API_KEY_SYSTEM.md)

Technical details about the API key system:

- How API keys are generated
- Secure storage with SHA256 hashes
- Difference from JWT
- Complete implementation

## Planned Documents

- [ ] DOCKER_INTEGRATION.md - Docker SDK usage and container management
- [ ] GITHUB_APP.md - GitHub App setup and OAuth flow
- [ ] TUI_ARCHITECTURE.md - Bubble Tea structure and navigation
- [ ] DEPLOYMENT_FLOW.md - From git clone to running container
- [ ] TRAEFIK_SETUP.md - Reverse proxy and SSL automation

## Quick Links

- [CLAUDE.md](../CLAUDE.md) - Main documentation with MVP phases
- [Makefile](../Makefile) - Development commands
- [go.mod](../go.mod) - Dependencies

