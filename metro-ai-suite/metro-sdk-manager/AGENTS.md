# AGENTS.md — OEP SDK Manager

You are an AI assistant working on the OEP SDK Manager project. This document provides instructions, patterns, and boundaries for making changes to this codebase.

## Project Overview

OEP SDK Manager is a comprehensive development tool that streamlines discovering, installing, and managing multiple SDKs for Intel edge AI applications. It consists of two main parts:

1. **Documentation UI** (`docs/`) — A Sphinx-based documentation site with an embedded interactive SDK selector built with HTML, CSS, and JavaScript. Includes tutorial sections for each SDK.
2. **Installation Scripts** (`scripts/`) — Bash scripts that install each SDK by pulling Docker images and cloning source repositories from GitHub.

### SDKs

| SDK | Install Script | Tutorials | Description |
|:----|:---------------|:----------|:------------|
| OEP Vision AI SDK | `scripts/oep-vision-ai-sdk.sh` | `docs/user-guide/oep-vision-ai-sdk/` | DLStreamer, OpenVINO, Pipeline Server, SceneScape — visual AI inference & spatial analytics |
| OEP Gen AI SDK | `scripts/oep-gen-ai-sdk.sh` | `docs/user-guide/oep-gen-ai-sdk/` | Audio Analyzer, VLM serving, embedding, document ingestion |
| Visual AI Demo Kit | `scripts/visual-ai-demo-kit.sh` | `docs/user-guide/visual-ai-demo-kit/` | DLStreamer Pipeline Server, Node-RED, Grafana, MQTT, MediaMTX |

Each SDK installs Docker container images and clones GitHub repositories (`edge-ai-libraries`, `edge-ai-suites`).

## Repository Structure

```
metro-sdk-manager/
├── docs/                              # Sphinx documentation source
│   ├── _static/
│   │   ├── installer/
│   │   │   ├── selector.html          # Interactive SDK selector UI
│   │   │   ├── style.css              # Selector styles
│   │   │   ├── config.js              # SDK options, versions, install commands
│   │   │   └── iframe-resizer.js      # Responsive iframe helper
│   │   ├── logo.svg
│   │   └── one-edge-platform-login-title.png
│   └── user-guide/
│       ├── index.md                   # Landing page (embeds installer iframe)
│       ├── release-notes.md
│       ├── oep-vision-ai-sdk/       # Vision SDK tutorials (get-started + tutorials 1-6)
│       ├── oep-gen-ai-sdk/          # Gen AI SDK tutorials (get-started)
│       └── visual-ai-demo-kit/        # Demo Kit tutorials (get-started + tutorials 1-3)
├── scripts/
│   ├── oep-vision-ai-sdk.sh         # Vision SDK installer
│   ├── oep-gen-ai-sdk.sh            # Gen AI SDK installer
│   └── visual-ai-demo-kit.sh          # Visual AI Demo Kit installer
├── .gitignore
├── README.md
└── AGENTS.md                          # This file
```

> **Note:** Files such as `Makefile`, `requirements.txt`, `VERSION`, `dict.txt`, `docconf/`, `docs/conf.py`, and `docs/substitutions.txt` are generated/managed by the CI pipeline and listed in `.gitignore`. They are not tracked in this repo.

## Tech Stack

- **Documentation**: Sphinx 8.2, pydata-sphinx-theme, MyST Markdown
- **Installer UI**: Vanilla HTML/CSS/JS (embedded via iframe in Sphinx output)
- **Install Scripts**: Bash (strict mode: `set -euo pipefail`)
- **Docker Images**: Each SDK defines its images array in the install script
- **Source Repos**: GitHub repos (edge-ai-libraries, edge-ai-suites) cloned at specific release branches
- **Linting**: ShellCheck, markdownlint, doc8, pylint, black, yamllint, reuse (license), Trivy
- **Build System**: GNU Make with Python virtualenv (provided by CI; `Makefile` and `requirements.txt` are gitignored)

## Core Commands

> **Note:** The `Makefile`, `requirements.txt`, and Sphinx config files are provided by CI and are gitignored.
> Build and lint commands (`make build`, `make lint`, etc.) are only available in the CI pipeline.
> Locally, you can run the install scripts and lint with standalone tools as shown below.

### Running Install Scripts

```bash
# Install a specific SDK
bash scripts/oep-vision-ai-sdk.sh
bash scripts/oep-gen-ai-sdk.sh
bash scripts/visual-ai-demo-kit.sh

# Skip specific steps
bash scripts/oep-vision-ai-sdk.sh --skip-system-check --skip-docker --skip-images --skip-git-clone

# Show help for any script
bash scripts/oep-vision-ai-sdk.sh --help
```

### Local Linting (standalone tools)

```bash
# Lint shell scripts
shellcheck scripts/*.sh

# Lint Markdown files
markdownlint-cli2 "**/*.md"
```

## Code Style & Conventions

### Shell Scripts (`scripts/`)

- Shebang: `#!/bin/bash`
- Strict mode: `set -euo pipefail` at the top
- Quote all variable expansions: `"${var}"` not `$var`
- Use Google Shell Style Guide comment headers for functions
- Each script defines its `repositories=()` and `images=()` arrays at the top
- Support `--skip-system-check`, `--skip-docker`, `--skip-images`, `--skip-git-clone`, `--help` flags
- Color-coded output using `info()`, `warn()`, `err()`, `success()` helper functions

### Documentation (`docs/`)

- Write in MyST Markdown (`.md` files)
- Use ATX-style headings (`#`, `##`, `###`)
- One sentence per line for cleaner diffs
- Images go in `images/` subdirectory next to the tutorial
- Use `<!--hide_directive ... hide_directive-->` for Sphinx-only directives that should not render on GitHub

### Installer UI (`docs/_static/installer/`)

- Vanilla JS — no frameworks or bundlers
- All SDK options, versions, and install commands are defined in `config.js`
- `selector.html` is embedded as an iframe in the Sphinx docs landing page
- Keep the UI self-contained with no external HTTP fetches

### License Headers

All new source files must include:

```
# SPDX-FileCopyrightText: (C) 2025 Intel Corporation
# SPDX-License-Identifier: Apache-2.0
```

Use the language-appropriate comment syntax (`#`, `//`, `/* */`).

## Adding a New SDK

1. Create the install script in `scripts/<sdk-name>.sh` following the existing pattern (define `repositories`, `images`, `NAME`).
2. Add a tutorial directory under `docs/user-guide/<sdk-name>/` with at minimum `get-started.md`.
3. Add the SDK entry to `docs/_static/installer/config.js` (options, components, and install commands).
4. Register the tutorial in the toctree in `docs/user-guide/index.md`.

## Security Rules

- **Never** commit secrets, tokens, passwords, or API keys.
- Docker image tags must reference specific versions, not `latest` or mutable tags.
- All dependencies must be compatible with Apache-2.0.
- Install scripts must validate inputs and fail safely on errors (`set -euo pipefail`).

## File Modification Rules

1. When updating Docker image versions in `scripts/`, also update the corresponding component lists in `docs/_static/installer/config.js`.
2. Do not add gitignored build/config files (`Makefile`, `requirements.txt`, `VERSION`, `docconf/`, `docs/conf.py`, etc.) — those are managed by CI.
3. Test install scripts with `--skip-images` for a dry-run validation.
