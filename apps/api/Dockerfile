FROM node:20-alpine AS base


FROM base AS builder
RUN apk add --no-cache libc6-compat
RUN apk update && upgrade
WORKDIR /app
RUN npm install --ignore-scripts -g turbo
COPY . .
RUN turbo prune --scope=api --docker


ARG SENTRY_ORG=${SENTRY_ORG}
ARG SENTRY_PROJECT=${SENTRY_PROJECT}
ARG SENTRY_ENVIRONMENT=${SENTRY_ENVIRONMENT}
ARG SENTRY_AUTH_TOKEN=${SENTRY_AUTH_TOKEN}
ARG DATABASE_URL=${DATABASE_URL}



# Add lockfile and package.json's of isolated subworkspace
FROM base AS installer
RUN apk add --no-cache libc6-compat
RUN apk update && upgrade
WORKDIR /app


# install the dependencies
RUN npm install --ignore-scripts -g pnpm
COPY .gitignore .gitignore
COPY --from=builder /app/out/json/ .
COPY --from=builder /app/out/pnpm-lock.yaml ./pnpm-lock.yaml
COPY --from=builder /app/turbo.json ./turbo.json
COPY --from=builder /app/apps/api/src/prisma ./src/prisma
RUN pnpm install --no-cache --ignore-scripts
RUN pnpm run prisma generate 

# Build the project

FROM node:16-alpine AS sourcer
WORKDIR /app
COPY --from=installer /app/ .
COPY --from=builder /app/out/full/ .
COPY .gitignore .gitignore
RUN pnpm turbo run build --scope=api --include-dependencies --no-deps


FROM node:20-alpine as runner
WORKDIR /app
COPY --from=sourcer /app/ .
ENV PORT=4200
ENV HOSTNAME "0.0.0.0"
EXPOSE 4200
CMD [ "node", "apps/api/dist/main.js" ]