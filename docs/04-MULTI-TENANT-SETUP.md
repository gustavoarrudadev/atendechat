# 👥 Configuração Multi-Tenant (Múltiplas Instâncias)

## 📋 Índice
- [Visão Geral](#visão-geral)
- [Arquitetura Multi-Tenant](#arquitetura-multi-tenant)
- [Estratégias de Implementação](#estratégias-de-implementação)
- [Configuração no Easypanel](#configuração-no-easypanel)
- [Gerenciamento de Recursos](#gerenciamento-de-recursos)
- [Isolamento de Dados](#isolamento-de-dados)

---

## 🎯 Visão Geral

O Atendechat suporta múltiplas instâncias (multi-tenant) para atender diferentes clientes com isolamento completo de dados.

### Benefícios do Multi-Tenant:
✅ **Isolamento**: Cada cliente tem seus próprios dados e configurações
✅ **Escalabilidade**: Adicione novos clientes facilmente
✅ **Customização**: White label por cliente
✅ **Segurança**: Bancos de dados separados
✅ **Gestão**: Controle individual de recursos

---

## 🏗️ Arquitetura Multi-Tenant

### Estratégia 1: Banco de Dados Compartilhado (Mais Simples)

```
┌─────────────────────────────────────────┐
│          Easypanel Project              │
│                                         │
│  ┌──────────────┐  ┌──────────────┐   │
│  │  Backend 1   │  │  Frontend 1  │   │
│  │  (cliente1)  │  │  (cliente1)  │   │
│  └──────┬───────┘  └──────────────┘   │
│         │                               │
│  ┌──────┴───────┐  ┌──────────────┐   │
│  │  Backend 2   │  │  Frontend 2  │   │
│  │  (cliente2)  │  │  (cliente2)  │   │
│  └──────┬───────┘  └──────────────┘   │
│         │                               │
│    ┌────┴────┐      ┌──────────┐      │
│    │ Postgres│      │  Redis   │      │
│    │(shared) │      │(shared)  │      │
│    └─────────┘      └──────────┘      │
└─────────────────────────────────────────┘
```

**Características:**
- 1 banco PostgreSQL compartilhado
- 1 Redis compartilhado
- Múltiplos backends + frontends
- Isolamento via `companyId` nas tabelas

**Vantagens:**
- ✅ Mais econômico
- ✅ Mais fácil de gerenciar
- ✅ Menos recursos necessários

**Desvantagens:**
- ⚠️ Menor isolamento
- ⚠️ Performance compartilhada

---

### Estratégia 2: Banco de Dados Separado (Mais Seguro)

```
┌─────────────────────────────────────────┐
│       Easypanel Project - Cliente 1     │
│                                         │
│  ┌──────────────┐  ┌──────────────┐   │
│  │  Backend     │  │  Frontend    │   │
│  └──────┬───────┘  └──────────────┘   │
│         │                               │
│    ┌────┴────┐      ┌──────────┐      │
│    │Postgres │      │  Redis   │      │
│    └─────────┘      └──────────┘      │
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│       Easypanel Project - Cliente 2     │
│                                         │
│  ┌──────────────┐  ┌──────────────┐   │
│  │  Backend     │  │  Frontend    │   │
│  └──────┬───────┘  └──────────────┘   │
│         │                               │
│    ┌────┴────┐      ┌──────────┐      │
│    │Postgres │      │  Redis   │      │
│    └─────────┘      └──────────┘      │
└─────────────────────────────────────────┘
```

**Características:**
- Projeto Easypanel separado por cliente
- Banco e Redis dedicados
- Isolamento completo

**Vantagens:**
- ✅ Máximo isolamento
- ✅ Performance independente
- ✅ Backups independentes
- ✅ Escalabilidade por cliente

**Desvantagens:**
- ⚠️ Mais caro
- ⚠️ Mais complexo de gerenciar

---

## 🛠️ Estratégias de Implementação

### Opção A: Múltiplos Backends no Mesmo Projeto

**Recomendado para:** Até 10 clientes

#### Passo 1: Criar Projeto Base

1. Crie projeto: `atendechat-saas`
2. Configure PostgreSQL e Redis compartilhados

---

#### Passo 2: Adicionar Cliente 1

1. **Backend Cliente 1:**
   - Nome: `backend-cliente1`
   - Domínio: `api-cliente1.seudominio.com`
   - Variáveis de ambiente:
     ```bash
     BACKEND_URL=https://api-cliente1.seudominio.com
     FRONTEND_URL=https://app-cliente1.seudominio.com
     DB_HOST=atendechat-postgres
     REDIS_URI=redis://:senha@atendechat-redis:6379
     USER_LIMIT=5
     CONNECTIONS_LIMIT=3
     ```

2. **Frontend Cliente 1:**
   - Nome: `frontend-cliente1`
   - Domínio: `app-cliente1.seudominio.com`
   - Variáveis de ambiente:
     ```bash
     REACT_APP_BACKEND_URL=https://api-cliente1.seudominio.com
     STACK_NAME=cliente1
     REACT_APP_COLOR=#682EE3
     REACT_APP_TAB_NAME=Cliente 1 - Atendechat
     ```

---

#### Passo 3: Adicionar Cliente 2

Repita o processo acima com:
- `backend-cliente2` → `api-cliente2.seudominio.com`
- `frontend-cliente2` → `app-cliente2.seudominio.com`
- Altere `STACK_NAME=cliente2`
- Altere limites conforme plano do cliente

---

### Opção B: Projetos Separados (Recomendado)

**Recomendado para:** Clientes grandes ou +10 clientes

#### Template de Projeto

Crie um template e duplique para cada cliente:

1. **Criar Projeto Template:**
   - Nome: `atendechat-template`
   - Configure tudo uma vez
   - Documente as variáveis

2. **Duplicar para Cada Cliente:**
   - Easypanel permite duplicar projetos
   - Ajuste apenas variáveis específicas
   - Domínios únicos

---

## 🔧 Configuração no Easypanel

### Script de Automação (Opcional)

Crie um script para adicionar novos clientes:

```bash
#!/bin/bash
# add-cliente.sh

CLIENT_NAME=$1
CLIENT_DOMAIN=$2
USER_LIMIT=$3
CONNECTIONS_LIMIT=$4
COLOR=$5

echo "🚀 Adicionando cliente: $CLIENT_NAME"

# Variáveis
BACKEND_DOMAIN="api-${CLIENT_NAME}.${CLIENT_DOMAIN}"
FRONTEND_DOMAIN="app-${CLIENT_NAME}.${CLIENT_DOMAIN}"

# Gerar secrets únicos
JWT_SECRET=$(openssl rand -base64 64)
JWT_REFRESH_SECRET=$(openssl rand -base64 64)

echo "✅ Backend URL: https://${BACKEND_DOMAIN}"
echo "✅ Frontend URL: https://${FRONTEND_DOMAIN}"
echo "✅ JWT Secret gerado"

# Copiar para clipboard ou salvar em arquivo seguro
cat > "cliente-${CLIENT_NAME}.env" <<EOF
# Cliente: $CLIENT_NAME
BACKEND_URL=https://${BACKEND_DOMAIN}
FRONTEND_URL=https://${FRONTEND_DOMAIN}
USER_LIMIT=${USER_LIMIT}
CONNECTIONS_LIMIT=${CONNECTIONS_LIMIT}
JWT_SECRET=${JWT_SECRET}
JWT_REFRESH_SECRET=${JWT_REFRESH_SECRET}
REACT_APP_COLOR=${COLOR}
STACK_NAME=${CLIENT_NAME}
EOF

echo "📄 Configuração salva em: cliente-${CLIENT_NAME}.env"
echo "⚠️  ATENÇÃO: Guarde este arquivo em local seguro!"
```

**Uso:**
```bash
chmod +x add-cliente.sh
./add-cliente.sh empresa1 seudominio.com 10 5 "#FF5733"
```

---

## 📊 Gerenciamento de Recursos

### Definir Limites por Cliente

No Easypanel, configure limites de recursos:

#### Cliente Pequeno (Plano Básico):
```yaml
Backend:
  CPU: 0.25 core
  RAM: 256MB
  Replicas: 1
  USER_LIMIT: 3
  CONNECTIONS_LIMIT: 2

Frontend:
  CPU: 0.25 core
  RAM: 256MB
  Replicas: 1
```

#### Cliente Médio (Plano Pro):
```yaml
Backend:
  CPU: 0.5 core
  RAM: 512MB
  Replicas: 1
  USER_LIMIT: 10
  CONNECTIONS_LIMIT: 5

Frontend:
  CPU: 0.25 core
  RAM: 256MB
  Replicas: 1
```

#### Cliente Grande (Plano Enterprise):
```yaml
Backend:
  CPU: 1.0 core
  RAM: 1GB
  Replicas: 2
  USER_LIMIT: 50
  CONNECTIONS_LIMIT: 20

Frontend:
  CPU: 0.5 core
  RAM: 512MB
  Replicas: 2
```

---

## 🔒 Isolamento de Dados

### Banco de Dados Compartilhado

O Atendechat já possui isolamento nativo por `companyId`:

```sql
-- Todas as tabelas têm companyId
SELECT * FROM Users WHERE companyId = 1;
SELECT * FROM Tickets WHERE companyId = 1;
SELECT * FROM Messages WHERE companyId = 1;
```

**Como funciona:**
1. Cada cliente é uma "Company" no banco
2. Primeiro acesso cria a Company automaticamente
3. Todos os dados vinculados à Company
4. Impossível acessar dados de outra Company

---

### Banco de Dados Separado

Para isolamento total, use bancos separados:

#### PostgreSQL com Múltiplos Databases:

```bash
# Cliente 1
DB_NAME=atendechat_cliente1
DB_USER=atendechat_cliente1
DB_PASS=senha_cliente1

# Cliente 2
DB_NAME=atendechat_cliente2
DB_USER=atendechat_cliente2
DB_PASS=senha_cliente2
```

---

## 🎨 White Label por Cliente

### Customização Individual

Cada cliente pode ter:

1. **Logo Personalizado:**
   - Pasta: `brands/cliente1/`
   - Arquivos: `logo.png`, `favicon.ico`
   - Configurar: `STACK_NAME=cliente1`

2. **Cor do Tema:**
   ```bash
   REACT_APP_COLOR=#FF5733  # Cliente 1 (Vermelho)
   REACT_APP_COLOR=#682EE3  # Cliente 2 (Roxo)
   REACT_APP_COLOR=#00C853  # Cliente 3 (Verde)
   ```

3. **Nome da Aba:**
   ```bash
   REACT_APP_TAB_NAME=Empresa ABC - Atendimento
   REACT_APP_TAB_NAME=XYZ Suporte
   ```

---

## 📈 Monitoramento Multi-Tenant

### Dashboard Centralizado

Configure alertas por cliente:

```yaml
Cliente 1:
  - CPU > 80%
  - RAM > 90%
  - Downtime > 5min
  - Erros > 100/hora

Cliente 2:
  - CPU > 80%
  - RAM > 90%
  - Downtime > 5min
```

---

### Métricas por Cliente

Monitore individualmente:
- Uso de recursos (CPU, RAM)
- Número de usuários ativos
- Número de tickets/mês
- Volume de mensagens
- Uptime

---

## 💰 Billing e Controle

### Variáveis de Controle

```bash
# Limites por plano
USER_LIMIT=10           # Máximo de usuários
CONNECTIONS_LIMIT=5     # Máximo de WhatsApp
```

### Bloqueio Automático

O sistema já verifica limites:
- Bloqueia criação de novos usuários se `USER_LIMIT` atingido
- Bloqueia novas conexões WhatsApp se `CONNECTIONS_LIMIT` atingido

---

## 🔄 Migração entre Planos

### Upgrade de Plano

1. Atualizar variáveis:
   ```bash
   USER_LIMIT=20          # Era 10
   CONNECTIONS_LIMIT=10   # Era 5
   ```

2. Ajustar recursos no Easypanel:
   - CPU: 0.5 → 1.0 core
   - RAM: 512MB → 1GB

3. Reiniciar serviço

---

### Downgrade de Plano

⚠️ **ATENÇÃO**: Verificar se limites não estão sendo ultrapassados

1. Verificar uso atual
2. Remover excedentes (se necessário)
3. Atualizar variáveis
4. Ajustar recursos
5. Reiniciar

---

## 📋 Checklist: Adicionar Novo Cliente

- [ ] Escolher nome único para o cliente
- [ ] Configurar domínios DNS
- [ ] Criar serviços no Easypanel
- [ ] Gerar JWT secrets únicos
- [ ] Configurar variáveis de ambiente
- [ ] Definir limites (USER_LIMIT, CONNECTIONS_LIMIT)
- [ ] Configurar white label (cor, logo, nome)
- [ ] Fazer primeiro deploy
- [ ] Testar acesso
- [ ] Criar usuário administrador
- [ ] Configurar backups
- [ ] Configurar alertas
- [ ] Documentar credenciais (cofre seguro)

---

## 🆘 Troubleshooting Multi-Tenant

### Problema: Cliente não consegue criar usuários

**Causa**: `USER_LIMIT` atingido
**Solução**: 
1. Verificar quantos usuários existem
2. Aumentar `USER_LIMIT` se upgrade de plano
3. Remover usuários inativos se necessário

---

### Problema: Dados de clientes misturados

**Causa**: `companyId` incorreto (muito raro)
**Solução**:
1. Verificar logs
2. Revisar queries do banco
3. Contactar suporte

---

## 📞 Próximos Passos

- Leia: [05-SSL-DOMAINS.md](./05-SSL-DOMAINS.md)
- Configure: Automação de onboarding
- Implemente: Sistema de billing

---

**✅ Multi-tenant configurado com sucesso!**
