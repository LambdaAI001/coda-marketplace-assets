# Coda — AI Coding Agent for VS Code

<div align="center">

<img src="coda.png" alt="Coda Logo" width="120" />

### An AI coding agent for VS Code that checks its own work

[![Marketplace Version](https://img.shields.io/visual-studio-marketplace/v/express-genix.coda-ai?color=34d399&label=Marketplace&logo=visual-studio-code)](https://marketplace.visualstudio.com/items?itemName=express-genix.coda-ai)
[![Installs](https://img.shields.io/visual-studio-marketplace/i/express-genix.coda-ai?color=38bdf8)](https://marketplace.visualstudio.com/items?itemName=express-genix.coda-ai)
[![Rating](https://img.shields.io/visual-studio-marketplace/r/express-genix.coda-ai?color=fbbf24)](https://marketplace.visualstudio.com/items?itemName=express-genix.coda-ai)
[![License: MIT](https://img.shields.io/badge/License-MIT-emerald.svg)](LICENSE)
[![Official Website](https://img.shields.io/badge/Website-coda--ai.dev-purple)](https://www.coda-ai.dev)
[![Security Policy](https://img.shields.io/badge/Security-Policy-blue)](SECURITY.md)

[Official Website](https://www.coda-ai.dev) • [Install Extension](https://marketplace.visualstudio.com/items?itemName=express-genix.coda-ai) • [Report Issue](https://github.com/LambdaAI001/coda-marketplace-assets/issues) • [Security Advisory](https://github.com/LambdaAI001/coda-marketplace-assets/security/advisories/new)

</div>

---

## 📌 About This Repository

This repository is the public home of **Coda** (`express-genix.coda-ai`): issues, security reports, discussions, and the [release notes](CHANGELOG.md) for every version.

- **Bugs & Issues:** [Open a Bug Report](https://github.com/LambdaAI001/coda-marketplace-assets/issues)
- **Feature Requests:** [Suggest a feature](https://github.com/LambdaAI001/coda-marketplace-assets/issues)
- **Security Vulnerabilities:** [Submit Private Disclosure](https://github.com/LambdaAI001/coda-marketplace-assets/security/advisories/new)
- **Discussions & Feedback:** [GitHub Discussions](https://github.com/LambdaAI001/coda-marketplace-assets/discussions)

---

## 🚀 Why Coda?

Coda is an AI coding agent that runs inside VS Code. It edits your code, then checks its work: it runs your project's build and tests after changing code and treats a failing exit code as something to fix, not a success to report.

1. **Semantic LSP Intelligence:** Language Server Protocol integration (`goToDefinition`, `findReferences`, AST refactoring) instead of blind regex grep.
2. **Contractual Verification:** A strict Delivery Ledger that checks actual exit codes (`tsc`, `lint`, `tests_run`) rather than trusting conversational claims.
3. **Defense-in-Depth Safety:** Protected paths (`.env`, `.git/`, lockfiles), recursive secret scrubbing, and command risk tiers.
4. **Sub-Agent Delegation:** Isolated explorer sub-agents on cheap model tiers producing bounded 4-part distillation summaries to prevent parent context pollution.
5. **Built on VS Code:** Jupyter notebooks through VS Code's notebook API, debugger inspection through the Debug Adapter Protocol, and a review of every change (Keep or Undo per change, file or all) above the chat box and in the editor.

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
Open the Coda panel from the Activity Bar and pick one of **37 providers**, then a model from that provider's own list:
- **Popular:** OpenAI, Anthropic, Mistral, Groq, xAI, DeepSeek, OpenRouter, GitHub Copilot.
- **Free and local:** Google Gemini (free tier), Ollama (runs on your machine, no key), Ollama Cloud, Pollinations, VS Code Models (your Copilot subscription), OmniRoute.
- **Enterprise and cloud:** AWS Bedrock, Azure OpenAI, Google Vertex AI, Vercel AI Gateway, Cloudflare, GitLab Duo, SAP AI Core, Snowflake Cortex.
- **More:** Cohere, Together AI, Fireworks AI, Perplexity, Cerebras, Alibaba Cloud, DeepInfra, NVIDIA NIM, Venice, LLM Gateway; and any OpenAI-compatible server (LM Studio, vLLM and others).

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
- **Official Website:** [coda-ai.dev](https://www.coda-ai.dev)
- **Release notes:** [CHANGELOG.md](CHANGELOG.md) (also on the [Marketplace's Changelog tab](https://marketplace.visualstudio.com/items/express-genix.coda-ai/changelog))
- **Security Policy:** [SECURITY.md](SECURITY.md)
- **Contributing Guidelines:** [CONTRIBUTING.md](CONTRIBUTING.md)

---

## 📄 License

Coda is free software released under the **[MIT License](LICENSE)**.

---

<div align="center">
  <sub>Visual Studio Code and "VS Code" are registered trademarks of Microsoft Corporation. Coda is an independent project and is not affiliated with, sponsored, or endorsed by Microsoft.</sub>
</div>
