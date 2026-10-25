# leancosts

Two skills that let Claude Code act on [leancosts](https://leancosts.com), the read-only cloud cost product: take one savings opportunity from the register to executed, and connect a cloud account to its first sync. leancosts never writes to your cloud. Every change runs from your terminal, with your own cloud credentials, after you confirm it.

## Setup

1. Create a Personal Access Token in leancosts: user menu, Settings, Tokens (`app.leancosts.com/settings/api-tokens`).
2. Install the plugin. Claude Code asks for the token once and keeps it in your system's credential store.

```text
/plugin install leancosts --marketplace leancosts/leancosts-plugins
```

## What it connects to

The plugin adds one MCP server, `https://api.leancosts.com/api/mcp`, and sends your token to it as a bearer token. Every step a skill takes is a call to that server's tools. It sends nothing anywhere else.

## What it runs on your machine

- **remediate**: accepts the opportunity, files and approves its change request, downloads the script leancosts rendered for it, and runs `./<script> --dry-run`, which changes nothing. Only after you answer yes does it run the script with your cloud credentials. The script then posts a receipt to leancosts.
- **connect**: reads which cloud identity you are signed in to (`aws sts get-caller-identity`, `az account show`, `gcloud config list`), shows the read-only grant commands, and runs them only after you answer yes.

Each skill handles one item per run and stops on any refusal.

## Docs

https://docs.leancosts.com/how-to/use-with-an-agent/

## License

MIT (see LICENSE).
