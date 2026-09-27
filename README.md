# first-repo
Creative world

## Claude Code skills

- `caveman` (`.claude/skills/caveman/`) — terse, no-filler output mode. Invoke with `/caveman`.
  Vendored from [Shawnchee/caveman-skill](https://github.com/Shawnchee/caveman-skill) @ `82af154` (MIT).

## Plugin marketplace

This repo is a Claude Code plugin marketplace (`.claude-plugin/marketplace.json`). Install `caveman` anywhere:

```
/plugin marketplace add masterpeteworld/first-repo
/plugin install caveman@first-repo
```

## MCP servers

`.mcp.json` registers the Perplexity MCP server (pinned to `1.3.0`). The key is read from the `PERPLEXITY_API_KEY` environment variable — set it in your shell or cloud environment settings; never commit it.
