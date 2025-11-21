# Build stage
FROM node:20-alpine AS builder

# Install pnpm
RUN corepack enable && corepack prepare pnpm@9.6.0 --activate

WORKDIR /app

# Copy workspace files
COPY package.json pnpm-workspace.yaml pnpm-lock.yaml ./
COPY .npmrc ./
COPY LICENSE.md ./

# Copy all packages and apps
COPY packages/ ./packages/
COPY apps/ ./apps/
COPY scripts/ ./scripts/

# Install dependencies
RUN pnpm install --frozen-lockfile

# Build all packages in dependency order
RUN pnpm run build:metadata
RUN pnpm run build:resolver
RUN pnpm run build:core
RUN pnpm run build:icons
RUN pnpm run build:forms
RUN pnpm run build:themes
RUN pnpm run build:lib
RUN pnpm run build:module

# Build showcase
RUN pnpm run build:showcase

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
