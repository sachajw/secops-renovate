# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Renovate is an automated dependency update tool that creates pull requests to update dependencies in software projects. It supports 90+ package managers and multiple VCS platforms (GitHub, GitLab, Bitbucket, Azure DevOps, Gitea, Forgejo, Gerrit).

**Key Characteristics:**
- Stateless design (repository state is source of truth, no database)
- Synchronous repository processing (to avoid API rate limits)
- Cascading configuration system (specific overrides general)
- 100% test coverage requirement
- License: AGPL-3.0-only

## Prerequisites

- Node.js ^24.11.0 (use Volta or .nvmrc)
- pnpm ^10.0.0
- Git >=2.45.1
- C++ compiler (for native dependencies)

## Common Commands

```bash
pnpm install          # Install dependencies
pnpm build            # Build the project (clean, generate, compile)
pnpm test             # Run all checks (lint + type-check + tests with coverage)
pnpm check            # Quick local CI check (recommended during development)
pnpm vitest <path>    # Run specific tests (e.g., pnpm vitest platform/gitlab)
pnpm vitest <file> -t "<test name>"  # Run specific test
pnpm lint-fix         # Auto-fix linting issues
pnpm start            # Run Renovate CLI
pnpm debug            # Run with Chrome debugger
```

## Project Structure

```
lib/
├── config/           # Configuration parsing and options
├── modules/
│   ├── datasource/   # Package registry integrations (npm, PyPI, Maven, etc.)
│   ├── manager/      # Package manager implementations (npm, pip, Docker, etc.)
│   ├── platform/     # VCS platform integrations (GitHub, GitLab, Bitbucket, etc.)
│   └── versioning/   # Versioning schemes (semver, pep440, etc.)
├── util/             # Utilities (HTTP, cache, git, fs, etc.)
├── workers/
│   ├── global/       # Entry point, orchestration, autodiscovery
│   └── repository/   # Per-repository processing
└── renovate.ts       # Main entry point

test/                 # Test files (mirrors lib structure)
tools/                # Build and development tools
docs/                 # Documentation source
```

## Code Style Guidelines

### TypeScript Conventions
- Use **named exports** (no default exports)
- Prefer **function declarations** over arrow functions for named functions
- Use `interface` over `type` for TypeScript declarations
- Avoid enums (use unions or immutable objects instead)
- Prefer `satisfies` operator over `as` casts
- Use ES6 module syntax with `.ts` extension imports
- Avoid `Array()` constructor (use bracket notation or `Array.from`)

### Logging
- `DEBUG/INFO`: Inline metadata in message (e.g., `logger.debug(\`Generated branch: ${name}\`)`)
- `WARN/ERROR/FATAL`: Use structured metadata (e.g., `logger.warn({ presetName }, 'Failed to look up preset')`)

### HTTP Requests
Always use `Http` class from `util/http` for HTTP requests (handles auth and caching):
```ts
import { Http } from '../../../util/http';
const http = new Http('some-host-type');
const body = (await http.getJson<Response>(url)).body;
```

### Dates and Times
Use the `Luxon` package with UTC for time zone independence.

## Testing Guidelines

- 100% test coverage is required for all code
- Test files use `*.spec.ts` naming, placed adjacent to source
- Use `it.each` with tagged template literals for parameterized tests
- Prefer `vi.spyOn` for mocking over `vi.fn()` reassignment
- Use `Fixture` class for loading test fixtures (`Fixture.get()`, `Fixture.getJson()`)
- Avoid `toMatchSnapshot` except for huge objects/strings
- Separate Arrange/Act/Assert phases with newlines
- Use `v8 ignore next` comments only for truly unreachable code

## Architecture Notes

1. **Entry Point**: `lib/renovate.ts` -> Global worker -> Repository workers
2. **Module Pattern**: Each module (datasource, manager, platform, versioning) follows a consistent structure with `index.ts`, `types.ts`, and `*.spec.ts`
3. **Configuration**: All config options defined in `lib/config/options/index.ts`
4. **Test Sharding**: Tests are sharded via `tools/test/shards.ts` for CI parallelization

## Important Files

- `lib/config/options/index.ts` - All configuration options
- `docs/usage/configuration-options.md` - Configuration documentation
- `tools/test/shards.ts` - Test sharding configuration
- `docs/development/` - Development documentation

## AI Assistance Disclosure

Any AI assistance used in contributions must be disclosed in pull requests per the contributing guidelines.
