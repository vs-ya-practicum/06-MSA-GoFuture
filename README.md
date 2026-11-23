# GoFuture. Проектная работа спринта 6

## Задание 2. Проектируем Event-Driven архитектуру

Событийная платформа GoFuture использует региональные топики Kafka для обмена сообщениями. Camunda координирует назначение водителя, а Flink рассчитывает показатели спроса и предложения для динамического ценообразования.

- [ADR событийной платформы](Task2/01-event-driven-platform.adr.ru.md) описывает модель событий, схему топиков Kafka, Saga назначения водителя и механизмы надёжной доставки.
- [Проект мониторинга](Task2/04-monitoring-design.ru.md) обосновывает выбор подходов, определяет метрики и указывает источники их сбора.
- [Исходный код диаграммы C2](Task2/diagrams/02-event-driven-platform.container.c4.puml) показывает целевую архитектуру и связи инструментов мониторинга с источниками метрик.
- [Исходный код диаграммы BPMN](Task2/diagrams/03-driver-assignment-saga.bpmn.mmd) показывает шаги назначения водителя и компенсации при отмене.

<details>
<summary>GoFuture Event-Driven Platform Container</summary>

![GoFuture Event-Driven Platform Container](Task2/diagrams/.svg/02-event-driven-platform.container.c4.svg)

</details>
