# Packaging & publishing ECC

This is the maintainer runbook for shipping ECC as an installable package. **Nothing here runs
automatically and nothing publishes on your behalf.** Publishing to a public registry is an
outward-facing act — you run the final commands, with your own npm credentials, after you have
verified the tarball yourself.

## What is already set up

- `package.json` is publish-ready: real package name (`engineering-context-compiler`), `main` +
  `types` + `exports` pointing at `dist/`, a `files` allowlist (only `dist/`, `README.md`,
  `LICENSE` ship — verified 80.7 kB tarball, 283 files, no source or tests), `bin` entries for the
  three CLIs (`ecc`, `ecc-mcp`, `ecc-pr-context`), `publishConfig.access: "public"`, and a
  `prepublishOnly` guard that re-runs `build` + `test` before any publish can proceed.
- `tsconfig.build.json` emits type declarations (`.d.ts`) and source maps into `dist/`, so
  TypeScript consumers get full types.
- `typescript` is a **devDependency** (build-time only); the runtime dependencies a consumer
  installs are just `@modelcontextprotocol/sdk` and `zod`.

## Name

The bare name `ecc` is already taken on npm (unrelated package). This package is therefore named
**`engineering-context-compiler`** (confirmed available on the public registry as of 2026-09-27).
The CLI binary names are independent of the package name and remain `ecc` / `ecc-mcp` /
`ecc-pr-context`. If you would rather publish under a scope you control, change `name` to
`@<your-npm-user>/engineering-context-compiler` and keep `publishConfig.access: "public"`.

## Verify locally before publishing (no registry involved)

```bash
npm ci                 # clean install from the lockfile
npm run build          # tsc -> dist/ (with .d.ts)
npm test               # vitest: expect 191/191 across 43 files
npm pack               # writes engineering-context-compiler-0.1.0.tgz; inspect the file list

# Install the packed tarball into a scratch dir and exercise the real bin:
mkdir -p /tmp/ecc-verify && cd /tmp/ecc-verify && npm init -y >/dev/null
npm install /path/to/engineering-context-compiler-0.1.0.tgz
npx ecc context "explain the memory retriever module" --path /path/to/some/repo --budget 800
```

`npm pack --dry-run` prints the exact tarball contents without writing a file — use it to confirm
only `dist/`, `README.md`, `LICENSE`, and `package.json` are included.

## Publish (your call — outward-facing)

```bash
npm login                       # your npm account
npm publish                     # prepublishOnly runs build + test first; publishConfig makes it public
```

Bump `version` (`npm version patch|minor|major`) before republishing — the registry rejects a
re-publish of an existing version. Tag a matching git release if you keep releases in sync.

## Use as an MCP server

After a global install (`npm install -g engineering-context-compiler`), the `ecc-mcp` binary is a
stdio MCP server exposing one tool, `compile_engineering_context` (inputs: `task`, optional `path`,
optional `tokenBudget`). Register it with any MCP client — e.g. Claude Desktop:

```json
{
  "mcpServers": {
    "ecc": { "command": "ecc-mcp" }
  }
}
```

Without a global install, point `command` at the built entry instead:
`"command": "node", "args": ["/absolute/path/to/dist/mcp/index.js"]`.

The server is deterministic and makes no network calls; it reads only the repository `path` passed
to the tool (defaulting to the server process's working directory).

## Honest status

ECC has **zero external adoption** and has not been published to npm. This runbook makes it
*publishable and locally installable*; it does not manufacture users. Whether/when to publish, and
under what name and version, is the maintainer's decision.
