# Anthropic Quickstarts Development Guide

This repository contains multiple independent quickstart projects demonstrating Claude's capabilities. Each project has its own dependencies, tooling, and conventions.

## Repository Structure

```
anthropic-quickstarts/
├── agents/                    # Minimal Python agent reference implementation (~300 LOC)
├── computer-use-demo/         # Claude computer control demo (Python + Docker)
├── customer-support-agent/    # Customer support chatbot (Next.js)
├── financial-data-analyst/    # Financial analysis platform (Next.js)
├── .github/workflows/         # CI/CD: Docker builds + tests for computer-use-demo
├── pyproject.toml             # Root pyright config pointing to computer-use-demo venv
└── .pre-commit-config.yaml    # Pre-commit hooks (ruff + pyright) for computer-use-demo
```

---

## Agents (`/agents/`)

Minimal, educational Python implementation of an LLM agent loop (~300 lines). Not an SDK — meant to be read and extended.

### Key Files
- `agent.py` — Core agent loop
- `tools/` — Tool management and execution
- `utils/` — Message history utilities
- `agent_demo.ipynb` — Jupyter notebook demo

### Setup
```bash
pip install anthropic mcp
```

### Code Style
- Python 3.8+ compatible
- Supports native tools and MCP (Model Context Protocol) server tools

---

## Computer-Use Demo (`/computer-use-demo/`)

Reference implementation of Claude's computer control capabilities. Runs inside a Docker container with a full desktop environment (Xvfb, X11vnc, noVNC, LibreOffice, Firefox).

### Setup & Development
```bash
./setup.sh                        # Create venv, install deps, set up pre-commit
docker build . -t computer-use-demo:local
docker run \
  -e ANTHROPIC_API_KEY=$ANTHROPIC_API_KEY \
  -v $(pwd)/computer_use_demo:/home/computeruse/computer_use_demo/ \
  -v $HOME/.anthropic:/home/computeruse/.anthropic \
  -p 5900:5900 -p 8501:8501 -p 6080:6080 -p 8080:8080 \
  -it computer-use-demo:local
```

### Ports
| Port | Service |
|------|---------|
| 8501 | Streamlit UI |
| 8080 | Combined landing page |
| 5900 | VNC |
| 6080 | noVNC (browser-based) |

### Testing & Code Quality
```bash
ruff check .          # Lint
ruff format .         # Format
pyright               # Type check
pytest                # Run all tests
pytest tests/path_to_test.py::test_name -v  # Run single test
```

### Ruff Configuration (`ruff.toml`)
- **Selected rules**: A, ASYNC, B, E, F, I, PIE, RUF200, T20, UP, W
- **Ignored**: E501 (line length), ASYNC230
- **isort**: `combine-as-imports = true`
- **Format**: docstring code formatting enabled

### Code Style
- **Python**: snake_case for functions/variables, PascalCase for classes
- **Imports**: isort with `combine-as-imports`
- **Error handling**: Use `ToolError` / `ToolFailure` for tool errors
- **Types**: Add type annotations for all parameters and return values
- **Classes**: Use dataclasses for structured data, abstract base classes for interfaces
- **Comments**: Prefer self-documenting code; avoid inline comments

### Supported Models
- Claude Sonnet 4 (`claude-sonnet-4-5-20251022`, latest default)
- Claude Opus 4 (`claude-opus-4-5-20251022`)
- Supports Anthropic API, AWS Bedrock, and Google Vertex AI providers

### Docker Details
- **Base image**: Ubuntu 22.04
- **Python**: 3.11.6 (via pyenv)
- **Display**: Configurable WIDTH/HEIGHT (default 1024×768), DISPLAY_NUM=1
- **User**: `computeruse` (non-root with passwordless sudo)

---

## Customer Support Agent (`/customer-support-agent/`)

Full-featured AI-powered customer support chatbot built with Next.js 14 and the Anthropic SDK. Integrates with Amazon Bedrock knowledge bases.

### Setup & Development
```bash
npm install
cp .env.local.example .env.local   # Set ANTHROPIC_API_KEY (and AWS creds if using Bedrock)
npm run dev          # Full UI (both sidebars)
npm run dev:left     # Left sidebar only
npm run dev:right    # Right sidebar only
npm run dev:chat     # Chat area only
npm run lint
npm run build        # Full UI build
```

### Environment Variables (`.env.local`)
```
ANTHROPIC_API_KEY=your_key
# Optional for Bedrock knowledge base:
BAWS_ACCESS_KEY_ID=
BAWS_SECRET_ACCESS_KEY=
BAWS_REGION=
BAWS_BEDROCK_KNOWLEDGE_BASE_ID=
```

### Key Features
- Real-time thinking display and debug info
- User mood detection and escalation
- Switchable models: Claude 3 Haiku, Claude 3.5 Sonnet
- Amazon Bedrock knowledge base RAG integration
- AWS Amplify deployment ready

### Code Style
- **TypeScript**: Strict mode with proper interfaces
- **Components**: Function components with React hooks
- **UI library**: shadcn/ui components
- **Styling**: TailwindCSS
- **Linting**: ESLint extends `next/core-web-vitals`

### Deployment (AWS Amplify)
Requires IAM service role with `AmazonBedrockFullAccess` policy. See `README.md` for full Amplify YAML configuration.

---

## Financial Data Analyst (`/financial-data-analyst/`)

Interactive financial data analysis platform with Claude-powered insights and Recharts visualizations.

### Setup & Development
```bash
npm install
# Set ANTHROPIC_API_KEY in .env.local
npm run dev
npm run lint
npm run build
```

### Key Features
- Multi-format file uploads: TXT, CSV, PDF, images
- Interactive chart types: line, bar, multi-bar, area, stacked area, pie
- Claude 3 Haiku and 3.5 Sonnet model support
- Edge runtime optimization
- PDF parsing via PDF.js

### Code Style
- **TypeScript**: Strict mode with proper type definitions
- **Components**: Function components with type annotations
- **Visualization**: Recharts 2.x library
- **State management**: React hooks
- **Linting**: ESLint extends `next` with custom rules (no HTML entities, no next/font)

---

## CI/CD (`.github/workflows/`)

### `tests.yaml` — Triggered on PRs and pushes to `main` affecting `computer-use-demo/`
1. **Ruff** lint check (`astral-sh/ruff-action@v1`)
2. **Pyright** type checking (Python 3.11.6, installs `dev-requirements.txt`)
3. **Pytest** unit tests with JUnit XML output

### `build.yaml` — Docker multi-platform builds
- Builds `amd64` and `arm64` images in parallel
- On PR: builds only (no push)
- On push to `main`: pushes to `ghcr.io/anthropics/anthropic-quickstarts` with short SHA + `latest` tags
- Health checks: Streamlit (:8501), VNC (:5900), noVNC (:6080), landing page (:8080)

---

## General Conventions

### Git
- Signed commits required (see `CONTRIBUTING.md`)
- PRs should use the provided pull request template

### Python Version
- **Development/CI**: Python 3.11.6 (pinned via pyenv for computer-use-demo)
- **Minimum supported**: Python 3.8 (agents project)
- Pyright venv path: `computer-use-demo/.venv`

### Node.js Version
- **Minimum**: 18.17.0 (required by Next.js 14)

### Adding a New Quickstart
1. Create a new top-level directory
2. Add its own `README.md`, dependency files, and lint configuration
3. Update this `CLAUDE.md` and the root `README.md`
4. Add CI/CD workflows as needed

### Claude API Usage
All projects use the `@anthropic-ai/sdk` (Node.js) or `anthropic` (Python) packages. Default to the latest capable models:
- **Claude Sonnet 4.6**: `claude-sonnet-4-6` (recommended default)
- **Claude Opus 4.6**: `claude-opus-4-6`
- **Claude Haiku 4.5**: `claude-haiku-4-5-20251001`
