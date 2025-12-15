# ============================================
# STAGE 1: Build Stage
# ============================================
# Use official Node.js LTS image for building the React application
FROM node:18-alpine AS build

# Update packages to patch known vulnerabilities
RUN apk update && apk upgrade --no-cache

# Set working directory inside the container
WORKDIR /app

# Copy package files first (dependency layer caching)
COPY package*.json ./

# Install dependencies with clean install for reproducible builds
RUN npm ci --only=production

# Copy application source code
COPY . .

# Build the React application for production
RUN npm run build

# ============================================
# STAGE 2: Production Stage
# ============================================
# Use official Nginx stable release on Alpine (automatically maintained)
FROM nginx:stable-alpine

# Update all system packages to latest security patches
RUN apk update && apk upgrade --no-cache && \
    rm -rf /var/cache/apk/*

# Remove default Nginx configuration
RUN rm /etc/nginx/conf.d/default.conf

# Copy custom Nginx configuration
COPY nginx.conf /etc/nginx/conf.d/

# Copy built React application from build stage
COPY --from=build /app/build /usr/share/nginx/html

# Expose port 3000 (as configured in nginx.conf)
EXPOSE 3000

# Add health check for container monitoring
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD wget --quiet --tries=1 --spider http://localhost:3000 || exit 1

# Run Nginx in foreground (required for Docker)
CMD ["nginx", "-g", "daemon off;"]
