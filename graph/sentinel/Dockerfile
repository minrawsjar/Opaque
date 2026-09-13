# Standalone, read-only Graph privacy companion. Build context is repository root:
# docker build -f graph/sentinel/Dockerfile .
FROM oven/bun:1.3.14-alpine

RUN addgroup -S sentinel && adduser -S -G sentinel sentinel
WORKDIR /app

# Graph has one workspace-local dependency and the generated client is created
# at container start from Railway runtime variables, never baked with a secret.
COPY packages/protocol-types ./packages/protocol-types
COPY graph ./graph
WORKDIR /app/graph
RUN bun install --frozen-lockfile

USER sentinel
ENV PORT=8787

# Railway injects GRAPH_GATEWAY_API_KEY at runtime. The Graph Client generator
# must run after that injection because it resolves the Gateway endpoint.
CMD ["sh", "-c", "bun ./node_modules/.bin/graphclient build --dir companion && exec bun sentinel/http-entry.ts"]
