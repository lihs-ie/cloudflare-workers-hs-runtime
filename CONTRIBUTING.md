# Contributing

Use Node 24 LTS and pnpm 12. Install dependencies with `corepack pnpm install`, then run `pnpm check` before opening a pull request.

Use Conventional Commits. Branch names may start with `feat/`, `fix/`, `docs/`, `test/`, `chore/`, or `release/`. The `codex/` prefix is prohibited in this repository.

Public APIs require English TSDoc. Keep the English README canonical and update `README.ja.md` in the same change when user-facing behavior changes. Every `if` statement must use braces.

Do not commit `dist`, coverage output, packed tarballs, generated Worker types, or local Wrangler state.
