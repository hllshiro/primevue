# Build stage
FROM node:20-alpine AS builder

# Install pnpm
RUN corepack enable && corepack prepare pnpm@9.6.0 --activate

WORKDIR /app

# Copy workspace files
COPY package.json pnpm-workspace.yaml pnpm-lock.yaml ./
COPY .npmrc ./

# Copy all packages and apps
COPY packages/ ./packages/
COPY apps/ ./apps/
COPY scripts/ ./scripts/

# Install dependencies
RUN pnpm install --frozen-lockfile

# Build dependencies first
RUN pnpm --filter primevue build
RUN pnpm --filter @primevue/core build
RUN pnpm --filter @primevue/forms build
RUN pnpm --filter themes build
RUN pnpm --filter @primevue/nuxt-module build

# Build showcase
RUN pnpm --filter showcase build

# Production stage
FROM node:20-alpine AS runner

WORKDIR /app

# Copy built output
COPY --from=builder /app/apps/showcase/.output /app/.output

# Expose port
EXPOSE 3000

# Set environment variables
ENV NODE_ENV=production
ENV NITRO_PORT=3000
ENV NITRO_HOST=0.0.0.0

# Run the application
CMD ["node", ".output/server/index.mjs"]
