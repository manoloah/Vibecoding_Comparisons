# CreateAnyting_GoalTracker Deep Dive

## Stack & Architecture
- Hybrid monorepo containing Expo app (`apps/mobile`) and Bun-based web app (`apps/web`), both JavaScript-first.
- Heavy reliance on generated scaffolding from CreateAnything: polyfills, patches, and sandbox messaging (`window.parent.postMessage`) to interface with the authoring environment.
- Mobile UI implemented in plain React Native components but imports CSS files and browser-specific utilities.

## Code Quality
- Inconsistent typing (JS + TS mix), large inline style objects, limited modularization.
- Generated plumbing (`__create` folders, sandbox error boundaries) increases cognitive load with little business value.
- Network calls hard-coded to relative `/api/...` paths and invoked directly inside components without abstraction.

## Backend & Data Layer
- Assumes a co-located web server providing `/api` endpoints (present in `apps/web`) but mobile code never configures a base URL or auth.
- No shared types; responses assumed via implicit shapes.
- Absence of offline storage or caching strategy on the mobile side.

## Scalability & Maintainability
- Dependency list is bloated (audio, maps, video, 3D, Uploadcare, etc.) even though most features are unused—hurts bundle size and complicates upgrades.
- Direct DOM/window usage (`window.parent.postMessage`, `AlertModal` from web polyfills) will crash on native builds.
- Patch-package hooks for Expo CLI/runtime add maintenance risk; upgrades likely to break.

## Safety & Compliance
- No authentication, input validation, or secure storage; fetches rely on implicit server trust.
- Requests to `/api/...` without HTTPS configuration or token handling.
- Broad Android permissions (record/modify audio) without justification could hinder store review.

## Release & Store Readiness
- Expo SDK 53 is already deprecated; stores require recent SDKs.
- Native builds will fail because `window` references run on native thread.
- Missing bundle identifiers (`ios`) and production packages; requires trimming dependencies and rewriting web-only code.

## Overall
Generates a demo-friendly sandbox experience but not production-suitable. Significant refactor needed (remove sandbox coupling, define API clients, add typing, upgrade SDK) before considering app store submission.

