FROM node:20-alpine

# Create a non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Set working directory
WORKDIR /usr/src/app

# Copy package.json files first (for caching)
COPY package*.json ./

# Install dependencies safely, ignoring post-install scripts
RUN npm ci --only=production --ignore-scripts && npm cache clean --force

# Copy only the source code
COPY src/ ./src

# Set ownership to non-root user
RUN chown -R appuser:appgroup /usr/src/app

# Switch to non-root user
USER appuser

# Environment
ENV NODE_ENV=production
ENV PORT=3000

EXPOSE 3000

CMD ["node", "index.js"]
