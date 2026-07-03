# syntax=docker/dockerfile:1
# Vexa Terminal (v0.12) — Next.js 15 app with a CUSTOM server (server.mjs) that proxies the `/ws`
# WebSocket upgrade to the gateway server-side. Because the server is custom (not Next's standalone
# server.js), we ship the pruned production node_modules rather than `output: standalone` — server.mjs
# imports `next`, `@next/env`, and `ws`, which a standalone trace would not all include.
#
# Build context: the REPO ROOT (compose: `context: ../..`), installed with pnpm against the
# workspace lockfile — the terminal is a first-class workspace member and can depend on
# `@vexa/*` bricks (add `transpilePackages` in next.config.ts when importing one). The context
# is trimmed by the adjacent Dockerfile.dockerignore (whitelist: lockfile + workspace manifests
# + clients/terminal). The old per-app package-lock.json/npm build is gone.

# node:22 — tinykeys@4 requires node >=22; pin to match engines. pnpm pinned to the repo's
# packageManager version (installed via npm — no corepack network/signature surprises).
FROM node:22-slim AS build
RUN npm i -g pnpm@11.7.0
WORKDIR /repo
COPY pnpm-lock.yaml pnpm-workspace.yaml package.json ./
# workspace manifests only (Dockerfile.dockerignore whitelists package.json under core/) —
# pnpm needs every importer present to honor --frozen-lockfile.
COPY core core
COPY clients/terminal clients/terminal
RUN pnpm install --frozen-lockfile --filter @vexa/terminal
# Google Analytics measurement id (G-XXXXXXXXXX). NEXT_PUBLIC_* is inlined into the client bundle at
# BUILD time, so it must be present BEFORE `next build` (pass via --build-arg / compose build.args).
ARG NEXT_PUBLIC_GA_MEASUREMENT_ID=""
ENV NEXT_PUBLIC_GA_MEASUREMENT_ID=$NEXT_PUBLIC_GA_MEASUREMENT_ID
RUN pnpm --filter @vexa/terminal build

FROM node:22-slim AS runtime
ENV NODE_ENV=production \
    PORT=3000 \
    HOST=0.0.0.0
RUN npm i -g pnpm@11.7.0
WORKDIR /repo
COPY pnpm-lock.yaml pnpm-workspace.yaml package.json ./
COPY core core
COPY clients/terminal/package.json clients/terminal/package.json
# Prod-only deps (keeps next/react/ws; drops the build toolchain), PLUS typescript — Next loads the
# TypeScript next.config.ts at runtime and would otherwise try to auto-install it (no network in a
# pod). typescript is installed in an ISOLATED dir and symlinked in: running npm inside the
# pnpm-managed tree re-runs dependency `prepare` scripts (husky) even with --ignore-scripts.
RUN pnpm install --frozen-lockfile --prod --filter @vexa/terminal \
    && mkdir -p /opt/ts && cd /opt/ts && npm i --no-save typescript@5 \
    && ln -s /opt/ts/node_modules/typescript /repo/clients/terminal/node_modules/typescript \
    && pnpm store prune && npm cache clean --force
WORKDIR /repo/clients/terminal
# The built app + the custom server + runtime config it reads.
COPY --from=build /repo/clients/terminal/.next ./.next
COPY clients/terminal/public ./public
COPY clients/terminal/server.mjs clients/terminal/next.config.ts clients/terminal/tsconfig.json ./
# next-env.d.ts is gitignored (Next.js generates it during `next build`); take it from the build stage.
COPY --from=build /repo/clients/terminal/next-env.d.ts ./
COPY clients/terminal/src ./src
EXPOSE 3000
CMD ["node", "server.mjs"]
