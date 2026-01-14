# =====================================================
# ATENDECHAT BACKEND - DOCKERFILE OTIMIZADO
# Easypanel Ready | Multi-stage Build | npm
# =====================================================

FROM node:20-alpine AS base

WORKDIR /app

# Instalar dependências do sistema
RUN apk add --no-cache \
    git \
    openssh-client \
    netcat-openbsd \
    tzdata \
    python3 \
    make \
    g++

# Configurar timezone
ENV TZ=America/Sao_Paulo
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && \
    echo $TZ > /etc/timezone

# =====================================================
# STAGE 1: DEPENDENCIES
# =====================================================
FROM base AS dependencies

# Copiar package files
COPY backend/package.json backend/package-lock.json* ./

# Instalar dependências
RUN npm install --omit=dev --legacy-peer-deps && \
    npm install fluent-ffmpeg @ffmpeg-installer/ffmpeg --legacy-peer-deps && \
    npm cache clean --force

# =====================================================
# STAGE 2: BUILD
# =====================================================
FROM base AS build

# Copiar node_modules da stage anterior
COPY --from=dependencies /app/node_modules ./node_modules

# Copiar package files
COPY backend/package*.json ./

# Copiar código fonte
COPY backend/ .

# Criar diretório de certificados
RUN mkdir -p /app/certs

# Build da aplicação TypeScript
RUN npm run build

# =====================================================
# STAGE 3: PRODUCTION
# =====================================================
FROM node:20-alpine AS production

WORKDIR /app

# Instalar apenas dependências essenciais para runtime
RUN apk add --no-cache \
    netcat-openbsd \
    tzdata

# Configurar timezone
ENV TZ=America/Sao_Paulo
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && \
    echo $TZ > /etc/timezone

# Copiar node_modules de produção
COPY --from=dependencies /app/node_modules ./node_modules

# Copiar código buildado
COPY --from=build /app/dist ./dist

# Copiar arquivos necessários
COPY --from=build /app/package*.json ./

# Criar diretórios necessários
RUN mkdir -p /app/certs /app/public

# Copiar e preparar entrypoint
COPY backend/docker-entrypoint.sh /app/docker-entrypoint.sh
RUN chmod +x /app/docker-entrypoint.sh

# Expor porta
EXPOSE 3000

# Variáveis de ambiente
ENV NODE_ENV=production
ENV HOST=0.0.0.0
ENV PORT=3000

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=60s --retries=3 \
    CMD nc -z localhost 3000 || exit 1

# Executar aplicação
CMD ["/app/docker-entrypoint.sh"]
