# Contexto compartido — EnergyShark E1 (G5ArquiSis)

Archivo de contexto a nivel de organización (RDOC04), referenciado desde `backend` y `frontend`.
Resume lo que cualquier integrante (o herramienta) necesita saber antes de tocar el código.

## El sistema

Nodo de energía de una ciudad para la E1 de IIC2173. Se comunica con la central por RabbitMQ
(cola y routing key `city.{CODE}`), mantiene su propio ledger (budget y balance energético),
negocia energía con la central y reporta su posición al cierre de cada ciclo.

- Ciudad asignada: Talca, código `TAL`. Donde este documento dice `{CODE}`, en nuestro nodo es `TAL` (cola `city.TAL`).
- Ciclo: 2 horas; ventana de negociación en los últimos 20 minutos; `negotiation-report` en los
  últimos 5 minutos de la ventana.

## Repositorios

| Repo | Qué contiene |
|---|---|
| `backend` | `master` (API FastAPI + Postgres) y `connector` (consumidor del broker), heredados de la E0 |
| `frontend` | SPA (React + Vite), servida desde S3 + CloudFront |
| `contratos` | Este repo: schemas de mensajes, OpenAPI, este archivo |

## URLs de producción

| URL | Qué es |
|---|---|
| https://app.melchort.me | Frontend (S3 + CloudFront) |
| https://api.melchort.me | API, detrás de API Gateway. Es la única que usa el frontend |
| https://melchort.me | El backend directo por Nginx; se mantiene por la E0 y es el origen del gateway |

## Reglas que no se rompen

- Todo mensaje publicado lleva la propiedad AMQP `user_id = city.{CODE}` y `cityId` en el cuerpo.
- `msgId` es siempre un UUID nuevo; una respuesta referencia el mensaje original en `data.target`.
- `idpk` nunca es igual a `msgId`. Un reintento conserva el `idpk`; una corrección usa uno nuevo.
- Un `idpk` ya aplicado nunca se vuelve a aplicar al ledger, pero se registra como duplicado.
- No se hace ACK de ACKs, NACK de NACKs ni error de errores. Reintentos siempre con tope y backoff.
- Nunca subir `.env` ni `.pem` a ningún repo.
- El CORS lo configura API Gateway. El backend no agrega headers de CORS (nada de `CORSMiddleware`):
  duplicados, el navegador rechaza la respuesta. Un origen nuevo se pide al rol de deploy.
- Los requisitos no variables de la E0 siguen vigentes: /history paginado y filtrable, contenedores master y connector con HEALTHCHECK, Nginx en el host.

## Registro de uso de IA

El enunciado (RDOC02) pide AI logs "acordes al uso real", con prompts y flujos relevantes por
integrante, o una declaración explícita de no-uso. El curso no entrega un formato: este es el del
grupo y se usa igual en `backend` y `frontend`.

**Dónde.** Un archivo por integrante: `docs/ai-logs/<usuario-github>.md`, en el repo donde quedó
el trabajo. Lo que se haga en `contratos` se registra en `backend`.

**Qué se registra.**

| Tipo de uso | Qué se hace |
|---|---|
| Agéntico (la IA escribe en el codebase) | Una entrada por tarea, siempre |
| Chat | Una entrada solo si influyó en una decisión o en código que quedó en el repo |
| Autocompletado | Se declara una vez en el encabezado del archivo, sin entradas |
| Ninguno | El archivo existe igual, con la frase "Declaro no haber usado IA en esta entrega." |

**Cuándo.** La entrada va en el mismo PR que el trabajo que registra, no al final de la entrega.

**Formato de cada entrada** (plantilla en `docs/ai-logs/_plantilla.md` de cada repo):

```markdown
## AAAA-MM-DD — Título corto de la tarea

- **Herramienta y modo:** herramienta, modelo y modo (agéntico, chat o autocompletado).
- **Tarea:** qué se quería lograr, en una línea.
- **Prompts relevantes:** citados textualmente; solo los que definieron el resultado.
- **Qué produjo la IA:** archivos, recursos o decisiones que salieron de ahí.
- **Verificación:** cómo se comprobó que funciona (tests, CI, prueba manual).
- **Correcciones del integrante:** qué se rechazó, corrigió o cambió de lo propuesto.
- **Referencia:** PR o commits.
```

Los prompts se citan sin secretos: nunca credenciales, tokens ni contenido de un `.env`.

## Protocolo v3: Mercado, Wallet y Reglas que no se rompen

En la Entrega 2 la central dejó de negociar directamente y pasó a operar la **Wallet** (fuente de verdad en créditos y energía) y la liquidación al cierre. Las ciudades comercian entre sí:

### 1. Mecánica del Mercado
- **Ofertas públicas:** Se publican en el exchange `market` con routing key `market.offer.{CODE}` (para TAL: `market.offer.TAL`). Llegan a la cola `market.{CODE}` de todas las ciudades. **No se responden con ACK**.
- **Posturas de compra (Bids):** El comprador envía un mensaje `bid` directamente a la cola de la ciudad vendedora (`city.{VENDEDOR}`) a través del exchange `energy.x`.
- **Validación de Identidad:** Todo mensaje directo entre ciudades lleva la propiedad AMQP `user_id = city.{CODE}` coincidente con su `cityId`. Si no coincide o falta, el receptor responde **NACK IDENTITY_MISMATCH (403)** dirigido a la cola de quien publicó (no a la ciudad usurpada).
- **Swaps en la Wallet:** 
  1. Al aceptar un bid, el vendedor propone un swap en la wallet (`POST /swaps`). La wallet responde `202 Accepted`.
  2. Al confirmarse la propuesta, la wallet emite el evento `swap-proposed` con el `swapToken` JWT (firmado con ES256).
  3. El vendedor envía `bid-accepted` con dicho token al comprador.
  4. El comprador valida criptográficamente el token contra el JWKS de la wallet (`/.well-known/jwks.json`) y sus claims (`sub`, `proposer`, `give`, `take`, `expiresAt`).
  5. Si es válido, el comprador acepta el swap en la wallet (`POST /swaps/{id}/accept`) usando `Idempotency-Key`.
  6. La wallet liquida atómicamente el intercambio y emite `swap-settled` a ambas ciudades; la central emite `trade-recorded` cobrando el transporte según la `distance-table`.

### 2. Reglas que no se rompen (v3)
- **Cero ACKs en el mercado:** Nunca responder con ACK ni NACK a mensajes provenientes del exchange `market` (`offer`, `offer-updated`, `offer-withdrawn`, `market-closed`).
- **Límites de tasa:** Respetar los topes estrictos: máximo 60 operaciones de mercado por minuto, 10 request/min, 120 escrituras y 300 lecturas por minuto a la wallet (con manejo de 429 y cabecera `Retry-After`).
- **Disciplina de Idempotencia:** Reintentar escrituras ante timeouts/502/504 usando siempre la MISMA clave de idempotencia (`idpk`). Una operación nueva o corregida usa una nueva clave.
- **Ventanas y plazos:** No ofertar después de `lastOfferAt`, no enviar bids tras `lastBidAt`, y enviar el `negotiation-report` obligatoriamente entre `reportOpensAt` y `validUntil` para evitar la multa de 50.000 créditos.

## Decisiones de arquitectura

Ver los ADRs en `backend/docs/adr/`:
- AD1: Topología del consumo (actualizado a 3 conexiones: observer, city, market).
- AD2 / AD5: Persistencia del ledger y consistencia con la wallet.
- AD3: Timeouts, disciplina de reintentos y descarte de mensajes.
- AD4: Serialización atómica de bids en el vendedor (lock por oferta).
- AD6: Estrategia configurable sin redeploy.

