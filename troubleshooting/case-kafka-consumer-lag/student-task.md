# Практика: «Kafka Consumer lag растет»

## Сценарий

В проде деградировал сервис `repoNotification`: уведомления приходят пользователям с большой задержкой.

### Контекст

- Топик: `repo.events`
- Партиции: `12`
- Consumer group: `repo-notification-v1`
- Producer: `repoAnalytics`
- Consumer: `repoNotification`

### Симптомы

- Alert `KafkaConsumerLagHigh` сработал в 11:42 UTC.
- P95 задержки доставки уведомлений выросла с 1.5s до 28s.
- Backlog consumer group продолжает расти.

## Твоя задача как on-call инженера

Используя артефакты из папки `artifacts/`, предложи:

1. **План диагностики по шагам** (что смотришь и в каком порядке).
2. **3–5 проверяемых гипотез** о причине роста lag.
3. **План стабилизации в первые 30 минут** (что можно сделать быстро).
4. **План окончательного решения** (устранение root cause).
5. **Пост-инцидентные action items** (мониторинг, архитектура, процессы).

## Ограничения

- Нельзя менять схему событий в Kafka в момент инцидента.
- Нельзя отключать producer.
- Допускается горизонтальное масштабирование consumer.

## Подсказка по формату ответа

Используй шаблон:

```text
1) Symptom validation
2) Logs findings
3) Metrics findings
4) Tracing findings
5) Hypothesis + verification
6) Mitigation plan (T+30m)
7) Permanent fix plan
8) Preventive controls
```


## Hands-on режим (на вашем k8s-стенде)

Если преподаватель запускает live-сценарий, используй `practical-lab-k8s.md` как источник фактических команд (`kubectl`, метрики, tracing) и приложи в ответ конкретные шаги с командами/графиками.
