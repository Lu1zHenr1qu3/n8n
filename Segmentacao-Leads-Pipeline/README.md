# Segmentação de Leads por Pipeline - Organização Automática de Oportunidades

## 📋 Visão Geral

Esta automação recebe webhooks do GoHighLevel (GHL) quando há atualizações em oportunidades/leads e organiza os dados em diferentes planilhas do Google Sheets baseado no estágio do pipeline. Permite segmentação automática de leads por status (Ganho, Venda Perdida, Desqualificado, etc.).

## 🎯 Objetivo

- Receber dados de leads do GoHighLevel via webhook
- Organizar leads em planilhas separadas por estágio do pipeline
- Manter histórico completo de interações e dados de marketing
- Facilitar análise e acompanhamento de conversão

## 📊 Fluxo do Workflow

![Workflow Visual](./Captura%20de%20tela%202026-01-22%20122639.png)

### Fluxo Detalhado:

1. **Webhook** → Recebe dados do GHL quando há atualização de oportunidade
2. **Code** → Processa e formata dados de marketing (UTM, campanhas)
3. **Edit Fields** → Organiza todos os campos em estrutura padronizada
4. **Switch** → Roteia para planilha correta baseado no estágio
5. **Google Sheets** → Salva/atualiza dados na planilha correspondente

## 🔧 Nodes Utilizados

### 1. Webhook
- **Tipo:** `n8n-nodes-base.webhook`
- **Função:** Recebe POST do GoHighLevel
- **Método:** POST
- **Path:** `/09a11316-b9a4-490e-b074-deb52911efd0`
- **Dados recebidos:**
  - Dados do contato (nome, email, telefone)
  - Dados do pipeline (nome, estágio, status)
  - Dados de marketing (UTM, campanhas, referrer)
  - Dados customizados (tratamentos, datas, valores)

### 2. Code
- **Tipo:** `n8n-nodes-base.code`
- **Função:** Processa e formata dados de marketing e mensagens
- **Processamento:**
  - Extrai dados "Last" (última interação)
  - Formata mensagens (remove quebras de linha, aspas problemáticas)
  - Normaliza campos de marketing
- **Campos processados:**
  - Last Session Source, Last Campaign, Last UTM Source/Medium/Content
  - Last Referrer, Last Campaign ID, Last FB/Google ClickId
  - Last UTM Campaign/Keyword/Term, Last Ad Name/ID
  - Primeiro_Mensagem, Last_Mensagem (formatadas)

### 3. Edit Fields
- **Tipo:** `n8n-nodes-base.set`
- **Função:** Organiza todos os campos em estrutura final
- **Campos mapeados:**
  - **Dados do Contato:**
    - contact_id, Primeiro_Nome, Sobrenome, Nome_Completo
    - Telefone, Email, Sexo, Data_de_Nascimento
  - **Dados de Marketing (Primeiro):**
    - Primeiro_Session_Source, Primeiro_Campaign
    - Primeiro_UTM_Source/Medium/Content/Campaign
    - Primeiro_Referrer, Primeiro_Campaign_ID
    - Primeiro_FB_ClickId, Primeiro_Google_ClickId
    - Primeiro_Ctwa_Clid, Primeiro_Ad_Name/ID
  - **Dados de Marketing (Último):**
    - Last_Session_Source, Last_Campaign
    - Last_UTM_Source/Medium/Content/Campaign
    - Last_Referrer, Last_Campaign_ID
    - Last_FB_ClickId, Last_Google_ClickId
    - Last_Ctwa_Clid, Last_Ad_Name/ID
  - **Dados de Pipeline:**
    - Pipeline, Estagio_Pipeline, Status, Valor
  - **Dados Clínicos:**
    - Primeiro_Tratamento_de_Interesse, Tratamentos
    - Status_do_orcamento
  - **Datas:**
    - Data_de_Preenchimento (timestamp atual)
    - Data_de_criacao, Data_do_agendamento
    - Data_da_1_consulta, Data_1_comparecimento
  - **Localização:**
    - Cidade, Estado, Pais, Codigo_Postal
  - **Mensagens:**
    - Primeiro_Mensagem, Last_Mensagem

### 4. Switch
- **Tipo:** `n8n-nodes-base.switch`
- **Função:** Roteia para planilha correta baseado no estágio
- **Regras de roteamento:**
  - **Totalmente Desclassificado** → Planilha correspondente
  - **Totalmente Desqualificado** → Planilha correspondente
  - **Desqualificado** → Planilha correspondente
  - **SQL** → Planilha correspondente
  - **Venda Perdida** → Planilha "Venda Perdida"
  - **Ganho** → Planilha correspondente
- **Campo de decisão:** `pipleline_stage` (do webhook)

### 5. Google Sheets (AppendOrUpdate)
- **Tipo:** `n8n-nodes-base.googleSheets`
- **Função:** Salva ou atualiza registro na planilha
- **Operação:** `appendOrUpdate`
- **Documento:** "Planilas de Contas"
- **Aba:** Varia conforme roteamento do Switch
- **Matching:** Por `contact_id` (evita duplicatas)
- **Campos salvos:** Todos os campos do Edit Fields

## 🔌 Integrações Externas

### GoHighLevel (GHL)
- **Tipo:** CRM/Plataforma de Marketing
- **Integração:** Webhook (POST)
- **Evento:** Atualização de Oportunidade/Lead
- **Dados enviados:**
  - Informações completas do contato
  - Dados do pipeline e estágio
  - Histórico de marketing (UTM, campanhas)
  - Dados customizados (tratamentos, datas)

### Google Sheets
- **Tipo:** Planilhas online
- **Autenticação:** OAuth2
- **Documento:** "Planilas de Contas"
- **Abas utilizadas:**
  - Venda Perdida
  - Ganho
  - Desqualificado
  - Totalmente Desqualificado
  - Totalmente Desclassificado
  - SQL
- **Operação:** AppendOrUpdate (atualiza se existe, cria se não existe)

## 📈 Dados Processados

### Estrutura de Dados

**Dados do Contato:**
- Identificação: contact_id, nome completo, email, telefone
- Demográficos: sexo, data de nascimento, localização

**Dados de Marketing:**
- **First Touch:** Primeira interação (UTM, campanha, referrer)
- **Last Touch:** Última interação (UTM, campanha, referrer)
- **Tracking:** Click IDs (Facebook, Google), Campaign IDs

**Dados de Pipeline:**
- Pipeline name, estágio atual, status
- Valor da oportunidade
- Datas importantes (criação, agendamento, comparecimento)

**Dados Clínicos:**
- Tratamentos de interesse
- Status do orçamento
- Tratamentos realizados

**Mensagens:**
- Primeira mensagem enviada
- Última mensagem enviada

## ⚙️ Lógica de Negócio

1. **Segmentação Automática:**
   - Leads são automaticamente organizados por estágio
   - Cada estágio tem sua própria planilha

2. **Prevenção de Duplicatas:**
   - Usa `contact_id` como chave única
   - AppendOrUpdate atualiza registros existentes

3. **Formatação de Dados:**
   - Mensagens são limpas (sem quebras de linha problemáticas)
   - Dados de marketing são normalizados
   - Timestamps são formatados corretamente

4. **Rastreamento Completo:**
   - Mantém histórico de First Touch e Last Touch
   - Preserva dados de campanhas e UTM
   - Registra todas as interações

## 🚀 Execução

- **Trigger:** Webhook (evento-driven)
- **Frequência:** Sempre que há atualização no GHL
- **Duração estimada:** 1-3 segundos por execução
- **Tratamento de erros:** Logs no n8n

## 📝 Notas Técnicas

- O webhook é público e deve ser configurado no GHL
- Credenciais do Google Sheets são OAuth2
- O campo `pipleline_stage` tem typo do GHL (mantido para compatibilidade)
- Mensagens são formatadas para evitar problemas em CSV/Excel
- Data de preenchimento é gerada no momento da execução

## 🎯 Casos de Uso

1. **Análise de Conversão:**
   - Comparar leads por estágio
   - Identificar padrões de conversão

2. **Rastreamento de Marketing:**
   - Verificar eficácia de campanhas
   - Analisar First Touch vs Last Touch

3. **Gestão de Pipeline:**
   - Acompanhar leads por estágio
   - Identificar gargalos no funil

4. **Relatórios:**
   - Exportar dados para análise
   - Criar dashboards no Google Sheets
