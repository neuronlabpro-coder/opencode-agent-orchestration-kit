# Installation

## Test without global install

```bash
git clone https://github.com/<owner>/opencode-agent-orchestration-kit.git
cd opencode-agent-orchestration-kit
export OPENCODE_CONFIG_DIR="$PWD/opencode"
source env.example
(cd opencode && npm install)
opencode
```

## Global install

```bash
./install.sh
(cd "${OPENCODE_CONFIG_DIR:-$HOME/.config/opencode}" && npm install)
```

Default target:

```bash
${OPENCODE_CONFIG_DIR:-$HOME/.config/opencode}
```

Use a custom target:

```bash
./install.sh --target "$HOME/.config/opencode"
```

Use `--force` only when you want to overwrite existing `AGENTS.md` and `opencode.json`.

## Auth and models

```bash
opencode auth login
opencode models openai --refresh
```

Then export model variables from `env.example`.

## Existing config

If `opencode.json` already exists, the installer preserves it unless `--force` is used. Add the Superpowers plugin manually if needed:

```json
{
  "plugin": ["superpowers@git+https://github.com/obra/superpowers.git"]
}
```

If `tui.json` already exists, the installer preserves it unless `--force` is
used. Add the bundled token usage plugin manually if needed:

```json
{
  "plugin": ["./plugins/token-tree-usage.tsx"]
}
```
