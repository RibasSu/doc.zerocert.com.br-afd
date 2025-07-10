# Documentação Técnica da API de Validação e Análise de Arquivos AFD

## Visão Geral

Esta API permite validar e extrair informações de arquivos AFD (Arquivo-Fonte de Dados) nos formatos Portaria 1510 e Portaria 671. A API realiza validações de integridade, estrutura e conteúdo, além de extrair informações essenciais sobre a empresa e funcionários.

## Autenticação

Todas as requisições à API devem incluir um token de autenticação no cabeçalho HTTP:

```
Authorization: Bearer TOKEN_HERE
```

## Endpoints Disponíveis

### 1. Informações da API

**Requisição:**
```
GET /api/v1/afd
```

**Resposta:**
```json
{
  "message": "API unificada para análise e validação de arquivos AFD",
  "version": "1.0.0",
  "supportedFormats": ["Portaria 1510", "Portaria 671"]
}
```

### 2. Processamento Assíncrono

**Requisição:**
```
POST /api/v1/afd/process
Content-Type: multipart/form-data

[arquivo binário com nome "afdFile"]
```

**Resposta Inicial (202 Accepted):**
```json
{
  "jobId": "550e8400-e29b-41d4-a716-446655440000"
}
```

**Verificação de Status:**
```
GET /api/v1/afd/status/550e8400-e29b-41d4-a716-446655440000
```

**Resposta de Status (em processamento):**
```json
{
  "status": "processing",
  "startedAt": "2023-06-24T10:30:00.000Z"
}
```

**Resposta de Status (concluído):**
```json
{
  "status": "complete",
  "result": {
    "algoritmoValidacao": "CRC-16 + SHA-256",
    "layout": "Portaria 671",
    "empresa": { ... },
    "funcionarios": [ ... ],
    "validacao": { ... },
    "estatisticas": { ... },
    "validacaoDetalhada": { ... }
  }
}
```

### 3. Processamento Síncrono

**Requisição:**
```
POST /api/v1/afd/process-sync
Content-Type: multipart/form-data

[arquivo binário com nome "afdFile"]
```

**Resposta:**
```json
{
  "status": "complete",
  "result": {
    "algoritmoValidacao": "CRC-16 + SHA-256",
    "layout": "Portaria 671",
    "empresa": { ... },
    "funcionarios": [ ... ],
    "validacao": { ... },
    "estatisticas": { ... },
    "validacaoDetalhada": { ... }
  }
}
```

### 4. Validação Detalhada

**Requisição:**
```
POST /api/v1/afd/validate
Content-Type: multipart/form-data

[arquivo binário com nome "afdFile"]
```

**Resposta:**
```json
{
  "status": "complete",
  "result": {
    "isValid": true,
    "errors": [],
    "warnings": [],
    "recordCounts": {
      "type1": 1,
      "type2": 1,
      "type3": 10,
      "type4": 1,
      "type5": 14,
      "type6": 10,
      "type7": 0,
      "type9": 1
    }
  }
}
```

## Estrutura da Resposta

A resposta completa de processamento contém as seguintes seções:

### Informações Gerais

- `algoritmoValidacao`: Algoritmo utilizado para validação ("CRC-16" ou "CRC-16 + SHA-256")
- `layout`: Formato do arquivo ("Portaria 1510" ou "Portaria 671")

### Dados da Empresa

- `empresa.razaoSocial`: Nome da empresa
- `empresa.cnpjCpf`: CNPJ ou CPF do empregador
- `empresa.tipoIdentificador`: Tipo de identificador ("CNPJ" ou "CPF")
- `empresa.cei`: Número CEI (quando aplicável)
- `empresa.cno`: Número CNO/CAEPF (quando aplicável)
- `empresa.periodoInicial`: Data inicial do período (formato YYYY-MM-DD)
- `empresa.periodoFinal`: Data final do período (formato YYYY-MM-DD)

### Lista de Funcionários

Array de objetos com as seguintes propriedades:

- `id`: Identificador do funcionário (PIS ou CPF)
- `nome`: Nome do funcionário
- `tipo`: Tipo de identificador ("PIS" ou "CPF")
- `status`: Status atual ("Ativo" ou "Excluído")
- `operacao`: Última operação ("Inclusão" ou "Exclusão")

### Validação Básica

- `validacao.integridadeArquivo`: Indica se o arquivo passou na validação de integridade (CRC-16/SHA-256)
- `validacao.validadePeriodo`: Indica se o período inicial é anterior ao período final
- `validacao.contagemRegistros`: Indica se a contagem de registros confere com o trailer
- `validacao.erros`: Lista de mensagens de erro (vazia se não houver erros)

### Estatísticas

- `estatisticas.totalFuncionarios`: Número total de funcionários encontrados
- `estatisticas.funcionariosAtivos`: Número de funcionários ativos
- `estatisticas.funcionariosExcluidos`: Número de funcionários excluídos
- `estatisticas.totalMarcacoes`: Número total de marcações de ponto
- `estatisticas.periodoCobertura`: Período coberto pelo arquivo (início e fim)

### Validação Detalhada

- `validacaoDetalhada.isValid`: Indica se o arquivo é válido segundo todas as regras
- `validacaoDetalhada.errors`: Lista de erros detalhados
- `validacaoDetalhada.warnings`: Lista de avisos (não impeditivos)
- `validacaoDetalhada.recordCounts`: Contagem de registros por tipo

## Códigos de Erro HTTP

- `200 OK`: Requisição processada com sucesso
- `202 Accepted`: Requisição aceita para processamento assíncrono
- `400 Bad Request`: Parâmetros inválidos ou arquivo não fornecido
- `401 Unauthorized`: Token de autenticação não fornecido
- `403 Forbidden`: Token de autenticação inválido
- `404 Not Found`: Recurso não encontrado (ex: jobId inválido)

## Erros Comuns

- "Nenhum arquivo foi enviado. Use a chave 'afdFile'." - Verifique se o arquivo está sendo enviado com o nome correto
- "Conteúdo do arquivo inválido ou não fornecido." - Verifique se o arquivo está em formato texto válido
- "Arquivo AFD vazio ou inválido (mínimo de cabeçalho e trailer)." - O arquivo deve conter pelo menos um registro de cabeçalho e um trailer
- "Período inicial deve ser anterior ao período final" - Verifique as datas no cabeçalho do arquivo
- "X registros com falha na verificação CRC-16" - Verifique a integridade do arquivo
- "Contagem de registros tipo X não confere" - Verifique se o trailer contém a contagem correta de registros

## Exemplos de Uso

### Exemplo com cURL

```bash
# Processamento síncrono
curl -X POST \
  -H "Authorization: Bearer TOKEN_HERE" \
  -F "afdFile=@caminho/para/seu/arquivo.afd" \
  https://api.zerotls.com.br/api/v1/afd/process-sync

# Processamento assíncrono
curl -X POST \
  -H "Authorization: Bearer TOKEN_HERE" \
  -F "afdFile=@caminho/para/seu/arquivo.afd" \
  https://api.zerotls.com.br/api/v1/afd/process

# Verificação de status
curl -X GET \
  -H "Authorization: Bearer TOKEN_HERE" \
  https://api.zerotls.com.br/api/v1/afd/status/550e8400-e29b-41d4-a716-446655440000
```

## Limitações

- O tamanho máximo do arquivo para processamento síncrono é de 10MB
- Para arquivos maiores, utilize o processamento assíncrono
- A API suporta apenas arquivos em formato texto (não binários)
- Os resultados de processamento assíncrono são mantidos por 24 horas
