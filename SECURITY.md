# Security Policy

## Supported versions

Only the latest published version of Coda AI on the
[VS Code Marketplace](https://marketplace.visualstudio.com/items?itemName=express-genix.coda-ai)
receives security fixes.

## Reporting a vulnerability

Please do not report security issues in public GitHub issues.

Report them privately using GitHub's
[private vulnerability reporting](https://github.com/LambdaAI001/coda-marketplace-assets/security/advisories/new).

Include as much of the following as you can:

- A description of the issue and its potential impact
- Steps to reproduce, or a proof of concept
- The Coda AI version, VS Code version and operating system

## What to expect

- We aim to acknowledge reports within 5 business days.
- We will keep you updated while we investigate and work on a fix.
- Once a fix is released, we are happy to credit you in the changelog
  unless you prefer to stay anonymous.

## Scope

Examples of issues we want to hear about:

- Bypassing approval gates for deletions or unsafe commands
- Access to protected paths such as keys, credentials or `.env` files
- Leaking API keys or code to anything other than the configured provider
- Bypassing the network egress allow/deny policy
- Unsafe handling of imported skills or MCP configuration
