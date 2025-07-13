# Documentação Técnica da API de Validação e Análise de Arquivos AFD

## Visão Geral

Esta API permite validar e extrair informações de arquivos AFD (Arquivo-Fonte de Dados) nos formatos Portaria 1510 e Portaria 671. A API realiza validações de integridade, estrutura e conteúdo, além de extrair informações essenciais sobre a empresa e funcionários.

## Autenticação e Acesso

### Obtenção de Token

Para utilizar a API, você precisa obter um token de acesso. Siga os passos abaixo:

1. Acesse sua conta em [https://api.zerocert.com.br/admin/login](https://api.zerocert.com.br/admin/login)
2. Após o login, acesse o painel administrativo
3. Na seção "Meu Perfil", você encontrará seu token de API
4. Utilize este token em todas as suas requisições

### Uso do Token

Todas as requisições à API devem incluir um token de autenticação no cabeçalho HTTP:

```
Authorization: Bearer TOKEN_HERE
```

### Restrições de IP

Para maior segurança, você pode configurar restrições de IP para seu token:

1. Acesse o painel administrativo em [https://api.zerocert.com.br/admin](https://api.zerocert.com.br/admin)
2. Navegue até "Meu Perfil" > "Restrições de IP"
3. Adicione os endereços IP que terão permissão para usar seu token

**Nota:** A funcionalidade de restrição de IP está disponível apenas em planos específicos.

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
  "version": "1.3.0",
  "supportedFormats": ["Portaria 1510", "Portaria 671"],
  "features": [
    "Validação de estrutura e integridade",
    "Detecção automática de layout",
    "Suporte a arquivos P7S para verificação de assinatura digital (REP P 671)"
  ]
}
```

### 2. Processamento Assíncrono

**Requisição:**
```
POST /api/v1/afd/process
Content-Type: multipart/form-data

[arquivo binário com nome "afdFile"]
[arquivo binário com nome "p7sFile"] (opcional, para verificação de assinatura digital)
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
    "estatisticas": { ... },
    "validacao": { ... }
  }
}
```

### 3. Processamento Síncrono

**Requisição:**
```
POST /api/v1/afd/process-sync
Content-Type: multipart/form-data

[arquivo binário com nome "afdFile"]
[arquivo binário com nome "p7sFile"] (opcional, para verificação de assinatura digital)
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
    "estatisticas": { ... },
    "validacao": { ... }
  }
}
```

### 4. Validação

**Requisição:**
```
POST /api/v1/afd/validate
Content-Type: multipart/form-data

[arquivo binário com nome "afdFile"]
[arquivo binário com nome "p7sFile"] (opcional, para verificação de assinatura digital)
```

**Resposta:**
```json
{
  "status": "complete",
  "result": {
    "valido": true,
    "integridadeArquivo": true,
    "validadePeriodo": true,
    "contagemRegistros": true,
    "erros": [],
    "avisos": [],
    "detalhesRegistros": {
    "tipo1": 1,
    "tipo2": 1,
    "tipo3": 10,
    "tipo4": 1,
    "tipo5": 14,
      "tipo6": 10,
      "tipo7": 0,
      "trailer": 1,
      "assinatura": 1
    },
    "crcErrors": 0,
    "algoritmoDetectado": "MODBUS",
    "multiplosAlgoritmosCompativeis": false
  }
}
```

## Estrutura da Resposta

A resposta completa de processamento contém as seguintes seções:

### Exemplo de Resposta Completa

```json
{
  "status": "complete",
  "result": {
    "algoritmoValidacao": "CRC-16 + SHA-256",
    "layout": "Portaria 671",
    "empresa": {
      "razaoSocial": "EMPRESA EXEMPLO LTDA",
      "cnpjCpf": "12345678901234",
      "tipoIdentificador": "CNPJ",
      "cei": null,
      "cno": null,
      "periodoInicial": "2023-01-01",
      "periodoFinal": "2023-01-31"
    },
    "funcionarios": [
      {
        "id": "12345678901",
        "nome": "FUNCIONARIO EXEMPLO",
        "tipo": "CPF",
        "status": "Ativo",
        "operacao": "Inclusão"
      }
    ],
    "estatisticas": {
      "totalFuncionarios": 1,
      "funcionariosAtivos": 1,
      "funcionariosExcluidos": 0,
      "totalMarcacoes": 10,
      "periodoCobertura": {
        "inicio": "2023-01-01",
        "fim": "2023-01-31"
      }
    },
    "validacao": {
      "valido": false,
      "integridadeArquivo": false,
      "validadePeriodo": true,
      "contagemRegistros": false,
      "erros": [
        "Contador type6 inconsistente: esperado 10, encontrado 9"
      ],
      "avisos": [],
      "detalhesRegistros": {
        "tipo1": 1,
        "tipo2": 1,
        "tipo3": 10,
        "tipo4": 1,
        "tipo5": 1,
        "tipo6": 9,
        "tipo7": 0,
        "trailer": 1,
        "assinatura": 0
      },
      "crcErrors": 0,
      "algoritmoDetectado": "MODBUS",
      "multiplosAlgoritmosCompativeis": false,
       "assinaturaDigital": {
         "presente": true,
         "verificada": true,
         "detalhes": {
           "mensagem": "Arquivo P7S encontrado e linha de assinatura digital presente no AFD",
           "tipoArquivo": "REP P 671",
           "tamanhoP7S": 2048
         }
       }
    }
  }
}
```

## Estrutura da Resposta
### Informações Gerais

- `algoritmoValidacao`: Algoritmo utilizado para validação ("CRC-16" ou "CRC-16 + SHA-256")
- `layout`: Formato do arquivo ("Portaria 1510" ou "Portaria 671")
- `fileHash`: Hash SHA-256 do arquivo enviado

### Exemplo Completo de Resposta

```json
{
  "status": "complete",
  "result": {
    "algoritmoValidacao": "CRC-16 + SHA-256",
    "layout": "Portaria 671",
    "empresa": {
      "razaoSocial": "EMPRESA LTDA",
      "cnpjCpf": "12345678000109",
      "tipoIdentificador": "CNPJ",
      "cei": null,
      "cno": "00000000000000",
      "periodoInicial": "2000-01-01",
      "periodoFinal": "2025-07-09"
    },
    "funcionarios": [
      {
        "id": "12345678909",
        "nome": "NOME FUNCIONARIO",
        "tipo": "CPF",
        "status": "Ativo",
        "operacao": "Inclusão"
      },
      {
        "id": "12345678909",
        "nome": "NOME FUNCIONARIO",
        "tipo": "CPF",
        "status": "Ativo",
        "operacao": "Inclusão"
      }
    ],
    "estatisticas": {
      "totalFuncionarios": 7,
      "funcionariosAtivos": 7,
      "funcionariosExcluidos": 0,
      "totalMarcacoes": 0,
      "periodoCobertura": {
        "inicio": "2000-01-01",
        "fim": "2025-07-09"
      }
    },
    "validacao": {
      "valido": false,
      "integridadeArquivo": false,
      "validadePeriodo": true,
      "contagemRegistros": true,
      "erros": [
        "Linha 2: Falha na verificação CRC-16. Esperado: 37E2, Encontrado: 7196",
        "Linha 9: Falha na verificação CRC-16. Esperado: E54F, Encontrado: CF64"
      ],
      "avisos": [
        "Linha 2: Resultados de CRC-16 para diferentes algoritmos: {\"CCITT-FFFF\":\"37E2\",\"KERMIT\":\"0001\",\"XMODEM\":\"9128\",\"MODBUS\":\"B51E\",\"CCITT-Default\":\"5EF6\"}",
        "Linha 9: Resultados de CRC-16 para diferentes algoritmos: {\"CCITT-FFFF\":\"E54F\",\"KERMIT\":\"6D76\",\"XMODEM\":\"FBE6\",\"MODBUS\":\"E06B\",\"CCITT-Default\":\"FF59\"}"
      ],
      "detalhesRegistros": {
        "tipo1": 1,
        "tipo2": 1,
        "tipo3": 0,
        "tipo4": 0,
        "tipo5": 7,
        "tipo6": 0,
        "tipo7": 0,
        "trailer": 1
      },
      "crcErrors": 2,
      "algoritmoDetectado": "KERMIT",
      "multiplosAlgoritmosCompativeis": false
    }
  },
  "fileHash": "d131f56245567bed18458775fed493304d27f8b21e7412289bc8265dfe9628e8"
}
```

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

### Estatísticas

- `estatisticas.totalFuncionarios`: Número total de funcionários encontrados
- `estatisticas.funcionariosAtivos`: Número de funcionários ativos
- `estatisticas.funcionariosExcluidos`: Número de funcionários excluídos
- `estatisticas.totalMarcacoes`: Número total de marcações de ponto
- `estatisticas.periodoCobertura`: Período coberto pelo arquivo (início e fim)

### Validação

- `validacao.valido`: Indica se o arquivo é válido segundo todas as regras
- `validacao.integridadeArquivo`: Indica se o arquivo passou na validação de integridade (CRC-16/SHA-256)
- `validacao.validadePeriodo`: Indica se o período inicial é anterior ao período final
- `validacao.contagemRegistros`: Indica se a contagem de registros confere com o trailer
- `validacao.erros`: Lista completa de todos os erros de validação (integridade do arquivo, contagem de registros, etc.)
- `validacao.avisos`: Lista de avisos (não impeditivos)
- `validacao.detalhesRegistros`: Contagem detalhada de registros por tipo
- `validacao.crcErrors`: Número de registros com erros na validação CRC-16
- `validacao.algoritmoDetectado`: Algoritmo CRC-16 detectado no arquivo (ex: "MODBUS", "KERMIT", "CCITT-FFFF", "XMODEM", "CCITT-Default")
- `validacao.multiplosAlgoritmosCompativeis`: Indica se múltiplos algoritmos CRC-16 são compatíveis com o arquivo

## Códigos de Erro HTTP

- `200 OK`: Requisição processada com sucesso
- `202 Accepted`: Requisição aceita para processamento assíncrono
- `400 Bad Request`: Parâmetros inválidos ou arquivo não fornecido
- `401 Unauthorized`: Token de autenticação não fornecido
- `403 Forbidden`: Token de autenticação inválido
- `404 Not Found`: Recurso não encontrado (ex: jobId inválido)

## Erros Comuns

### Erros HTTP

- "Nenhum arquivo foi enviado. Use a chave 'afdFile'." - Verifique se o arquivo está sendo enviado com o nome correto
- "Conteúdo do arquivo inválido ou não fornecido." - Verifique se o arquivo está em formato texto válido
- "Arquivo AFD vazio ou inválido (mínimo de cabeçalho e trailer)." - O arquivo deve conter pelo menos um registro de cabeçalho e um trailer

### Erros de Validação (encontrados em `validacao.erros`)

- "Período inicial deve ser anterior ao período final" - Verifique as datas no cabeçalho do arquivo
- "X registros com falha na verificação CRC-16" - Verifique a integridade do arquivo
- "Contador typeX inconsistente: esperado Y, encontrado Z" - Verifique se o trailer contém a contagem correta de registros

## Exemplos de Integração

### cURL

```bash
# Processamento síncrono
curl -X POST \
  -H "Authorization: Bearer TOKEN_HERE" \
  -F "afdFile=@caminho/para/seu/arquivo.afd" \
  https://api.zerocert.com.br/api/v1/afd/process-sync

# Processamento assíncrono
curl -X POST \
  -H "Authorization: Bearer TOKEN_HERE" \
  -F "afdFile=@caminho/para/seu/arquivo.afd" \
  https://api.zerocert.com.br/api/v1/afd/process

# Verificação de status
curl -X GET \
  -H "Authorization: Bearer TOKEN_HERE" \
  https://api.zerocert.com.br/api/v1/afd/status/550e8400-e29b-41d4-a716-446655440000
```

### PHP

```php
<?php
// Processamento síncrono
$token = "SEU_TOKEN_AQUI";
$arquivo = new CURLFile('/caminho/para/seu/arquivo.afd', 'text/plain', 'afdFile');

$ch = curl_init();

curl_setopt($ch, CURLOPT_URL, "https://api.zerocert.com.br/api/v1/afd/process-sync");
curl_setopt($ch, CURLOPT_POST, 1);
curl_setopt($ch, CURLOPT_POSTFIELDS, ["afdFile" => $arquivo]);
curl_setopt($ch, CURLOPT_HTTPHEADER, ["Authorization: Bearer $token"]);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);

$response = curl_exec($ch);
$httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);

curl_close($ch);

if ($httpCode == 200) {
    $resultado = json_decode($response, true);
    // Processar o resultado
    print_r($resultado);
} else {
    echo "Erro na requisição: $httpCode";
    echo $response;
}
?>
```

### Python

```python
import requests

# Processamento síncrono
def processar_arquivo_afd(caminho_arquivo, token):
    url = "https://api.zerocert.com.br/api/v1/afd/process-sync"
    headers = {"Authorization": f"Bearer {token}"}
    
    with open(caminho_arquivo, 'rb') as arquivo:
        files = {"afdFile": arquivo}
        response = requests.post(url, headers=headers, files=files)
    
    if response.status_code == 200:
        return response.json()
    else:
        print(f"Erro: {response.status_code}")
        print(response.text)
        return None

# Processamento assíncrono
def iniciar_processamento_afd(caminho_arquivo, token):
    url = "https://api.zerocert.com.br/api/v1/afd/process"
    headers = {"Authorization": f"Bearer {token}"}
    
    with open(caminho_arquivo, 'rb') as arquivo:
        files = {"afdFile": arquivo}
        response = requests.post(url, headers=headers, files=files)
    
    if response.status_code == 202:
        return response.json()["jobId"]
    else:
        print(f"Erro: {response.status_code}")
        print(response.text)
        return None

def verificar_status_processamento(job_id, token):
    url = f"https://api.zerocert.com.br/api/v1/afd/status/{job_id}"
    headers = {"Authorization": f"Bearer {token}"}
    
    response = requests.get(url, headers=headers)
    
    if response.status_code == 200:
        return response.json()
    else:
        print(f"Erro: {response.status_code}")
        print(response.text)
        return None

# Exemplo de uso
token = "SEU_TOKEN_AQUI"
caminho_arquivo = "/caminho/para/seu/arquivo.afd"

# Para processamento síncrono
resultado = processar_arquivo_afd(caminho_arquivo, token)
print(resultado)

# Para processamento assíncrono
job_id = iniciar_processamento_afd(caminho_arquivo, token)
if job_id:
    status = verificar_status_processamento(job_id, token)
    print(status)
```

### JavaScript (Node.js)

```javascript
const fs = require('fs');
const axios = require('axios');
const FormData = require('form-data');

// Configuração
const token = 'SEU_TOKEN_AQUI';
const apiUrl = 'https://api.zerocert.com.br/api/v1/afd';

// Processamento síncrono
async function processarArquivoAFD(caminhoArquivo) {
  try {
    const formData = new FormData();
    formData.append('afdFile', fs.createReadStream(caminhoArquivo));
    
    const response = await axios.post(`${apiUrl}/process-sync`, formData, {
      headers: {
        ...formData.getHeaders(),
        'Authorization': `Bearer ${token}`
      }
    });
    
    return response.data;
  } catch (error) {
    console.error('Erro ao processar arquivo:', error.response ? error.response.data : error.message);
    throw error;
  }
}

// Processamento assíncrono
async function iniciarProcessamentoAFD(caminhoArquivo) {
  try {
    const formData = new FormData();
    formData.append('afdFile', fs.createReadStream(caminhoArquivo));
    
    const response = await axios.post(`${apiUrl}/process`, formData, {
      headers: {
        ...formData.getHeaders(),
        'Authorization': `Bearer ${token}`
      }
    });
    
    return response.data.jobId;
  } catch (error) {
    console.error('Erro ao iniciar processamento:', error.response ? error.response.data : error.message);
    throw error;
  }
}

async function verificarStatusProcessamento(jobId) {
  try {
    const response = await axios.get(`${apiUrl}/status/${jobId}`, {
      headers: {
        'Authorization': `Bearer ${token}`
      }
    });
    
    return response.data;
  } catch (error) {
    console.error('Erro ao verificar status:', error.response ? error.response.data : error.message);
    throw error;
  }
}

// Exemplo de uso
(async () => {
  try {
    // Processamento síncrono
    const resultado = await processarArquivoAFD('/caminho/para/seu/arquivo.afd');
    console.log('Resultado:', resultado);
    
    // Processamento assíncrono
    const jobId = await iniciarProcessamentoAFD('/caminho/para/seu/arquivo.afd');
    console.log('Job ID:', jobId);
    
    // Verificar status (em um caso real, você pode implementar polling)
    const status = await verificarStatusProcessamento(jobId);
    console.log('Status:', status);
  } catch (error) {
    console.error('Falha na operação:', error);
  }
})();
```

### Java

```java
import java.io.File;
import java.io.IOException;
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.nio.file.Files;
import java.nio.file.Path;

public class ZeroCertApiClient {
    private static final String API_URL = "https://api.zerocert.com.br/api/v1/afd";
    private final String token;
    private final HttpClient httpClient;

    public ZeroCertApiClient(String token) {
        this.token = token;
        this.httpClient = HttpClient.newBuilder().build();
    }

    // Processamento síncrono
    public String processarArquivoAFD(String caminhoArquivo) throws IOException, InterruptedException {
        String boundary = "----" + System.currentTimeMillis();
        File arquivo = new File(caminhoArquivo);
        
        // Preparar o corpo multipart/form-data
        byte[] fileContent = Files.readAllBytes(arquivo.toPath());
        String requestBody = "--" + boundary + "\r\n" +
                "Content-Disposition: form-data; name=\"afdFile\"; filename=\"" + arquivo.getName() + "\"\r\n" +
                "Content-Type: text/plain\r\n\r\n" +
                new String(fileContent) + "\r\n" +
                "--" + boundary + "--";

        HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create(API_URL + "/process-sync"))
                .header("Content-Type", "multipart/form-data; boundary=" + boundary)
                .header("Authorization", "Bearer " + token)
                .POST(HttpRequest.BodyPublishers.ofString(requestBody))
                .build();

        HttpResponse<String> response = httpClient.send(request, HttpResponse.BodyHandlers.ofString());

        if (response.statusCode() == 200) {
            return response.body();
        } else {
            throw new IOException("Erro na requisição: " + response.statusCode() + " - " + response.body());
        }
    }

    // Processamento assíncrono
    public String iniciarProcessamentoAFD(String caminhoArquivo) throws IOException, InterruptedException {
        // Implementação similar ao método acima, mas usando o endpoint /process
        // ...
        return "jobId"; // Retorna o jobId da resposta
    }

    public String verificarStatusProcessamento(String jobId) throws IOException, InterruptedException {
        HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create(API_URL + "/status/" + jobId))
                .header("Authorization", "Bearer " + token)
                .GET()
                .build();

        HttpResponse<String> response = httpClient.send(request, HttpResponse.BodyHandlers.ofString());

        if (response.statusCode() == 200) {
            return response.body();
        } else {
            throw new IOException("Erro na requisição: " + response.statusCode() + " - " + response.body());
        }
    }

    public static void main(String[] args) {
        try {
            ZeroCertApiClient client = new ZeroCertApiClient("SEU_TOKEN_AQUI");
            String resultado = client.processarArquivoAFD("/caminho/para/seu/arquivo.afd");
            System.out.println("Resultado: " + resultado);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

## Tratamento de Erros e Boas Práticas

### Tratamento de Erros

Ao integrar com a API ZeroCert, é importante implementar um tratamento de erros adequado:

1. **Verifique sempre o código HTTP de resposta**:
   - 2xx: Sucesso na operação
   - 4xx: Erro do cliente (verifique os parâmetros enviados)
   - 5xx: Erro do servidor (tente novamente mais tarde)

2. **Implemente retry com backoff exponencial**:
   - Para erros 5xx ou timeouts, implemente tentativas com intervalos crescentes
   - Exemplo: 1ª tentativa após 1s, 2ª após 2s, 3ª após 4s, etc.
   - Limite o número máximo de tentativas (ex: 5)

3. **Valide localmente quando possível**:
   - Verifique o formato do arquivo AFD antes de enviar
   - Confirme se o token está presente no cabeçalho
   - Verifique se o arquivo não excede o limite do seu plano

### Boas Práticas

1. **Processamento Assíncrono vs. Síncrono**:
   - Use processamento assíncrono para arquivos grandes
   - Use processamento síncrono apenas para arquivos pequenos ou quando precisar do resultado imediatamente

2. **Monitoramento de Uso**:
   - Monitore regularmente seu uso da API através do painel administrativo
   - Configure alertas para quando estiver próximo do limite do seu plano

3. **Segurança**:
   - Nunca exponha seu token de API no código do lado do cliente
   - Utilize as restrições de IP para aumentar a segurança
   - Armazene o token em variáveis de ambiente ou cofres de segredos

4. **Cache**:
   - Implemente cache local para resultados de validação frequentes
   - Evite reprocessar o mesmo arquivo repetidamente

5. **Logs**:
   - Mantenha logs detalhados das chamadas à API e respostas
   - Inclua IDs de job para rastreabilidade

## Planos e Limites de Uso

### Planos Disponíveis

| Plano | Requisições/mês | Requisições/minuto | Restrições de IP | Suporte |
|-------|----------------|---------------------------|-----------------|----------|
| Starter | 1.000 | 60 | Sim | Email |
| Profissional | 10.000 | 120 | Sim | Email, Chat |
| Avançado | 50.000 | 180 | Sim | Email, Chat, Telefone |
| Enterprise | Sob consulta | Sob consulta | Sim | Dedicado |

### Monitoramento de Uso

Você pode monitorar o uso da sua API através do painel administrativo:

1. Acesse [https://api.zerocert.com.br/admin](https://api.zerocert.com.br/admin)
2. Navegue até "Uso da API"
3. Visualize estatísticas detalhadas de uso, incluindo:
   - Total de requisições por período
   - Distribuição por endpoint
   - Histórico de erros
   - Exportação de relatórios em CSV

## Limitações Técnicas

- A API suporta apenas arquivos em formato texto (não binários)
- Os resultados de processamento assíncrono são mantidos por 7 dias

## Formatos de Arquivo AFD Suportados

A API ZeroCert suporta os seguintes formatos de arquivo AFD:

### Portaria 1510/2009

- **Estrutura**: Registros de tamanho fixo com 9 dígitos de NSR + 1 dígito de tipo + dados específicos
- **Tipos de Registro**:
  - Tipo 1: Cabeçalho (deve ser o primeiro registro)
  - Tipo 2: Registro de inclusão ou alteração da identificação da empresa
  - Tipo 3: Registro de marcação de ponto
  - Tipo 4: Registro de ajuste do relógio de tempo real
  - Tipo 5: Registro de inclusão ou alteração ou exclusão de empregado
  - Tipo 6: Eventos sensíveis
  - Tipo 9: Trailer (deve ser o último registro)
- **Validação**: CRC-16 para cada registro
- **Assinatura**: Opcional, após o trailer

### Portaria 671/2021

- **Estrutura**: Similar à Portaria 1510, com campos adicionais e formato de data ISO
- **Tipos de Registro**:
  - Tipo 1: Cabeçalho (302 caracteres)
  - Tipo 2: Registro de empregador (200 caracteres)
  - Tipo 3: Registro de marcação de ponto (200 caracteres)
  - Tipo 4: Registro de ajuste do relógio (200 caracteres)
  - Tipo 5: Registro de empregado (200 caracteres)
  - Tipo 6: Registro de eventos sensíveis (200 caracteres)
  - Tipo 7: Registro de marcação pré-assinalada (137 caracteres)
  - Tipo 9: Trailer
- **Validação**: CRC-16 + SHA-256
- **Formato de Data**: ISO 8601 (YYYY-MM-DDThh:mm:ss)
- **Assinatura Digital**: Pode conter ou não assinatura digital com certificado e-CNPJ
  - Quando assinado digitalmente, é necessário enviar também o arquivo .p7s correspondente
  - A API verifica a presença da linha indicativa de assinatura digital ("ASSINATURA_DIGITAL_EM_ARQUIVO_P7S") no AFD
  - A verificação completa da assinatura digital é realizada quando o arquivo .p7s é fornecido junto com o AFD

### Detecção Automática

A API detecta automaticamente o formato do arquivo com base em:
- Presença de datas no formato ISO 8601 (indicativo de Portaria 671)
- Estrutura e tamanho dos registros

### Verificação de Assinatura Digital

A API suporta a verificação de assinatura digital para arquivos AFD REP P 671:

1. **Envio de Arquivos**:
   - Envie o arquivo AFD com o campo `afdFile`
   - Envie o arquivo P7S correspondente com o campo `p7sFile`

2. **Processo de Verificação**:
   - A API verifica se o arquivo AFD é do tipo REP P 671
   - Verifica a presença da linha indicativa de assinatura digital no AFD ("ASSINATURA_DIGITAL_EM_ARQUIVO_P7S")
   - Verifica a existência do arquivo P7S correspondente
   - **Nota**: A verificação criptográfica completa da assinatura será implementada em uma versão futura

3. **Resultado da Verificação**:
   - O resultado é retornado no campo `assinaturaDigital` da resposta
   - Inclui informações sobre a presença e validade da assinatura
   - Fornece detalhes adicionais sobre a verificação
### Algoritmos de Validação

- A API detecta e utiliza automaticamente o algoritmo de validação apropriado
- Suporta múltiplos algoritmos de CRC-16 para compatibilidade com diferentes fabricantes

## Suporte e Contato

### Canais de Suporte

O suporte varia de acordo com o plano contratado:

- **Email**: Disponível para todos os planos em suporte@zerocert.com.br
- **Chat**: Disponível para planos Profissional e superiores
- **Telefone**: Disponível para planos Empresarial e superiores
- **Suporte Dedicado**: Exclusivo para planos Personalizados

### Documentação e Recursos

- **Base de Conhecimento**: [https://zerocert.com.br/kb](https://zerocert.com.br/kb)
- **FAQ**: [https://zerocert.com.br/faq](https://zerocert.com.br/faq)
- **Changelog**: [https://zerocert.com.br/changelog](https://zerocert.com.br/changelog)

### Reportando Problemas

Ao reportar um problema, inclua as seguintes informações:

1. ID do token utilizado (não envie o token completo)
2. Data e hora da ocorrência
3. ID do job (para processamento assíncrono)
4. Código de erro recebido
5. Descrição detalhada do problema
6. Trecho do arquivo AFD que gerou o erro (se aplicável)

### Feedback e Sugestões

Enviamos atualizações regulares com base no feedback dos usuários. Envie suas sugestões para feedback@zerocert.com.br
