# Стратегия комплаенса GoFuture

<b>Содержание</b>

- [Межрегиональная передача](#межрегиональная-передача)
- [Допуск к эксплуатации](#допуск-к-эксплуатации)

GoFuture согласует передачу и обработку персональных данных в парном регионе до включения репликации и аварийного переключения. Согласование охватывает данные пассажиров и водителей, заказы поездок, координаты и сведения об оплате. Команда проверяет, какие из этих данных содержат реплики PostgreSQL, копии топиков Kafka, журналы Zeebe, контрольные точки Flink и резервные копии. [Схема репликации](02-replication-schema.md)

## Межрегиональная передача

<table style="font-size: 0.85rem;">
  <thead>
    <tr>
      <th valign="top" scope="col" style="vertical-align: top;">Направление</th>
      <th valign="top" scope="col" style="vertical-align: top;">Цель</th>
      <th valign="top" scope="col" style="vertical-align: top;">Что проверить до включения</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td valign="top" style="vertical-align: top;">Малайзия → Сингапур</td>
      <td valign="top" style="vertical-align: top;">Обрабатывать запросы пользователей из Малайзии по основному маршруту.</td>
      <td valign="top" style="vertical-align: top;">Проверить применимость малайзийских правил к первичной передаче персональных данных в Сингапур. Согласовать условия этой передачи до запуска маршрута. <a href="https://www.pdp.gov.my/en/akta/personal-data-protection-guidelines-on-cross-border-transfer-of-personal-data-cbpdt/">Руководство Малайзии</a></td>
    </tr>
    <tr>
      <td valign="top" style="vertical-align: top;">Сингапур → Джакарта</td>
      <td valign="top" style="vertical-align: top;">Копировать данные маршрута Сингапура и обрабатывать его запросы в Джакарте при отказе.</td>
      <td valign="top" style="vertical-align: top;">Подтвердить, что данные при передаче из Сингапура получают сопоставимый уровень защиты по PDPA. Для данных, которые поступили из Малайзии, проверить условия дальнейшей передачи в Джакарту. <a href="https://www.pdpc.gov.sg/-/media/Files/PDPC/PDF-Files/Advisory-Guidelines/AG-on-Key-Concepts/Advisory-Guidelines-on-Key-Concepts-in-the-PDPA-17-May-2022.pdf">Разъяснения PDPC, раздел 19</a> · <a href="https://www.pdp.gov.my/en/akta/personal-data-protection-guidelines-on-cross-border-transfer-of-personal-data-cbpdt/">Руководство Малайзии</a></td>
    </tr>
    <tr>
      <td valign="top" style="vertical-align: top;">Джакарта → Сингапур</td>
      <td valign="top" style="vertical-align: top;">Копировать данные маршрута Джакарты и обрабатывать его запросы в Сингапуре при отказе.</td>
      <td valign="top" style="vertical-align: top;">Определить условие передачи из Индонезии по статье 56 закона № 27/2022. Сначала проверить уровень защиты в стране получателя. При необходимости проверить обязательные гарантии или применимость согласия. <a href="https://peraturan.bpk.go.id/Home/Download/224884/UU%20Nomor%2027%20Tahun%202022.pdf">Закон Индонезии, статья 56</a></td>
    </tr>
    <tr>
      <td valign="top" style="vertical-align: top;">Сан-Паулу → Чили</td>
      <td valign="top" style="vertical-align: top;">Копировать данные маршрута Сан-Паулу и обрабатывать его запросы в Чили при отказе.</td>
      <td valign="top" style="vertical-align: top;">Определить правовое основание обработки и механизм передачи по LGPD. Проверить требования к хранению и обработке данных в Чили на дату запуска. <a href="https://www.gov.br/anpd/pt-br/assuntos/assuntos-internacionais/transferencia-internacional-de-dados/international-affairs">Правила ANPD</a> · <a href="https://www.bcn.cl/leychile/Navegar?idNorma=141599">Закон Чили № 19.628</a> · <a href="https://www.bcn.cl/leychile/Navegar?idNorma=1209272">Поправки № 21.719</a></td>
    </tr>
  </tbody>
</table>

## Допуск к эксплуатации

1. Ответственный за защиту персональных данных ведёт реестр передач. Для каждого потока он фиксирует состав данных, цель, получателя, страны хранения и доступа, срок хранения и основание передачи. Внешний юрист по местному праву проверяет применимые требования, правовое основание и условия передачи. Уполномоченный руководитель GoFuture утверждает передачу, а ответственный за защиту персональных данных фиксирует заключение и решение в реестре. При добавлении рынка, получателя или нового вида данных команда пересматривает запись.
2. Команда платформы включает репликацию и резервный маршрут только для согласованных потоков. Она ограничивает регионы назначения и права на изменение конфигурации, шифрует данные при передаче и хранении и сохраняет журнал изменений. Пока согласование не завершено, данные остаются в основном регионе, а резервный маршрут для соответствующих операций закрыт. [Рекомендации AWS по работе в нескольких регионах](https://docs.aws.amazon.com/prescriptive-guidance/latest/security-reference-architecture/multi-region-architecture.html)
3. Перед запуском команда эксплуатации проверяет, что реплики и копии находятся в согласованных регионах и что API Gateway принимает в резервном регионе только разрешённые операции. Результаты проверок и конфигурацию команда сохраняет вместе с записью о согласовании. При аварии она использует заранее согласованные маршруты. [Схема геомаршрутизации](03-geo-routing.md) · [Сценарии отказа](04-failover-scenario.md)
