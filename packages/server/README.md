# claude-blocker

CLI tool and server for [Claude Blocker](https://github.com/t3-content/claude-blocker) — block distracting websites unless Claude Code is actively running inference.

## Installation

```bash
npm install -g claude-blocker
# or
npx claude-blocker
```

## Quick Start

```bash
# First time setup (configures Claude Code hooks)
npx claude-blocker --setup

# The server will start automatically after setup
```

## Usage

```bash
# Start server (default port 8765)
npx claude-blocker

# Start with setup (configures hooks if not already done)
npx claude-blocker --setup

# Custom port
npx claude-blocker --port 9000

# Remove hooks from Claude Code
npx claude-blocker --remove

# Show help
npx claude-blocker --help
```

### Google Cloud Shell Developer Connect

When running in remote environments like Google Cloud Shell Developer Connect, configure the server URL before setup:

```bash
# Export the server URL (replace with your Cloud Shell forwarded URL)
export CLAUDE_BLOCKER_URL=https://8765-cs-xxxxxxxxx.cloudshell.dev

# Run setup - hooks will use the custom URL
npx claude-blocker --setup

# Start the server
npx claude-blocker
```

**Important:** The `CLAUDE_BLOCKER_URL` environment variable must be set **before** running `--setup` so the hooks are configured with the correct URL. If you change the URL later, you'll need to run `--remove` and then `--setup` again.

## How It Works

1. **Hooks** — The `--setup` command adds hooks to `~/.claude/settings.json` that notify the server when:
   - You submit a prompt (`UserPromptSubmit`)
   - Claude uses a tool (`PreToolUse`)
   - Claude finishes (`Stop`)
   - A session starts/ends (`SessionStart`, `SessionEnd`)

2. **Server** — Runs on localhost and:
   - Tracks all active Claude Code sessions
   - Knows when sessions are "working" vs "idle"
   - Broadcasts state via WebSocket to the Chrome extension

3. **Extension** — Connects to the server and:
   - Blocks configured sites when no sessions are working
   - Shows a modal overlay (soft block, not network block)
   - Updates in real-time without page refresh

## API

### HTTP Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/status` | GET | Returns current state (sessions, blocked status) |
| `/hook` | POST | Receives hook payloads from Claude Code |

### WebSocket

Connect to `ws://localhost:8765/ws` to receive real-time state updates:

```json
{
  "type": "state",
  "blocked": true,
  "sessions": 1,
  "working": 0
}
```

## Programmatic Usage

```typescript
import { startServer } from 'claude-blocker';

// Start on default port (8765)
startServer();

// Or custom port
startServer(9000);
```

## Requirements

- Node.js 18+
- [Claude Code](https://claude.ai/claude-code)

## License

MIT
