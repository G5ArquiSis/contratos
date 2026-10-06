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

## Decisiones de arquitectura

Ver los ADRs en `backend/docs/adr/` (AD1 topología del consumo, AD2 persistencia del ledger,
AD3 timeouts de negociación).
