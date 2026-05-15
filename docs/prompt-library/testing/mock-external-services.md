---
id: mock-external-services
title: Mock External Services
sidebar_label: 🤖 Mock External Services
description: Replace external API calls and database interactions with test doubles.
---

:::note
Replace the placeholder prompt below with the version your team has validated. Add example output, tips, and any org-specific context (namespace, SDK language, coding standards).
:::

## Prompt

```
Create test doubles for the external services used in the following Temporal activities in [Python/Go/TypeScript/Java].

[Paste your activity code]

External dependencies to mock:
[List them, e.g.:
- HTTP client calling payments API (base URL: https://api.payments.internal)
- PostgreSQL database (via SQLAlchemy / GORM / pg)
- Redis cache client]

Requirements:
- Create reusable mock/stub implementations suitable for both unit and integration tests
- Mocks should support configuring success responses, error responses, and call assertions
- Include helper functions to set up common test scenarios
- Use the idiomatic mocking approach for [testing framework]
```

## Example output

> Add an example of what good output looks like for your team's use cases.

## Tips

- Prefer interface-based mocks over patching — they're easier to maintain as code evolves.
- Ask for a test fixture or factory to reduce setup boilerplate across test files.

## Related prompts

- [Unit Test Activities](./unit-test-activities.md)
- [Integration Testing](./integration-testing.md)
