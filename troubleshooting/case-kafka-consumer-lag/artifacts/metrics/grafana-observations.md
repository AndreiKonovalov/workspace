# Grafana observations (срез 11:40–11:45 UTC)

1. **Kafka / Consumer Lag**
   - `repo-notification-v1` lag: 12k → 182k за 5 минут.
   - Growth rate ~560 msg/s.

2. **Notification service / Processing**
   - `processing_duration_ms p95`: 0.4s → 5.2s.
   - `error_rate`: 0.6% → 8.9%.

3. **External Provider panel**
   - `provider_latency p95`: 0.35s → 3.1s.
   - `provider_timeout_rate`: 0.2% → 12.7%.

4. **Analytics producer panel**
   - throughput стабильный (19–20 rps), p95 publish < 15ms.

5. **Kafka broker health**
   - Under-replicated partitions = 0.
   - Broker CPU умеренный (<55%), network in/out без аномалий.

