# Builder stage
FROM node:18-alpine AS builder

# Install bash
RUN apk add --no-cache bash git

# Set working directory
WORKDIR /app

# Copy dependency files
COPY package.json package-lock.json ./
ENV CI=1

# Install dependencies
RUN npm ci --legacy-peer-deps

# Copy application files
COPY nginx/ ./nginx/
COPY assets/ ./assets/
COPY locale/ ./locale/
COPY scripts/ ./scripts/
COPY src/ ./src/
COPY vite.config.js tsconfig.json ./
COPY *.d.ts ./
COPY schema.graphql ./
COPY introspection*.json ./
COPY .featureFlags/ ./.featureFlags/

# Configure build arguments and environment variables
ARG API_URL
ENV API_URL=${API_URL:-http://localhost:8000/graphql/}

ARG APP_MOUNT_URI
ENV APP_MOUNT_URI=${APP_MOUNT_URI:-/dashboard/}

ARG APPS_MARKETPLACE_API_URL
ENV APPS_MARKETPLACE_API_URL=${APPS_MARKETPLACE_API_URL:-https://apps.saleor.io/api/v2/saleor-apps}

ARG APPS_TUNNEL_URL_KEYWORDS
ENV APPS_TUNNEL_URL_KEYWORDS=${APPS_TUNNEL_URL_KEYWORDS}

ARG STATIC_URL
ENV STATIC_URL=${STATIC_URL:-/dashboard/}

ARG SKIP_SOURCEMAPS
ENV SKIP_SOURCEMAPS=${SKIP_SOURCEMAPS:-true}

ARG LOCALE_CODE
ENV LOCALE_CODE=${LOCALE_CODE:-EN}

# Build the application
RUN npm run build

# Runner stage
FROM nginx:stable-alpine AS runner

# Set working directory for runner stage
WORKDIR /app

# Copy built files and configurations
COPY --from=builder /app/build/ /app/
COPY ./nginx/default.conf /etc/nginx/conf.d/default.conf
COPY ./nginx/replace-env-vars.sh /docker-entrypoint.d/50-replace-env-vars.sh

# Make the env vars script executable
RUN chmod +x /docker-entrypoint.d/50-replace-env-vars.sh

# Add health check
HEALTHCHECK --interval=30s --timeout=3s \
  CMD wget --quiet --tries=1 --spider http://localhost:9000 || exit 1

# Expose port 9000
EXPOSE 9000
