# 🔐 Documentação Completa de Variáveis de Ambiente

## 📋 Índice
- [Backend Variables](#backend-variables)
- [Frontend Variables](#frontend-variables)
- [Variáveis de Build](#variáveis-de-build)
- [Geração de Secrets](#geração-de-secrets)
- [Variáveis por Ambiente](#variáveis-por-ambiente)

---

## 🔧 Backend Variables

### Variáveis Obrigatórias

#### NODE_ENV
```bash
NODE_ENV=production
```
- **Descrição**: Define o ambiente de execução
- **Valores**: `development`, `production`, `test`
- **Obrigatório**: ✅ Sim
- **Padrão**: `production` no Easypanel
- **Impacto**: Afeta logs, cache, otimizações

---

#### BACKEND_URL
```bash
BACKEND_URL=https://api.seudominio.com
```
- **Descrição**: URL completa do backend (API)
- **Formato**: `https://` + domínio (sem barra no final)
- **Obrigatório**: ✅ Sim
- **Exemplo**: `https://api.atendechat.com`
- **Uso**: CORS, webhooks, links de email

---

#### FRONTEND_URL
```bash
FRONTEND_URL=https://app.seudominio.com
```
- **Descrição**: URL completa do frontend (painel)
- **Formato**: `https://` + domínio (sem barra no final)
- **Obrigatório**: ✅ Sim
- **Exemplo**: `https://app.atendechat.com`
- **Uso**: CORS, redirecionamentos, links

---

#### PORT
```bash
PORT=3000
```
- **Descrição**: Porta interna do backend no container
- **Obrigatório**: ✅ Sim
- **Padrão**: `3000`
- **Nota**: Não altere, o Easypanel gerencia portas externas

---

#### PROXY_PORT
```bash
PROXY_PORT=443
```
- **Descrição**: Porta do proxy reverso (HTTPS)
- **Obrigatório**: ✅ Sim
- **Padrão**: `443` (HTTPS)
- **Nota**: Gerenciado automaticamente pelo Easypanel

---

### Banco de Dados (PostgreSQL)

#### DB_DIALECT
```bash
DB_DIALECT=postgres
```
- **Descrição**: Tipo de banco de dados
- **Valor**: `postgres` (fixo)
- **Obrigatório**: ✅ Sim
- **Nota**: Sistema usa PostgreSQL exclusivamente

---

#### DB_HOST
```bash
DB_HOST=atendechat-postgres
```
- **Descrição**: Hostname do servidor PostgreSQL
- **Valor Easypanel**: Nome do serviço PostgreSQL
- **Obrigatório**: ✅ Sim
- **Exemplo**: `atendechat-postgres`, `atendechat-db`

---

#### DB_PORT
```bash
DB_PORT=5432
```
- **Descrição**: Porta do PostgreSQL
- **Obrigatório**: ✅ Sim
- **Padrão**: `5432`

---

#### DB_USER
```bash
DB_USER=atendechat
```
- **Descrição**: Usuário do banco de dados
- **Obrigatório**: ✅ Sim
- **Nota**: Use o mesmo nome configurado no PostgreSQL

---

#### DB_PASS
```bash
DB_PASS=SuaSenhaSegura123!
```
- **Descrição**: Senha do banco de dados
- **Obrigatório**: ✅ Sim
- **Segurança**: 
  - ⚠️ Nunca commitar no Git
  - ⚠️ Use senhas fortes (mínimo 16 caracteres)
  - ✅ Gere no Easypanel automaticamente

---

#### DB_NAME
```bash
DB_NAME=atendechat
```
- **Descrição**: Nome do banco de dados
- **Obrigatório**: ✅ Sim
- **Padrão**: `atendechat`

---

### JWT (Autenticação)

#### JWT_SECRET
```bash
JWT_SECRET=abc123def456...
```
- **Descrição**: Chave secreta para assinatura de tokens JWT
- **Obrigatório**: ✅ Sim
- **Formato**: String aleatória de 64+ caracteres
- **Segurança**:
  - ⚠️ **CRÍTICO**: Nunca commitar no Git
  - ⚠️ Alterar invalida todas as sessões ativas
  - ✅ Use valores diferentes para cada instância

**Como Gerar:**
```bash
# Método 1: OpenSSL (Linux/Mac)
openssl rand -base64 64

# Método 2: Node.js
node -e "console.log(require('crypto').randomBytes(64).toString('base64'))"

# Método 3: Online
# https://randomkeygen.com/ (use "CodeIgniter Encryption Keys")
```

---

#### JWT_REFRESH_SECRET
```bash
JWT_REFRESH_SECRET=xyz789uvw012...
```
- **Descrição**: Chave secreta para tokens de refresh
- **Obrigatório**: ✅ Sim
- **Formato**: String aleatória de 64+ caracteres
- **Nota**: Deve ser DIFERENTE do `JWT_SECRET`

**Como Gerar:** Use os mesmos métodos acima

---

### Redis (Cache e Filas)

#### REDIS_URI
```bash
REDIS_URI=redis://:SenhaDo Redis@atendechat-redis:6379
```
- **Descrição**: URI de conexão completa do Redis
- **Formato**: `redis://:<senha>@<host>:<porta>`
- **Obrigatório**: ✅ Sim
- **Componentes**:
  - `redis://` - Protocolo
  - `:<senha>` - Senha do Redis (com `:` antes)
  - `@atendechat-redis` - Host do Redis
  - `:6379` - Porta do Redis

**Exemplo completo:**
```bash
REDIS_URI=redis://:minhaSenha123@atendechat-redis:6379
```

---

#### REDIS_OPT_LIMITER_MAX
```bash
REDIS_OPT_LIMITER_MAX=1
```
- **Descrição**: Número máximo de requisições por janela de tempo
- **Obrigatório**: ❌ Não
- **Padrão**: `1`
- **Uso**: Rate limiting de requisições

---

#### REDIS_OPT_LIMITER_DURATION
```bash
REDIS_OPT_LIMITER_DURATION=3000
```
- **Descrição**: Duração da janela de rate limiting (milissegundos)
- **Obrigatório**: ❌ Não
- **Padrão**: `3000` (3 segundos)
- **Uso**: Previne spam de requisições

---

### Limites e Capacidade

#### USER_LIMIT
```bash
USER_LIMIT=10
```
- **Descrição**: Número máximo de usuários/atendentes permitidos
- **Obrigatório**: ✅ Sim
- **Valores**: Número inteiro positivo
- **Uso**: Controle de planos/licenciamento
- **Nota**: Configure de acordo com o plano do cliente

---

#### CONNECTIONS_LIMIT
```bash
CONNECTIONS_LIMIT=10
```
- **Descrição**: Número máximo de conexões WhatsApp permitidas
- **Obrigatório**: ✅ Sim
- **Valores**: Número inteiro positivo
- **Uso**: Controle de planos/licenciamento
- **Nota**: Configure de acordo com o plano do cliente

---

#### CLOSED_SEND_BY_ME
```bash
CLOSED_SEND_BY_ME=true
```
- **Descrição**: Fecha automaticamente tickets quando cliente envia mensagem
- **Obrigatório**: ❌ Não
- **Valores**: `true`, `false`
- **Padrão**: `true`

---

### Campanhas (WhatsApp Marketing)

#### CAMPAIGN_RATE_LIMIT
```bash
CAMPAIGN_RATE_LIMIT=10000
```
- **Descrição**: Limite de mensagens por campanha
- **Obrigatório**: ❌ Não
- **Padrão**: `10000`
- **Uso**: Previne bloqueios do WhatsApp

---

#### CAMPAIGN_BATCH_SIZE
```bash
CAMPAIGN_BATCH_SIZE=50
```
- **Descrição**: Número de mensagens enviadas por lote
- **Obrigatório**: ❌ Não
- **Padrão**: `50`
- **Uso**: Controla velocidade de envio

---

### Gerencianet / Efí (Pagamentos PIX)

> **Nota**: Opcional - Use apenas se tiver integração com pagamentos

#### GERENCIANET_SANDBOX
```bash
GERENCIANET_SANDBOX=false
```
- **Descrição**: Ativa modo sandbox (testes)
- **Obrigatório**: ❌ Não (se usar Gerencianet)
- **Valores**: `true` (testes), `false` (produção)
- **Padrão**: `false`

---

#### GERENCIANET_CLIENT_ID
```bash
GERENCIANET_CLIENT_ID=Client_Id_...
```
- **Descrição**: Client ID da API Gerencianet
- **Obrigatório**: ❌ Não (se usar Gerencianet)
- **Obter em**: Painel Gerencianet/Efí

---

#### GERENCIANET_CLIENT_SECRET
```bash
GERENCIANET_CLIENT_SECRET=Client_Secret_...
```
- **Descrição**: Client Secret da API Gerencianet
- **Obrigatório**: ❌ Não (se usar Gerencianet)
- **Segurança**: ⚠️ Nunca commitar no Git

---

#### GERENCIANET_PIX_CERT
```bash
GERENCIANET_PIX_CERT=producao-123456-cert.pem
```
- **Descrição**: Nome do arquivo de certificado PIX
- **Obrigatório**: ❌ Não (se usar Gerencianet)
- **Localização**: Pasta `backend/certs/`

---

#### GERENCIANET_PIX_KEY
```bash
GERENCIANET_PIX_KEY=suachavepix@email.com
```
- **Descrição**: Chave PIX cadastrada na Gerencianet
- **Obrigatório**: ❌ Não (se usar Gerencianet)
- **Tipos**: Email, Telefone, CPF/CNPJ, Aleatória

---

### Email (Recuperação de Senha)

> **Nota**: Opcional - Use para enviar emails de recuperação de senha

#### MAIL_HOST
```bash
MAIL_HOST=smtp.gmail.com
```
- **Descrição**: Servidor SMTP para envio de emails
- **Obrigatório**: ❌ Não (se usar emails)
- **Exemplos**:
  - Gmail: `smtp.gmail.com`
  - Outlook: `smtp-mail.outlook.com`
  - SendGrid: `smtp.sendgrid.net`

---

#### MAIL_PORT
```bash
MAIL_PORT=465
```
- **Descrição**: Porta do servidor SMTP
- **Obrigatório**: ❌ Não (se usar emails)
- **Valores**:
  - `465` - SSL
  - `587` - TLS
  - `25` - Não seguro (não recomendado)

---

#### MAIL_USER
```bash
MAIL_USER=seu@email.com
```
- **Descrição**: Email de autenticação SMTP
- **Obrigatório**: ❌ Não (se usar emails)

---

#### MAIL_PASS
```bash
MAIL_PASS=SuaSenhaDeApp
```
- **Descrição**: Senha de autenticação SMTP
- **Obrigatório**: ❌ Não (se usar emails)
- **Gmail**: Use "Senha de App" (não a senha normal)
- **Segurança**: ⚠️ Nunca commitar no Git

---

#### MAIL_FROM
```bash
MAIL_FROM=noreply@seudominio.com
```
- **Descrição**: Email remetente (From)
- **Obrigatório**: ❌ Não (se usar emails)
- **Nota**: Pode ser diferente de `MAIL_USER`

---

### Outras Variáveis

#### npm_package_version
```bash
npm_package_version=6.0.1
```
- **Descrição**: Versão do sistema
- **Obrigatório**: ❌ Não
- **Uso**: Exibição de versão no frontend

---

## 🎨 Frontend Variables

### REACT_APP_BACKEND_URL
```bash
REACT_APP_BACKEND_URL=https://api.seudominio.com
```
- **Descrição**: URL da API backend
- **Obrigatório**: ✅ Sim
- **Formato**: `https://` + domínio (sem barra no final)
- **Uso**: Todas as requisições do frontend

---

### REACT_APP_HOURS_CLOSE_TICKETS_AUTO
```bash
REACT_APP_HOURS_CLOSE_TICKETS_AUTO=24
```
- **Descrição**: Horas para fechamento automático de tickets
- **Obrigatório**: ❌ Não
- **Padrão**: `24` (24 horas)
- **Uso**: Tickets inativos fecham automaticamente

---

### STACK_NAME
```bash
STACK_NAME=cliente1
```
- **Descrição**: Nome da instância/stack
- **Obrigatório**: ❌ Não
- **Uso**: Identificação de assets (logos, certificados)
- **Nota**: Útil para white label

---

### REACT_APP_COLOR
```bash
REACT_APP_COLOR=#682EE3
```
- **Descrição**: Cor primária do tema
- **Obrigatório**: ❌ Não
- **Formato**: Hexadecimal (`#RRGGBB`)
- **Padrão**: `#682EE3` (roxo)
- **Uso**: Customização de tema

---

### REACT_APP_TAB_NAME
```bash
REACT_APP_TAB_NAME=Atendechat
```
- **Descrição**: Nome exibido na aba do navegador
- **Obrigatório**: ❌ Não
- **Padrão**: `Atendechat`
- **Uso**: White label / branding

---

## 🏗️ Variáveis de Build (Docker)

Estas variáveis são usadas durante o build do Docker:

### Backend Build Args

```dockerfile
ARG STACK_NAME
ARG ENABLE_FINANCIAL
ARG GERENCIANET_PIX_CERT
```

### Frontend Build Args

```dockerfile
ARG REACT_APP_BACKEND_URL
ARG REACT_APP_HOURS_CLOSE_TICKETS_AUTO
ARG STACK_NAME
ARG REACT_APP_COLOR
ARG REACT_APP_TAB_NAME
```

---

## 🔑 Geração de Secrets

### JWT Secrets

**Linux/Mac:**
```bash
openssl rand -base64 64
```

**Windows (PowerShell):**
```powershell
[Convert]::ToBase64String((1..64|%{Get-Random -Max 256}))
```

**Node.js:**
```bash
node -e "console.log(require('crypto').randomBytes(64).toString('base64'))"
```

**Python:**
```bash
python3 -c "import secrets; print(secrets.token_urlsafe(64))"
```

---

### Senhas de Banco e Redis

**Gere senhas fortes:**
```bash
# Linux/Mac
openssl rand -base64 32

# Ou use geradores online
https://passwordsgenerator.net/
```

**Critérios de segurança:**
- ✅ Mínimo 16 caracteres
- ✅ Letras maiúsculas e minúsculas
- ✅ Números
- ✅ Caracteres especiais
- ❌ Sem palavras do dicionário
- ❌ Sem sequências óbvias (123, abc)

---

## 🌍 Variáveis por Ambiente

### Desenvolvimento Local

```bash
NODE_ENV=development
BACKEND_URL=http://localhost:3000
FRONTEND_URL=http://localhost:3001
DB_HOST=localhost
REDIS_URI=redis://:senha@localhost:6379
```

---

### Staging/Homologação

```bash
NODE_ENV=production
BACKEND_URL=https://api-staging.seudominio.com
FRONTEND_URL=https://app-staging.seudominio.com
DB_HOST=atendechat-postgres-staging
REDIS_URI=redis://:senha@atendechat-redis-staging:6379
```

---

### Produção

```bash
NODE_ENV=production
BACKEND_URL=https://api.seudominio.com
FRONTEND_URL=https://app.seudominio.com
DB_HOST=atendechat-postgres
REDIS_URI=redis://:senha@atendechat-redis:6379
```

---

## ✅ Checklist de Variáveis

Antes de fazer deploy, verifique:

### Backend:
- [ ] `NODE_ENV` configurado
- [ ] `BACKEND_URL` e `FRONTEND_URL` corretos
- [ ] `PORT` definido (3000)
- [ ] `DB_*` todas configuradas
- [ ] `JWT_SECRET` e `JWT_REFRESH_SECRET` únicos e seguros
- [ ] `REDIS_URI` correta
- [ ] `USER_LIMIT` e `CONNECTIONS_LIMIT` definidos

### Frontend:
- [ ] `REACT_APP_BACKEND_URL` correto
- [ ] `STACK_NAME` definido (se white label)
- [ ] `REACT_APP_COLOR` customizado (se necessário)

### Segurança:
- [ ] Nenhum secret commitado no Git
- [ ] Senhas fortes geradas
- [ ] JWT secrets únicos por instância
- [ ] Arquivo `.env` no `.gitignore`

---

## 📞 Suporte

Para dúvidas sobre variáveis de ambiente:
- Veja exemplos em: `backend/.env.example`
- Documentação: `docs/01-EASYPANEL-SETUP.md`
- Troubleshooting: `docs/07-TROUBLESHOOTING.md`

---

**✅ Todas as variáveis documentadas!**
