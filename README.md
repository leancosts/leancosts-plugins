# leancosts plugins

Skills for Claude Code and Codex that take a leancosts savings opportunity from the register to executed, and connect a cloud to its first sync, with your own cloud credentials. leancosts only reads your cloud.

This repository is a copy. The source lives with the product and is exported by a script; do not open pull requests here.

## Claude Code

```text
/plugin marketplace add leancosts/leancosts-plugins
/plugin install leancosts@leancosts
```

Then run `/mcp`, pick leancosts, choose Authenticate and approve in the browser. There is no key to copy.

## Codex

```bash
codex plugin marketplace add leancosts/leancosts-plugins
codex plugin add leancosts@leancosts
codex mcp login leancosts
```

The last command opens the browser to sign in. There is no key to copy.

## Docs

https://docs.leancosts.com/how-to/use-with-an-agent/

## License

MIT, for the plugin in this repository only (see LICENSE). The leancosts product keeps its own terms.
