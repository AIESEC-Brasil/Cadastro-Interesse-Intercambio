# Build multi-stage para a imagem local de teste do servidor Next.js.
FROM node:24.18-alpine AS base
WORKDIR /app
ENV NEXT_TELEMETRY_DISABLED=1
RUN npm install --global npm@12.2.1

# Instala dependências uma vez para permitir cache independente do código-fonte.
FROM base AS dependencies
COPY package.json package-lock.json ./
RUN npm ci

# Gera o bundle standalone consumido pela imagem final.
FROM base AS builder
COPY --from=dependencies /app/node_modules ./node_modules
COPY . .
ARG NEXT_PUBLIC_URL_DIRECIONADA
ENV NEXT_PUBLIC_URL_DIRECIONADA=${NEXT_PUBLIC_URL_DIRECIONADA}
RUN npm run build

# Executa somente os artefatos de produção e aponta a API para o serviço Compose.
FROM base AS runner
ENV NODE_ENV=production \
    HOSTNAME=0.0.0.0 \
    PORT=3000 \
    EXTERNAL_API_URL=http://backend:5000

RUN addgroup --system --gid 1001 nodejs \
    && adduser --system --uid 1001 nextjs
COPY --from=builder --chown=nextjs:nodejs /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs
EXPOSE 3000
CMD ["node", "server.js"]
