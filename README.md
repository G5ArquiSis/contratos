# EnergyShark E1 — Contratos

Repositorio de contratos a nivel de organización (RDOC04). Es la fuente de verdad de todo lo que
`backend` y `frontend` se prometen entre sí y con la central.

| Carpeta / archivo | Contenido | Responsable |
|---|---|---|
| [`schemas/`](schemas/) | JSON Schema de cada mensaje de la mecánica (protocolo v2) | A |
| [`openapi/`](openapi/) | OpenAPI de nuestra API HTTP | B y C (endpoints), revisa D |
| [`AGENTS.md`](AGENTS.md) | Contexto compartido del proyecto, referenciado por ambos repos | Todos |

## Reglas

- Un cambio de contrato entra por PR **antes** que el código que lo implementa en `backend` o `frontend`.
- El PR de un cambio de contrato lo revisa al menos una persona de cada lado afectado.
