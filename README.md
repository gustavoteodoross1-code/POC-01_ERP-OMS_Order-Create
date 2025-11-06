# POC-01_ERP-OMS_Order-Create

POC de integração no **SAP Integration Suite** para criar pedidos no OMS.  
O iFlow **aceita somente POST** no endpoint `/http/orders`, valida método e propaga **X-Correlation-Id** (gera um novo se ausente).  
Receiver atual: **Postman Mock** (para simulação).

---

## 🔗 Artefatos e Release

- Export do iFlow (para transporte):  
  `iflow/export/ERP_to_OMS_OrderCreate_IFLOW_default.zip`
- Export do iFlow (para documentação):  
  `iflow/export/ERP_to_OMS_OrderCreate_IFLOW_merged.zip`
- Releases do projeto: **aba _Releases_ do GitHub** (ex.: `v0.1.0 – iFlow export`)

> Dica: baixe o ZIP **default** para transportar entre tenants/ambientes.  
> O ZIP **merged** é útil para consulta/inspeção local.

---

## 🧭 Fluxo (alto nível)
Client (curl/Postman) ──HTTPS Sender──▶ iFlow

Content Modifier (headers/body)

SetCorrelationId (gera se faltar)

Router (GET → 405 | POST → segue)

Request-Reply ──HTTP Receiver (Mock OMS)
◀── Response (201)



---

## ✅ Comportamento esperado

- `POST /http/orders` → **201 Created**  
  Retorna JSON simples: `{"status":"created","message":"string"}`
- `GET /http/orders` → **405 Method Not Allowed**  
  Retorna JSON: `{ "error": "Method Not Allowed", "allowed": ["POST"] }`
- Header **X-Correlation-Id**:
  - Se enviado: é preservado e propagado.
  - Se ausente: o iFlow gera (`iflow-<timestamp>`).

---

## 🧪 Como testar (curl)

> Substitua `<HOST>` e credenciais. Ex.:  
> `https://<seu-subdominio>.cfapps.us10-001.hana.ondemand.com`

### POST deve retornar 201
CID="sanity-$(date +%s)"
curl -i -X POST "<HOST>/http/orders" \
  -u "<usuario>:<senha>" \
  -H "Content-Type: application/json" \
  -H "X-Correlation-Id: $CID" \
  --data '{"orderId":"SO-1001","total":100}'

### POST deve retornar 201
curl -i -X GET "<HOST>/http/orders" \
  -u "<usuario>:<senha>"

### Enviando payload de arquivo (exemplo)
curl -i -X POST "<HOST>/http/orders" \
  -u "<usuario>:<senha>" \
  -H "Content-Type: application/json" \
  --data @mock/mock_oms_order.json

## 🧰 Como testar (Postman)

1. Importe tests/postman_collection.json.

2. Crie um Environment com variáveis:

host = https://<seu-subdominio>.cfapps.us10-001.hana.ondemand.com
user, pass
cid (opcional)

3. Rode POST Create Sales Order in OMS.
Esperado: Status 201 com corpo {"status":"created","message":"string"}.

## 🔧 Parâmetros/Config (iFlow)

- EXT_OMS_ADDRESS: URL do HTTP Receiver (OMS/Mock).
- EXT_HTTP_REQ_HEADERS: cabeçalhos do Request (ex.: Content-Type=application/json;X-Correlation-Id=${header.X-Correlation-Id}).
- Timeout/Retry/HTTP Error Codes por externalização no adapter HTTP.

Para mudar o destino (Mock → OMS real), altere EXT_OMS_ADDRESS no adapter HTTP do Receiver e faça Deploy.

## 🖥️ Monitoramento (Integration Suite)

1. Monitor → Message Processing.
2. Filtre por ERP_to_OMS_OrderCreate_IFLOW.
3. Abra o Message e veja:

- Properties: sap_messageprocessinglogid, sap_mplcorrelationid.
- Logs: sequência dos steps, headers propagados e tempos.

## 🗂️ Estrutura do repositório
api/          # OpenAPI / contracts (quando aplicável)
docs/         # Runbook, operação e troubleshooting
iflow/
  └─ export/  # ZIPs exportados do iFlow (default/merged)
mock/         # Exemplo de payload (mock_oms_order.json)
tests/        # Postman collection / configs de teste
README.md

## 🆘 Troubleshooting

401 Unauthorized
Credenciais inválidas. Valide usuário/senha do runtime.

403 Forbidden
Falta de role/política no tenant/serviço. Ajuste Role Collection.

405 Method Not Allowed
Você chamou GET; o endpoint aceita apenas POST.

500 Internal Server Error
Abra o MPL (Message Processing Log) e identifique o step (ex.: Receiver/Content Modifier) e o motivo.





