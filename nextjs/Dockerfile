# Build arguments
ARG NODE_VERSION=22.14.0-alpine
ARG PNPM_VERSION=10.13.1
ARG PORT=3011
ARG USER_ID=1001
ARG GROUP_ID=1001

# Stage 1: Dependencies
FROM node:${NODE_VERSION} AS deps
ARG PNPM_VERSION

WORKDIR /app

# Install node-gyp dependencies and pnpm
RUN apk add --no-cache libc6-compat python3 make g++ openssl
RUN npm install -g pnpm@${PNPM_VERSION}

COPY package.json pnpm-lock.yaml ./

# Install dependencies
RUN pnpm install --frozen-lockfile

# Stage 2: Builder
FROM node:${NODE_VERSION} AS builder
ARG PNPM_VERSION

WORKDIR /app

# Install pnpm
RUN npm install -g pnpm@${PNPM_VERSION}

# Copy all files from deps
COPY --from=deps /app ./
COPY . .

# Environment variables
ENV NEXT_TELEMETRY_DISABLED=1
ENV NODE_ENV=production

# Build
RUN pnpm build

# Stage 3: Runner
FROM node:${NODE_VERSION} AS runner
ARG USER_ID
ARG GROUP_ID
ARG PORT

WORKDIR /app

# Create non-root user
RUN addgroup --system --gid ${GROUP_ID} nodejs
RUN adduser --system --uid ${USER_ID} nextjs

# Copy necessary files
COPY --from=builder /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static
COPY --from=builder /app/node_modules ./node_modules

# Switch to non-root user
USER nextjs

# Expose port
EXPOSE ${PORT}

# Set hostname
ENV HOSTNAME="0.0.0.0"
ENV PORT=${PORT}

# Start the application
CMD ["node", "server.js"]
