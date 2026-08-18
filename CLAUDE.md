# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

@AGENTS.md

## Project

Arcade Vault — a platform for playing games online and competing for the highest score (per README.md, in Spanish). The codebase is currently the unmodified `create-next-app` scaffold (App Router, TypeScript, Tailwind v4); no game or vault features have been implemented yet.

This project follows Spec Driven Design via the `/spec` and `/spec-impl` skills from https://github.com/Klerith/fernando-skills (installed with `npx skills@latest add Klerith/fernando-skills`). Look for and use those skills when planning and implementing new features.

## Commands

- `npm run dev` — start the dev server
- `npm run build` — production build
- `npm run start` — run the production build
- `npm run lint` — run ESLint (flat config in `eslint.config.mjs`, based on `eslint-config-next`)

No test runner is configured yet.

## Skills

Usa siempre /frontend-design para diseñar la interfaz de usuario

## Architecture

- App Router under `app/` (`app/layout.tsx`, `app/page.tsx`). Path alias `@/*` maps to the repo root (see `tsconfig.json`).
- Styling via Tailwind CSS v4 through `@tailwindcss/postcss` (see `postcss.config.mjs`), with global styles in `app/globals.css`.
- **Next.js 16.3.1 has breaking changes from what you may know.** Before writing any Next.js code, read the relevant guide under `node_modules/next/dist/docs/` (`01-app`, `02-pages`, `03-architecture`, `04-community`) and heed deprecation notices — see `AGENTS.md` for details.
