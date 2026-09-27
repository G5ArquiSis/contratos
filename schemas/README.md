# Schemas de mensajes (protocolo v2)

Un archivo JSON Schema por tipo de mensaje. Todos comparten el envelope base:
`idpk`, `msgId`, `type`, `timestamp` (obligatorios), `cityId` (mensajes nuestros) o `sender`
(mensajes de la central), y el contenido en `data`.

| Tipo | Dirección | Archivo |
|---|---|---|
| envelope base | ambas | `envelope.schema.json` |
| `ack` | ambas | |
| `nack` | ambas | |
| `error` | central → ciudad | |
| `status-statement` | central → ciudad | |
| `transfer` | ambas | |
| `demand-statement` | central → ciudad | |
| `distance-table` | central → ciudad | |
| `negotiation-proposal` | ciudad → central | |
| `give` / `take` | central → ciudad | |
| `negotiation-report` | ciudad → central | |
| `request` | ciudad → central | |
