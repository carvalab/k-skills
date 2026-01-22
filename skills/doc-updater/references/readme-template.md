# README Update Template

Standard template for project README files.

## Template

```markdown
# Project Name

Brief description of what the project does.

## Setup

\`\`\`bash
# Installation
npm install

# Environment variables
cp .env.example .env.local
# Fill in required values

# Development
npm run dev

# Build
npm run build
\`\`\`

## Architecture

See [docs/CODEMAPS/INDEX.md](docs/CODEMAPS/INDEX.md) for detailed architecture.

### Key Directories

- `src/app` - Application entry points
- `src/components` - Reusable components
- `src/lib` - Utility libraries

## Features

- **Feature 1** - Description
- **Feature 2** - Description
- **Feature 3** - Description

## Documentation

- [Setup Guide](docs/GUIDES/setup.md)
- [API Reference](docs/GUIDES/api.md)
- [Architecture](docs/CODEMAPS/INDEX.md)

## Development

\`\`\`bash
# Run tests
npm test

# Lint
npm run lint

# Type check
npm run typecheck
\`\`\`

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md)

## License

MIT
```

## Section Guidelines

### Setup Section
- List actual commands from package.json
- Include all required environment variables
- Note any system dependencies (Node version, etc.)

### Architecture Section
- Link to codemaps, don't duplicate
- List key directories with one-line descriptions
- Keep concise - details belong in codemaps

### Features Section
- List user-facing features
- Use consistent formatting
- Link to relevant docs/guides

### Development Section
- Include all common dev commands
- Match package.json scripts exactly
- Note any gotchas or prerequisites

## Updating Process

1. Read current README.md
2. Compare with actual project state
3. Update Setup with current commands
4. Verify all links work
5. Update architecture if changed
6. Preserve any custom sections
