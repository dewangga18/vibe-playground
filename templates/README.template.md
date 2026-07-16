# README Template — Instructions for the Agent

Generate `README.md` at the root of the current project.

## Steps

**1. Check for existing file**
- If `README.md` already exists, show the current content and ask:
  > "README.md already exists. Overwrite?"
- Wait for explicit confirmation before continuing.

**2. Collect info**
- Read `./AGENTS.md` if it exists — Stack, Commands, and Folder structure are already there.
- Scan the project root for additional context: `package.json` (name, description, scripts),
  `go.mod`, `Cargo.toml`, `pyproject.toml`, `Makefile`, etc.
- For requirements and troubleshooting, prefer what's documented over what you infer.

**3. Generate `README.md`**

Use the format below. Skip "Schemes / Build Configuration" if the project has no build variants.
Keep it factual — no marketing language, no repeating what's in AGENTS.md.


# \<Project Name>

<One-line tagline: what it is, what stack it's built with>

## About
What this project does and its main feature(s). 2-4 sentences.

## Requirements
- <e.g. Node.js 20+, Go 1.22+, Xcode 16 / iOS 17 SDK, Docker 24+>
- <package manager version, e.g. pnpm 9+>
- <any account/access needed, e.g. VPN, API keys>

## Getting Started
```bash
git clone <repo-url>
cd <project>
<install command>
<setup/config command>  # e.g. cp .env.example .env
<run command>
```

## Project Structure
```
├── <entry-point>/     # where execution starts
├── <core-modules>/    # main business logic
├── <shared>/          # shared components/utils
├── <config>/          # environment/build config
└── <infra>/           # CI, Docker, build tooling
```

## Dependencies
- Install: `<command>`
- Add: `<command>`
- Update: `<command>`

## Schemes / Build Configuration *(omit if not applicable)*

| Scheme / Variant | Target | Config |
| --- | --- | --- |
| `<name>` | `<target>` | `<Debug/Release, env file>` |

## Troubleshooting
- <common issue → fix>
