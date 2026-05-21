# OpenCode MCP Configuration

This repository contains my OpenCode configuration with MCP (Model Context Protocol) servers. All sensitive values are stored in environment variables, making this repo safe to push to GitHub.

## Setup

1. **Clone this repo**

2. **Copy the environment template**
   ```bash
   cp .env.example .env
   ```

3. **Fill in your secrets** in `.env`
   ```bash
   # Edit .env with your actual API keys and tokens
   ```

4. **Install opencode if you haven't already**
   ```bash
   npm install -g opencode
   # or use npx
   ```

5. **Load the environment and start opencode**
   
   You have two options:

   ### Option A: Export variables manually (current session only)
   ```bash
   export $(cat .env | grep -v '^#' | xargs)
   opencode
   ```

   ### Option B: Use a shell wrapper (recommended)
   Create a script or alias that loads `.env` before starting opencode:
   ```bash
   # In your .zshrc or .bashrc
   alias opencode-mcp='export $(cat ~/.config/opencode/.env | grep -v "^#" | xargs) && opencode'
   ```

   ### Option C: Use direnv (best for projects)
   If you have [direnv](https://direnv.net/) installed:
   ```bash
   cp .env .envrc  # direnv uses .envrc
   direnv allow
   ```

## How it works

The `opencode.json` file references environment variables using `${VAR_NAME}` syntax. For example:

```json
{
  "mcp": {
    "github": {
      "type": "remote",
      "url": "${GITHUB_MCP_URL}",
      "headers": {
        "Authorization": "Bearer ${GITHUB_TOKEN}"
      }
    }
  }
}
```

This means:
- The actual secrets live only in your `.env` file
- `.env` is listed in `.gitignore` and will never be committed
- You can safely push `opencode.json` to GitHub

## Adding new MCP servers

1. Add the server configuration to `opencode.json`
2. If it requires secrets, add the env var names to `.env.example`
3. Update your local `.env` with the actual values
4. Restart opencode for changes to take effect

## Security tips

- **Never commit `.env`** — it's in `.gitignore` but double-check before pushing
- **Rotate tokens regularly** — especially GitHub personal access tokens
- **Use fine-grained tokens** — give each MCP the minimum permissions it needs
- **Consider a password manager** — store tokens in 1Password/Bitwarden and inject them via CLI

## Structure

```
.
├── opencode.json      # Safe to commit — uses env var references
├── .env.example       # Safe to commit — template with empty values
├── .env               # NEVER commit — your actual secrets live here
├── .gitignore         # Excludes .env and other sensitive files
└── README.md          # This file
```

## Troubleshooting

### "Environment variable not found" error
Make sure you've exported the variables before starting opencode:
```bash
export $(cat .env | grep -v '^#' | xargs)
```

### Changes not taking effect
OpenCode loads config at startup and does not hot-reload. **Quit and restart** after any config changes.

### Need to add a new secret
1. Add the variable name to `.env.example` (with empty or fake value)
2. Add the actual value to your local `.env`
3. Reference it in `opencode.json` with `${VAR_NAME}`
