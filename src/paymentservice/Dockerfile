FROM node:20-alpine

# Create app user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Set working directory
WORKDIR /usr/src/app

# Copy package files only
COPY package*.json ./

# Install dependencies safely
RUN npm ci --only=production && npm cache clean --force

# Copy only source code
COPY src/ ./src

# Change ownership to non-root user
RUN chown -R appuser:appgroup /usr/src/app

# Switch to non-root user
USER appuser

# Set environment
ENV NODE_ENV=production
ENV PORT=3000

EXPOSE 3000

CMD ["node", "index.js"]
