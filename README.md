# Temporal Platform Hub

A foundational template for Platform teams to create an internal Temporal knowledge base. Built with [Docusaurus](https://docusaurus.io/).

**See it live:** [go.temporal.io/platform-hub](https://go.temporal.io/platform-hub)

## Why this exists

Platform teams adopting Temporal often face the same challenges:

- **Onboarding friction** - Engineers spend weeks learning Temporal concepts scattered across docs, Slack threads, and tribal knowledge
- **Repeated support questions** - Platform teams answer the same "how do I..." questions repeatedly
- **Inconsistent patterns** - Without clear guidance, teams reinvent solutions and create tech debt
- **Missing organizational context** - Official Temporal docs are excellent, but don't cover your Namespace conventions, deployment patterns, or compliance requirements

Temporal Platform Hub provides a customizable starting point to address these challenges.

## What's included

| Section | Description |
|---------|-------------|
| [Temporal Overview](docs/temporal-overview.md) | Customizable intro to Temporal with space for your org's metrics and use cases |
| [Decision Framework](docs/decision-framework.md) | Help developers determine if Temporal fits their use case |
| [Getting Started](docs/getting-started.md) | Environment setup and first Workflow guide |
| [Learning Paths](docs/learning-path.md) | Structured progression from basics to advanced patterns |
| [Architecture](docs/architecture.md) | Document your Namespace conventions, Worker deployment, and connectivity |
| [Cost](docs/cost.md) | Temporal Cloud pricing model and cost optimization |
| [Shared Responsibility](docs/shared-responsibility.md) | Define team responsibilities for Temporal applications |
| [Patterns](docs/patterns.md) | Common Workflow design patterns with code samples |
| [Troubleshooting](docs/troubleshooting.md) | Observability and debugging guidance |
| [Support](docs/support.md) | Support model and escalation paths |
| [FAQs](docs/faqs.md) | Common questions and answers |

Each page includes instructions to help you adapt the content for your organization.

## Use this template

1. Fork this repository
2. Update `docusaurus.config.ts` with your organization's details
3. Replace "ABC Financial" references throughout the docs with your organization
4. Follow the `:::note` banners on each page for specific customization guidance

## Getting started

### Prerequisites

- Node.js 18+
- yarn

### Installation

```bash
yarn install
```

### Local development

```bash
yarn start
```

This starts a local dev server at `http://localhost:3000`. Changes are reflected live without restarting.

### Deploy

```bash
USE_SSH=true GIT_USER=<USERNAME> yarn deploy
```

Build and deploy the changes to GitHub Pages.

## License

MIT
