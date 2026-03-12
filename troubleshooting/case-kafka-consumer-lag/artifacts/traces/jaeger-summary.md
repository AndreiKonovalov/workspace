# Jaeger trace summary

## Trace: `ProcessRepoEvent` (slow path)

- Total duration: `6.7s`
- Spans:
  1. `KafkaConsumer.poll` — `32ms`
  2. `DeserializeEvent` — `8ms`
  3. `BuildNotificationPayload` — `14ms`
  4. `HTTP POST notifyx /send` — `6120ms` (retry x2)
  5. `CommitOffset` — `5ms`

## Trace: `ProcessRepoEvent` (normal path)

- Total duration: `410ms`
- Спаны:
  1. `KafkaConsumer.poll` — `26ms`
  2. `DeserializeEvent` — `7ms`
  3. `BuildNotificationPayload` — `15ms`
  4. `HTTP POST notifyx /send` — `352ms`
  5. `CommitOffset` — `4ms`

## Вывод

Узкое место локализовано во внешнем вызове провайдера уведомлений.

