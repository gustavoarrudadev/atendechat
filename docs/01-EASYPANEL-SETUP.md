# 🚀 Guia Completo de Setup no Easypanel

## 📋 Índice
- [Visão Geral](#visão-geral)
- [Pré-requisitos](#pré-requisitos)
- [Configuração Inicial](#configuração-inicial)
- [Criação de Serviços](#criação-de-serviços)
- [Configuração de Variáveis de Ambiente](#configuração-de-variáveis-de-ambiente)
- [Deploy](#deploy)
- [Verificação e Testes](#verificação-e-testes)

---

## 🎯 Visão Geral

Este guia descreve como fazer o deploy completo do **Atendechat** no **Easypanel**, eliminando a necessidade de:
- ❌ AAPanel
- ❌ Instalação manual de dependências
- ❌ Scripts bash de instalação
- ❌ Configuração manual de Nginx
- ❌ Configuração manual de SSL/Certbot
- ❌ Gerenciamento manual de PM2

O Easypanel automatiza **tudo isso** com uma interface visual moderna e simples.

---

## 📦 Pré-requisitos

### 1. **Conta no Easypanel**
- Acesse [easypanel.io](https://easypanel.io)
- Crie uma conta ou faça login
- Tenha um servidor conectado ao Easypanel

### 2. **Repositório Git**
- Repositório Git com o código do Atendechat
- Acesso configurado (GitHub, GitLab, Bitbucket)
- Branch principal configurada (main/master)

### 3. **Domínios Configurados**
- Domínio para o Frontend (ex: `app.seudominio.com`)
- Domínio para o Backend/API (ex: `api.seudominio.com`)
- DNS apontando para o servidor do Easypanel

---

## 🛠️ Configuração Inicial

### Passo 1: Criar Novo Projeto

1. Acesse o dashboard do Easypanel
2. Clique em **"Create Project"**
3. Digite o nome do projeto: `atendechat-cliente1` (ou nome desejado)
4. Clique em **"Create"**

### Passo 2: Conectar Repositório Git

1. Dentro do projeto, clique em **"Add Service"**
2. Selecione **"From Git Repository"**
3. Escolha o provedor Git (GitHub/GitLab/Bitbucket)
4. Autorize o acesso se necessário
5. Selecione o repositório **codatendechat**
6. Selecione a branch (geralmente `main` ou `master`)

---

## 🗄️ Criação de Serviços

O Atendechat precisa de **4 serviços** no Easypanel:

### 1. PostgreSQL (Banco de Dados)

1. No projeto, clique em **"Add Service"**
2. Selecione **"PostgreSQL"**
3. Configure:
   - **Nome**: `atendechat-postgres`
   - **Versão**: PostgreSQL 14 ou 15
   - **Database Name**: `atendechat`
   - **Username**: `atendechat`
   - **Password**: Gere uma senha segura (o Easypanel pode gerar automaticamente)
4. Clique em **"Create"**

**Anote as credenciais geradas** - você precisará delas nas variáveis de ambiente.

---

### 2. Redis (Cache e Filas)

1. No projeto, clique em **"Add Service"**
2. Selecione **"Redis"**
3. Configure:
   - **Nome**: `atendechat-redis`
   - **Versão**: Redis 7 (latest)
   - **Password**: Gere uma senha segura
4. Clique em **"Create"**

**Anote a senha do Redis** - você precisará dela nas variáveis de ambiente.

---

### 3. Backend (API)

1. No projeto, clique em **"Add Service"**
2. Selecione **"App"** (aplicação customizada)
3. Configure:
   - **Nome**: `atendechat-backend`
   - **Source**: Git Repository (já conectado)
   - **Build Method**: Dockerfile
   - **Dockerfile Path**: `backend/Dockerfile`
   - **Context Path**: `.` (root do repositório)

4. **Configurar Porta**:
   - **Internal Port**: `3000`
   - **External Port**: Deixe o Easypanel atribuir automaticamente

5. **Configurar Domínio**:
   - Clique em **"Add Domain"**
   - Digite: `api.seudominio.com`
   - Habilite **SSL Automático** (Let's Encrypt)

6. **Configurar Variáveis de Ambiente** (veja seção específica abaixo)

7. Clique em **"Create"**

---

### 4. Frontend (Interface)

1. No projeto, clique em **"Add Service"**
2. Selecione **"App"** (aplicação customizada)
3. Configure:
   - **Nome**: `atendechat-frontend`
   - **Source**: Git Repository (mesmo repositório)
   - **Build Method**: Dockerfile
   - **Dockerfile Path**: `frontend/Dockerfile`
   - **Context Path**: `.` (root do repositório)

4. **Configurar Porta**:
   - **Internal Port**: `3001`
   - **External Port**: Deixe o Easypanel atribuir automaticamente

5. **Configurar Domínio**:
   - Clique em **"Add Domain"**
   - Digite: `app.seudominio.com`
   - Habilite **SSL Automático** (Let's Encrypt)

6. **Configurar Variáveis de Ambiente** (veja seção específica abaixo)

7. Clique em **"Create"**

---

## 🔐 Configuração de Variáveis de Ambiente

### Backend (atendechat-backend)

Na seção **Environment Variables** do serviço Backend, adicione:

```bash
# Node
NODE_ENV=production

# URLs
BACKEND_URL=https://api.seudominio.com
FRONTEND_URL=https://app.seudominio.com
PROXY_PORT=443
PORT=3000

# Database (PostgreSQL)
DB_DIALECT=postgres
DB_HOST=atendechat-postgres
DB_PORT=5432
DB_USER=atendechat
DB_PASS=<senha_do_postgres>
DB_NAME=atendechat

# JWT Secrets (GERE NOVOS VALORES SEGUROS!)
JWT_SECRET=<gerar_valor_seguro_64_chars>
JWT_REFRESH_SECRET=<gerar_valor_seguro_64_chars>

# Redis
REDIS_URI=redis://:<senha_do_redis>@atendechat-redis:6379
REDIS_OPT_LIMITER_MAX=1
REDIS_OPT_LIMITER_DURATION=3000

# Limites da Instância
USER_LIMIT=10
CONNECTIONS_LIMIT=10
CLOSED_SEND_BY_ME=true

# Gerencianet (Opcional - Sistema de Pagamentos)
GERENCIANET_SANDBOX=false
GERENCIANET_CLIENT_ID=
GERENCIANET_CLIENT_SECRET=
GERENCIANET_PIX_CERT=
GERENCIANET_PIX_KEY=

# Email (Opcional - Recuperação de Senha)
MAIL_HOST=smtp.gmail.com
MAIL_USER=
MAIL_PASS=
MAIL_FROM=
MAIL_PORT=465

# Campanhas
CAMPAIGN_RATE_LIMIT=10000
CAMPAIGN_BATCH_SIZE=50

# Versão
npm_package_version=6.0.1
```

**⚠️ IMPORTANTE:**
- Substitua `<senha_do_postgres>` pela senha do PostgreSQL
- Substitua `<senha_do_redis>` pela senha do Redis
- **GERE NOVOS JWT SECRETS** (use geradores online ou comando: `openssl rand -base64 64`)
- Substitua os domínios pelos seus domínios reais

---

### Frontend (atendechat-frontend)

Na seção **Environment Variables** do serviço Frontend, adicione:

```bash
# Backend URL
REACT_APP_BACKEND_URL=https://api.seudominio.com

# Configurações
REACT_APP_HOURS_CLOSE_TICKETS_AUTO=24

# Stack Name (Nome da Instância)
STACK_NAME=cliente1

# Customização (Opcional)
REACT_APP_COLOR=#682EE3
REACT_APP_TAB_NAME=Atendechat
```

**⚠️ IMPORTANTE:**
- Substitua `https://api.seudominio.com` pela URL real do seu backend
- `STACK_NAME` deve ser único para cada instância (cliente)

---

## 🚀 Deploy

### Ordem de Deploy Recomendada:

1. **PostgreSQL** → Aguarde ficar "Running"
2. **Redis** → Aguarde ficar "Running"
3. **Backend** → Aguarde build e deploy completo
4. **Frontend** → Aguarde build e deploy completo

### Monitoramento do Build:

1. Clique no serviço (Backend ou Frontend)
2. Vá para a aba **"Logs"**
3. Acompanhe o processo de build em tempo real
4. Aguarde a mensagem de sucesso

### Comandos Automáticos no Deploy:

O backend executa automaticamente:
- `npm install` (instalação de dependências)
- `npm run build` (compilação TypeScript)
- `npx sequelize db:migrate` (migrations do banco)
- `npx sequelize db:seed:all` (seeds iniciais)

Isso é configurado no arquivo `backend/docker-entrypoint.sh`.

---

## ✅ Verificação e Testes

### 1. Verificar Status dos Serviços

No dashboard do Easypanel, verifique se todos os serviços estão com status **"Running"** (verde):
- ✅ atendechat-postgres
- ✅ atendechat-redis
- ✅ atendechat-backend
- ✅ atendechat-frontend

---

### 2. Testar Backend (API)

Acesse no navegador:
```
https://api.seudominio.com
```

Você deve ver uma resposta (pode ser erro 404, mas isso significa que está funcionando).

Teste o health check:
```
https://api.seudominio.com/health
```

---

### 3. Testar Frontend

Acesse no navegador:
```
https://app.seudominio.com
```

Você deve ver a tela de login do Atendechat.

---

### 4. Credenciais Padrão (Seeds)

Após o primeiro deploy, o sistema cria um usuário administrador padrão:

```
Email: admin@atendechat.com
Senha: admin123
```

**⚠️ ATENÇÃO:** Altere essa senha imediatamente após o primeiro login!

---

### 5. Verificar Logs

Se algo não funcionar, verifique os logs:

1. Clique no serviço com problema
2. Vá para a aba **"Logs"**
3. Verifique mensagens de erro
4. Copie e analise o erro

---

## 🔧 Comandos Úteis

### Reiniciar Serviço

1. Clique no serviço
2. Clique no botão **"Restart"**

### Redeployar (Rebuild)

1. Clique no serviço
2. Clique em **"Deploy"**
3. Aguarde o rebuild completo

### Executar Migrations Manualmente

1. Clique no serviço Backend
2. Vá para a aba **"Console"**
3. Execute:
```bash
npx sequelize db:migrate
```

### Executar Seeds Manualmente

1. No console do Backend:
```bash
npx sequelize db:seed:all
```

---

## 📊 Recursos e Escalabilidade

### Ajustar Recursos (CPU/RAM)

1. Clique no serviço
2. Vá para **"Settings"**
3. Ajuste:
   - **CPU Limit**: Limite de CPU
   - **Memory Limit**: Limite de RAM
   - **Replicas**: Número de instâncias (para escalabilidade)

### Recomendações de Recursos:

**Backend:**
- CPU: 0.5 - 1.0 core
- RAM: 512MB - 1GB
- Replicas: 1-3 (dependendo do tráfego)

**Frontend:**
- CPU: 0.25 - 0.5 core
- RAM: 256MB - 512MB
- Replicas: 1-2

**PostgreSQL:**
- CPU: 0.5 - 1.0 core
- RAM: 512MB - 2GB

**Redis:**
- CPU: 0.25 - 0.5 core
- RAM: 256MB - 512MB

---

## 🆘 Resolução de Problemas Comuns

### Backend não inicia

**Causa**: Variáveis de ambiente incorretas
**Solução**: Revise todas as variáveis, especialmente:
- `DB_HOST`, `DB_USER`, `DB_PASS`
- `REDIS_URI`
- `JWT_SECRET` e `JWT_REFRESH_SECRET`

---

### Frontend não carrega

**Causa**: `REACT_APP_BACKEND_URL` incorreto
**Solução**: 
1. Verifique se a URL do backend está correta
2. Certifique-se que o backend está rodando
3. Redeploy o frontend

---

### Migrations não executam

**Causa**: Banco de dados não acessível
**Solução**:
1. Verifique se o PostgreSQL está rodando
2. Teste a conexão no console do backend:
```bash
psql -h atendechat-postgres -U atendechat -d atendechat
```

---

### SSL não funciona

**Causa**: DNS não propagado ou configuração incorreta
**Solução**:
1. Verifique se o DNS aponta para o servidor correto
2. Aguarde propagação do DNS (até 48h)
3. No Easypanel, force renovação do certificado

---

## 🎯 Próximos Passos

Após o setup inicial:

1. 📖 Leia o guia: [02-DEPLOYMENT-GUIDE.md](./02-DEPLOYMENT-GUIDE.md)
2. 🔒 Configure segurança adicional
3. 📊 Configure monitoramento e alertas
4. 🔄 Configure backups automáticos
5. 👥 Configure múltiplas instâncias (multi-tenant)

---

## 📞 Suporte

- **Documentação Easypanel**: [docs.easypanel.io](https://docs.easypanel.io)
- **Documentação Atendechat**: Veja a pasta `docs/`

---

**🎉 Parabéns! Seu Atendechat está rodando no Easypanel!**
