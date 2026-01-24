# Use official Node.js image as base
FROM node:20-alpine

# Set working directory
WORKDIR /usr/src/app

# Copy package.json and package-lock.json first for caching
COPY package*.json ./

# Install dependencies in a single RUN instruction
# Chaining commands, cleaning up caches, and using alphabetical order
RUN npm ci --only=production && \
    npm cache clean --force

# Copy application source code
COPY . .

# Set environment variables (if needed)
ENV NODE_ENV=production
ENV PORT=3000

# Expose application port
EXPOSE 3000

# Start the application
CMD ["node", "index.js"]
