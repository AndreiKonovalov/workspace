# Instructor Guide: разбор кейса «Kafka Consumer lag растет»

## Ожидаемый root cause

Основная причина: в `repoNotification` появилась деградация на этапе внешнего HTTP-вызова к Notification Provider, из-за чего увеличилось среднее время обработки сообщения на consumer и вырос lag.

Дополнительный фактор: consumer неэффективно использует batch/poll цикл (маленький `max.poll.records` и частые долгие ретраи), что усиливает backlog.

## Как должен рассуждать стажер

### 1) Подтвердить масштаб проблемы

- По метрикам: рост `kafka_consumergroup_lag` и `records-lag-max`.
- По бизнес-метрике: увеличение задержки доставки уведомлений.
- По логам: ошибки/таймауты внешнего провайдера и увеличение `processing_time_ms`.

### 2) Отсечь ложные гипотезы

- Producer не деградировал (стабильный throughput и без ошибок отправки).
- Kafka broker жив (нет массовых ошибок ISR/under-replicated partitions).
- Проблема локализована в consumer path.

### 3) Подтвердить узкое место

- Jaeger: спаны `POST /send` занимают 80–90% времени end-to-end обработки.
- Логи consumer: повторные retries с backoff, повышенная латентность внешнего API.

## Ожидаемый план стабилизации (первые 30 минут)

1. Увеличить количество реплик `repoNotification` (например, `3 -> 8`) при контроле ребалансировки.
2. Временно уменьшить retry budget/таймаут внешнего провайдера, чтобы снизить залипание воркеров.
3. Настроить circuit breaker/fallback на degraded mode (если допустимо бизнесом).
4. Проверить, что lag перестал расти (перешел в плато/снижение).

## Ожидаемый permanent fix

1. Перевести отправку в провайдер на асинхронный outbox/queue внутри consumer pipeline.
2. Разделить hot partitions/keys при необходимости, выровнять нагрузку по partition.
3. Пересмотреть consumer tuning:
   - `max.poll.records`
   - `max.poll.interval.ms`
   - `fetch.min.bytes`
   - bounded retries + DLQ
4. Добавить SLO и алерты:
   - lag growth rate
   - processing duration p95/p99
   - external dependency latency/error budget

## Что считать хорошим ответом стажера

- Есть четкий порядок диагностики (логи → метрики → трейсы).
- Есть различие short-term mitigation vs long-term remediation.
- Есть измеримые критерии успеха (lag delta, latency p95, error rate).
- Учитываются риски (ребаланс, duplicate processing, потеря сообщений).


## Проведение в live-формате (Kubernetes)

Для практической демонстрации используйте `practical-lab-k8s.md`: там есть controlled failure, список проверок и порядок стабилизации, который можно выполнить прямо в кластере.
