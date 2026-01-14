# 🗄️ Migrations e Gerenciamento de Banco de Dados

## 📋 Índice
- [Visão Geral](#visão-geral)
- [Estrutura de Migrations](#estrutura-de-migrations)
- [Execução Automática](#execução-automática)
- [Execução Manual](#execução-manual)
- [Seeds (Dados Iniciais)](#seeds-dados-iniciais)
- [Rollback](#rollback)
- [Troubleshooting](#troubleshooting)

---

## 🎯 Visão Geral

O Atendechat usa **Sequelize** como ORM para gerenciar o banco de dados PostgreSQL.

**Migrations** são arquivos que definem mudanças no esquema do banco:
- ✅ Criar tabelas
- ✅ Adicionar colunas
- ✅ Remover colunas
- ✅ Criar índices
- ✅ Alterar tipos de dados
- ✅ Adicionar constraints (foreign keys, unique, etc.)

---

## 📁 Estrutura de Migrations

### Localização

```
backend/
├── src/
│   └── database/
│       ├── migrations/
│       │   ├── 20210101000000-create-users.ts
│       │   ├── 20210101000001-create-companies.ts
│       │   ├── 20210101000002-create-contacts.ts
│       │   └── ... (160+ migrations)
│       └── seeds/
│           ├── 20210101000000-create-default-settings.js
│           ├── 20210101000001-create-admin-user.js
│           └── ... (seeds iniciais)
```

---

### Ordem de Execução

As migrations são executadas em ordem cronológica (timestamp no nome):

```
20210101000000 → 20210101000001 → 20210101000002 → ...
```

O Sequelize mantém controle no banco (tabela `SequelizeMeta`):

```sql
SELECT * FROM "SequelizeMeta";

-- Resultado:
--  name
-- ------------------------------------
--  20210101000000-create-users.js
--  20210101000001-create-companies.js
--  20210101000002-create-contacts.js
```

---

## ⚙️ Execução Automática

### Durante o Deploy no Easypanel

As migrations são executadas **automaticamente** durante o deploy via `docker-entrypoint.sh`:

```bash
# backend/docker-entrypoint.sh

#!/bin/sh
echo "🔄 Aguardando banco de dados..."
# (aguarda PostgreSQL estar pronto)

echo "🗄️  Executando migrations..."
npx sequelize db:migrate

echo "🌱 Executando seeds..."
npx sequelize db:seed:all

echo "🚀 Iniciando servidor..."
node dist/server.js
```

**Fluxo no Easypanel:**
1. Build do Docker image
2. Container inicia
3. `docker-entrypoint.sh` é executado
4. Migrations aplicadas automaticamente
5. Seeds aplicados (se necessário)
6. Servidor inicia

---

## 🛠️ Execução Manual

### Acessar Console do Backend

1. No Easypanel, clique no serviço **Backend**
2. Vá para a aba **"Console"** ou **"Terminal"**
3. Um terminal interativo será aberto

---

### Comandos de Migration

#### Executar Migrations Pendentes

```bash
# Aplica todas as migrations que ainda não foram executadas
npx sequelize db:migrate
```

**Output esperado:**
```
Sequelize CLI [Node: 20.x.x, CLI: 6.x.x, ORM: 5.x.x]

Loaded configuration file "src/database/config.ts".
Using environment "production".
== 20230515120000-add-column-to-users: migrating =======
== 20230515120000-add-column-to-users: migrated (0.045s)
```

---

#### Ver Status das Migrations

```bash
# Lista todas as migrations e seu status
npx sequelize db:migrate:status
```

**Output:**
```
up  20210101000000-create-users.js
up  20210101000001-create-companies.js
up  20210101000002-create-contacts.js
down 20230515120000-add-new-feature.js  ← Pendente
```

- **up**: Migration executada
- **down**: Migration pendente

---

#### Reverter Última Migration (Rollback)

```bash
# Desfaz a última migration executada
npx sequelize db:migrate:undo
```

⚠️ **ATENÇÃO:** Use com cuidado em produção!

---

#### Reverter Todas as Migrations

```bash
# Desfaz TODAS as migrations (limpa o banco)
npx sequelize db:migrate:undo:all
```

⚠️ **PERIGO:** Isso apaga TODAS as tabelas! Use apenas em desenvolvimento!

---

#### Reverter até Migration Específica

```bash
# Desfaz migrations até chegar em uma específica
npx sequelize db:migrate:undo:all --to 20210101000002-create-contacts.js
```

---

## 🌱 Seeds (Dados Iniciais)

### O que são Seeds?

Seeds são scripts que populam o banco com **dados iniciais** necessários:

- ✅ Configurações padrão do sistema
- ✅ Usuário administrador inicial
- ✅ Filas padrão de atendimento
- ✅ Tags padrão
- ✅ Mensagens rápidas padrão

---

### Executar Seeds

```bash
# Executa todos os seeds
npx sequelize db:seed:all
```

**⚠️ IMPORTANTE:**
- Seeds podem ser executados múltiplas vezes
- Alguns seeds verificam se dados já existem antes de criar
- Seeds NÃO são versionados como migrations (não há controle de "já executado")

---

### Ver Lista de Seeds

```bash
ls -la src/database/seeds/
```

Seeds disponíveis:
- `20210101000000-create-default-settings.js` - Configurações padrão
- `20210101000001-create-admin-user.js` - Usuário admin
- `20210101000002-create-default-queues.js` - Filas padrão
- `20210101000003-create-default-tags.js` - Tags padrão

---

### Dados Padrão Criados pelos Seeds

#### Usuário Administrador

```
Email: admin@atendechat.com
Senha: admin123
Perfil: Administrador
```

⚠️ **IMPORTANTE:** Altere essa senha após primeiro login!

#### Configurações Padrão

```yaml
- Nome da empresa
- Horário de expediente
- Mensagem de ausência
- Tempo de inatividade
- Limites de conexão
```

---

## 🔄 Criar Nova Migration

### Quando Criar?

Crie uma nova migration quando precisar:
- Adicionar nova tabela
- Adicionar/remover colunas
- Alterar tipo de dados
- Criar índices
- Modificar estrutura do banco

---

### Comando para Criar

```bash
# Gera um arquivo de migration
npx sequelize migration:generate --name nome-da-migration
```

**Exemplo:**
```bash
npx sequelize migration:generate --name add-premium-field-to-companies
```

Isso cria:
```
src/database/migrations/20260114120000-add-premium-field-to-companies.js
```

---

### Estrutura de uma Migration

```javascript
'use strict';

module.exports = {
  // Executado quando aplica a migration
  up: async (queryInterface, Sequelize) => {
    await queryInterface.addColumn('Companies', 'premium', {
      type: Sequelize.BOOLEAN,
      defaultValue: false,
      allowNull: false
    });
  },

  // Executado quando reverte a migration
  down: async (queryInterface, Sequelize) => {
    await queryInterface.removeColumn('Companies', 'premium');
  }
};
```

---

### Operações Comuns

#### Criar Tabela

```javascript
up: async (queryInterface, Sequelize) => {
  await queryInterface.createTable('TableName', {
    id: {
      type: Sequelize.INTEGER,
      primaryKey: true,
      autoIncrement: true,
      allowNull: false
    },
    name: {
      type: Sequelize.STRING,
      allowNull: false
    },
    createdAt: {
      type: Sequelize.DATE,
      allowNull: false
    },
    updatedAt: {
      type: Sequelize.DATE,
      allowNull: false
    }
  });
}
```

---

#### Adicionar Coluna

```javascript
up: async (queryInterface, Sequelize) => {
  await queryInterface.addColumn('TableName', 'newColumn', {
    type: Sequelize.STRING,
    allowNull: true
  });
}
```

---

#### Remover Coluna

```javascript
up: async (queryInterface, Sequelize) => {
  await queryInterface.removeColumn('TableName', 'oldColumn');
}
```

---

#### Criar Índice

```javascript
up: async (queryInterface, Sequelize) => {
  await queryInterface.addIndex('TableName', ['columnName'], {
    name: 'idx_table_column',
    unique: false
  });
}
```

---

#### Adicionar Foreign Key

```javascript
up: async (queryInterface, Sequelize) => {
  await queryInterface.addConstraint('ChildTable', {
    fields: ['parentId'],
    type: 'foreign key',
    name: 'fk_child_parent',
    references: {
      table: 'ParentTable',
      field: 'id'
    },
    onDelete: 'CASCADE',
    onUpdate: 'CASCADE'
  });
}
```

---

## 🔙 Rollback (Reverter Migrations)

### Quando Fazer Rollback?

- ⚠️ Migration causou erro
- ⚠️ Dados corrompidos
- ⚠️ Estrutura incorreta
- ⚠️ Problema de performance

---

### Processo de Rollback

1. **Fazer Backup do Banco** (SEMPRE!)

```bash
# No console do PostgreSQL no Easypanel
pg_dump -U atendechat atendechat > backup_antes_rollback.sql
```

2. **Verificar Qual Migration Reverter**

```bash
npx sequelize db:migrate:status
```

3. **Executar Rollback**

```bash
# Reverter última migration
npx sequelize db:migrate:undo
```

4. **Verificar Banco**

```bash
# Conectar ao PostgreSQL
psql -U atendechat -d atendechat

# Verificar tabelas
\dt

# Verificar estrutura de tabela
\d+ TableName
```

5. **Corrigir Migration**

Edite o arquivo da migration e corrija o erro.

6. **Aplicar Novamente**

```bash
npx sequelize db:migrate
```

---

## 🆘 Troubleshooting

### Erro: "Relation does not exist"

**Causa:** Tabela não foi criada

**Solução:**
```bash
# Verificar status das migrations
npx sequelize db:migrate:status

# Executar migrations pendentes
npx sequelize db:migrate
```

---

### Erro: "Column already exists"

**Causa:** Migration tentando adicionar coluna que já existe

**Solução:**

1. **Verificar se migration já foi executada:**
```bash
npx sequelize db:migrate:status
```

2. **Se aparecer como "up", pular essa migration:**
```sql
-- Adicionar manualmente no banco
INSERT INTO "SequelizeMeta" (name) 
VALUES ('20230515120000-nome-da-migration.js');
```

---

### Erro: "Cannot run migration... previous migration failed"

**Causa:** Migration anterior falhou e deixou o banco em estado inconsistente

**Solução:**

1. **Verificar última migration executada:**
```bash
npx sequelize db:migrate:status
```

2. **Reverter migration problemática:**
```bash
npx sequelize db:migrate:undo
```

3. **Corrigir problema e executar novamente:**
```bash
npx sequelize db:migrate
```

---

### Erro: "Database connection refused"

**Causa:** Backend não consegue conectar ao PostgreSQL

**Solução:**

1. **Verificar se PostgreSQL está rodando:**
```bash
# No Easypanel, verificar status do serviço PostgreSQL
```

2. **Verificar variáveis de ambiente:**
```bash
DB_HOST=atendechat-postgres
DB_PORT=5432
DB_USER=atendechat
DB_PASS=<senha_correta>
DB_NAME=atendechat
```

3. **Testar conexão manual:**
```bash
psql -h atendechat-postgres -U atendechat -d atendechat
```

---

### Migration Demorou Muito (Timeout)

**Causa:** Migration complexa em tabela grande

**Solução:**

1. **Executar em horário de baixo tráfego**
2. **Aumentar timeout no Sequelize config:**

```javascript
// src/database/config.ts
dialectOptions: {
  statement_timeout: 60000 // 60 segundos
}
```

3. **Quebrar migration grande em várias menores**

---

## 📊 Boas Práticas

### ✅ FAZER:

- ✅ Sempre fazer backup antes de migrations em produção
- ✅ Testar migrations em ambiente de staging primeiro
- ✅ Criar migration reversa (down) funcional
- ✅ Usar transações em migrations complexas
- ✅ Documentar migrations complexas com comentários
- ✅ Versionar migrations no Git
- ✅ Executar migrations em horário de baixo tráfego

---

### ❌ NÃO FAZER:

- ❌ Editar migrations antigas já executadas em produção
- ❌ Deletar migrations que já foram executadas
- ❌ Commitar dados sensíveis em seeds
- ❌ Executar migrations manualmente em produção sem backup
- ❌ Fazer alterações diretas no banco (sempre via migration)

---

## 🔍 Comandos Úteis do PostgreSQL

```bash
# Conectar ao banco
psql -U atendechat -d atendechat

# Listar tabelas
\dt

# Descrever tabela
\d+ TableName

# Ver tamanho do banco
SELECT pg_size_pretty(pg_database_size('atendechat'));

# Ver tabelas maiores
SELECT 
  tablename,
  pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename))
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC
LIMIT 10;

# Sair do psql
\q
```

---

## 📋 Checklist de Migration

Antes de aplicar migration em produção:

- [ ] Testada em desenvolvimento
- [ ] Testada em staging
- [ ] Backup do banco criado
- [ ] Migration reversa (down) funcional
- [ ] Documentação atualizada
- [ ] Equipe notificada
- [ ] Horário de baixo tráfego escolhido
- [ ] Plano de rollback definido
- [ ] Monitoramento ativo

---

## 📞 Próximos Passos

- Troubleshooting: [07-TROUBLESHOOTING.md](./07-TROUBLESHOOTING.md)
- Backups: [02-DEPLOYMENT-GUIDE.md](./02-DEPLOYMENT-GUIDE.md#backups)

---

**✅ Migrations gerenciadas com segurança!**
