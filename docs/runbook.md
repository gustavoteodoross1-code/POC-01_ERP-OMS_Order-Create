# Runbook — ERP_to_OMS_OrderCreate_IFLOW

## Objetivo
Criar pedidos no OMS (mock) via HTTPS, aceitando apenas **POST** e propagando **X-Correlation-Id**.

## Endpoints
- Runtime HTTPS Sender: `/http/orders` (apenas **POST**)

## Parâmetros Externalizados (principais)
- `EXT_OMS_ADDRESS`: URL do HTTP Receiver (OMS/Mock).
- `EXT_HTTP_REQ_HEADERS`: `Content-Type=application/json;X-Correlation-Id=${header.X-Correlation-Id}`

## Operação — Como validar rápido
1. Faça **POST** (curl/Postman) → espere **201**.
2. Faça **GET** → espere **405**.
3. No **Monitor → Message Processing**, filtre pelo iFlow, abra MPL e valide headers/logs.

## Troubleshooting (atalhos)
- 401 → credenciais do runtime.
- 403 → roles/autorizações no tenant.
- 405 → método incorreto (usar POST).
- 500 → abrir MPL e ver step que falhou (Receiver/Content Modifier).

## Deploy / Mudança de destino
Trocar `EXT_OMS_ADDRESS` no adapter HTTP Receiver e **Deploy**.
