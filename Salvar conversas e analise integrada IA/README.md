# Salvar Conversas e Análise Integrada IA

## 📋 Visão Geral

Sistema avançado que exporta conversas completas do GoHighLevel, processa com Inteligência Artificial (Claude e OpenAI) para análise de sentimento, qualidade e insights, e armazena no Supabase com embeddings vetoriais para busca semântica. Inclui sistema de tratamento de erros dedicado.

## 🎯 Objetivo

- Exportar conversas completas do GoHighLevel (com paginação)
- Processar transcrições com IA para análise de qualidade e sentimento
- Gerar embeddings vetoriais para busca semântica
- Armazenar dados estruturados no Supabase
- Monitorar e registrar erros automaticamente

## 📊 Fluxo do Workflow Principal

<img width="2487" height="397" alt="1" src="https://github.com/user-attachments/assets/d82b9a92-3e05-471e-a03a-1e6cc09030e3" />
<img width="2093" height="752" alt="2" src="https://github.com/user-attachments/assets/5fe84328-31d5-46b8-8fb9-4cc087520a81" />




### Fluxo Detalhado:

1. **Webhook/Trigger** → Recebe evento do GHL (atualização de oportunidade)
2. **Normalizar Payload GHL** → Padroniza dados do webhook
3. **Preparar Dados** → Extrai campos essenciais (contact_id, pipeline, etc.)
4. **Buscar Conversation ID** → Encontra ID da conversa no GHL
5. **Extrair Conversation ID** → Isola ID e metadados
6. **Buscar Mensagens (Paginação)** → Busca todas as mensagens com paginação
7. **Prep Transcript** → Processa e formata transcrição completa
8. **Verificar Transcrição** → Valida se há transcrição válida
9. **Análise com Claude** → Analisa conversa com IA (sentimento, qualidade, insights)
10. **Truncar Script** → Limita tamanho para API (se necessário)
11. **Gerar Embedding** → Cria vetor de embedding da análise
12. **Salvar no Supabase** → Armazena dados completos com embedding

## 🔧 Nodes Utilizados (Workflow Principal)

### 1. Normalizar Payload GHL
- **Tipo:** `n8n-nodes-base.code`
- **Função:** Normaliza e padroniza dados do webhook GHL
- **Processamento:**
  - Extrai dados do contato (nome, email, telefone)
  - Normaliza dados de pipeline e estágio
  - Processa dados clínicos e de marketing
  - Organiza First Touch e Last Touch
  - Converte tags para array
- **Saída:** Estrutura padronizada com todos os campos relevantes

### 2. Preparar Dados
- **Tipo:** `n8n-nodes-base.set`
- **Função:** Extrai campos essenciais do payload normalizado
- **Campos extraídos:**
  - `contact_id`, `contact_name`, `contact_email`, `contact_phone`
  - `pipeline_name`, `pipeline_stage`, `pipeline_id`
  - `location_id`, `sdr_nome`, `sdr_email`
  - `tratamento_interesse`, `tags`

### 3. Buscar Conversation ID
- **Tipo:** `n8n-nodes-base.httpRequest`
- **Função:** Busca ID da conversa no GHL
- **Endpoint:** `https://services.leadconnectorhq.com/conversations/search`
- **Método:** GET
- **Parâmetros:** `locationId`, `contactId`, `pageSize=1000`
- **Headers:** Authorization Bearer Token, Version, Accept

### 4. Extrair Conversation ID
- **Tipo:** `n8n-nodes-base.set`
- **Função:** Extrai ID da primeira conversa encontrada
- **Campos:** `conversation_id`, `canal_raw`, `locationId`

### 5. Buscar Msg 1
- **Tipo:** `n8n-nodes-base.httpRequest`
- **Função:** Busca primeira página de mensagens
- **Endpoint:** `https://services.leadconnectorhq.com/conversations/{id}/messages`
- **Parâmetros:** `locationId`, `pageSize=1000`

### 6. ACC
- **Tipo:** `n8n-nodes-base.set`
- **Função:** Acumula mensagens da primeira página
- **Campos:** `acc` (array de mensagens), `lastId1`, `next1` (has next page)

### 7. Next Page Conversas
- **Tipo:** `n8n-nodes-base.code`
- **Função:** Implementa paginação completa de mensagens
- **Lógica:**
  - Continua buscando enquanto `nextPage === true`
  - Filtra mensagens do sistema (tipo 28, 31, 32, 33, 34, 35)
  - Remove mensagens com `body` vazio ou `#switch`
  - Remove mensagens com status `failed`
  - Limite de segurança: 100.000 mensagens
- **Retorna:** Array completo de mensagens válidas

### 8. Prep Transcript
- **Tipo:** `n8n-nodes-base.code`
- **Função:** Processa e formata transcrição completa
- **Processamento:**
  - Extrai todas as mensagens (incluindo páginas aninhadas)
  - Classifica mensagens por role (LEAD, SDR, WORKFLOW)
  - Formata timestamps (dd/MM HH:mm)
  - Processa anexos (áudio, imagem, PDF, documento, vídeo)
  - Extrai eventos do sistema (agendamento, pagamento, status)
  - Gera transcrição em formato texto
  - Cria resumo estatístico
- **Saída:**
  - `transcricao`: Texto completo formatado
  - `lead_nome`: Nome do contato
  - `resumo`: Estatísticas (total, lead, sdr, workflow, período)
  - `eventos`: Array de eventos do sistema
  - `audios_para_transcrever`: Array de áudios encontrados
  - `imagens_para_descrever`: Array de imagens encontradas
  - `documentos_para_ler`: Array de documentos encontrados

### 9. Verificar Transcrição
- **Tipo:** `n8n-nodes-base.if`
- **Função:** Valida se há transcrição válida
- **Condição:** `transcricao` não está vazio
- **True:** Continua para análise com IA
- **False:** Usa valores fallback

### 10. Fallback (Sem Transcrição)
- **Tipo:** `n8n-nodes-base.set`
- **Função:** Define valores padrão quando não há transcrição
- **Valores:**
  - `score_qualidade`: 0
  - `sentimento`: "sem_analise"
  - `feedback_ia`: "Transcrição vazia - não foi possível analisar"

### 11. Análise com Claude (Anthropic)
- **Tipo:** `n8n-nodes-base.httpRequest` ou `n8n-nodes-base.anthropic`
- **Função:** Analisa conversa com Claude AI
- **Modelo:** Claude Sonnet 4 (ou similar)
- **Prompt:** Análise de qualidade, sentimento, insights, recomendações
- **Saída:** JSON estruturado com análise completa

### 12. Truncar Script
- **Tipo:** `n8n-nodes-base.code`
- **Função:** Limita tamanho da transcrição para API
- **Limite:** 15.000 caracteres (~3.75k tokens)
- **Estratégia:** Mantém início (50%) e fim (50%)
- **Motivo:** Claude tem limite de 8.192 tokens total

### 13. Preparar Embedding da Análise
- **Tipo:** `n8n-nodes-base.code`
- **Função:** Extrai JSON da análise do Claude para embedding
- **Processamento:**
  - Remove markdown code blocks
  - Extrai JSON puro
  - Converte para string JSON

### 14. Gerar Embedding (OpenAI)
- **Tipo:** `n8n-nodes-base.httpRequest`
- **Função:** Gera vetor de embedding da análise
- **Endpoint:** `https://api.openai.com/v1/embeddings`
- **Modelo:** `text-embedding-3-small`
- **Input:** JSON stringificado da análise
- **Output:** Array de 1536 dimensões

### 15. Extrair Embedding
- **Tipo:** `n8n-nodes-base.code`
- **Função:** Combina análise + embedding + metadados
- **Processamento:**
  - Extrai embedding do output OpenAI
  - Extrai JSON da análise do Claude
  - Combina com campos essenciais do Prep Transcript
  - Remove dados desnecessários (mensagens_json, transcricao_markdown)

### 16. Merge Análise + Fallback
- **Tipo:** `n8n-nodes-base.merge`
- **Função:** Une branches (com IA ou sem IA)
- **Estratégia:** Merge por índice

### 17. Calcular Status
- **Tipo:** `n8n-nodes-base.set`
- **Função:** Calcula flags de status baseado em tags
- **Campos:**
  - `agendou`: true se tem tag 'agendado', 'confirmado', 'compareceu', 'ganho'
  - `compareceu`: true se tem tag 'ganho', 'compareceu'
  - `fechou_venda`: true se tem tag 'ganho'

### 18. Salvar no Supabase
- **Tipo:** `n8n-nodes-base.httpRequest` ou `n8n-nodes-base.postgres`
- **Função:** Insere/atualiza registro no Supabase
- **Endpoint:** `https://qyrkyvoilfaxppbvtkpi.supabase.co/rest/v1/conversations`
- **Método:** POST (ou UPSERT)
- **Dados salvos:**
  - Metadados da conversa (contact_id, conversation_id, pipeline)
  - Transcrição completa
  - Análise da IA (JSON)
  - Embedding vetorial (1536 dimensões)
  - Estatísticas e resumo
  - Status calculados

## 🔧 Workflow de Tratamento de Erros

![Error Handler](./Error%20Handler%20-%20GHL%20Conversas.png)

### Fluxo do Error Handler:

1. **Error Trigger** → Captura erros de qualquer workflow
2. **Format Error Message** → Formata dados do erro
3. **Preparar Body para Supabase** → Estrutura dados para salvamento
4. **Salvar Erro no Supabase** → Registra erro na tabela `error_logs`

### Nodes do Error Handler:

#### 1. Error Trigger
- **Tipo:** `n8n-nodes-base.errorTrigger`
- **Função:** Captura erros de execução
- **Escopo:** Workflow "GHL - Exportar Conversas (novo)"

#### 2. Format Error Message
- **Tipo:** `n8n-nodes-base.code`
- **Função:** Extrai e formata informações do erro
- **Campos extraídos:**
  - `workflow_name`, `execution_id`
  - `error_type`, `error_message`
  - `node_name`, `stack_trace`
  - `contact_id`, `conversation_id` (se disponível)

#### 3. Preparar Body para Supabase
- **Tipo:** `n8n-nodes-base.code`
- **Função:** Estrutura dados para inserção
- **Campos:** Todos os campos do Format Error Message

#### 4. Salvar Erro no Supabase
- **Tipo:** `n8n-nodes-base.httpRequest`
- **Função:** Salva erro na tabela `error_logs`
- **Endpoint:** `https://qyrkyvoilfaxppbvtkpi.supabase.co/rest/v1/error_logs`
- **Método:** POST
- **Headers:** Authorization (Supabase API Key), Content-Type, Prefer

## 🔌 Integrações Externas

### GoHighLevel (GHL)
- **Tipo:** CRM/Plataforma de Comunicação
- **APIs utilizadas:**
  - Conversations Search API
  - Messages API (com paginação)
- **Autenticação:** Bearer Token (PIT token)
- **Dados extraídos:**
  - Conversas completas
  - Mensagens (texto, anexos, metadados)
  - Eventos do sistema

### Anthropic Claude API
- **Tipo:** IA Generativa
- **Modelo:** Claude Sonnet 4
- **Função:** Análise de conversas
- **Análise realizada:**
  - Qualidade da conversa (score 0-100)
  - Sentimento (positivo, neutro, negativo)
  - Insights e recomendações
  - Identificação de oportunidades
  - Análise de objeções

### OpenAI API
- **Tipo:** IA Generativa
- **Modelo:** `text-embedding-3-small`
- **Função:** Geração de embeddings vetoriais
- **Dimensões:** 1536
- **Uso:** Busca semântica no Supabase

### Supabase
- **Tipo:** Backend as a Service (PostgreSQL)
- **Função:** Armazenamento de dados
- **Tabelas utilizadas:**
  - `conversations`: Dados completos das conversas
  - `error_logs`: Logs de erros
- **Recursos:**
  - PostgreSQL com extensões
  - Busca vetorial (pgvector)
  - REST API
  - Autenticação

## 📈 Dados Processados

### Estrutura de Dados no Supabase

**Tabela `conversations`:**
- `id`: UUID (chave primária)
- `contact_id`: ID do contato no GHL
- `conversation_id`: ID da conversa no GHL
- `contact_name`, `contact_email`, `contact_phone`
- `pipeline_name`, `pipeline_stage`
- `sdr_nome`, `sdr_email`
- `transcricao`: Texto completo da conversa
- `transcricao_texto`: Versão limpa (sem markdown)
- `analise_ia`: JSON completo da análise do Claude
- `embedding`: Array de 1536 dimensões (vetor)
- `score_qualidade`: 0-100
- `sentimento`: string (positivo, neutro, negativo)
- `feedback_ia`: Texto com insights
- `resumo`: JSON com estatísticas
- `eventos`: Array de eventos do sistema
- `agendou`, `compareceu`, `fechou_venda`: boolean
- `created_at`, `updated_at`: timestamps

**Tabela `error_logs`:**
- `id`: UUID
- `workflow_name`: Nome do workflow
- `execution_id`: ID da execução
- `error_type`: Tipo do erro
- `error_message`: Mensagem do erro
- `node_name`: Nome do node que falhou
- `contact_id`, `conversation_id`: Contexto (se disponível)
- `stack_trace`: Stack trace completo
- `full_error`: Objeto completo do erro
- `created_at`: Timestamp

## ⚙️ Lógica de Negócio

1. **Paginação Completa:**
   - Busca todas as mensagens, não apenas primeira página
   - Implementa loop de paginação automático
   - Limite de segurança: 100.000 mensagens

2. **Filtragem Inteligente:**
   - Remove mensagens do sistema (tipo 28, 31, 32, 33, 34, 35)
   - Remove mensagens com `#switch`
   - Remove mensagens com status `failed`
   - Mantém apenas mensagens úteis

3. **Processamento de Anexos:**
   - Identifica tipo de anexo (áudio, imagem, PDF, documento, vídeo)
   - Indexa anexos para processamento futuro
   - Marca anexos na transcrição com IDs

4. **Análise com IA:**
   - Trunca transcrição se necessário (limite de tokens)
   - Mantém início e fim (mais relevante)
   - Gera análise estruturada em JSON

5. **Busca Semântica:**
   - Embeddings permitem busca por similaridade
   - Encontra conversas similares
   - Agrupa conversas por padrões

6. **Tratamento de Erros:**
   - Captura todos os erros automaticamente
   - Registra contexto completo
   - Facilita debugging

## 🚀 Execução

- **Trigger:** Webhook (evento-driven) ou manual
- **Frequência:** Sempre que há atualização no GHL
- **Duração estimada:** 10-30 segundos (dependendo do tamanho da conversa)
- **Tratamento de erros:** Error Handler dedicado

## 📝 Notas Técnicas

- O workflow processa conversas de qualquer tamanho
- Implementa paginação robusta para grandes volumes
- Usa truncamento inteligente para respeitar limites de API
- Embeddings são gerados apenas da análise (não da transcrição completa)
- Error Handler é workflow separado para isolamento
- Credenciais são armazenadas no n8n (não expostas)

## 🎯 Casos de Uso

1. **Análise de Qualidade:**
   - Identificar conversas de alta/baixa qualidade
   - Comparar performance de SDRs
   - Encontrar padrões de sucesso

2. **Busca Semântica:**
   - Encontrar conversas similares
   - Agrupar leads por perfil
   - Identificar tendências

3. **Monitoramento:**
   - Acompanhar taxa de agendamento
   - Analisar sentimento dos leads
   - Identificar problemas recorrentes

4. **Otimização:**
   - Melhorar scripts de venda
   - Treinar SDRs com exemplos
   - Ajustar estratégias baseado em dados
