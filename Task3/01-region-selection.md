# Обоснование выбора регионов развёртывания

К концу первого года GoFuture планирует развернуть платформу как минимум в четырёх регионах. Задание 3 требует низкой задержки для пользователей Юго-Восточной Азии и Южной Америки. Целевая доступность платформы составляет 99,99%.

В каталоге Yandex Cloud два региона, в России и Казахстане. GoFuture выбирает AWS для развёртывания в четырёх регионах. Платформа использует EC2, ELB/ALB и RDS из описания спринта. Резерв для каждого основного региона находится в той же части света.

<table style="font-size: 0.85rem;">
  <thead>
    <tr>
      <th valign="top" scope="col" style="vertical-align: top;">Регион AWS</th>
      <th valign="top" scope="col" style="vertical-align: top;">Роль региона</th>
      <th valign="top" scope="col" style="vertical-align: top;">Обоснование выбора</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th valign="top" scope="rowgroup" colspan="3" style="vertical-align: top; text-align: left;">Юго-Восточная Азия</th>
    </tr>
    <tr>
      <td valign="top" style="vertical-align: top;">Сингапур (<code>ap-southeast-1</code>)</td>
      <td valign="top" style="vertical-align: top;">Обслуживает трафик из Сингапура и Малайзии. При отказе региона в Джакарте принимает его трафик.</td>
      <td valign="top" style="vertical-align: top;">Из Сингапура сетевая задержка до региона составила 1,5 мс. По выборке из 11 городов Малайзии задержка составила 11,2 мс. Этот показатель служит ориентиром для Куала-Лумпура. <a href="https://pingcloud.io/best-country-by-region/2026-10/aws-singapore-sg/">PingCloud, октябрь 2026</a></td>
    </tr>
    <tr>
      <td valign="top" style="vertical-align: top;">Джакарта (<code>ap-southeast-3</code>)</td>
      <td valign="top" style="vertical-align: top;">Обслуживает трафик из Индонезии. При отказе региона в Сингапуре принимает его трафик.</td>
      <td valign="top" style="vertical-align: top;">По выборке из 36 городов Индонезии сетевая задержка до региона в Джакарте составила 12,2 мс. Для той же выборки задержка до региона в Сингапуре составила 24,3 мс. Из Сингапура до резервного региона в Джакарте задержка составила 16,2 мс. <a href="https://pingcloud.io/best-region-by-country/indonesia/">PingCloud, Индонезия</a> · <a href="https://pingcloud.io/best-region-by-country/singapore/">PingCloud, Сингапур</a></td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <th valign="top" scope="rowgroup" colspan="3" style="vertical-align: top; text-align: left;">Южная Америка</th>
    </tr>
    <tr>
      <td valign="top" style="vertical-align: top;">Сан-Паулу (<code>sa-east-1</code>)</td>
      <td valign="top" style="vertical-align: top;">Обслуживает трафик из Южной Америки.</td>
      <td valign="top" style="vertical-align: top;">Сан-Паулу — единственный общедоступный регион AWS в Южной Америке. Установление TCP-соединения с региональной точкой подключения DynamoDB заняло около 2 мс из Сан-Паулу и 33 мс из Буэнос-Айреса. <a href="https://docs.aws.amazon.com/global-infrastructure/latest/regions/aws-regions.html">Регионы AWS</a> · <a href="https://tracemapper.com/blog/cloud-latency-33-cities">TraceMapper, сентябрь 2026</a></td>
    </tr>
    <tr>
      <td valign="top" style="vertical-align: top;">Южная Америка (Чили)</td>
      <td valign="top" style="vertical-align: top;">При отказе региона в Сан-Паулу принимает трафик из Южной Америки.</td>
      <td valign="top" style="vertical-align: top;">Размещение в Чили сохраняет резерв в Южной Америке. 11 ноября 2026 года AWS сообщила, что планирует открыть регион для всех клиентов к концу года. Этот срок укладывается в план развёртывания GoFuture. <a href="https://www.aboutamazon.com/news/aws/aws-south-america-chile-region-launch">Статус региона AWS в Чили</a></td>
    </tr>
  </tbody>
</table>

Для остальных стран Юго-Восточной Азии Route 53 выбирает Сингапур или Джакарту по задержке. Route 53 учитывает только регионы, которые готовы принять этот трафик. [Схема геомаршрутизации](03-geo-routing.md)

GoFuture размещает экземпляры сервисов в нескольких зонах доступности, чтобы сохранить работу при отказе одной зоны. При отказе региона его трафик принимает резервный регион. Для достижения доступности 99,99% резервный регион должен принять нагрузку и восстановить критические операции в пределах допустимого времени простоя. Резервный регион принимает операции, если его юрисдикция допускает передачу и обработку данных этих операций.

AWS указывает точки подключения EC2, ELB/ALB и RDS для Сингапура, Джакарты и Сан-Паулу. GoFuture подключит Чили после открытия региона для всех клиентов и подтверждения поддержки требуемых конфигураций этих сервисов. [Регионы AWS](https://docs.aws.amazon.com/global-infrastructure/latest/regions/aws-regions.html) · [EC2](https://docs.aws.amazon.com/general/latest/gr/ec2-service.html) · [ELB](https://docs.aws.amazon.com/general/latest/gr/elb.html) · [RDS](https://docs.aws.amazon.com/general/latest/gr/rds-service.html)

Сетевые измерения в таблице характеризуют задержку соединения. Перед запуском GoFuture сравнит время ответа API из выбранных городов через основные и резервные регионы. Эта проверка охватит запросы из Сан-Паулу и Буэнос-Айреса через Чили. CloudWatch Internet Monitor позволяет сравнить время до первого байта по городам на основе наблюдаемого трафика. [CloudWatch Internet Monitor](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-IM-insights.html)
