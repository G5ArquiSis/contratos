# Schemas de mensajes (protocolo v3)

Un JSON Schema (draft 2020-12) por tipo de mensaje. Cada schema fija su `type`, quién lo emite, canal/routing key y la estructura esperada de `data`.

## Mensajes del protocolo v3

### 1. Feed del Mercado (exchange `market`)
Publicación masiva de ofertas y eventos de mercado. No llevan ACK.
- [`offer`](offer.schema.json): Publicación de oferta (`market.offer.{CODE}`).
- [`offer-updated`](offer-updated.schema.json): Actualización de volumen restante o precio (`market.offer.{CODE}`).
- [`offer-withdrawn`](offer-withdrawn.schema.json): Retiro voluntario de una oferta (`market.offer.{CODE}`).
- [`market-closed`](market-closed.schema.json): Notificación de cierre de mercado emitida por la central (`market.closed`).

### 2. Mensajes Directos entre Ciudades (exchange `energy.x`, rk `city.{OTRA}`)
Mensajes punto a punto con validación obligatoria de `user_id == city.{CODE}`, envelope y ACK.
- [`bid`](bid.schema.json): Postura de compra enviada a la cola del vendedor.
- [`bid-accepted`](bid-accepted.schema.json): Aceptación del bid por parte del vendedor, conteniendo el `swapToken` JWT (ES256) de la wallet.
- [`bid-rejected`](bid-rejected.schema.json): Rechazo de postura (código 409, precede ACK del bid).

### 3. Notificaciones y Liquidación de la Central (exchange `energy.x`, rk `city.{CODE}`)
- [`status-statement`](status-statement.schema.json): Abre el ciclo y mercado (energía vendible, parámetros de mercado, límites y fin de ventana).
- [`transfer`](transfer.schema.json): Financiamiento de créditos con `balanceAfter`, `creditLine` y posible `penalty`.
- [`distance-table`](distance-table.schema.json): Tabla de rutas y costo de transporte, indicando rutas habilitadas/deshabilitadas y tarifa punitiva `disabledRouteTransportCost`.
- [`trade-recorded`](trade-recorded.schema.json): Notificación a ambas partes tras liquidación de un swap, informando costo de transporte.
- [`imbalance-settlement`](imbalance-settlement.schema.json): Liquidación final por desbalance (CR y kWh) al cierre del ciclo.
- [`negotiation-report`](negotiation-report.schema.json): Reporte obligatorio que la ciudad envía a la central en la ventana final del ciclo.
- [`request`](request.schema.json): Solicitud de reenvío de status-statement o distance-table.
- [`ack`](ack.schema.json), [`nack`](nack.schema.json), [`error`](error.schema.json): Confirmaciones y errores de protocolo.

### 4. Eventos de la Wallet (central → ciudad)
Notificaciones asíncronas del estado de swaps en la wallet:
- [`swap-proposed`](swap-proposed.schema.json): Swap propuesto exitosamente en la wallet.
- [`swap-settled`](swap-settled.schema.json): Swap aceptado y liquidado atómicamente.
- [`swap-rejected`](swap-rejected.schema.json): Swap rechazado por falta de saldo u otra restricción.
- [`swap-expired`](swap-expired.schema.json): Swap expirado sin aceptación.
- [`swap-cancelled`](swap-cancelled.schema.json): Swap cancelado por el proponente.

### 5. Mensajes Retirados (E1 → E2)
La central responde `TYPE_RETIRED` (410) si se intentan usar:
- [`demand-statement`](demand-statement.schema.json)
- [`give-take`](give-take.schema.json)
- [`negotiation-proposal`](negotiation-proposal.schema.json)
