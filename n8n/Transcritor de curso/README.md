# Transcritor de Curso - Automação de Transcrição de Áudio

## 📋 Visão Geral

Esta automação processa arquivos de áudio armazenados no Google Drive, transcreve automaticamente usando AssemblyAI e salva as transcrições de volta no Google Drive em formato texto. Ideal para transcrever cursos, palestras, podcasts ou qualquer conteúdo em áudio.

## 🎯 Objetivo

- Monitorar pasta do Google Drive por novos arquivos de áudio
- Transcrever automaticamente usando AssemblyAI
- Salvar transcrições formatadas no Google Drive
- Processar múltiplos arquivos em sequência

## 📊 Fluxo do Workflow

![Workflow Visual](./Captura%20de%20tela%202026-01-22%20122746.png)

### Fluxo Detalhado:

1. **Manual Trigger** → Inicia o processo (pode ser agendado)
2. **Buscando arquivos dentro da pasta** → Lista arquivos na pasta do Google Drive
3. **Baixando os arquivos** → Faz download dos arquivos de áudio
4. **Fazendo Upload do arquivo** → Envia arquivo para AssemblyAI
5. **Fazendo a transcricao** → Inicia processo de transcrição
6. **Verificando se a transcricao do arquivo esta correta** → Verifica status da transcrição
7. **If** → Verifica se transcrição está completa
8. **Wait** → Aguarda 3 minutos se ainda estiver processando
9. **Convert to File** → Converte texto para arquivo
10. **Google Drive** → Salva transcrição no Google Drive

## 🔧 Nodes Utilizados

### 1. When clicking 'Execute workflow'
- **Tipo:** `n8n-nodes-base.manualTrigger`
- **Função:** Inicia o workflow manualmente
- **Nota:** Pode ser substituído por Schedule Trigger para automação

### 2. Buscando arquivos dentro da pasta
- **Tipo:** `n8n-nodes-base.googleDrive`
- **Função:** Lista arquivos na pasta especificada
- **Operação:** List files
- **Filtros:**
  - `driveId`: "My Drive"
  - `folderId`: ID da pasta "Audios para serem transcritos"
  - `whatToSearch`: "files"
- **Limite:** 40 arquivos por execução
- **Retorna:** Lista de arquivos com metadados (id, name, mimeType, etc.)

### 3. Baixando os arquivos
- **Tipo:** `n8n-nodes-base.googleDrive`
- **Função:** Faz download dos arquivos de áudio
- **Operação:** Download
- **Parâmetro:** `fileId` (do node anterior)
- **Retorna:** Binary data do arquivo

### 4. Fazendo Upload do arquivo
- **Tipo:** `n8n-nodes-base.httpRequest`
- **Função:** Envia arquivo para AssemblyAI
- **Método:** POST
- **Endpoint:** `https://api.assemblyai.com/v2/upload`
- **Headers:**
  - `Transfer-Encoding`: "chunked"
  - `Authorization`: API Key do AssemblyAI
- **Body:** Binary data do arquivo
- **Content-Type:** `binaryData`
- **Retorna:** `upload_url` (URL temporária do arquivo)

### 5. Fazendo a transcricao
- **Tipo:** `n8n-nodes-base.httpRequest`
- **Função:** Inicia processo de transcrição
- **Método:** POST
- **Endpoint:** `https://api.assemblyai.com/v2/transcript`
- **Headers:** Authorization (API Key)
- **Body (JSON):**
  ```json
  {
    "audio_url": "{{ upload_url }}",
    "language_code": "pt",
    "punctuate": true,
    "format_text": true
  }
  ```
- **Retorna:** `id` (ID da transcrição), `status` (queued/processing/completed)

### 6. Verificando se a transcricao do arquivo esta correta
- **Tipo:** `n8n-nodes-base.httpRequest`
- **Função:** Verifica status da transcrição
- **Método:** GET
- **Endpoint:** `https://api.assemblyai.com/v2/transcript/{{ id }}`
- **Headers:** Authorization (API Key)
- **Retorna:** 
  - `status`: "queued", "processing", ou "completed"
  - `text`: Texto transcrito (quando completo)

### 7. If
- **Tipo:** `n8n-nodes-base.if`
- **Função:** Verifica se transcrição está completa
- **Condição:** `status === "completed"`
- **True:** Continua para salvar transcrição
- **False:** Aguarda e verifica novamente

### 8. Wait
- **Tipo:** `n8n-nodes-base.wait`
- **Função:** Aguarda antes de verificar novamente
- **Duração:** 3 minutos
- **Motivo:** Dar tempo para AssemblyAI processar

### 9. Convert to File
- **Tipo:** `n8n-nodes-base.convertToFile`
- **Função:** Converte texto para arquivo
- **Operação:** `toText`
- **Source Property:** `text` (da transcrição)
- **Retorna:** Binary data do arquivo de texto

### 10. Google Drive
- **Tipo:** `n8n-nodes-base.googleDrive`
- **Função:** Salva transcrição no Google Drive
- **Operação:** Upload
- **Nome do arquivo:** Gerado dinamicamente
  - Formato: `Transcricao_{nome_base}_{numero}.txt`
  - Exemplo: `Transcricao_Curso_Ladeira_Automacoes_inteligentes_N8N_1_txt`
- **Pasta destino:** "N8N" (ID da pasta)
- **Drive:** "My Drive"

## 🔌 Integrações Externas

### Google Drive
- **Tipo:** Armazenamento em nuvem
- **Autenticação:** OAuth2
- **Operações utilizadas:**
  - **List files:** Lista arquivos em pasta
  - **Download:** Baixa arquivos de áudio
  - **Upload:** Salva transcrições
- **Pastas utilizadas:**
  - **Origem:** "Audios para serem transcritos"
  - **Destino:** "N8N"

### AssemblyAI
- **Tipo:** API de transcrição de áudio
- **Autenticação:** API Key (Header Authorization)
- **Endpoints utilizados:**
  - `/v2/upload` - Upload de arquivo
  - `/v2/transcript` - Iniciar transcrição
  - `/v2/transcript/{id}` - Verificar status e obter resultado
- **Configurações:**
  - `language_code`: "pt" (Português)
  - `punctuate`: true (adiciona pontuação)
  - `format_text`: true (formata texto)
- **Limites:**
  - Suporta vários formatos de áudio (MP3, WAV, M4A, etc.)
  - Processamento assíncrono
  - Status: queued → processing → completed

## 📈 Dados Processados

### Entrada
- **Arquivos de áudio:** MP3, WAV, M4A, OGG, etc.
- **Origem:** Google Drive (pasta específica)
- **Metadados:** Nome do arquivo, ID, tipo MIME

### Processamento
- **Upload:** Arquivo enviado para AssemblyAI
- **Transcrição:** Áudio convertido para texto
- **Formatação:** Texto pontuado e formatado

### Saída
- **Arquivo de texto:** `.txt`
- **Nome:** `Transcricao_{nome_base}_{numero}.txt`
- **Destino:** Google Drive (pasta "N8N")
- **Conteúdo:** Texto transcrito completo

## ⚙️ Lógica de Negócio

1. **Processamento Assíncrono:**
   - AssemblyAI processa áudio de forma assíncrona
   - Workflow verifica status periodicamente
   - Aguarda 3 minutos entre verificações

2. **Nomenclatura de Arquivos:**
   - Extrai número do arquivo original (ex: "1.mp3" → "1")
   - Gera nome baseado no curso (configurável)
   - Formato final: `Transcricao_{base}_{numero}_txt`

3. **Tratamento de Erros:**
   - Se transcrição não estiver pronta, aguarda
   - Loop de verificação até completar
   - Timeout implícito (múltiplas tentativas)

4. **Processamento em Lote:**
   - Processa até 40 arquivos por execução
   - Cada arquivo é processado sequencialmente
   - Pode ser executado múltiplas vezes para processar todos

## 🚀 Execução

- **Trigger:** Manual (pode ser agendado)
- **Frequência:** Conforme necessidade
- **Duração estimada:** 
  - Upload: 10-30 segundos
  - Transcrição: 1-5 minutos (dependendo do tamanho do áudio)
  - Total: 2-10 minutos por arquivo
- **Limite:** 40 arquivos por execução

## 📝 Notas Técnicas

- O workflow processa arquivos sequencialmente (não em paralelo)
- AssemblyAI suporta vários formatos de áudio
- O nome do arquivo de saída é gerado dinamicamente
- A pasta de destino é fixa ("N8N")
- Credenciais são armazenadas no n8n (não expostas)
- O workflow está configurado como `active: false` (desativado)

## 🎯 Casos de Uso

1. **Transcrição de Cursos:**
   - Converter aulas em áudio para texto
   - Criar material de estudo
   - Facilitar busca em conteúdo

2. **Transcrição de Reuniões:**
   - Converter gravações de reuniões
   - Criar atas automáticas
   - Facilitar revisão

3. **Transcrição de Podcasts:**
   - Gerar transcrições para SEO
   - Criar conteúdo escrito
   - Facilitar acessibilidade

4. **Processamento em Lote:**
   - Transcrever múltiplos arquivos
   - Automatizar workflow de produção
   - Economizar tempo manual

## 🔄 Melhorias Possíveis

1. **Agendamento Automático:**
   - Substituir Manual Trigger por Schedule Trigger
   - Executar periodicamente (ex: diariamente)

2. **Processamento Paralelo:**
   - Processar múltiplos arquivos simultaneamente
   - Reduzir tempo total de processamento

3. **Notificações:**
   - Enviar notificação quando transcrição completar
   - Alertar em caso de erro

4. **Validação de Formato:**
   - Verificar se arquivo é de áudio antes de processar
   - Pular arquivos já transcritos

5. **Mover Arquivos:**
   - Mover arquivos processados para pasta "Processados"
   - Evitar reprocessamento
