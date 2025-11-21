# Backend Dockerfile for EcoBuddy API
FROM node:18-alpine

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy application source
COPY . .

# Expose port (DigitalOcean will use the PORT environment variable)
EXPOSE 3001

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
  CMD node -e "require('http').get('http://localhost:' + (process.env.PORT || 3001) + '/api/health', \
    (r) => { process.exit(r.statusCode === 200 ? 0 : 1); })"

# Start the application
CMD ["npm", "start"]
