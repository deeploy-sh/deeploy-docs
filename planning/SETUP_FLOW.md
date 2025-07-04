# The BEST Setup Flow Ever 🔥

## 1. Server Install (2 min)
```bash
ssh root@hetzner
curl -sSL https://get.deeploy.io | bash

✅ Deeploy installed!
🔑 API Key: dpl_K9sJ2dH8G7F6D5S4A3
📡 Server: http://142.251.40.14:8090

Save this API key! Now install the CLI on your machine.
```

## 2. CLI Install & Connect (1 min)
```bash
brew install deeploy
deeploy

Welcome to Deeploy! Let's connect to your server.

Server URL: 142.251.40.14
API Key: dpl_K9sJ2dH8G7F6D5S4A3

⠋ Connecting...
✅ Connected to server!

Now let's connect GitHub for deployments:
Press ENTER to open GitHub authorization...
```

## 3. GitHub Auth (30 sec)
```bash
Opening https://github.com/apps/deeploy/installations/new

[Browser opens]
→ User selects repos
→ GitHub redirects to http://142.251.40.14:8090/github/callback
→ Server shows: "✅ GitHub connected! Return to your CLI"
```

## 4. CLI Auto-Update (Magic! ✨)
```bash
[CLI polls in background]

✅ Connected to server
✅ Connected to GitHub (user: axeladrian)

Press ENTER to continue to dashboard...

[ENTER]

┌─ Deeploy Dashboard ─────────────────────┐
│ Projects (0)         › Create New       │
│                                         │
│ [n] New Project                         │
│ [?] Help                                │
│ [q] Quit                                │
└─────────────────────────────────────────┘
```

## Why This is Amazing 🎯

1. **No Registration!** - API key is enough
2. **Forced GitHub Setup** - But in-flow, not annoying
3. **Auto-Update Magic** - CLI detects when GitHub is connected
4. **Zero Web UI** - Everything in terminal (except GitHub OAuth)

## The Polling Trick 🪄

```go
// In CLI during GitHub auth
func waitForGitHubConnection() {
    spinner := spinner.New()
    spinner.Start()
    
    for {
        status, _ := api.GetSetupStatus()
        if status.GitHubConnected {
            spinner.Stop()
            fmt.Println("✅ Connected to GitHub!")
            break
        }
        time.Sleep(2 * time.Second)
    }
}
```

## Even Better: OAuth Device Flow! 🤯

Instead of opening browser:

```bash
To connect GitHub, visit:
https://github.com/login/device

Enter code: ABCD-1234

⠋ Waiting for authorization...
✅ GitHub connected!
```

(Like GitHub CLI `gh auth login` does!)

## Server Landing Page

When someone visits the IP in browser:

```
┌────────────────────────────────┐
│   🚀 Deeploy Server Active     │
│                                │
│   This is a headless server.   │
│   Use the CLI to interact:     │
│                                │
│   brew install deeploy         │
│   deeploy connect YOUR-IP      │
│                                │
│   Docs: deeploy.io/quickstart  │
└────────────────────────────────┘
```

## Flow Improvements:

### 1. Skip GitHub Option
```bash
Connect GitHub now? (recommended) [Y/n]: n
⚠️  Skipping GitHub (only public repos available)
```

### 2. Domain Setup Wizard
```bash
✅ Connected to GitHub

Configure domain? [Y/n]: y
Domain: app.example.com
→ Configuring DNS...
✅ Domain configured!
```

### 3. First Deploy Prompt
```bash
Ready to deploy your first app? [Y/n]: y
→ Opening project creation...
```

## This is THE Flow! 🚀

Simple, fast, forced-opinionated but not annoying. Can't get better than this!