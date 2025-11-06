mandei ver no seu README e… dá pra ficar bem melhor ✅

Hoje ele está todo “espremido” (quase sem quebras de linha), alguns títulos/trechos se misturam e os exemplos de `curl` perderam o host/variáveis — isso pode confundir quem for rodar.

Abaixo vai uma versão **pronta-pra-uso** (só copiar e colar no `README.md`). Eu já corrigi formatação, quebras, seções, e deixei os exemplos com variáveis claras (`RUNTIME`, `USER`, `PASS`, `CID`). Mantive tudo que você documentou (exports, comportamento, testes, monitoramento, troubleshooting), mas mais limpo.

---

# POC-01 · ERP → OMS · Order Create

POC de integração no **SAP Integration Suite** para criar pedidos no OMS.
O iFlow **aceita somente `POST`** no endpoint `/http/orders`, valida o método e propaga **`X-Correlation-Id`** (gera um novo se ausente).
Receiver atual: **Postman Mock** (simulação).

---

## Artefatos & Release

* Export do iFlow (para **transporte**):
  `iflow/export/ERP_to_OMS_OrderCreate_IFLOW_default.zip`
* Export do iFlow (para **documentação/inspeção**):
  `iflow/export/ERP_to_OMS_OrderCreate_IFLOW_merged.zip`
* Releases do projeto: **aba *Releases* no GitHub** (ex.: `v0.1.0 – iFlow export`)

> **Dica:** use o ZIP **default** para transportar entre tenants/ambientes.
> O ZIP **merged** é útil para consulta local (parâmetros mesclados).

---

## Fluxo (alto nível)

```
Client (curl/Postman)
   └─HTTPS Sender → Content Modifier (headers/body)
                     → SetCorrelationId (gera se faltar)
                     → Router (GET → 405 | POST → segue)
                     → Request-Reply → HTTP Receiver (Mock OMS)
                                      ← Response (201)
```

---

## ✅ Comportamento esperado

* `POST /http/orders` → **201 Created**
  Corpo (exemplo): `{"status":"created","message":"string"}`
* `GET /http/orders` → **405 Method Not Allowed**
  Corpo: `{ "error": "Method Not Allowed", "allowed": ["POST"] }`
* Header **`X-Correlation-Id`**

  * Se **enviado**: preservado e propagado.
  * Se **ausente**: o iFlow gera (`iflow-<timestamp>`).

---

## Como testar (curl)

Defina as variáveis conforme seu runtime e credenciais do **tenant de runtime**:

```bash
export RUNTIME="https://<subaccount>.<domain>/http/orders"    # ex.: https://9115d201trial.it-cpitrial06-rt.cfapps.us10-001.hana.ondemand.com/http/orders
export USER="seu.usuario@example.com"
export PASS="sua-senha"
export CID="sanity-$(date +%s)"
```

### 1) POST deve retornar 201

```bash
curl -i -X POST "$RUNTIME" \
  -u "$USER:$PASS" \
  -H "Content-Type: application/json" \
  -H "X-Correlation-Id: $CID" \
  --data '{"orderId":"SO-1001","total":100}'
```

### 2) GET deve retornar 405

```bash
curl -i -X GET "$RUNTIME" \
  -u "$USER:$PASS"
```

### 3) Enviando payload de arquivo

```bash
# exemplo de payload em mock/mock_oms_order.json
curl -i -X POST "$RUNTIME" \
  -u "$USER:$PASS" \
  -H "Content-Type: application/json" \
  --data @mock/mock_oms_order.json
```

---

## Como testar (Postman)

1. Importe `tests/postman_collection.json`.
2. Crie um Environment com variáveis:

   * `host` = `https://<subaccount>.cfapps.us10-001.hana.ondemand.com`
   * `user`, `pass`
   * `cid` (opcional)
3. Rode **POST Create Sales Order in OMS**.
   Esperado: **201** com corpo `{"status":"created","message":"string"}`.

---

## Parâmetros / Config (iFlow)

* `EXT_OMS_ADDRESS`: URL do **HTTP Receiver** (OMS/Mock).
* `EXT_HTTP_REQ_HEADERS`: cabeçalhos do Request
  (ex.: `Content-Type=application/json;X-Correlation-Id=${header.X-Correlation-Id}`).
* Timeout / Retry / HTTP Error Codes: **externalizados** no adapter HTTP.

> Para mudar o destino (Mock → OMS real), altere `EXT_OMS_ADDRESS` no adapter **HTTP Receiver** e faça **Deploy**.

---

## 🔎 Monitoramento (Integration Suite)

1. **Monitor** → **Message Processing**.
2. Filtre por `ERP_to_OMS_OrderCreate_IFLOW`.
3. Abra a mensagem e confira:

   * **Properties**: `sap_messageprocessinglogid`, `sap_mplcorrelationid`.
   * **Logs**: sequência dos steps, headers propagados e tempos.

---

## Estrutura do repositório

```
api/     # OpenAPI / contratos (quando aplicável)
docs/    # Runbook, operação e troubleshooting
iflow/
  └─ export/   # ZIPs exportados do iFlow (default/merged)
mock/    # Exemplo de payload (mock_oms_order.json)
tests/   # Postman collection / configs de teste
README.md
```

---

## Troubleshooting

| Sintoma                       | Causa comum                                    | Ação sugerida                              |
| ----------------------------- | ---------------------------------------------- | ------------------------------------------ |
| **401 Unauthorized**          | Credenciais inválidas                          | Valide usuário/senha do runtime            |
| **403 Forbidden**             | Falta de role/política no tenant/serviço       | Ajuste Role Collection / permissões        |
| **405 Method Not Allowed**    | Chamou `GET`; endpoint aceita apenas `POST`    | Use `POST`                                 |
| **500 Internal Server Error** | Falha em step (ex.: Receiver/Content Modifier) | Abra o **MPL** e identifique o step/motivo |

---

## Release Notes (exemplo)

* `v0.1.0 – iFlow export`

  * Export do iFlow `ERP_to_OMS_OrderCreate_IFLOW`
  * ZIP para transporte: `iflow/export/ERP_to_OMS_OrderCreate_IFLOW_default.zip`
  * ZIP para documentação: `iflow/export/ERP_to_OMS_OrderCreate_IFLOW_merged.zip`
  * Validações: `GET → 405`, `POST → 201` com `X-Correlation-Id`
