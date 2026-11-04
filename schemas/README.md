# Schemas de mensajes (protocolo v2)

Un JSON Schema (draft 2020-12) por tipo de mensaje. Cada uno extiende `envelope.schema.json`
con `allOf` y fija su `type`, quién lo emite y la forma de `data`.

El envelope exige `idpk`, `msgId`, `type`, `timestamp` y `data`. Los mensajes nuestros llevan
`cityId` y los de la central llevan `sender: "central"`. Hay una regla que JSON Schema no puede
expresar y que valida `connector`: `idpk` debe ser distinto de `msgId`.

Los schemas no cierran `additionalProperties`, porque la central puede agregar campos (por
ejemplo, `data.market` en el status-statement). Lo que validan es lo que nosotros leemos o
enviamos.

| Tipo | Dirección | Archivo |
|---|---|---|
| envelope base | ambas | `envelope.schema.json` |
| `ack` | ambas | `ack.schema.json` |
| `nack` | ambas | `nack.schema.json` |
| `error` | central → ciudad | `error.schema.json` |
| `status-statement` | central → ciudad | `status-statement.schema.json` |
| `transfer` | ambas | `transfer.schema.json` |
| `demand-statement` | central → ciudad | `demand-statement.schema.json` |
| `distance-table` | central → ciudad | `distance-table.schema.json` |
| `negotiation-proposal` | ciudad → central | `negotiation-proposal.schema.json` |
| `give` / `take` | central → ciudad | `give-take.schema.json` |
| `negotiation-report` | ciudad → central | `negotiation-report.schema.json` |
| `request` | ciudad → central | `request.schema.json` |

Se verificaron con los ejemplos del enunciado E1 v1.1, con los mensajes que arman `connector`
y `master`, y con mensajes reales de producción.

## API HTTP

`../openapi/openapi.yaml` se genera desde la app con `backend/scripts/exportar_openapi.py`; no
se edita a mano.
