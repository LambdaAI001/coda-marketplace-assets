# Coda — AI Coding Agent for VS Code

<div align="center">

<img src="coda.png" alt="Coda Logo" width="120" />

### The Autonomous Software Engineering Harness Engineered for VS Code

[![Marketplace Version](https://img.shields.io/visual-studio-marketplace/v/express-genix.coda-ai?color=34d399&label=Marketplace&logo=visual-studio-code)](https://marketplace.visualstudio.com/items?itemName=express-genix.coda-ai)
[![Installs](https://img.shields.io/visual-studio-marketplace/i/express-genix.coda-ai?color=38bdf8)](https://marketplace.visualstudio.com/items?itemName=express-genix.coda-ai)
[![Rating](https://img.shields.io/visual-studio-marketplace/r/express-genix.coda-ai?color=fbbf24)](https://marketplace.visualstudio.com/items?itemName=express-genix.coda-ai)
[![License: MIT](https://img.shields.io/badge/License-MIT-emerald.svg)](LICENSE)
[![Official Website](https://img.shields.io/badge/Website-coda.express--genix.dev-purple)](https://github.com/LambdaAI001/coda-website)
[![Security Policy](https://img.shields.io/badge/Security-Policy-blue)](SECURITY.md)

[Official Website](https://github.com/LambdaAI001/coda-website) • [Install Extension](https://marketplace.visualstudio.com/items?itemName=express-genix.coda-ai) • [Report Issue](https://github.com/LambdaAI001/coda-marketplace-assets/issues) • [Security Advisory](https://github.com/LambdaAI001/coda-marketplace-assets/security/advisories/new)

</div>

---

## 📌 About This Repository

This repository is the **official public community hub, issue tracker, release notes registry, and documentation portal** for **Coda** (`express-genix.coda-ai`).

- **Bugs & Issues:** [Open a Bug Report](https://github.com/LambdaAI001/coda-marketplace-assets/issues)
- **Feature Requests:** [Propose a Superpower](https://github.com/LambdaAI001/coda-marketplace-assets/issues)
- **Security Vulnerabilities:** [Submit Private Disclosure](https://github.com/LambdaAI001/coda-marketplace-assets/security/advisories/new)
- **Discussions & Feedback:** [GitHub Discussions](https://github.com/LambdaAI001/coda-marketplace-assets/discussions)

---

## 🚀 Why Coda?

Most AI coding assistants are simple chat wrappers inside an iframe or basic prompt-to-regex loops that hallucinate call graphs, conflate file reads with test executions, and silently corrupt context.

**Coda is an industrial-grade, autonomous software engineering harness engineered natively for the VS Code extension host.** It combines:

1. **Semantic LSP Intelligence:** Language Server Protocol integration (`goToDefinition`, `findReferences`, AST refactoring) instead of blind regex grep.
2. **Contractual Verification:** A strict Delivery Ledger that checks actual exit codes (`tsc`, `lint`, `tests_run`) rather than trusting conversational claims.
3. **Defense-in-Depth Safety:** Protected paths (`.env`, `.git/`, lockfiles), recursive secret scrubbing, and command risk tiers.
4. **Sub-Agent Delegation:** Isolated explorer sub-agents on cheap model tiers producing bounded 4-part distillation summaries to prevent parent context pollution.
5. **Native IDE Superpowers:** Official Jupyter Notebook tooling (`NotebookEdit`), adapter-aware DAP debugger inspection, and safe loopback browser preview.

---

## ⚡ Quick Start

### 1. Install via VS Code Command Palette
Press `Ctrl+P` (or `Cmd+P` on macOS) and run:
```bash
ext install express-genix.coda-ai
```

Or open directly in VS Code:
```
vscode:extension/express-genix.coda-ai
```

### 2. Configure Your Provider
Open the Coda panel from the Activity Bar and pick from **35+ supported providers**:
- **Commercial:** Anthropic (Claude 3.7 / 3.5 Sonnet), OpenAI (GPT-4.5, o3-mini), Google Gemini 2.0, DeepSeek R1, Groq, Cerebras.
- **Local & Private (100% Offline):** Ollama, LM Studio, vLLM, Venice AI, Generic OpenAI-compatible.
- **Enterprise Cloud:** AWS Bedrock, Azure OpenAI, Google Vertex AI, Snowflake Cortex.
- **Zero Setup:** Native VS Code Language Models (`vscode-lm` / GitHub Copilot subscription).

API keys are stored securely using VS Code's native `SecretStorage` API — never logged or exfiltrated.

---

## 🛡️ Verification Gate Architecture

```
┌─────────────────────────────────────────────────────────────┐
│  ✓ Delivery Ledger:                                         │
│  [✓ tsc --noEmit: exit 0]                                   │
│  [✓ vitest run tests/auth: 42 passed]                       │
│  [✓ secrets: recursively scrubbed]                          │
│  [✓ diff: +18 -4 across 2 files]                            │
└─────────────────────────────────────────────────────────────┘
```

Coda never marks a task satisfied based on model assertions. It executes authoritative project commands, validates process exit codes, and audits git state before reporting completion.

---

## 📦 Community Links & Governance

- **Extension Marketplace:** [express-genix.coda-ai](https://marketplace.visualstudio.com/items?itemName=express-genix.coda-ai)
- **Official Documentation Website:** [coda-website](https://github.com/LambdaAI001/coda-website)
- **Official Live Changelog:** [VS Code Marketplace Changelog](https://marketplace.visualstudio.com/items/express-genix.coda-ai/changelog)
- **Security Policy:** [SECURITY.md](SECURITY.md)
- **Contributing Guidelines:** [CONTRIBUTING.md](CONTRIBUTING.md)

---

## 📄 License

Coda is open-source software released under the **[MIT License](LICENSE)**.

---

<div align="center">
  <sub>Visual Studio Code and "VS Code" are registered trademarks of Microsoft Corporation. Coda is an independent project and is not affiliated with, sponsored, or endorsed by Microsoft.</sub>
</div>
