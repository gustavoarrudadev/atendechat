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

# Copiar apenas package files para cache otimizado
COPY backend/package*.json ./

# Instalar dependências (npm)
RUN npm ci --only=production --legacy-peer-deps && \
    npm cache clean --force

# Instalar dependências específicas
RUN npm install fluent-ffmpeg @ffmpeg-installer/ffmpeg --legacy-peer-deps

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

# Copiar certificados (se existirem)
COPY certificates ./certs-temp

# Executar script de certificados
COPY backend/copy_cert_assets.sh ./
RUN chmod +x copy_cert_assets.sh && \
    ./copy_cert_assets.sh || true && \
    rm -rf ./certs-temp

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
COPY --from=build /app/certs ./certs

# Copiar e preparar entrypoint
COPY backend/docker-entrypoint.sh /app/docker-entrypoint.sh
RUN chmod +x /app/docker-entrypoint.sh

# Criar diretórios necessários
RUN mkdir -p /app/public

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
