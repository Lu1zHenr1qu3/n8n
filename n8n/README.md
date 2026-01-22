# Portfólio de Automações n8n

[![n8n](https://img.shields.io/badge/n8n-FF6D5B?style=flat&logo=n8n&logoColor=white)](https://n8n.io)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

Este repositório contém um conjunto de automações profissionais desenvolvidas com n8n, demonstrando integrações avançadas entre diferentes sistemas e APIs para resolver problemas reais de negócio. Todos os workflows incluem documentação completa, tratamento de erros e estão prontos para uso em produção.

## 🚀 Sobre

Portfólio de automações desenvolvidas com n8n demonstrando integrações avançadas entre sistemas. Inclui: sincronização de dados médicos (Biologix-Airtable), segmentação automática de leads por pipeline (GHL-Google Sheets), análise de conversas com IA (Claude/OpenAI + Supabase) e transcrição automática de áudio (AssemblyAI).

## 📋 Índice

1. [Sincronização Biologix-Airtable](#1-sincronização-biologix-airtable)
2. [Segmentação de Leads por Pipeline](#2-segmentação-de-leads-por-pipeline)
3. [Salvar Conversas e Análise Integrada IA](#3-salvar-conversas-e-análise-integrada-ia)
4. [Transcritor de Curso](#4-transcritor-de-curso)

---

## 1. Sincronização Biologix-Airtable

**Descrição:** Automação que sincroniza dados de exames de ronco e sono da API Biologix com o Airtable, atualizando informações de pacientes e exames automaticamente.

**Frequência:** Execução diária às 8h

**Ferramentas Externas:**
- API Biologix Sleep (autenticação e consulta de exames)
- Airtable (armazenamento de dados de pacientes e exames)

**Documentação Completa:** [Sincronizacao-Biologix-Airtable/README.md](./Sincronizacao-Biologix-Airtable/README.md)

---

## 2. Segmentação de Leads por Pipeline

**Descrição:** Workflow que recebe webhooks do GoHighLevel (GHL) e organiza os dados de leads em diferentes planilhas do Google Sheets baseado no estágio do pipeline.

**Ferramentas Externas:**
- GoHighLevel (webhook de atualização de oportunidades)
- Google Sheets (armazenamento organizado por estágio)

**Documentação Completa:** [Segmentacao-Leads-Pipeline/README.md](./Segmentacao-Leads-Pipeline/README.md)

---

## 3. Salvar Conversas e Análise Integrada IA

**Descrição:** Sistema avançado que exporta conversas do GoHighLevel, processa com IA (Claude/OpenAI) para análise de sentimento e qualidade, e armazena no Supabase com embeddings para busca semântica.

**Ferramentas Externas:**
- GoHighLevel API (exportação de conversas e mensagens)
- Anthropic Claude API (análise de conversas com IA)
- OpenAI API (geração de embeddings)
- Supabase (banco de dados PostgreSQL com busca vetorial)

**Documentação Completa:** [Salvar conversas e analise integrada IA/README.md](./Salvar%20conversas%20e%20analise%20integrada%20IA/README.md)

---

## 4. Transcritor de Curso

**Descrição:** Automação que processa arquivos de áudio do Google Drive, transcreve usando AssemblyAI e salva as transcrições de volta no Google Drive.

**Ferramentas Externas:**
- Google Drive (armazenamento de arquivos de áudio e transcrições)
- AssemblyAI (transcrição de áudio para texto)

**Documentação Completa:** [Transcritor de curso/README.md](./Transcritor%20de%20curso/README.md)

---

## 🔒 Segurança

**IMPORTANTE:** Este repositório foi sanitizado para remover:
- Credenciais de API
- Chaves de autenticação
- Tokens pessoais
- Dados sensíveis

Todas as credenciais devem ser configuradas no ambiente n8n antes de executar os workflows.

**📖 Guia de Configuração:** Consulte [CONFIGURACAO_CREDENCIAIS.md](./CONFIGURACAO_CREDENCIAIS.md) para instruções detalhadas sobre como configurar as credenciais necessárias.

---

## 📝 Notas

- Todos os workflows foram desenvolvidos para uso em produção
- As automações incluem tratamento de erros e validações
- Os workflows são modulares e podem ser adaptados para diferentes necessidades
