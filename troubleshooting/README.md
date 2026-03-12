# Kafka Troubleshooting Lab

Набор учебных кейсов для стажеров по диагностике проблем в паре микросервисов:

- `repoAnalytics` — Kafka producer (генерирует события аналитики репозиториев).
- `repoNotification` — Kafka consumer (читает события и отправляет уведомления).

## Цель

Отработать системный подход к troubleshooting:

1. Проверка симптома (SLA/SLO, алерты, impact).
2. Анализ логов.
3. Анализ метрик (Prometheus/Grafana).
4. Проверка распределенных трассировок (Jaeger).
5. Формирование гипотез, проверка и план стабилизации.

## Что внутри

- `case-kafka-consumer-lag/student-task.md` — задание для стажера.
- `case-kafka-consumer-lag/instructor-guide.md` — разбор и ожидаемый ход диагностики.
- `case-kafka-consumer-lag/practical-lab-k8s.md` — пошаговый hands-on сценарий для Kubernetes (инъекция деградации, диагностика, mitigation).
- `case-kafka-consumer-lag/artifacts/logs/` — синтетические логи producer/consumer и брокера.
- `case-kafka-consumer-lag/artifacts/metrics/` — срезы метрик и описание графиков.
- `case-kafka-consumer-lag/artifacts/traces/` — выдержки из трассировок Jaeger.

## Формат проведения занятия

- 10 минут: чтение инцидента и уточнение симптома.
- 20 минут: анализ артефактов и построение гипотез.
- 15 минут: презентация плана диагностики и remediation.
- 15 минут: разбор с преподавателем (root cause, short-term и long-term меры).

