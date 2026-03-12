# Practical Lab (Kubernetes): Kafka Consumer Lag Troubleshooting

Этот сценарий позволяет **вживую** воспроизвести рост consumer lag в `repoNotification` на вашем k8s-стенде.

## 0) Предусловия

- В кластере подняты:
  - `repoAnalytics` (producer)
  - `repoNotification` (consumer)
  - Kafka
  - Prometheus + Grafana
  - Zipkin (есть, но не полностью настроен)
- Есть доступ к `kubectl` и namespace, где работают сервисы.

> Ниже переменные можно подставить под ваш кластер.

```bash
export NS=platform
export NOTIF_DEPLOY=repo-notification
export ANALYTICS_DEPLOY=repo-analytics
export CONSUMER_GROUP=repo-notification-v1
export TOPIC=repo.events
```

## 1) Baseline (до инцидента)

Снимите базовые показатели (5–10 минут):

1. Lag consumer group.
2. `repoNotification` latency p95/p99.
3. Ошибки и таймауты внешнего notification provider.
4. Producer throughput/publish latency.

Пример оперативных команд:

```bash
kubectl -n $NS get pods
kubectl -n $NS logs deploy/$NOTIF_DEPLOY --since=10m | tail -n 100
kubectl -n $NS logs deploy/$ANALYTICS_DEPLOY --since=10m | tail -n 100
```

## 2) Создание controlled failure (рост lag)

### Вариант A (рекомендуется): замедлить egress у consumer

Если `repoNotification` вызывает внешний provider по HTTP, внесите искусственную задержку через `tc` в pod.

```bash
POD=$(kubectl -n $NS get pod -l app=$NOTIF_DEPLOY -o jsonpath='{.items[0].metadata.name}')
kubectl -n $NS exec -it $POD -- sh -c "tc qdisc add dev eth0 root netem delay 2500ms 500ms"
```

Ожидаемый эффект через 2–5 минут:
- растут timeout/retry в consumer-логах,
- растет `kafka_consumergroup_lag`.

Откат:

```bash
kubectl -n $NS exec -it $POD -- sh -c "tc qdisc del dev eth0 root"
```

### Вариант B: уменьшить пропускную способность consumer

```bash
kubectl -n $NS scale deploy/$NOTIF_DEPLOY --replicas=1
```

При стабильном producer throughput lag начнет накапливаться.

## 3) Что должен сделать стажер во время инцидента

### Логи

Проверить признаки деградации в `repoNotification`:

```bash
kubectl -n $NS logs deploy/$NOTIF_DEPLOY --since=15m | rg "timeout|retry|processing|lag|rebalance"
```

### Метрики (Prometheus/Grafana)

Проверить:
- `kafka_consumergroup_lag{group="repo-notification-v1"}`
- `processing_duration_ms` (p95/p99)
- `provider_latency_ms`/`timeout_rate`
- Producer publish latency/throughput

### Трассировка (Zipkin)

Минимальная проверка:
- есть ли трейсы `ProcessRepoEvent`,
- какой span занимает основное время (`HTTP POST /send`).

## 4) Быстрая стабилизация (T+30m)

1. Горизонтально масштабировать consumer:

```bash
kubectl -n $NS scale deploy/$NOTIF_DEPLOY --replicas=8
```

2. Временно снизить “залипание” воркеров:
- уменьшить timeout к provider,
- ограничить retries,
- включить circuit breaker/degraded mode (если есть).

3. Контроль результата:
- lag перестал расти,
- p95 latency пошла вниз,
- timeout-rate снижается.

## 5) Как быстро донастроить Zipkin (чтобы не был “формально поднят”)

Проверьте, что оба сервиса отправляют trace в Zipkin collector.

Пример env (Spring):

```yaml
env:
  - name: MANAGEMENT_TRACING_SAMPLING_PROBABILITY
    value: "1.0"
  - name: MANAGEMENT_ZIPKIN_TRACING_ENDPOINT
    value: "http://zipkin:9411/api/v2/spans"
```

Минимальный smoke-test:
1. Сгенерировать нагрузку producer.
2. Открыть Zipkin UI.
3. Убедиться, что видны трейсы от `repoAnalytics` и `repoNotification` с общим trace-id.

## 6) Критерии успешной практики

- Стажер локализует узкое место (downstream provider latency).
- Дает **пошаговый** план диагностики (logs → metrics → traces).
- Отделяет **mitigation** от **permanent fix**.
- Подтверждает эффект mitigation метриками.

