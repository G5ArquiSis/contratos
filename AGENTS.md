# Contexto compartido — EnergyShark E1 (G5ArquiSis)

Archivo de contexto a nivel de organización (RDOC04), referenciado desde `backend` y `frontend`.
Resume lo que cualquier integrante (o herramienta) necesita saber antes de tocar el código.

## El sistema

Nodo de energía de una ciudad para la E1 de IIC2173. Se comunica con la central por RabbitMQ
(cola y routing key `city.{CODE}`), mantiene su propio ledger (budget y balance energético),
negocia energía con la central y reporta su posición al cierre de cada ciclo.

- Ciudad asignada: <!-- completar -->
- Ciclo: 2 horas; ventana de negociación en los últimos 20 minutos; `negotiation-report` en los
  últimos 5 minutos de la ventana.

## Repositorios

| Repo | Qué contiene |
|---|---|
| `backend` | `master` (API FastAPI + Postgres) y `connector` (consumidor del broker), heredados de la E0 |
| `frontend` | SPA (React + Vite), servida desde S3 + CloudFront |
| `contratos` | Este repo: schemas de mensajes, OpenAPI, este archivo |

## Reglas que no se rompen

- Todo mensaje publicado lleva la propiedad AMQP `user_id = city.{CODE}` y `cityId` en el cuerpo.
- `msgId` es siempre un UUID nuevo; una respuesta referencia el mensaje original en `data.target`.
- `idpk` nunca es igual a `msgId`. Un reintento conserva el `idpk`; una corrección usa uno nuevo.
- Un `idpk` ya aplicado nunca se vuelve a aplicar al ledger, pero se registra como duplicado.
- No se hace ACK de ACKs, NACK de NACKs ni error de errores. Reintentos siempre con tope y backoff.
- Nunca subir `.env` ni `.pem` a ningún repo.
- Los requisitos no variables de la E0 siguen vigentes: /history paginado y filtrable, contenedores master y connector con HEALTHCHECK, Nginx en el host.

## Decisiones de arquitectura

Ver los ADRs en `backend/docs/adr/` (AD1 topología del consumo, AD2 persistencia del ledger,
AD3 timeouts de negociación).
