# 🚀 Atendechat - Deploy no Easypanel

## 📖 Visão Geral

O **Atendechat** é uma solução completa de atendimento via WhatsApp que aumenta a produtividade e organização das equipes de suporte e vendas.

Este repositório agora está **totalmente otimizado** para deploy no **Easypanel**, eliminando a necessidade de instaladores complexos e configurações manuais.

---

## ⚡ Deploy Rápido

**Tempo estimado: 15 minutos** ⏱️

### Método 1: Guia Rápido (Recomendado)

Siga o [**QUICKSTART.md**](./QUICKSTART.md) para fazer deploy em 15 minutos!

### Método 2: Documentação Completa

Para configuração detalhada, siga a [documentação completa](docs/01-EASYPANEL-SETUP.md).

---

## 📚 Documentação

### 📁 Guias Principais

| Documento | Descrição |
|-----------|-----------|
| [🚀 Quickstart](./QUICKSTART.md) | Deploy rápido em 15 minutos |
| [01 - Easypanel Setup](docs/01-EASYPANEL-SETUP.md) | Guia completo de configuração |
| [02 - Deployment Guide](docs/02-DEPLOYMENT-GUIDE.md) | Deploy, atualizações e CI/CD |
| [03 - Environment Variables](docs/03-ENVIRONMENT-VARIABLES.md) | Todas as variáveis documentadas |
| [04 - Multi-Tenant Setup](docs/04-MULTI-TENANT-SETUP.md) | Configurar múltiplas instâncias |
| [05 - SSL & Domains](docs/05-SSL-DOMAINS.md) | Configuração de domínios e SSL |
| [06 - Database Migrations](docs/06-DATABASE-MIGRATIONS.md) | Gerenciamento de banco de dados |
| [07 - Troubleshooting](docs/07-TROUBLESHOOTING.md) | Resolução de problemas |
| [08 - Migration from AAPanel](docs/08-MIGRATION-FROM-AAPANEL.md) | Migrar do AAPanel para Easypanel |

---

## 🏗️ Arquitetura

```
┌─────────────────────────────────────────┐
│          Easypanel (Gestão)             │
│                                         │
│  ┌──────────────┐  ┌──────────────┐   │
│  │   Backend    │  │   Frontend   │   │
│  │   (Node.js)  │  │   (React)    │   │
│  │   Port 3000  │  │   Port 3001  │   │
│  └──────┬───────┘  └──────────────┘   │
│         │                               │
│    ┌────┴────┐      ┌──────────┐      │
│    │Postgres │      │  Redis   │      │
│    │  (DB)   │      │ (Cache)  │      │
│    └─────────┘      └──────────┘      │
│                                         │
│  SSL: Let's Encrypt (Automático)       │
│  Proxy: Nginx/Traefik (Automático)     │
└─────────────────────────────────────────┘
```

---

## ✨ Funcionalidades

### 💬 Atendimento Multi-Canal
- ✅ WhatsApp integrado (via Baileys)
- ✅ Múltiplas conexões simultâneas
- ✅ Atendimento em tempo real
- ✅ Histórico completo de conversas

### 👥 Gestão de Equipe
- ✅ Usuários e permissões
- ✅ Filas de atendimento
- ✅ Distribuição automática de tickets
- ✅ Transferência entre atendentes

### 🤖 Automação
- ✅ Respostas rápidas
- ✅ Chatbots configuráveis
- ✅ Campanhas de mensagens em massa
- ✅ Agendamento de mensagens
- ✅ Fluxos de atendimento (Flow Builder)

### 📊 Relatórios e Analytics
- ✅ Dashboard em tempo real
- ✅ Relatórios de atendimento
- ✅ Métricas de performance
- ✅ Histórico de campanhas

### 🎨 White Label
- ✅ Customização de cores
- ✅ Logo personalizado
- ✅ Nome da empresa
- ✅ Multi-tenant (múltiplos clientes)

---

## 🛠️ Stack Tecnológica

### Backend
- **Node.js** 20.x
- **Express** (API REST)
- **TypeScript**
- **Sequelize** (ORM)
- **PostgreSQL** (Banco de dados)
- **Redis** (Cache e filas)
- **Baileys** (WhatsApp integration)
- **Socket.io** (WebSocket real-time)

### Frontend
- **React** 17.x
- **Material-UI** 4.x
- **Socket.io Client**
- **Axios**
- **React Router**
- **Zustand** (State management)

### DevOps
- **Docker** (Containers)
- **Easypanel** (Orquestração)
- **Nginx** (Proxy reverso - automático)
- **Let's Encrypt** (SSL - automático)

---

## 📦 Requisitos Mínimos

### Servidor
- **RAM**: 2 GB mínimo, 4 GB recomendado
- **CPU**: 2 vCPUs mínimo
- **Disco**: 50 GB SSD
- **SO**: Ubuntu 20.04/22.04 LTS

### Domínios
- 2 domínios ou subdomínios:
  - `api.seudominio.com` (Backend)
  - `app.seudominio.com` (Frontend)

### Serviços
- **Easypanel** instalado e configurado
- **Git** (para clonar repositório)

---

## 🚀 Deploy no Easypanel

### Opção 1: Deploy Rápido (15 minutos)

```bash
# 1. Acesse o Easypanel
https://seu-easypanel.com:3000

# 2. Siga o guia rápido
docs/QUICKSTART.md

# 3. Pronto! ✅
```

### Opção 2: Deploy Completo (30 minutos)

```bash
# 1. Leia a documentação completa
docs/01-EASYPANEL-SETUP.md

# 2. Configure passo a passo
# 3. Deploy com customizações avançadas
```

---

## 🔐 Segurança

### Práticas Implementadas

✅ **Autenticação JWT**  
✅ **Senhas criptografadas (bcrypt)**  
✅ **HTTPS obrigatório**  
✅ **CORS configurado**  
✅ **Rate limiting**  
✅ **Variáveis de ambiente seguras**  
✅ **Sanitização de inputs**  
✅ **Headers de segurança**

### Recomendações

1. Use senhas fortes (mínimo 16 caracteres)
2. Gere JWT secrets únicos e aleatórios
3. Nunca commite arquivos `.env`
4. Mantenha sistema atualizado
5. Configure backups automáticos
6. Monitore logs regularmente

---

## 📊 Performance

### Otimizações

- ✅ Redis para cache
- ✅ Índices no banco de dados
- ✅ Compressão Gzip
- ✅ Lazy loading no frontend
- ✅ Code splitting
- ✅ Assets otimizados
- ✅ Connection pooling

### Escalabilidade

- ✅ Horizontal (múltiplas replicas)
- ✅ Load balancing automático
- ✅ Cache distribuído (Redis)
- ✅ Database read replicas (opcional)

---

## 🔄 Atualizações

### Deploy Automático

Configure webhook no Easypanel para deploy automático ao fazer push no Git:

```bash
git add .
git commit -m "feat: nova funcionalidade"
git push origin main

# Easypanel faz deploy automaticamente! 🚀
```

### Deploy Manual

1. Acesse o Easypanel
2. Vá para o serviço (Backend ou Frontend)
3. Clique em **"Deploy"**
4. Aguarde o build
5. Pronto! ✅

---

## 🆘 Suporte

### Documentação

- [📖 Docs Completa](docs/)
- [⚡ Quickstart](./QUICKSTART.md)
- [🔧 Troubleshooting](docs/07-TROUBLESHOOTING.md)

### Comunidade

- Easypanel Docs: [docs.easypanel.io](https://docs.easypanel.io)
- Issues: [GitHub Issues](https://github.com/seu-usuario/codatendechat/issues)

---

## 📝 Changelog

### v6.0.1 (Atual)
- ✅ Otimizado para Easypanel
- ✅ Documentação completa
- ✅ SSL automático
- ✅ Deploy em 15 minutos
- ✅ Multi-tenant simplificado

### Versões Anteriores

Ver [CHANGELOG.md](./CHANGELOG.md)

---

## 📜 Licença

Este projeto está sob licença proprietária.

Todos os direitos reservados © [Atendechat](https://atendechat.com)

---

## 🤝 Contribuindo

Para contribuir com o projeto:

1. Fork o repositório
2. Crie uma branch: `git checkout -b feature/nova-feature`
3. Commit suas mudanças: `git commit -m 'feat: adiciona nova feature'`
4. Push para a branch: `git push origin feature/nova-feature`
5. Abra um Pull Request

---

## 🎯 Roadmap

### Em Desenvolvimento
- [ ] Suporte a Instagram Direct
- [ ] Suporte a Telegram
- [ ] API pública (webhooks)
- [ ] Integração com CRMs
- [ ] App mobile (React Native)

### Planejado
- [ ] Inteligência Artificial (ChatGPT)
- [ ] Análise de sentimento
- [ ] Transcrição de áudio automática
- [ ] Relatórios avançados

---

## 📞 Contato

- **Website**: [atendechat.com](https://atendechat.com)
- **Email**: suporte@atendechat.com
- **Documentação**: [docs/](docs/)

---

## 🎉 Agradecimentos

Agradecemos a todos que contribuíram para tornar o Atendechat uma solução robusta e fácil de usar!

**Tecnologias utilizadas:**
- [Node.js](https://nodejs.org/)
- [React](https://react.dev/)
- [PostgreSQL](https://www.postgresql.org/)
- [Redis](https://redis.io/)
- [Easypanel](https://easypanel.io/)
- [Baileys](https://github.com/WhiskeySockets/Baileys)

---

**⚡ Deploy rápido, gestão fácil, resultados incríveis!**

**Feito com ❤️ pelo time Atendechat**
