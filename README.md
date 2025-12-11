# Target Automation

Serviço Node.js que automatiza consultas e exportações de ofertas aprovadas do Adobe Target, com foco nas atividades de "Trava Telas". O projeto expõe endpoints REST sob `/target` que autenticam via `client_credentials`, montam as chamadas corretas para a API do Target e retornam payloads prontos para uso ou atualização.

## Requisitos

- Node.js 18+
- Conta e credenciais válidas do Adobe Target

## Configuração

1. Copie o arquivo `.env.example` para `.env` e preencha com suas credenciais:

   ```bash
   cp .env.example .env
   ```

2. Preencha as variáveis obrigatórias no `.env`:

   | Variável               | Descrição |
   | ---------------------- | --------- |
   | `TENANT_ID`            | ID do tenant usado nos endpoints da Adobe (ex.: `abc123`). |
   | `CLIENT_ID`            | Client ID da integração configurada no Adobe. |
   | `CLIENT_SECRET`        | Client secret da integração. |
   | `API_KEY`              | API key (geralmente igual ao `CLIENT_ID`). |
   | `API_SCOPE`            | Escopos autorizados para o Adobe Target (ex.: `target.client`). |
   | `API_PORT` *(opcional)*| Porta HTTP do serviço (padrão `3001`). |
   | `TRAVA_TELAS_IDENTIFIER` *(opcional)* | Texto usado para localizar a atividade de Trava Telas (padrão `[APP] travaTelasHomeProd`). |

3. Instale as dependências:

   ```bash
   npm install
   ```

4. Inicie o servidor:

   ```bash
   npm start
   ```

O serviço ficará disponível em `http://localhost:<API_PORT>` (padrão `3001`).

## Endpoints principais (prefixo `/target`)

### `GET /automation/trava-telas`
Retorna as ofertas aprovadas da atividade "Trava Telas".

**Query params**

- `activityId` *(opcional)*: força a consulta para uma atividade específica (por ID numérico).

**Resposta**

```json
{
  "totalOffers": 3,
  "offers": [
    {
      "activityId": 12345,
      "activityName": "[APP] travaTelasHomeProd",
      "offerId": 67890,
      "offerType": "json",
      "offer": { /* payload completo da oferta */ },
      "experienceName": "Experience A",
      "audience": "ALL VISITORS"
    }
  ]
}
```

### `GET /automation/trava-telas/export`
Exporta o mesmo conteúdo do endpoint anterior como um arquivo JSON para download. Aceita o mesmo `activityId` opcional.

### `PUT /automation/trava-telas/update-date`
Atualiza (ou simula a atualização) das datas das ofertas aprovadas.

**Query params**

- `activityId` *(opcional)*: direciona a execução para uma atividade específica em modo de teste.

**Corpo da requisição (opcional)**

- `offers`: array com as ofertas que devem ser atualizadas. Se omitido, o serviço busca as ofertas aprovadas automaticamente.

**Resposta**

```json
{
  "message": "Offers updated successfully",
  "processedOffers": [ /* ofertas tocadas */ ]
}
```

> Para enviar payloads com corpo, certifique-se de configurar o cliente HTTP para usar `Content-Type: application/json`.

## Desenvolvimento

- `npm run dev` — Inicia o servidor com `nodemon` para recarga automática.
- `npm run lint` — Executa o ESLint usando a configuração Airbnb Base.

## Estrutura do projeto

```
src/
├── config/
│   └── environment.js    # Carrega e valida variáveis de ambiente
├── index.js              # Ponto de entrada do servidor Express (prefixo /target)
├── routes/
│   └── target.js         # Rotas da automação de Trava Telas
└── services/
    └── adobeTargetService.js # Cliente para autenticação e consumo da API do Target
```

## Observações

- O arquivo `.env` está no `.gitignore` para evitar exposição de segredos.
- Tokens de acesso são armazenados em cache em memória e renovados automaticamente antes de expirar.
- As chamadas ao Adobe Target são autenticadas com `client_credentials` e levam os headers exigidos pela API oficial.
