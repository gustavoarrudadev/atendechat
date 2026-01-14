# 📦 Guia de Deploy e Atualizações

## 📋 Índice
- [Deploy Inicial](#deploy-inicial)
- [Atualizações e Redeploy](#atualizações-e-redeploy)
- [CI/CD Automático](#cicd-automático)
- [Rollback](#rollback)
- [Backups](#backups)
- [Monitoramento](#monitoramento)

---

## 🚀 Deploy Inicial

### Pré-requisitos

Antes de fazer o deploy inicial, certifique-se de que:

✅ Todos os serviços foram criados no Easypanel
✅ Variáveis de ambiente estão configuradas
✅ Domínios estão apontando corretamente
✅ PostgreSQL e Redis estão rodando

---

### Fluxo de Deploy Inicial

#### 1. Deploy do Backend

O backend deve ser deployado primeiro pois:
- Executa as migrations do banco de dados
- Cria a estrutura inicial de tabelas
- Popula dados iniciais (seeds)

**Passos:**

1. Acesse o serviço `atendechat-backend` no Easypanel
2. Clique em **"Deploy"**
3. Aguarde o build completo (5-10 minutos na primeira vez)
4. Acompanhe os logs em tempo real
5. Verifique se aparece: `✅ Database migrated successfully`

**Verificação:**
```bash
# Teste a API
curl https://api.seudominio.com/health

# Deve retornar algo como:
{"status": "ok", "timestamp": "2026-01-14T..."}
```

---

#### 2. Deploy do Frontend

Após o backend estar rodando:

1. Acesse o serviço `atendechat-frontend` no Easypanel
2. Clique em **"Deploy"**
3. Aguarde o build (3-5 minutos)
4. Acompanhe os logs
5. Verifique se aparece: `✅ Build completed successfully`

**Verificação:**
```bash
# Acesse no navegador
https://app.seudominio.com

# Deve exibir a tela de login
```

---

#### 3. Primeiro Acesso

Após o deploy completo:

1. Acesse: `https://app.seudominio.com`
2. Faça login com as credenciais padrão:
   - **Email**: `admin@atendechat.com`
   - **Senha**: `admin123`
3. **IMPORTANTE**: Altere a senha imediatamente!

---

## 🔄 Atualizações e Redeploy

### Quando Fazer Redeploy?

Faça redeploy quando:
- Houver atualizações no código (git pull)
- Alterar variáveis de ambiente
- Corrigir bugs ou adicionar features
- Atualizar dependências

---

### Tipos de Atualizações

#### 1. Atualização Simples (Sem Alteração de Banco)

Quando **não há** alterações no banco de dados (novas migrations):

**Backend:**
1. Commit e push suas alterações no Git
2. No Easypanel, clique em `atendechat-backend`
3. Clique em **"Deploy"**
4. Aguarde o build
5. O serviço será reiniciado automaticamente

**Frontend:**
1. Commit e push suas alterações no Git
2. No Easypanel, clique em `atendechat-frontend`
3. Clique em **"Deploy"**
4. Aguarde o build
5. Limpe o cache do navegador (`Ctrl + F5`)

---

#### 2. Atualização com Migrations (Alteração de Banco)

Quando **há novas migrations**:

**⚠️ ATENÇÃO:** Faça backup do banco antes!

**Passos:**

1. **Fazer Backup do Banco:**
```bash
# No console do PostgreSQL no Easypanel
pg_dump -U atendechat atendechat > backup_$(date +%Y%m%d_%H%M%S).sql
```

2. **Deploy do Backend:**
```bash
# O backend executará automaticamente:
# - npm install (novas dependências)
# - npm run build (compilação)
# - npx sequelize db:migrate (migrations)
```

3. **Verificar Logs:**
```bash
# Procure por:
✅ Migrations executed successfully
✅ Server running on port 3000
```

4. **Testar API:**
```bash
curl https://api.seudominio.com/health
```

5. **Deploy do Frontend** (se necessário)

---

#### 3. Atualização de Variáveis de Ambiente

Quando alterar variáveis de ambiente:

1. No Easypanel, clique no serviço
2. Vá para **"Environment"**
3. Edite as variáveis necessárias
4. Clique em **"Save"**
5. Clique em **"Restart"** (o serviço reinicia automaticamente)

**Nota:** Não precisa fazer redeploy completo.

---

## 🤖 CI/CD Automático (Opcional)

### Configurar Deploy Automático no Easypanel

O Easypanel pode fazer deploy automático quando você faz push no Git.

#### Passo 1: Configurar Webhook

1. No serviço (Backend ou Frontend), vá para **"Settings"**
2. Habilite **"Auto Deploy"**
3. Selecione a branch (ex: `main` ou `production`)
4. Salve as configurações

#### Passo 2: Testar

```bash
# Faça uma alteração no código
git add .
git commit -m "test: deploy automático"
git push origin main

# O Easypanel iniciará o deploy automaticamente
```

---

### GitHub Actions (CI/CD Avançado)

Para testes automáticos antes do deploy:

Crie o arquivo `.github/workflows/deploy.yml`:

```yaml
name: Deploy to Easypanel

on:
  push:
    branches: [ main, production ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '20'
      
      - name: Install Backend Dependencies
        run: |
          cd backend
          npm install
      
      - name: Build Backend
        run: |
          cd backend
          npm run build
      
      - name: Run Tests
        run: |
          cd backend
          npm test
  
  deploy:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - name: Deploy to Easypanel
        run: |
          curl -X POST ${{ secrets.EASYPANEL_WEBHOOK_URL }}
```

**Configuração:**
1. No GitHub, vá para **Settings → Secrets**
2. Adicione o secret: `EASYPANEL_WEBHOOK_URL`
3. Cole a URL do webhook do Easypanel

---

## ⏮️ Rollback (Voltar Versão Anterior)

Se algo der errado após um deploy:

### Método 1: Rollback via Easypanel

1. Clique no serviço com problema
2. Vá para **"Deployments"** (histórico)
3. Encontre o deploy anterior (que funcionava)
4. Clique em **"Rollback to this version"**
5. Aguarde o processo

---

### Método 2: Rollback via Git

```bash
# Voltar para commit anterior
git log --oneline  # Ver histórico
git revert <commit-hash>  # Reverter commit específico
git push origin main

# Ou voltar para tag específica
git checkout v6.0.0
git push origin main --force  # ⚠️ Use com cuidado!
```

Depois force o redeploy no Easypanel.

---

## 💾 Backups

### Backup Automático no Easypanel

O Easypanel oferece backups automáticos do PostgreSQL:

1. Clique no serviço `atendechat-postgres`
2. Vá para **"Backups"**
3. Configure:
   - **Frequência**: Diário (recomendado)
   - **Retenção**: 7-30 dias
   - **Horário**: Madrugada (menor tráfego)

---

### Backup Manual

#### Backup Completo do Banco:

```bash
# No console do serviço PostgreSQL
pg_dump -U atendechat atendechat > backup_$(date +%Y%m%d_%H%M%S).sql

# Download do backup
# Use o painel do Easypanel para baixar o arquivo
```

#### Backup Específico (Tabelas):

```bash
# Backup de tabelas específicas
pg_dump -U atendechat -t Users -t Companies atendechat > backup_users_companies.sql
```

---

### Restaurar Backup

```bash
# No console do PostgreSQL
psql -U atendechat atendechat < backup_20260114_120000.sql
```

---

## 📊 Monitoramento

### Métricas do Easypanel

O Easypanel fornece métricas em tempo real:

1. Clique no serviço
2. Vá para **"Metrics"**
3. Visualize:
   - 📈 CPU Usage
   - 📈 Memory Usage
   - 📈 Network I/O
   - 📈 Disk I/O

---

### Logs em Tempo Real

```bash
# No Easypanel, aba "Logs"
# Ou use o CLI do Easypanel:
easypanel logs atendechat-backend --follow
```

---

### Alertas e Notificações

Configure alertas no Easypanel:

1. Vá para **"Project Settings"**
2. Configure **"Notifications"**
3. Adicione:
   - Email
   - Slack
   - Discord
   - Webhook customizado

Alertas disponíveis:
- ⚠️ Serviço inativo (downtime)
- ⚠️ Alto uso de CPU (>80%)
- ⚠️ Alto uso de memória (>90%)
- ⚠️ Build falhou
- ⚠️ Deploy falhou

---

### Health Checks

Configure health checks automáticos:

1. No serviço, vá para **"Health Checks"**
2. Configure:
   - **Path**: `/health`
   - **Interval**: 30 segundos
   - **Timeout**: 5 segundos
   - **Retries**: 3

O Easypanel reiniciará automaticamente o serviço se falhar.

---

## 🔍 Logs Importantes

### Backend - Logs a Monitorar:

```bash
✅ Server running on port 3000
✅ Database connected successfully
✅ Redis connected successfully
❌ Error connecting to database
❌ JWT authentication failed
❌ WebSocket connection error
```

---

### Frontend - Logs a Monitorar:

```bash
✅ Build completed successfully
✅ Serving on port 3001
❌ Failed to fetch from backend
❌ WebSocket connection failed
```

---

## 📈 Performance e Otimização

### Otimizações Recomendadas:

#### 1. Cache Redis
```bash
# Já configurado no sistema
# Verifique se o Redis está sendo usado:
redis-cli -h atendechat-redis ping
# Deve retornar: PONG
```

#### 2. Compressão Gzip
```bash
# Já configurado no Nginx do Easypanel
# Reduz tamanho dos assets em até 70%
```

#### 3. Cache de Assets Estáticos
```bash
# Configurado automaticamente no frontend
# Navegador cacheia CSS, JS, imagens
```

#### 4. CDN (Opcional)
Para grandes volumes de tráfego, considere usar CDN:
- Cloudflare (grátis)
- AWS CloudFront
- Fastly

---

## 🎯 Checklist de Deploy

Antes de cada deploy importante:

- [ ] Backup do banco de dados criado
- [ ] Código testado localmente
- [ ] Migrations revisadas
- [ ] Variáveis de ambiente verificadas
- [ ] Changelog atualizado
- [ ] Usuários notificados (se downtime)
- [ ] Horário de baixo tráfego escolhido
- [ ] Plano de rollback definido

---

## 📞 Suporte

- **Documentação**: Veja `docs/07-TROUBLESHOOTING.md`
- **Logs**: Sempre verifique os logs primeiro
- **Easypanel Support**: [docs.easypanel.io](https://docs.easypanel.io)

---

**✅ Deploy realizado com sucesso!**
