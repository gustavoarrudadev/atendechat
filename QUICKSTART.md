# ⚡ Guia Rápido de Início - Atendechat no Easypanel

## 🎯 Em 15 Minutos para Produção!

Este guia rápido mostra como fazer deploy do Atendechat no **Easypanel** em menos de 15 minutos.

---

## 📋 Pré-requisitos

✅ Conta no [Easypanel](https://easypanel.io)  
✅ Servidor conectado ao Easypanel  
✅ 2 domínios configurados:
   - `api.seudominio.com` (Backend)
   - `app.seudominio.com` (Frontend)

---

## 🚀 Passo a Passo

### 1️⃣ Criar Projeto (1 minuto)

1. Acesse o Easypanel
2. Clique em **"Create Project"**
3. Nome: `atendechat`
4. Clique em **"Create"**

---

### 2️⃣ Criar PostgreSQL (1 minuto)

1. Dentro do projeto, clique em **"Add Service"**
2. Selecione **"PostgreSQL"**
3. Configure:
   - **Nome**: `atendechat-postgres`
   - **Versão**: PostgreSQL 14
   - **Database**: `atendechat`
   - **Username**: `atendechat`
   - **Password**: (gere uma senha forte)
4. Clique em **"Create"**

⚠️ **Anote a senha do PostgreSQL!**

---

### 3️⃣ Criar Redis (1 minuto)

1. Clique em **"Add Service"**
2. Selecione **"Redis"**
3. Configure:
   - **Nome**: `atendechat-redis`
   - **Versão**: Redis 7
   - **Password**: (gere uma senha forte)
4. Clique em **"Create"**

⚠️ **Anote a senha do Redis!**

---

### 4️⃣ Criar Backend (5 minutos)

1. Clique em **"Add Service"** → **"App"**
2. Configure:
   - **Nome**: `atendechat-backend`
   - **Source**: Git Repository
   - **Repository**: `https://github.com/seu-usuario/codatendechat`
   - **Build Method**: Dockerfile
   - **Dockerfile Path**: `backend/Dockerfile`
   - **Context**: `.`
   - **Internal Port**: `3000`

3. **Adicionar Domínio**:
   - Domain: `api.seudominio.com`
   - SSL: ✅ Enable SSL
   - Redirect HTTP → HTTPS: ✅

4. **Variáveis de Ambiente** (copie e adapte):

```bash
NODE_ENV=production
BACKEND_URL=https://api.seudominio.com
FRONTEND_URL=https://app.seudominio.com
PROXY_PORT=443
PORT=3000

DB_DIALECT=postgres
DB_HOST=atendechat-postgres
DB_PORT=5432
DB_USER=atendechat
DB_PASS=SENHA_DO_POSTGRES
DB_NAME=atendechat

JWT_SECRET=GERE_UM_VALOR_AQUI
JWT_REFRESH_SECRET=GERE_OUTRO_VALOR_AQUI

REDIS_URI=redis://:SENHA_DO_REDIS@atendechat-redis:6379
REDIS_OPT_LIMITER_MAX=1
REDIS_OPT_LIMITER_DURATION=3000

USER_LIMIT=10
CONNECTIONS_LIMIT=10
CLOSED_SEND_BY_ME=true

CAMPAIGN_RATE_LIMIT=10000
CAMPAIGN_BATCH_SIZE=50

npm_package_version=6.0.1
```

**🔑 Gerar JWT Secrets:**
```bash
# Linux/Mac
openssl rand -base64 64
```

5. Clique em **"Create"**
6. Aguarde o build (5-7 minutos)

---

### 5️⃣ Criar Frontend (5 minutos)

1. Clique em **"Add Service"** → **"App"**
2. Configure:
   - **Nome**: `atendechat-frontend`
   - **Source**: Mesmo repositório
   - **Dockerfile Path**: `frontend/Dockerfile`
   - **Context**: `.`
   - **Internal Port**: `3001`

3. **Adicionar Domínio**:
   - Domain: `app.seudominio.com`
   - SSL: ✅ Enable SSL
   - Redirect HTTP → HTTPS: ✅

4. **Variáveis de Ambiente**:

```bash
REACT_APP_BACKEND_URL=https://api.seudominio.com
REACT_APP_HOURS_CLOSE_TICKETS_AUTO=24
STACK_NAME=default
REACT_APP_COLOR=#682EE3
REACT_APP_TAB_NAME=Atendechat
```

5. Clique em **"Create"**
6. Aguarde o build (3-5 minutos)

---

## ✅ Verificação

### Testar Backend

```bash
curl https://api.seudominio.com/health
```

### Testar Frontend

Acesse: `https://app.seudominio.com`

### Fazer Login

```
Email: admin@atendechat.com
Senha: admin123
```

⚠️ **IMPORTANTE:** Altere esta senha imediatamente!

---

## 📊 Status Esperado

Todos os serviços devem estar **"Running"** (verde):

- ✅ atendechat-postgres
- ✅ atendechat-redis
- ✅ atendechat-backend
- ✅ atendechat-frontend

---

## 🆘 Problemas Comuns

### Backend não inicia?

1. Verifique logs: `Backend → Logs`
2. Confirme todas as variáveis de ambiente
3. Verifique se PostgreSQL e Redis estão rodando

### Frontend não carrega?

1. Limpe cache do navegador (`Ctrl + Shift + Del`)
2. Verifique `REACT_APP_BACKEND_URL`
3. Confirme que backend está funcionando

### SSL não funciona?

1. Aguarde propagação do DNS (1-4 horas)
2. Verifique se DNS aponta para IP correto
3. Force renovação do certificado

---

## 📚 Próximos Passos

✅ Deploy completo? Continue com:

1. **Segurança**: 
   - Alterar senha do admin
   - Revisar variáveis de ambiente

2. **Configuração**:
   - Configurar limites (USER_LIMIT, CONNECTIONS_LIMIT)
   - Configurar white label (se necessário)

3. **Monitoramento**:
   - Configurar alertas
   - Configurar backups automáticos

4. **Documentação Completa**:
   - [Setup Completo](docs/01-EASYPANEL-SETUP.md)
   - [Deployment Guide](docs/02-DEPLOYMENT-GUIDE.md)
   - [Variáveis de Ambiente](docs/03-ENVIRONMENT-VARIABLES.md)
   - [Multi-Tenant](docs/04-MULTI-TENANT-SETUP.md)
   - [Troubleshooting](docs/07-TROUBLESHOOTING.md)

---

## 🎉 Pronto!

Seu Atendechat está rodando no Easypanel com:

- ✅ SSL automático
- ✅ Backups automáticos
- ✅ Monitoramento visual
- ✅ Escalabilidade fácil
- ✅ Deploy em 1 clique

**Tempo total: ~15 minutos** 🚀

---

## 💡 Dicas

- Use senhas fortes (mínimo 16 caracteres)
- Gere JWT secrets únicos para cada instância
- Configure backups automáticos
- Monitor recursos (CPU, RAM)
- Mantenha documentação atualizada

---

**Dúvidas?** Consulte a [documentação completa](docs/) ou [troubleshooting](docs/07-TROUBLESHOOTING.md).
