# leancosts

Two skills that let Claude Code act on [leancosts](https://leancosts.com), the read-only cloud cost product: take one savings opportunity from the register to executed, and connect a cloud account to its first sync. leancosts never writes to your cloud. Every change runs from your terminal, with your own cloud credentials, after you confirm it.

## Setup

1. Install the plugin.

```text
/plugin install leancosts --marketplace leancosts/leancosts-plugins
```

2. Sign in: run `/mcp`, pick leancosts, choose Authenticate and approve in the browser. There is no key to copy, and the connection acts with your own leancosts permissions.

## What it connects to

The plugin adds one MCP server, `https://api.leancosts.com/api/mcp`, which you sign in to with OAuth. Every leancosts step a skill takes is a call to that server's tools.

The commands the skills run on your machine use your own cloud CLI, so they reach your cloud provider (AWS, Azure, Google Cloud or Oracle Cloud) with your credentials, and only after you answer yes. The remediation script then reports its receipt to `api.leancosts.com`. The plugin sends nothing anywhere else.

## What it runs on your machine

- **remediate**: accepts the opportunity, files and approves its change request, downloads the script leancosts rendered for it, and runs `./<script> --dry-run`, which changes nothing. Only after you answer yes does it run the script with your cloud credentials. The script then posts a receipt to leancosts.
- **connect**: reads which cloud identity you are signed in to (`aws sts get-caller-identity`, `az account show`, `gcloud config list`), shows the read-only grant commands, and runs them only after you answer yes.

Each skill handles one item per run and stops on any refusal.

## Docs

https://docs.leancosts.com/how-to/use-with-an-agent/

## License

MIT (see LICENSE).
