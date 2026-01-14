# 🔄 Guia de Migração: AAPanel → Easypanel

## 📋 Índice
- [Visão Geral](#visão-geral)
- [Comparação](#comparação)
- [Preparação](#preparação)
- [Backup](#backup)
- [Processo de Migração](#processo-de-migração)
- [Pós-Migração](#pós-migração)
- [Rollback](#rollback)

---

## 🎯 Visão Geral

Este guia detalha o processo completo de migração de uma instalação Atendechat existente no **AAPanel** para o **Easypanel**.

### Por que Migrar?

**AAPanel (Instalação Atual):**
- ❌ Instalação manual complexa
- ❌ Scripts bash extensos
- ❌ Gerenciamento manual de dependências
- ❌ Configuração manual de Nginx
- ❌ Renovação manual de SSL
- ❌ Múltiplos pontos de falha
- ❌ Difícil de escalar
- ❌ Troubleshooting complexo

**Easypanel (Nova Abordagem):**
- ✅ Deploy com 1 clique
- ✅ Interface visual moderna
- ✅ Automação completa
- ✅ SSL automático (Let's Encrypt)
- ✅ Backups automáticos
- ✅ Monitoramento integrado
- ✅ Escalabilidade fácil
- ✅ Logs centralizados

---

## 📊 Comparação Detalhada

### Processo de Instalação

| Tarefa | AAPanel | Easypanel |
|--------|---------|-----------|
| Instalar Node.js | Manual (script) | Automático (Docker) |
| Instalar PostgreSQL | Manual (apt/yum) | 1 clique |
| Instalar Redis | Manual (Docker) | 1 clique |
| Configurar Nginx | Manual (arquivos conf) | Automático |
| Configurar SSL | Manual (Certbot) | Automático |
| Criar usuário deploy | Manual (useradd) | Não necessário |
| Clonar repositório | Manual (git clone) | Automático (Git URL) |
| Instalar dependências | Manual (npm install) | Automático (Dockerfile) |
| Executar migrations | Manual (npx sequelize) | Automático (entrypoint) |
| Iniciar PM2 | Manual | Não necessário |
| **Tempo Total** | **2-4 horas** | **15-30 minutos** |

---

### Manutenção e Atualizações

| Tarefa | AAPanel | Easypanel |
|--------|---------|-----------|
| Atualizar código | Manual (git pull + restart) | 1 clique ou automático |
| Renovar SSL | Manual (certbot renew) | Automático |
| Fazer backup | Manual (scripts cron) | Automático + 1 clique |
| Ver logs | SSH + tail -f | Interface web |
| Monitorar recursos | SSH + top/htop | Dashboard visual |
| Adicionar instância | Executar script completo | Duplicar projeto |

---

## 🛠️ Preparação

### 1. Inventário da Instalação Atual

Documente sua instalação AAPanel atual:

```bash
# Conectar via SSH ao servidor AAPanel
ssh deploy@seu-servidor-aapanel.com

# Informações para coletar:
```

**Banco de Dados:**
```bash
# Nome do banco
psql -U postgres -c "\l" | grep atendechat

# Usuário do banco
cat /home/deploy/sua-instancia/backend/.env | grep DB_USER

# Senha do banco
cat /home/deploy/sua-instancia/backend/.env | grep DB_PASS

# Tamanho do banco
psql -U postgres -d atendechat -c "SELECT pg_size_pretty(pg_database_size('atendechat'));"
```

**Redis:**
```bash
# Porta do Redis
docker ps | grep redis

# Senha do Redis
cat /home/deploy/sua-instancia/backend/.env | grep REDIS_URI
```

**Domínios:**
```bash
# Domínios configurados
ls -la /etc/nginx/sites-enabled/

# Ver configurações
cat /etc/nginx/sites-enabled/sua-instancia-backend
cat /etc/nginx/sites-enabled/sua-instancia-frontend
```

**Variáveis de Ambiente:**
```bash
# Backend
cat /home/deploy/sua-instancia/backend/.env

# Frontend
cat /home/deploy/sua-instancia/frontend/.env
```

**Versão do Sistema:**
```bash
# Versão do código
cd /home/deploy/sua-instancia
git log -1 --oneline
git branch
```

---

### 2. Preparar Servidor Easypanel

1. **Provisionar Servidor:**
   - VPS/Cloud (DigitalOcean, AWS, Hetzner, etc.)
   - Mínimo: 2 GB RAM, 2 vCPUs, 50 GB SSD
   - Ubuntu 20.04/22.04 LTS recomendado

2. **Instalar Easypanel:**
```bash
curl -sSL https://get.easypanel.io | sh
```

3. **Acessar Easypanel:**
   - URL: `http://IP-DO-SERVIDOR:3000`
   - Criar conta administrador
   - Fazer login

4. **Configurar SSH (opcional mas recomendado):**
```bash
# Adicionar sua chave SSH
ssh-copy-id root@ip-do-servidor-easypanel
```

---

## 💾 Backup

### ⚠️ CRÍTICO: Fazer Backup Completo

Antes de qualquer migração, faça backup completo!

#### 1. Backup do Banco de Dados

```bash
# No servidor AAPanel
sudo su - postgres

# Backup completo
pg_dump -U atendechat atendechat > /tmp/atendechat_backup_$(date +%Y%m%d_%H%M%S).sql

# Copiar para local seguro
scp /tmp/atendechat_backup_*.sql usuario@seu-pc:~/backups/

# Ou baixar via SFTP
```

**Verificar backup:**
```bash
# Ver tamanho do arquivo
ls -lh /tmp/atendechat_backup_*.sql

# Ver primeiras linhas
head -n 50 /tmp/atendechat_backup_*.sql
```

---

#### 2. Backup dos Arquivos Públicos

```bash
# Arquivos enviados (fotos, áudios, documentos)
cd /home/deploy/sua-instancia/backend
tar -czf public_backup_$(date +%Y%m%d).tar.gz public/

# Copiar para local seguro
scp public_backup_*.tar.gz usuario@seu-pc:~/backups/
```

---

#### 3. Backup das Variáveis de Ambiente

```bash
# Backend
cp /home/deploy/sua-instancia/backend/.env ~/backup_env_backend.txt

# Frontend
cp /home/deploy/sua-instancia/frontend/.env ~/backup_env_frontend.txt

# Copiar para local seguro
```

---

#### 4. Backup da Configuração do Nginx

```bash
# Nginx configs
sudo cp -r /etc/nginx/sites-enabled ~/backup_nginx/

# Certificados SSL (se manual)
sudo cp -r /etc/letsencrypt ~/backup_ssl/
```

---

## 🚀 Processo de Migração

### Estratégia Recomendada: Blue-Green Deployment

**Blue (AAPanel - Atual):** Sistema em produção
**Green (Easypanel - Novo):** Sistema em teste

1. Configurar Easypanel (Green)
2. Testar completamente
3. Redirecionar tráfego gradualmente
4. Monitorar
5. Desligar AAPanel (Blue) apenas após confirmar sucesso

---

### Passo 1: Configurar Easypanel (Paralelo ao AAPanel)

1. **Criar Projeto no Easypanel:**
   - Nome: `atendechat-producao`
   - Seguir guia: [01-EASYPANEL-SETUP.md](./01-EASYPANEL-SETUP.md)

2. **Criar Serviços:**
   - PostgreSQL
   - Redis
   - Backend
   - Frontend

3. **NÃO configurar domínios ainda** (AAPanel ainda está usando)
   - Use subdomínios temporários: `test-api.seudominio.com`

---

### Passo 2: Restaurar Backup do Banco

```bash
# 1. Upload do backup para servidor Easypanel
scp atendechat_backup_*.sql root@ip-easypanel:/tmp/

# 2. No Easypanel, acessar console do PostgreSQL
# Easypanel → atendechat-postgres → Console

# 3. Restaurar backup
psql -U atendechat -d atendechat < /tmp/atendechat_backup_*.sql

# 4. Verificar
psql -U atendechat -d atendechat
\dt  # Listar tabelas
SELECT COUNT(*) FROM "Users";  # Contar usuários
\q
```

---

### Passo 3: Restaurar Arquivos Públicos

```bash
# 1. Upload dos arquivos
scp public_backup_*.tar.gz root@ip-easypanel:/tmp/

# 2. Acessar console do Backend no Easypanel
cd /app

# 3. Extrair arquivos
tar -xzf /tmp/public_backup_*.tar.gz

# 4. Verificar
ls -la public/
```

---

### Passo 4: Configurar Variáveis de Ambiente

Use as mesmas variáveis do AAPanel, ajustando:

```bash
# Backend
# DB_HOST: localhost → atendechat-postgres
# REDIS_URI: 127.0.0.1 → atendechat-redis
# Mantenha: JWT_SECRET, DB_PASS, etc. (mesmos valores!)
```

**⚠️ IMPORTANTE:**
- `JWT_SECRET` deve ser o MESMO (senão desloga todos)
- `DB_PASS` deve ser o MESMO (já configurado no backup)
- `BACKEND_URL` e `FRONTEND_URL` mudam temporariamente para teste

---

### Passo 5: Testar Instalação Easypanel

1. **Testar Backend:**
```bash
curl https://test-api.seudominio.com/health
```

2. **Testar Frontend:**
   - Acessar: `https://test-app.seudominio.com`
   - Fazer login com usuário existente
   - Testar funcionalidades principais

3. **Verificar Dados:**
   - Usuários importados?
   - Contatos existem?
   - Tickets antigos visíveis?
   - Mensagens carregam?

---

### Passo 6: Migração de Domínios (Switchover)

**⏰ Escolha horário de baixo tráfego** (madrugada, fim de semana)

**Método 1: Migração Gradual (Recomendado)**

1. **Manter AAPanel funcionando**

2. **Adicionar domínios no Easypanel:**
   - `api.seudominio.com`
   - `app.seudominio.com`

3. **Atualizar DNS (TTL baixo primeiro):**
```bash
# Reduzir TTL para 300 (5 minutos)
# 24 horas antes da migração
Tipo: A
Host: api
Valor: IP-AAPANEL
TTL: 300  ← Reduzido
```

4. **No dia da migração, atualizar IP:**
```bash
Tipo: A
Host: api
Valor: IP-EASYPANEL  ← Novo IP
TTL: 300

Tipo: A
Host: app
Valor: IP-EASYPANEL  ← Novo IP
TTL: 300
```

5. **Aguardar propagação (5-30 minutos)**

6. **Monitorar ambos os servidores:**
   - Tráfego do AAPanel deve cair gradualmente
   - Tráfego do Easypanel deve aumentar

7. **Após 100% do tráfego migrado (24-48h):**
   - Aumentar TTL de volta: 3600

---

**Método 2: Migração Instantânea (Mais Simples)**

1. **Colocar AAPanel em manutenção:**
```bash
# Adicionar página de manutenção no Nginx
# "Sistema em manutenção, retornaremos em breve"
```

2. **Fazer backup final:**
```bash
pg_dump -U atendechat atendechat > backup_final.sql
```

3. **Atualizar DNS:**
```bash
api.seudominio.com → IP-EASYPANEL
app.seudominio.com → IP-EASYPANEL
```

4. **Aguardar propagação (1-4 horas)**

5. **Remover manutenção**

6. **Testar tudo**

---

## ✅ Pós-Migração

### Checklist

- [ ] Usuários conseguem fazer login?
- [ ] Dados históricos visíveis?
- [ ] WhatsApp conectando?
- [ ] Mensagens sendo enviadas/recebidas?
- [ ] Tickets funcionando?
- [ ] Campanhas funcionando?
- [ ] Arquivos (imagens, áudios) carregando?
- [ ] SSL ativo e funcionando?
- [ ] Performance satisfatória?
- [ ] Logs sem erros críticos?

---

### Monitoramento Inicial

Primeiras 24-48 horas são críticas:

```bash
# Verificar logs constantemente
# Easypanel → Backend → Logs
# Easypanel → Frontend → Logs

# Verificar métricas
# CPU, RAM, Network, Disk

# Verificar banco de dados
# Conexões ativas
# Queries lentas
```

---

### Comunicação com Usuários

**Antes da Migração:**
```
Prezados usuários,

Realizaremos uma manutenção programada no sistema para 
melhorias de infraestrutura.

Data: DD/MM/AAAA
Horário: 02:00 - 04:00
Duração estimada: 2 horas

Durante este período, o sistema ficará indisponível.
Pedimos desculpas pelo inconveniente.

Equipe Atendechat
```

**Após a Migração:**
```
Manutenção concluída com sucesso!

O sistema está funcionando normalmente com 
melhorias significativas de performance e estabilidade.

Caso identifique qualquer problema, entre em contato.

Obrigado pela compreensão!
Equipe Atendechat
```

---

## ⏮️ Rollback

Se algo der muito errado:

### Rollback Completo para AAPanel

```bash
# 1. Atualizar DNS de volta
api.seudominio.com → IP-AAPANEL
app.seudominio.com → IP-AAPANEL

# 2. Aguardar propagação DNS

# 3. Verificar se AAPanel está funcionando

# 4. Analisar logs do Easypanel para identificar problema

# 5. Corrigir problema no Easypanel (sem pressa)

# 6. Tentar migração novamente em outra janela
```

---

## 🗑️ Desativar AAPanel

**Apenas após confirmar sucesso total (1-2 semanas):**

```bash
# 1. Fazer backup final do AAPanel
pg_dump > backup_aapanel_final.sql

# 2. Parar serviços
sudo su - deploy
pm2 stop all
pm2 delete all

# 3. Parar Nginx
sudo systemctl stop nginx

# 4. Parar PostgreSQL
sudo systemctl stop postgresql

# 5. Parar Redis
docker stop redis-*

# 6. Cancelar servidor (se VPS separado)
# Ou
# 7. Desinstalar AAPanel (se mesmo servidor)
```

---

## 📊 Benefícios Pós-Migração

Após migrar para Easypanel, você terá:

✅ **Deploy 10x mais rápido** (15 min vs 2-4h)
✅ **SSL sempre atualizado** (automático)
✅ **Backups automáticos** (diários)
✅ **Monitoramento visual** (dashboards)
✅ **Logs centralizados** (interface web)
✅ **Escalabilidade fácil** (+ replicas = 1 clique)
✅ **Rollback rápido** (1 clique)
✅ **Multi-tenant simplificado** (duplicar projeto)
✅ **Manutenção reduzida** (95% menos trabalho manual)

---

## 📋 Timeline Recomendada

### Semana 1-2: Preparação
- Estudar documentação Easypanel
- Provisionar servidor teste
- Fazer backups completos
- Documentar configuração atual

### Semana 3: Teste
- Instalar no Easypanel (ambiente de teste)
- Restaurar dados
- Testes completos
- Ajustes e correções

### Semana 4: Migração
- Escolher data/hora
- Comunicar usuários
- Executar migração
- Monitoramento intensivo

### Semana 5-6: Estabilização
- Monitorar
- Ajustar performance
- Coletar feedback
- Documentar lições aprendidas

### Semana 7+: Desativação AAPanel
- Confirmar sucesso total
- Desativar AAPanel
- Cancelar recursos antigos

---

## 📞 Suporte

- **Documentação Easypanel**: [01-EASYPANEL-SETUP.md](./01-EASYPANEL-SETUP.md)
- **Troubleshooting**: [07-TROUBLESHOOTING.md](./07-TROUBLESHOOTING.md)
- **Easypanel Docs**: https://docs.easypanel.io

---

**🎉 Parabéns! Você migrou com sucesso para uma infraestrutura moderna e automatizada!**
