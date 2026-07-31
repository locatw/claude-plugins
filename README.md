# claude-plugins

Personal [Claude Code](https://code.claude.com/docs) plugins, distributed through a self-hosted marketplace catalog.

This repository is not listed in the official or community marketplaces.
It is added by URL, so only someone who knows the repository can install from it.

## Install

```bash
# In Claude Code
/plugin marketplace add locatw/claude-plugins
/plugin install loca@locatw
/reload-plugins
```

For non-interactive environments such as container images, declare it in `settings.json` instead:

```json
{
  "extraKnownMarketplaces": {
    "locatw": {
      "source": {
        "source": "git",
        "url": "https://github.com/locatw/claude-plugins.git"
      }
    }
  },
  "enabledPlugins": {
    "loca@locatw": true
  }
}
```

The URL is spelled out because the `owner/repo` shorthand clones over SSH by default, which fails where no SSH agent is available.

## Layout

- `.claude-plugin/marketplace.json` — the catalog that lists every plugin in this repository.
- `plugins/<name>/` — one plugin per directory, each with its own `.claude-plugin/plugin.json`.

## Plugins

| Plugin | Provides |
| :----- | :------- |
| `loca` | `/loca:commit` — stage files and write a WHY-focused commit message, confirming at each step |

## Versioning

No plugin declares a `version`, so Claude Code uses the commit SHA and treats every commit as a new version.
This removes version bumps and the risk of shipping a change without one.
The trade-off is that `claude plugin list` shows a SHA, so comparing two installs means checking `git log`.

## Update

```bash
claude plugin marketplace update locatw
claude plugin update loca@locatw
```

Third-party marketplaces have auto-update disabled by default, so updating is an explicit step.
`claude plugin install` reports the plugin as already installed and changes nothing, so `update` is the command that moves an existing install forward.
Restart Claude Code afterwards to apply the new version.
