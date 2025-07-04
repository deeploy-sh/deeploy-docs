# Simplified MVP Notes 📝

## Key Simplifications for MVP

### 1. Authentication: API Key Only

- **No setup tokens** - Just generate API key during install
- **No email/password** - API key is enough for MVP
- **No user management** - Single admin with master API key

### 2. The Simplified Flow

```bash
# 1. Install server
./install.sh
> API Key: dpl_xxx...  # Save this!

# 2. Connect CLI
deeploy connect
> Server: YOUR-IP
> API Key: dpl_xxx...
> ✅ Connected!

# 3. Connect GitHub (in TUI)
> Press 'g' to connect GitHub
> Opens browser for OAuth
> ✅ GitHub connected!

# 4. Deploy
> Create project → Add pod → Deploy!
```

### 3. What We're NOT Building (Yet)

- ❌ Setup UI/wizard
- ❌ Multiple users
- ❌ Email/password auth
- ❌ API key regeneration
- ❌ Web dashboard (except landing page)

### 4. What We ARE Building

- ✅ Simple API key auth
- ✅ TUI-only interface
- ✅ GitHub integration
- ✅ Docker deployments
- ✅ Traefik for domains

Keep it simple, ship it fast! 🚀

