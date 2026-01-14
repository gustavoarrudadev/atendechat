# 🔒 Configuração de Domínios e SSL

## 📋 Índice
- [Visão Geral](#visão-geral)
- [Configuração de DNS](#configuração-de-dns)
- [Configuração no Easypanel](#configuração-no-easypanel)
- [SSL Automático](#ssl-automático)
- [Domínios Customizados](#domínios-customizados)
- [Troubleshooting](#troubleshooting)

---

## 🎯 Visão Geral

O Easypanel gerencia automaticamente:
✅ Proxy reverso (Nginx/Traefik)
✅ Certificados SSL via Let's Encrypt
✅ Renovação automática de certificados
✅ Redirecionamento HTTP → HTTPS
✅ Configurações de segurança modernas

**Você NÃO precisa:**
❌ Instalar Nginx manualmente
❌ Configurar Certbot
❌ Criar arquivos de configuração
❌ Renovar certificados manualmente

---

## 🌐 Configuração de DNS

### Pré-requisitos

Você precisa de **2 domínios** por instância:
1. **Backend/API**: `api.seudominio.com` ou `api-cliente1.seudominio.com`
2. **Frontend/Painel**: `app.seudominio.com` ou `app-cliente1.seudominio.com`

---

### Obter IP do Servidor Easypanel

1. Acesse o Easypanel
2. Vá para **"Settings" → "Server"**
3. Anote o **IP público** do servidor

Exemplo: `203.0.113.45`

---

### Configurar Registros DNS

Acesse o painel DNS do seu provedor (Registro.br, GoDaddy, Cloudflare, etc.)

#### Opção 1: Subdomínios (Recomendado)

**Para Backend:**
```
Tipo: A
Host: api
Valor: 203.0.113.45
TTL: 3600
```

**Para Frontend:**
```
Tipo: A
Host: app
Valor: 203.0.113.45
TTL: 3600
```

**Resultado:**
- Backend: `api.seudominio.com`
- Frontend: `app.seudominio.com`

---

#### Opção 2: Multi-Tenant com Prefixos

**Cliente 1 - Backend:**
```
Tipo: A
Host: api-cliente1
Valor: 203.0.113.45
TTL: 3600
```

**Cliente 1 - Frontend:**
```
Tipo: A
Host: app-cliente1
Valor: 203.0.113.45
TTL: 3600
```

**Resultado:**
- Backend: `api-cliente1.seudominio.com`
- Frontend: `app-cliente1.seudominio.com`

---

#### Opção 3: Domínio Próprio do Cliente

Se o cliente tiver domínio próprio:

**No DNS do cliente:**
```
Tipo: A
Host: api
Valor: 203.0.113.45

Tipo: A
Host: app
Valor: 203.0.113.45
```

**Resultado:**
- Backend: `api.dominioocliente.com`
- Frontend: `app.dominioocliente.com`

---

### Verificar Propagação do DNS

Use ferramentas online para verificar:

```bash
# Via terminal (Linux/Mac)
dig api.seudominio.com
nslookup api.seudominio.com

# Via navegador
https://dnschecker.org/
https://www.whatsmydns.net/
```

**⏱️ Tempo de propagação:**
- Mínimo: 5-10 minutos
- Máximo: 48 horas
- Típico: 1-4 horas

---

## 🔧 Configuração no Easypanel

### Adicionar Domínio ao Backend

1. Acesse o serviço **Backend** no Easypanel
2. Vá para a aba **"Domains"**
3. Clique em **"Add Domain"**
4. Configure:
   - **Domain**: `api.seudominio.com`
   - **SSL**: Habilite **"Enable SSL"**
   - **Redirect HTTP to HTTPS**: Habilite
5. Clique em **"Save"**

O Easypanel irá automaticamente:
- ✅ Configurar proxy reverso
- ✅ Gerar certificado SSL via Let's Encrypt
- ✅ Ativar HTTPS

---

### Adicionar Domínio ao Frontend

1. Acesse o serviço **Frontend** no Easypanel
2. Vá para a aba **"Domains"**
3. Clique em **"Add Domain"**
4. Configure:
   - **Domain**: `app.seudominio.com`
   - **SSL**: Habilite **"Enable SSL"**
   - **Redirect HTTP to HTTPS**: Habilite
5. Clique em **"Save"**

---

## 🔐 SSL Automático

### Let's Encrypt

O Easypanel usa **Let's Encrypt** para certificados gratuitos:

**Características:**
- ✅ Certificados válidos por 90 dias
- ✅ Renovação automática (30 dias antes de expirar)
- ✅ Confiável por todos os navegadores
- ✅ Sem custo
- ✅ Ilimitado

---

### Processo Automático

Quando você adiciona um domínio:

1. **Easypanel verifica** se o DNS aponta corretamente
2. **Let's Encrypt valida** que você controla o domínio
3. **Certificado é emitido** em segundos
4. **HTTPS ativado** automaticamente
5. **Renovação agendada** automaticamente

---

### Verificar Status do SSL

1. No serviço, vá para **"Domains"**
2. Verifique o ícone do certificado:
   - 🟢 Verde = SSL ativo
   - 🟡 Amarelo = Processando
   - 🔴 Vermelho = Erro

3. Clique no domínio para ver detalhes:
   - Data de emissão
   - Data de expiração
   - Emissor (Let's Encrypt)

---

### Testar HTTPS

**No navegador:**
```
https://api.seudominio.com
https://app.seudominio.com
```

**Verificar certificado:**
1. Clique no cadeado 🔒 na barra de endereço
2. Clique em **"Certificado"** ou **"Certificate"**
3. Verifique:
   - ✅ Emissor: Let's Encrypt
   - ✅ Válido até: Data futura
   - ✅ Status: Válido

---

### Testar SSL Grade

Use ferramentas online:

```
https://www.ssllabs.com/ssltest/

# Digite seu domínio: api.seudominio.com
# Aguarde a análise (2-3 minutos)
# Grade esperada: A ou A+
```

---

## 🌍 Domínios Customizados

### Wildcard Domains (Opcional)

Para múltiplos clientes, use wildcard:

**No DNS:**
```
Tipo: A
Host: *
Valor: 203.0.113.45
```

**Resultado:**
- Qualquer subdomínio apontará para o servidor
- `cliente1.seudominio.com`
- `cliente2.seudominio.com`
- `qualquercoisa.seudominio.com`

**⚠️ Nota:** Você ainda precisa configurar cada domínio no Easypanel individualmente.

---

### Múltiplos Domínios no Mesmo Serviço

Você pode adicionar vários domínios para o mesmo serviço:

**Exemplo:**
- `api.seudominio.com` (principal)
- `api.outrodominio.com` (alternativo)
- `api-legacy.seudominio.com` (legado)

Todos apontam para o mesmo backend.

---

### Domínio Raiz (Root Domain)

Para usar o domínio raiz (sem `www` ou subdomínio):

**DNS:**
```
Tipo: A
Host: @
Valor: 203.0.113.45
```

**Resultado:**
- Frontend: `seudominio.com`
- Backend: `api.seudominio.com`

---

## 🛡️ Segurança Avançada

### Configurações Automáticas do Easypanel

O Easypanel já configura automaticamente:

✅ **TLS 1.2 e 1.3** (protocols modernos)
✅ **HSTS** (HTTP Strict Transport Security)
✅ **OCSP Stapling** (verificação de certificado rápida)
✅ **Ciphers modernos** (criptografia forte)
✅ **Redirecionamento HTTP → HTTPS**

---

### Headers de Segurança

O Easypanel adiciona headers:

```
Strict-Transport-Security: max-age=31536000
X-Frame-Options: SAMEORIGIN
X-Content-Type-Options: nosniff
X-XSS-Protection: 1; mode=block
Referrer-Policy: strict-origin-when-cross-origin
```

---

### Cloudflare (Opcional)

Para proteção adicional, use Cloudflare:

**Benefícios:**
- 🛡️ DDoS protection
- 🚀 CDN global (mais rápido)
- 🔥 Firewall WAF
- 📊 Analytics
- 🔒 SSL adicional (Cloudflare → Easypanel)

**Configuração:**

1. Adicione o domínio ao Cloudflare
2. Aponte os DNS para o Cloudflare (nameservers)
3. No Cloudflare, crie os registros A:
   ```
   api.seudominio.com → 203.0.113.45 (Proxied ☁️)
   app.seudominio.com → 203.0.113.45 (Proxied ☁️)
   ```
4. No Cloudflare, ative:
   - ✅ SSL/TLS → Full (strict)
   - ✅ Always Use HTTPS
   - ✅ Auto Minify (JS, CSS, HTML)
   - ✅ Brotli compression

---

## 🆘 Troubleshooting

### Problema: Certificado SSL não é gerado

**Causas comuns:**

1. **DNS não propagado**
   - Aguarde 1-4 horas
   - Verifique com `dig` ou `nslookup`

2. **DNS apontando incorretamente**
   - Confirme IP do servidor
   - Verifique registros A

3. **Porta 80/443 bloqueada**
   - Let's Encrypt precisa acessar porta 80
   - Verifique firewall do servidor

4. **Limite de rate do Let's Encrypt**
   - Máximo: 5 certificados/hora por domínio
   - Aguarde 1 hora e tente novamente

---

**Soluções:**

```bash
# 1. Verificar DNS
dig api.seudominio.com

# 2. Testar conectividade
curl http://api.seudominio.com

# 3. Verificar logs no Easypanel
# Aba "Logs" → procure por erros SSL/certificate

# 4. Remover e adicionar domínio novamente
# Easypanel → Domains → Remove → Add novamente
```

---

### Problema: HTTPS não funciona, mas HTTP sim

**Causa:** Certificado não foi gerado ou expirou

**Solução:**

1. Vá para **Domains** no Easypanel
2. Verifique status do certificado
3. Se expirado, clique em **"Renew Certificate"**
4. Se erro, remova e adicione o domínio novamente

---

### Problema: "Your connection is not private" (ERR_CERT_DATE_INVALID)

**Causa:** Certificado expirado

**Solução:**

1. No Easypanel, forçar renovação:
   - Domains → Renew Certificate
2. Aguardar 1-2 minutos
3. Limpar cache do navegador
4. Testar novamente

---

### Problema: Mixed Content (HTTP em página HTTPS)

**Causa:** Frontend carregando recursos via HTTP

**Solução:**

1. Verificar variável de ambiente:
   ```bash
   # Deve ser HTTPS, não HTTP
   REACT_APP_BACKEND_URL=https://api.seudominio.com
   ```

2. Redeploy do frontend

3. Limpar cache do navegador (`Ctrl + Shift + Del`)

---

### Problema: CORS Error após configurar domínio

**Causa:** Backend não reconhece o novo domínio do frontend

**Solução:**

1. Atualizar variável no backend:
   ```bash
   FRONTEND_URL=https://app.seudominio.com
   ```

2. Reiniciar backend

---

### Problema: Domínio funciona, mas sem HTTPS

**Causa:** SSL não foi habilitado

**Solução:**

1. Easypanel → Serviço → Domains
2. Clique no domínio
3. Habilite **"Enable SSL"**
4. Habilite **"Redirect HTTP to HTTPS"**
5. Save

---

## 📋 Checklist de Domínios

Antes de considerar completo:

### DNS:
- [ ] Registros A criados para backend
- [ ] Registros A criados para frontend
- [ ] DNS propagado (verificado com dig/nslookup)
- [ ] Apontando para IP correto do servidor

### Easypanel - Backend:
- [ ] Domínio adicionado
- [ ] SSL habilitado
- [ ] Certificado gerado com sucesso
- [ ] HTTPS funcionando
- [ ] Variável `BACKEND_URL` correta

### Easypanel - Frontend:
- [ ] Domínio adicionado
- [ ] SSL habilitado
- [ ] Certificado gerado com sucesso
- [ ] HTTPS funcionando
- [ ] Variável `REACT_APP_BACKEND_URL` correta

### Variáveis de Ambiente:
- [ ] `BACKEND_URL` com HTTPS
- [ ] `FRONTEND_URL` com HTTPS
- [ ] `REACT_APP_BACKEND_URL` com HTTPS
- [ ] Sem barras no final das URLs

### Testes:
- [ ] Backend acessível via HTTPS
- [ ] Frontend acessível via HTTPS
- [ ] Login funcionando
- [ ] WebSocket conectando
- [ ] Sem erros de Mixed Content
- [ ] Sem erros de CORS
- [ ] Grade SSL A ou A+ (ssllabs.com)

---

## 📞 Próximos Passos

- Configure múltiplos clientes: [04-MULTI-TENANT-SETUP.md](./04-MULTI-TENANT-SETUP.md)
- Configure backups: [02-DEPLOYMENT-GUIDE.md](./02-DEPLOYMENT-GUIDE.md#backups)
- Troubleshooting: [07-TROUBLESHOOTING.md](./07-TROUBLESHOOTING.md)

---

**🔒 Domínios e SSL configurados com segurança!**
