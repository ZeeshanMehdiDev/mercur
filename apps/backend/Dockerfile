FROM node:20-alpine AS base

# The web Dockerfile is copy-pasted into our main docs at /docs/handbook/deploying-with-docker.
# Make sure you update this Dockerfile, the Dockerfile in the web workspace and copy that over to Dockerfile in the docs.

FROM base AS builder
# Check https://github.com/nodejs/docker-node/tree/b4117f9333da4138b03a546ec926ef50a31506c3#nodealpine to understand why libc6-compat might be needed.
RUN apk update
RUN apk add --no-cache libc6-compat
WORKDIR /app
RUN yarn global add turbo@2.5.4
ENV TURBO_NO_TELEMETRY=1
COPY . .
RUN turbo prune api --docker

FROM base AS installer
WORKDIR /app
COPY --from=builder /app/out/json/ .
RUN yarn install
COPY --from=builder /app/out/full/ .
RUN yarn turbo build

FROM base AS runner
WORKDIR /app
RUN addgroup --system --gid 1001 medusa
RUN adduser --system --uid 1001 medusa
COPY --from=installer /app .
# Create necessary directories and set ownership
RUN mkdir -p /app/apps/backend/static /app/apps/backend/src/modules/algolia/migrations
RUN chown -R medusa:medusa /app
USER medusa
WORKDIR /app/apps/backend
ENV DATABASE_URL=postgresql://postgres:hakgYEfBUwxuutonEuvSkzFKBUUQLFvQ@ballast.proxy.rlwy.net:12067/railway
ENV REDIS_URL=redis://default:iAFOFRGyuCIcjJoGMladVCMkULnMCKCY@nozomi.proxy.rlwy.net:14773
CMD ["sh", "-c", "yarn db:migrate && yarn start"]