# onfra-skill

Agent skill for checking Celo wallet **financial reputation** via [OnFRA](https://onfra.xyz) (Onchain Financial Reputation Agent).

## Install

### Skills CLI (Cursor, Claude Code, etc.)

```bash
npx skills add github:jeffIshmael/onfra-skill
```

### Manual

```bash
git clone https://github.com/jeffIshmael/onfra-skill.git
cp -r onfra-skill/SKILL.md ~/.cursor/skills/onfra-check-wallet/SKILL.md
# or symlink into your project's .agents/skills/ folder
```

### Cursor project skill

```bash
mkdir -p .agents/skills/onfra-check-wallet
curl -o .agents/skills/onfra-check-wallet/SKILL.md \
  https://raw.githubusercontent.com/jeffIshmael/onfra-skill/main/SKILL.md
```

## What it does

Teaches AI agents how to:

- Analyze any Celo wallet (`POST /api/agent/analyze`)
- Verify `REP-{id}` passports (`GET /api/agent/verify/{id}`)
- Use MCP tools from `/.well-known/mcp.json`
- Handle x402 micropayments for external queries

## Publish your own fork

This folder is designed as a **standalone git repository**:

```bash
cd onfra-skill
git init
git add SKILL.md README.md reference.md
git commit -m "Add OnFRA wallet reputation skill"
git remote add origin git@github.com:YOUR_USER/onfra-skill.git
git push -u origin main
```

Then update the install command with your GitHub username.

## Links

- API: https://wallet-profile-orpin.vercel.app
- MCP: https://wallet-profile-orpin.vercel.app/.well-known/mcp.json
- ERC-8004: https://8004scan.io/agents/celo/9219
- Docs: https://onfra.xyz/api

## License

MIT
