# 1. Prune Phase: Create a lightweight context with Turbo
FROM node:lts-alpine AS pruner

WORKDIR /app
RUN npm install -g turbo
COPY . .
# Adjust the scope to target the Nest.js app
RUN turbo prune --scope=nest-app --docker

# 2. Install Phase: Install dependencies
FROM node:lts-alpine AS installer

WORKDIR /app
# Copy pruned files for dependency installation
COPY --from=pruner /app/out/json .
COPY --from=pruner /app/out/pnpm-lock.yaml ./pnpm-lock.yaml
COPY --from=pruner /app/turbo.json ./turbo.json
RUN npm install -g pnpm
RUN pnpm install --frozen-lockfile

# 3. Build Phase: Build the app
FROM node:lts-alpine AS builder

WORKDIR /app
COPY --from=installer /app/ .
COPY --from=pruner /app/out/full .
RUN npm install -g pnpm
RUN pnpm run build

# 4. Runner Phase: Production-ready container
FROM node:lts-alpine AS runner

WORKDIR /app
ENV NODE_ENV=production
# Copy only the necessary files for production
COPY --from=builder /app/ .

EXPOSE 4000

# Set the default command to run the Nest.js app
CMD ["node","apps/nest-app/dist/main"]
 
