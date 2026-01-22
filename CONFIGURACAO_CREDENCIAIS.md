# 🔐 Configuração de Credenciais

Este documento explica como configurar as credenciais necessárias para executar os workflows deste repositório.

## ⚠️ Importante

**NUNCA** commite credenciais reais no repositório. Sempre use variáveis de ambiente ou o sistema de credenciais do n8n.

## 📋 Credenciais Necessárias por Workflow

### 1. Sincronização Biologix-Airtable

**API Biologix Sleep:**
- `username`: Seu usuário da API Biologix
- `password`: Sua senha da API Biologix
- `centerID`: ID do centro (exemplo: 4798042LW)

**Airtable:**
- Personal Access Token do Airtable
- Base ID: `apphVHj0lGyVJqB4P`
- Tabelas: `Pacientes`, `Exames`

**Como configurar:**
1. No node "Requisição de autenticação da API Biologix1", substitua:
   - `YOUR_USERNAME` pelo seu username
   - `YOUR_PASSWORD` pela sua senha
2. Configure a credencial Airtable no n8n com seu Personal Access Token

---

### 2. Segmentação de Leads por Pipeline

**Google Sheets:**
- OAuth2 do Google Drive/Sheets
- Documento ID: `1ul0pVYoVsrckKM2YOKD_gBtcnYAOWr6wIcXbDIX5ceA`
- Abas: Venda Perdida, Ganho, Desqualificado, etc.

**GoHighLevel:**
- Webhook configurado no GHL para enviar atualizações

**Como configurar:**
1. Configure OAuth2 do Google no n8n
2. Configure o webhook no GHL apontando para a URL do n8n
3. Ajuste o Document ID e Sheet IDs conforme necessário

---

### 3. Salvar Conversas e Análise Integrada IA

**GoHighLevel (GHL):**
- API Token (PIT Token)
- Substitua `YOUR_GHL_API_TOKEN` em:
  - Node "Buscar Conversation ID" (header Authorization)
  - Node "Buscar Msg 1" (header Authorization)
  - Node "Next Page Conversas" (código JavaScript)

**Anthropic Claude:**
- API Key do Claude
- Configure no n8n como credencial OpenAI/Anthropic

**OpenAI:**
- API Key do OpenAI
- Usado para gerar embeddings
- Configure no n8n como credencial OpenAI

**Supabase:**
- API Key do Supabase
- URL do projeto: `https://qyrkyvoilfaxppbvtkpi.supabase.co`
- Tabelas: `conversations`, `error_logs`
- Configure no n8n como credencial Supabase

**Como configurar:**
1. No node "Buscar Conversation ID", substitua `YOUR_GHL_API_TOKEN`
2. No node "Buscar Msg 1", substitua `YOUR_GHL_API_TOKEN`
3. No código do node "Next Page Conversas", substitua `YOUR_GHL_API_TOKEN`
4. Configure credenciais do Claude, OpenAI e Supabase no n8n

---

### 4. Transcritor de Curso

**Google Drive:**
- OAuth2 do Google Drive
- Pasta origem: "Audios para serem transcritos"
- Pasta destino: "N8N"

**AssemblyAI:**
- API Key do AssemblyAI
- Substitua `YOUR_ASSEMBLYAI_API_KEY` no node "Fazendo Upload do arquivo"

**Como configurar:**
1. Configure OAuth2 do Google no n8n
2. No node "Fazendo Upload do arquivo", substitua `YOUR_ASSEMBLYAI_API_KEY`
3. Ajuste os IDs das pastas do Google Drive conforme necessário

---

## 🔧 Configuração no n8n

### Método Recomendado: Credenciais do n8n

1. Acesse **Settings** → **Credentials** no n8n
2. Crie credenciais para cada serviço:
   - **Airtable**: Personal Access Token
   - **Google**: OAuth2
   - **OpenAI/Anthropic**: API Key
   - **Supabase**: API Key
   - **AssemblyAI**: API Key (HTTP Header Auth)
3. Nos workflows, selecione as credenciais criadas ao invés de hardcode

### Método Alternativo: Variáveis de Ambiente

Para tokens que precisam estar no código (como no JavaScript):

1. Configure variáveis de ambiente no n8n:
   ```bash
   GHL_API_TOKEN=seu_token_aqui
   ASSEMBLYAI_API_KEY=sua_key_aqui
   ```

2. Use no código:
   ```javascript
   const authToken = process.env.GHL_API_TOKEN;
   ```

---

## ✅ Checklist de Segurança

- [ ] Todas as credenciais foram removidas dos arquivos JSON
- [ ] Credenciais estão configuradas no n8n (não no código)
- [ ] `.gitignore` está configurado corretamente
- [ ] Nenhuma credencial foi commitada no histórico do Git
- [ ] Tokens expirados foram revogados e novos gerados
- [ ] Acesso ao repositório está restrito

---

## 🆘 Problemas Comuns

**Erro: "Invalid credentials"**
- Verifique se as credenciais estão corretas no n8n
- Confirme que os tokens não expiraram
- Verifique permissões das credenciais

**Erro: "API rate limit exceeded"**
- Aguarde alguns minutos antes de tentar novamente
- Considere implementar rate limiting no workflow

**Erro: "Webhook not found"**
- Verifique se o webhook está ativo no n8n
- Confirme a URL do webhook no serviço externo (GHL)

---

## 📝 Notas

- Sempre use o sistema de credenciais do n8n quando possível
- Para tokens em código JavaScript, considere usar variáveis de ambiente
- Revise regularmente as credenciais e revogue tokens não utilizados
- Mantenha backups seguros das configurações (sem credenciais)
