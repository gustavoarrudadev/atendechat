# =====================================================
# ATENDECHAT FRONTEND - DOCKERFILE OTIMIZADO
# Easypanel Ready | Multi-stage Build | npm
# =====================================================

FROM node:20-alpine AS build

WORKDIR /app

# Copiar package files
COPY frontend/package*.json ./

# Instalar dependências
RUN npm install --legacy-peer-deps && \
    npm cache clean --force

# Copiar código fonte
COPY frontend/ .

# Remover .env se existir
RUN rm -f .env

# Configurar variáveis de ambiente para build
ARG REACT_APP_BACKEND_URL
ENV REACT_APP_BACKEND_URL=$REACT_APP_BACKEND_URL

ARG REACT_APP_HOURS_CLOSE_TICKETS_AUTO
ENV REACT_APP_HOURS_CLOSE_TICKETS_AUTO=$REACT_APP_HOURS_CLOSE_TICKETS_AUTO

ARG STACK_NAME
ENV STACK_NAME=$STACK_NAME

ARG REACT_APP_COLOR
ENV REACT_APP_COLOR=$REACT_APP_COLOR

ARG REACT_APP_TAB_NAME
ENV REACT_APP_TAB_NAME=$REACT_APP_TAB_NAME

# Criar diretório de brands e copiar assets (se existirem)
RUN mkdir -p ./brands

# Executar scripts de customização (se existirem)
RUN if [ -f copy_brand_assets.sh ]; then \
        chmod +x copy_brand_assets.sh && \
        ./copy_brand_assets.sh || true; \
    fi

RUN if [ -f update_tab_name.sh ]; then \
        chmod +x update_tab_name.sh && \
        ./update_tab_name.sh || true; \
    fi

RUN if [ -f update_app_color.sh ]; then \
        chmod +x update_app_color.sh && \
        ./update_app_color.sh || true; \
    fi

# Limpar brands
RUN rm -rf ./brands

# Build da aplicação React
RUN npm run build

# =====================================================
# STAGE 2: PRODUCTION
# =====================================================
FROM node:20-alpine AS production

WORKDIR /app

# Instalar serve globalmente
RUN npm install -g serve

# Copiar build da aplicação
COPY --from=build /app/build ./build

# Expor porta
EXPOSE 3001

# Variáveis de ambiente
ENV HOST=0.0.0.0
ENV PORT=3001

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=30s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://localhost:3001 || exit 1

# Servir aplicação
CMD ["serve", "-s", "build", "-l", "3001"]
