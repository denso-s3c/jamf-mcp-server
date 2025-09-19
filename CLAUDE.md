# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Model Context Protocol (MCP) server that enables AI assistants to interact with Jamf Pro for Apple device management. The server provides comprehensive device management capabilities through the Jamf Pro API.

## Essential Commands

### Development

```bash
npm run dev          # Run in development mode with hot reload
npm run build        # Compile TypeScript to JavaScript (dist/)
npm run serve        # Run production build
npm run inspector    # Launch MCP Inspector for debugging
```

### Testing

```bash
npm test             # Run all tests
npm test -- --coverage  # Run tests with coverage report
npm test -- path/to/test.spec.ts  # Run specific test file
```

### Linting

```bash
npm run lint         # Run ESLint on TypeScript files
npm run lint:fix     # Auto-fix linting issues
```

## Architecture

### Client Architecture Evolution

The codebase has multiple Jamf client implementations to handle different API versions and error scenarios:

1. **jamf-client.ts**: Modern API v2 client (primary)
2. **jamf-client-classic.ts**: Classic API v1 client (legacy endpoints)
3. **jamf-client-hybrid.ts**: Combines Modern + Classic APIs (seamless fallback)
4. **jamf-client-enhanced.ts**: Enhanced error handling with retry logic

The server automatically selects the appropriate client based on the `JAMF_ENHANCED_MODE` environment variable.

### MCP Integration Structure

- **Tools** (`src/tools/`): Define callable functions exposed to AI assistants
- **Resources** (`src/resources/`): Provide data sources that can be read
- **Prompts** (`src/prompts/`): Pre-defined workflow templates for common tasks

### Authentication Flow

The server supports two authentication methods:

1. **OAuth2**: Uses client credentials flow with automatic token refresh
2. **Basic Auth**: Username/password authentication (legacy)

Authentication is configured via environment variables and automatically selected based on available credentials.

### Error Handling Strategy

- Custom error classes in `src/utils/errors.ts` for typed error handling
- Axios interceptors for automatic retry on transient failures
- Enhanced mode includes exponential backoff and circuit breaker patterns

## Key Implementation Notes

### API Version Handling

When making API calls, the hybrid client automatically falls back from Modern API to Classic API if an endpoint is not available. This is critical for compatibility as Jamf Pro transitions between API versions.

### Testing Approach

- Unit tests mock the Jamf API responses using fixtures in `src/__tests__/fixtures/`
- Integration tests require a live Jamf Pro instance (skipped in CI)
- Coverage thresholds are enforced: 80% for functions/lines/statements, 70% for branches

### Configuration

- Local development: Use `.mcp.json` for API credentials (git-ignored)
- Production: Use environment variables (JAMF_SERVER_URL, JAMF_CLIENT_ID, etc.)
- Claude Desktop: Configure via `.claude/settings.local.json`

### Common Pitfalls

- Many Jamf API endpoints have different response formats between v1 and v2
- Device IDs can be strings or numbers depending on the API version
- Rate limiting may occur with frequent API calls - enhanced mode handles this automatically
- OAuth token refresh must be handled before expiration (built into enhanced client)

## Development Workflow

When adding new Jamf functionality:

1. Check if the endpoint exists in both Modern and Classic APIs
2. Implement in the appropriate client file(s)
3. Add the tool definition in `src/tools/index.ts`
4. Include proper Zod validation for parameters
5. Add tests with mocked API responses
6. Update the README.md with usage examples if it's a major feature

## MCP Server Specifics

This server runs as an MCP server that can be integrated with:

- Claude Desktop (via `.claude/settings.local.json`)
- Any MCP-compatible client

The server exposes ~50 tools for device management, policy deployment, script execution, and reporting. Each tool includes built-in parameter validation and confirmation requirements for destructive operations.
