# Matterhorn Skills

Curated Agent Skills for [Matterhorn IDE](https://matterhorn.dev) — the Web3 development environment.

Skills are reusable instruction packages that enhance the AI agent with domain-specific knowledge. They follow the open [Agent Skills](https://agentskills.io/) standard.

## Skills Catalog

### Anthropic Official (4)

| Skill | Description | Source |
|-------|-------------|--------|
| 🎨 [frontend-design](skills/anthropic/frontend-design) | Production-grade frontend interfaces | [anthropics/skills](https://github.com/anthropics/skills) |
| 🔧 [mcp-builder](skills/anthropic/mcp-builder) | MCP server development guide | [anthropics/skills](https://github.com/anthropics/skills) |
| ⚡ [skill-creator](skills/anthropic/skill-creator) | Create and improve agent skills | [anthropics/skills](https://github.com/anthropics/skills) |
| 🧪 [webapp-testing](skills/anthropic/webapp-testing) | Playwright web app testing | [anthropics/skills](https://github.com/anthropics/skills) |

### Uniswap Official (5)

| Skill | Description | Source |
|-------|-------------|--------|
| 🦄 [uniswap-hooks](skills/uniswap/uniswap-hooks) | V4 hook development + security foundations | [Uniswap/uniswap-ai](https://github.com/Uniswap/uniswap-ai) |
| 🦄 [uniswap-trading](skills/uniswap/uniswap-trading) | Swap integration + pay-with-any-token | [Uniswap/uniswap-ai](https://github.com/Uniswap/uniswap-ai) |
| 🦄 [uniswap-viem](skills/uniswap/uniswap-viem) | EVM integration with viem/wagmi | [Uniswap/uniswap-ai](https://github.com/Uniswap/uniswap-ai) |
| 🦄 [uniswap-driver](skills/uniswap/uniswap-driver) | Swap & liquidity planning | [Uniswap/uniswap-ai](https://github.com/Uniswap/uniswap-ai) |
| 🦄 [uniswap-cca](skills/uniswap/uniswap-cca) | CCA auction configuration & deployment | [Uniswap/uniswap-ai](https://github.com/Uniswap/uniswap-ai) |

### Matterhorn Web3 Core (5)

| Skill | Description |
|-------|-------------|
| 🛡️ [solidity-security](skills/matterhorn/solidity-security) | Security patterns, vulnerability prevention, audit prep |
| 🔍 [smart-contract-audit](skills/matterhorn/smart-contract-audit) | Systematic audit workflow with automated + manual review |
| ✅ [web3-testing](skills/matterhorn/web3-testing) | Unit, fuzz, invariant, and fork testing with Foundry/Hardhat |
| 🪙 [evm-tokens](skills/matterhorn/evm-tokens) | ERC-20/721/1155/4626/2981 with OpenZeppelin v5 |
| 💰 [defi-patterns](skills/matterhorn/defi-patterns) | Swaps, lending, flash loans, oracles, Permit2 |

### Matterhorn Developer Experience (3)

| Skill | Description |
|-------|-------------|
| 🖥️ [dapp-frontend](skills/matterhorn/dapp-frontend) | Wallet connection, tx lifecycle, wagmi/RainbowKit |
| 🔑 [account-abstraction](skills/matterhorn/account-abstraction) | ERC-4337, EIP-7702, paymasters, session keys |
| 🌉 [cross-chain](skills/matterhorn/cross-chain) | LayerZero, Wormhole, Axelar, Chainlink CCIP |

### Matterhorn Nice to Have (2)

| Skill | Description |
|-------|-------------|
| ⛽ [gas-optimization](skills/matterhorn/gas-optimization) | Storage packing, calldata tricks, assembly patterns |
| 🚀 [web3-deployment](skills/matterhorn/web3-deployment) | CREATE2, UUPS proxies, verification, multi-chain deploy |

## Structure

```
matterhorn-skills/
├── registry.json              # Master catalog of all skills
├── skills/
│   ├── anthropic/             # Exact clones from anthropics/skills (Apache-2.0)
│   │   ├── frontend-design/
│   │   ├── mcp-builder/
│   │   ├── skill-creator/
│   │   └── webapp-testing/
│   ├── uniswap/               # Exact clones from Uniswap/uniswap-ai (MIT)
│   │   ├── uniswap-hooks/
│   │   ├── uniswap-trading/
│   │   ├── uniswap-viem/
│   │   ├── uniswap-driver/
│   │   └── uniswap-cca/
│   └── matterhorn/            # Matterhorn original skills (MIT)
│       ├── solidity-security/
│       ├── smart-contract-audit/
│       ├── web3-testing/
│       ├── evm-tokens/
│       ├── defi-patterns/
│       ├── dapp-frontend/
│       ├── account-abstraction/
│       ├── cross-chain/
│       ├── gas-optimization/
│       └── web3-deployment/
```

## How It Works

The Matterhorn IDE reads `registry.json` to populate the Skills catalog in the Directory modal. When a user activates a skill:

1. The skill's `SKILL.md` content is copied to `.matterhorn/skills/{id}/SKILL.md`
2. The skill is registered in `.matterhorn/skills.json`
3. At session start, enabled skills are injected into the agent's system prompt

## Licenses

- **Anthropic skills**: Apache-2.0 (sourced from [anthropics/skills](https://github.com/anthropics/skills))
- **Uniswap skills**: MIT (sourced from [Uniswap/uniswap-ai](https://github.com/Uniswap/uniswap-ai))
- **Matterhorn skills**: MIT
