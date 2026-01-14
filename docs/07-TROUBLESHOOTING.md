# 🆘 Guia de Resolução de Problemas

## 📋 Índice
- [Problemas Comuns](#problemas-comuns)
- [Backend](#backend)
- [Frontend](#frontend)
- [Banco de Dados](#banco-de-dados)
- [Redis](#redis)
- [SSL/Domínios](#ssldomínios)
- [Performance](#performance)
- [Logs e Debugging](#logs-e-debugging)

---

## ⚡ Problemas Comuns

### 🔴 Serviço não Inicia

**Sintomas:**
- Status "Crashed" ou "Failed" no Easypanel
- Container reiniciando constantemente
- Logs mostrando erros

**Diagnóstico:**

1. **Verificar Logs:**
```bash
# No Easypanel, aba "Logs"
# Procure por:
- "Error:"
- "ECONNREFUSED"
- "Authentication failed"
- "Cannot find module"
```

2. **Verificar Variáveis de Ambiente:**
- Vá para "Environment" no serviço
- Confirme todas as variáveis obrigatórias
- Verifique erros de digitação

3. **Verificar Dependências:**
- PostgreSQL está rodando?
- Redis está rodando?
- Portas corretas configuradas?

**Soluções Comuns:**

```bash
# 1. Variável faltando ou incorreta
#    → Adicione/corrija a variável
#    → Clique em "Restart"

# 2. Banco de dados não acessível
#    → Verifique DB_HOST, DB_USER, DB_PASS
#    → Teste conexão manual

# 3. Migrations falharam
#    → Acesse console do backend
#    → Execute: npx sequelize db:migrate

# 4. Dependências não instaladas
#    → Force rebuild
#    → Clique em "Deploy"
```

---

### 🔴 Erro 500 (Internal Server Error)

**Sintomas:**
- API retorna erro 500
- Frontend exibe mensagens de erro
- Requisições falham

**Diagnóstico:**

1. **Verificar Logs do Backend:**
```bash
# Procure por stack traces
# Linhas começando com "Error:"
# Mensagens de exceção
```

2. **Testar Endpoint Específico:**
```bash
curl https://api.seudominio.com/health
curl https://api.seudominio.com/api/status
```

**Soluções Comuns:**

```bash
# 1. Erro de banco de dados
#    → Verificar migrations
#    → Verificar conexão com PostgreSQL

# 2. Erro de autenticação JWT
#    → Verificar JWT_SECRET e JWT_REFRESH_SECRET
#    → Devem ser diferentes e únicos

# 3. Erro de Redis
#    → Verificar REDIS_URI
#    → Testar conexão com Redis

# 4. Erro de código
#    → Verificar logs detalhados
#    → Revisar últimas alterações
#    → Considerar rollback
```

---

### 🔴 Frontend Não Carrega

**Sintomas:**
- Tela branca
- "Cannot GET /"
- Assets não carregam
- Console do navegador com erros

**Diagnóstico:**

1. **Verificar Console do Navegador (F12):**
```
- Network errors
- CORS errors
- Failed to load resource
- 404 errors
```

2. **Verificar Status do Serviço:**
```bash
# Easypanel → Frontend → Status
# Deve estar "Running" (verde)
```

**Soluções:**

```bash
# 1. Build não completou
#    → Verificar logs de build
#    → Fazer redeploy

# 2. REACT_APP_BACKEND_URL incorreto
#    → Corrigir variável
#    → Redeploy (variáveis de build precisam redeploy)

# 3. Assets não encontrados (404)
#    → Limpar cache do navegador (Ctrl + Shift + Del)
#    → Hard refresh (Ctrl + F5)

# 4. CORS error
#    → Verificar FRONTEND_URL no backend
#    → Deve bater com domínio atual
```

---

## 🔧 Backend

### Erro: "Cannot connect to database"

**Mensagem:**
```
Error: connect ECONNREFUSED atendechat-postgres:5432
```

**Causas:**
1. PostgreSQL não está rodando
2. Nome do host incorreto
3. Credenciais incorretas
4. Porta incorreta

**Solução:**

```bash
# 1. Verificar status do PostgreSQL
# Easypanel → atendechat-postgres → Status

# 2. Verificar variáveis
DB_HOST=atendechat-postgres  # Deve ser o nome do serviço
DB_PORT=5432
DB_USER=atendechat
DB_PASS=<senha_correta>
DB_NAME=atendechat

# 3. Testar conexão manual
# Console do backend:
psql -h atendechat-postgres -U atendechat -d atendechat

# 4. Se falhar, reiniciar PostgreSQL
# Easypanel → PostgreSQL → Restart
```

---

### Erro: "Redis connection failed"

**Mensagem:**
```
Error: Redis connection to atendechat-redis:6379 failed
```

**Causas:**
1. Redis não está rodando
2. URI incorreta
3. Senha incorreta

**Solução:**

```bash
# 1. Verificar Redis
# Easypanel → atendechat-redis → Status

# 2. Verificar variável
REDIS_URI=redis://:<senha>@atendechat-redis:6379
#               ↑ IMPORTANTE: dois pontos antes da senha

# 3. Testar conexão
# Console do Redis:
redis-cli -h atendechat-redis -a <senha>
PING  # Deve retornar: PONG

# 4. Reiniciar Redis se necessário
```

---

### Erro: "JWT malformed" ou "Invalid token"

**Mensagem:**
```
JsonWebTokenError: jwt malformed
Error: Invalid token
```

**Causas:**
1. JWT_SECRET alterado (invalidou tokens existentes)
2. JWT_SECRET não configurado
3. JWT_SECRET igual em ambientes diferentes

**Solução:**

```bash
# 1. Verificar se JWT_SECRET está configurado
# Backend → Environment → JWT_SECRET

# 2. Gerar novo se necessário
openssl rand -base64 64

# 3. ⚠️ ATENÇÃO: Alterar JWT_SECRET desloga todos os usuários
#    Avise os usuários antes!

# 4. Certifique-se que JWT_SECRET != JWT_REFRESH_SECRET

# 5. Reiniciar backend após alterar
```

---

### Erro: "Migrations failed"

**Mensagem:**
```
ERROR: relation "TableName" does not exist
Migration 20230515-add-column failed
```

**Solução:**

```bash
# 1. Acessar console do backend
# Easypanel → Backend → Console

# 2. Verificar status das migrations
npx sequelize db:migrate:status

# 3. Executar migrations pendentes
npx sequelize db:migrate

# 4. Se erro persistir, verificar:
#    - Sintaxe da migration
#    - Dependências entre migrations
#    - Rollback e reaplicar se necessário

# 5. Em último caso, restaurar backup do banco
```

---

## 🎨 Frontend

### Erro: "Failed to fetch" ou "Network Error"

**Sintomas:**
- Requisições falham
- Login não funciona
- Dados não carregam

**Causas:**
1. Backend não está acessível
2. CORS error
3. URL do backend incorreta

**Solução:**

```bash
# 1. Testar backend diretamente
curl https://api.seudominio.com/health

# 2. Verificar REACT_APP_BACKEND_URL
# Frontend → Environment
REACT_APP_BACKEND_URL=https://api.seudominio.com

# 3. Verificar CORS no backend
# Backend → Environment
FRONTEND_URL=https://app.seudominio.com

# 4. Verificar console do navegador (F12)
# Deve mostrar exatamente qual requisição falhou

# 5. Redeploy frontend se alterou variável
```

---

### Erro: "Mixed Content"

**Mensagem no Console:**
```
Mixed Content: The page at 'https://app...' was loaded over HTTPS, 
but requested an insecure resource 'http://api...'. 
This request has been blocked.
```

**Causa:**
- Frontend (HTTPS) tentando acessar backend (HTTP)

**Solução:**

```bash
# Garantir que backend esteja com HTTPS
REACT_APP_BACKEND_URL=https://api.seudominio.com
#                     ↑ HTTPS, não HTTP

# Redeploy frontend após corrigir
```

---

### Tela Branca / Blank Page

**Causas:**
1. Build falhou
2. Assets não encontrados
3. Erro de JavaScript no navegador
4. Rota do React Router não configurada

**Solução:**

```bash
# 1. Verificar console do navegador (F12)
# Procure por erros JavaScript

# 2. Verificar logs de build do frontend
# Easypanel → Frontend → Logs
# Procure por: "Build completed successfully"

# 3. Limpar cache do navegador
# Ctrl + Shift + Del → Limpar tudo

# 4. Hard refresh
# Ctrl + F5 (Windows/Linux)
# Cmd + Shift + R (Mac)

# 5. Verificar se serve está configurado corretamente
# Frontend Dockerfile deve ter:
# CMD ["serve", "-s", "build", "-l", "3001"]

# 6. Fazer redeploy se necessário
```

---

## 🗄️ Banco de Dados

### Banco está Lento

**Sintomas:**
- Requisições demoram muito
- Timeouts frequentes
- CPU do PostgreSQL alta

**Diagnóstico:**

```sql
-- Conexões ativas
SELECT count(*) FROM pg_stat_activity;

-- Queries lentas
SELECT pid, now() - query_start AS duration, query 
FROM pg_stat_activity 
WHERE state = 'active'
ORDER BY duration DESC;

-- Tamanho do banco
SELECT pg_size_pretty(pg_database_size('atendechat'));

-- Tabelas maiores
SELECT 
  schemaname||'.'||tablename AS table,
  pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC
LIMIT 10;
```

**Soluções:**

```bash
# 1. Adicionar índices em colunas muito consultadas
# CREATE INDEX idx_tickets_status ON "Tickets"(status);

# 2. Limpar dados antigos
# DELETE FROM "Messages" WHERE createdAt < NOW() - INTERVAL '6 months';

# 3. Executar VACUUM
# VACUUM ANALYZE;

# 4. Aumentar recursos do PostgreSQL no Easypanel
# CPU: 0.5 → 1.0 core
# RAM: 512MB → 1GB

# 5. Considerar sharding ou read replicas (casos extremos)
```

---

### Erro: "Too many connections"

**Mensagem:**
```
Error: too many connections for role "atendechat"
```

**Causa:**
- Pool de conexões não está sendo liberado
- Muitos backends rodando
- Leak de conexões

**Solução:**

```sql
-- Ver conexões atuais
SELECT * FROM pg_stat_activity 
WHERE datname = 'atendechat';

-- Matar conexões idle
SELECT pg_terminate_backend(pid)
FROM pg_stat_activity
WHERE datname = 'atendechat'
  AND state = 'idle'
  AND state_change < current_timestamp - INTERVAL '5 minutes';
```

```bash
# Ajustar pool de conexões no backend
# src/database/config.ts
pool: {
  max: 5,       # Reduzir se necessário
  min: 0,
  acquire: 30000,
  idle: 10000
}

# Reiniciar backend
```

---

## 🔴 Redis

### Redis Fora de Memória

**Mensagem:**
```
Error: OOM command not allowed when used memory > 'maxmemory'
```

**Causa:**
- Redis está cheio
- Muitos dados em cache
- Falta de política de eviction

**Solução:**

```bash
# 1. Ver uso de memória
redis-cli -h atendechat-redis -a <senha>
INFO memory

# 2. Ver chaves por tipo
redis-cli -h atendechat-redis -a <senha> --bigkeys

# 3. Limpar cache (CUIDADO!)
redis-cli -h atendechat-redis -a <senha>
FLUSHDB  # Limpa banco atual
# ou
FLUSHALL # Limpa tudo (USE COM CUIDADO!)

# 4. Aumentar memória do Redis no Easypanel
# RAM: 256MB → 512MB

# 5. Configurar política de eviction
# No Easypanel, adicionar variável:
# maxmemory-policy=allkeys-lru
```

---

## 🔒 SSL/Domínios

### Certificado SSL Não é Gerado

**Soluções por Causa:**

```bash
# 1. DNS não propagou
#    Aguardar 1-4 horas
#    Verificar: dig api.seudominio.com

# 2. DNS apontando errado
#    Corrigir IP no DNS
#    Aguardar propagação

# 3. Limite do Let's Encrypt
#    Máximo: 5 certificados/hora
#    Aguardar 1 hora

# 4. Porta 80 bloqueada
#    Verificar firewall do servidor
#    Let's Encrypt precisa acessar porta 80

# 5. Remover e adicionar domínio novamente
#    Easypanel → Domains → Remove → Add
```

---

### Erro: "Your connection is not private"

**Causa:**
- Certificado expirado
- Certificado inválido
- Relógio do servidor incorreto

**Solução:**

```bash
# 1. Forçar renovação
# Easypanel → Domains → Renew Certificate

# 2. Verificar data/hora do servidor
date

# 3. Limpar cache SSL do navegador
# Chrome: chrome://net-internals/#hsts
# Digite o domínio e clique em "Delete"

# 4. Testar com outro navegador/dispositivo

# 5. Verificar validade do certificado
# https://www.ssllabs.com/ssltest/
```

---

## 🚀 Performance

### Backend Lento

**Diagnóstico:**

```bash
# 1. Verificar uso de recursos
# Easypanel → Backend → Metrics
# CPU > 80% ? Aumentar recursos
# RAM > 90% ? Aumentar recursos

# 2. Verificar logs de performance
# Procure por queries lentas
# Procure por operações demoradas

# 3. Usar APM (Application Performance Monitoring)
# Considere: New Relic, Datadog, Sentry
```

**Soluções:**

```bash
# 1. Aumentar recursos
# CPU: 0.5 → 1.0 core
# RAM: 512MB → 1GB

# 2. Adicionar réplicas (load balancing)
# Easypanel → Backend → Replicas: 1 → 2

# 3. Otimizar queries do banco
# Adicionar índices
# Revisar queries N+1

# 4. Usar cache Redis mais agressivamente
# Cachear resultados de queries pesadas

# 5. Comprimir respostas (Gzip)
# Já configurado automaticamente no Easypanel
```

---

### Frontend Lento

**Soluções:**

```bash
# 1. Usar CDN (Cloudflare)
# Cacheia assets estáticos
# Reduz latência global

# 2. Otimizar imagens
# Comprimir/redimensionar
# Usar formatos modernos (WebP)

# 3. Code splitting
# Já configurado no Create React App

# 4. Lazy loading de componentes
# import() dinâmico

# 5. Service Worker / PWA
# Cache offline
```

---

## 🔍 Logs e Debugging

### Ver Logs em Tempo Real

```bash
# No Easypanel
Serviço → Logs → (refresh automático)

# Filtrar por tipo
Serviço → Logs → Filter: "error", "warning", etc.
```

---

### Logs Importantes do Backend

```bash
# Sucesso
✅ Server running on port 3000
✅ Database connected successfully
✅ Redis connected successfully
✅ Migrations executed

# Atenção
⚠️  Warning: High memory usage
⚠️  Deprecated API usage

# Erro
❌ Error connecting to database
❌ JWT authentication failed
❌ Migration failed
❌ WebSocket connection error
```

---

### Aumentar Verbosidade dos Logs

```bash
# Backend
# Adicionar variável:
LOG_LEVEL=debug

# Frontend (Build)
REACT_APP_DEBUG=true

# Reiniciar serviços
```

---

### Exportar Logs

```bash
# No Easypanel
Logs → Download Logs

# Ou via CLI (se disponível)
easypanel logs atendechat-backend > backend.log
```

---

## 📋 Checklist de Troubleshooting

Quando algo não funciona:

- [ ] Verificar status de todos os serviços (Running?)
- [ ] Verificar logs em tempo real
- [ ] Verificar variáveis de ambiente
- [ ] Testar conectividade (PostgreSQL, Redis)
- [ ] Verificar DNS e SSL
- [ ] Limpar cache do navegador
- [ ] Testar em navegador anônimo
- [ ] Verificar uso de recursos (CPU, RAM)
- [ ] Revisar últimas alterações (rollback se necessário)
- [ ] Consultar documentação específica

---

## 📞 Obter Ajuda

Se o problema persistir:

1. **Coletar Informações:**
   - Logs completos
   - Screenshots
   - Variáveis de ambiente (sem senhas!)
   - Passos para reproduzir

2. **Verificar Documentação:**
   - `docs/01-EASYPANEL-SETUP.md`
   - `docs/02-DEPLOYMENT-GUIDE.md`
   - `docs/03-ENVIRONMENT-VARIABLES.md`

3. **Comunidade:**
   - Easypanel Docs: https://docs.easypanel.io
   - Sequelize Docs: https://sequelize.org/docs
   - Stack Overflow

---

**🔧 Problema resolvido? Ótimo! Documente a solução para referência futura.**
