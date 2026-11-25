# Обеспечение качества данных

Статус **Реализовано** означает, что проект уже предусматривает эту меру. Статус **К реализации** относится к дополнительным проверкам конвейера данных.

<table style="font-size: 0.85rem;">
  <thead>
    <tr style="vertical-align: top;">
      <th scope="col" style="vertical-align: top;">Сценарий</th>
      <th scope="col" style="vertical-align: top;">Данные для проверки</th>
      <th scope="col" style="vertical-align: top;">Механизм</th>
      <th scope="col" style="vertical-align: top;">Какие ошибки предотвращает механизм</th>
      <th scope="col" style="vertical-align: top;">Статус и место проверки</th>
    </tr>
  </thead>
  <tbody>
    <tr style="vertical-align: top;">
      <th scope="row" style="vertical-align: top;">Динамическое ценообразование</th>
      <td style="vertical-align: top;">
        <ul style="margin: 0; padding-left: 1.2em;">
          <li>Поездки в ожидании водителя</li>
          <li>Назначения и освобождения водителей</li>
          <li>Координаты водителей и время наблюдения</li>
          <li>Расчёты стоимости поездок</li>
        </ul>
      </td>
      <td style="vertical-align: top;">
        <ul style="margin: 0; padding-left: 1.2em;">
          <li>Flink и Analytics проверяют обязательные поля событий.</li>
          <li>Flink учитывает только последние координаты каждого водителя, пока они актуальны.</li>
          <li>Pricing применяет базовую стоимость, если данные о спросе или предложении устарели или отсутствуют.</li>
        </ul>
      </td>
      <td style="vertical-align: top;">
        <ul style="margin: 0; padding-left: 1.2em;">
          <li>Расчёт стоимости попадает в неверную зону или период.</li>
          <li>Водитель попадает в число доступных в зоне, которую он уже покинул.</li>
          <li>Pricing рассчитывает надбавку по устаревшим данным.</li>
        </ul>
      </td>
      <td style="vertical-align: top;">
        <strong>Реализовано.</strong>
        <ul style="margin: 0; padding-left: 1.2em;">
          <li>Flink использует актуальные координаты.</li>
          <li>Pricing применяет базовую стоимость при потере актуальных данных о спросе или предложении.</li>
        </ul>
        <strong>К реализации.</strong>
        <ul style="margin: 0; padding-left: 1.2em;">
          <li>При чтении из Kafka задание Flink проверяет события для расчёта стоимости в реальном времени.</li>
          <li>Analytics проверяет записи о расчётах стоимости при чтении из Kafka.</li>
        </ul>
      </td>
    </tr>
    <tr style="vertical-align: top;">
      <th scope="row" style="vertical-align: top;">Прогнозирование спроса</th>
      <td style="vertical-align: top;">
        <ul style="margin: 0; padding-left: 1.2em;">
          <li>Заявки после подтверждения стоимости поездки</li>
          <li>Для каждой заявки нужны идентификатор поездки, время создания, регион и зона посадки.</li>
          <li>Для каждого окна нужны фактическое и прогнозное число заявок.</li>
        </ul>
      </td>
      <td style="vertical-align: top;">
        <ul style="margin: 0; padding-left: 1.2em;">
          <li>Аналитический конвейер учитывает каждую поездку один раз.</li>
          <li>Конвейер группирует заявки по времени создания.</li>
          <li>Перед публикацией конвейер сверяет фактическое число заявок с числом уникальных событий Booking за то же окно.</li>
        </ul>
      </td>
      <td style="vertical-align: top;">
        <ul style="margin: 0; padding-left: 1.2em;">
          <li>Повторные сообщения завышают спрос.</li>
          <li>Из-за задержки сообщения заявка попадает в неверное окно.</li>
          <li>При пропуске входных данных отчёт показывает нулевой спрос.</li>
        </ul>
      </td>
      <td style="vertical-align: top;">
        <strong>Реализовано.</strong>
        <ul style="margin: 0; padding-left: 1.2em;">
          <li>По заданию 2 потребители обрабатывают сообщения идемпотентно.</li>
          <li>В задании 4 аналитический конвейер группирует заявки по времени создания.</li>
        </ul>
        <strong>К реализации.</strong>
        <ul style="margin: 0; padding-left: 1.2em;">
          <li>Аналитический конвейер сверяет число заявок перед публикацией.</li>
          <li>Затем конвейер записывает фактические данные в ClickHouse или передаёт исторические данные в SageMaker AI.</li>
        </ul>
      </td>
    </tr>
    <tr style="vertical-align: top;">
      <th scope="row" style="vertical-align: top;">Обнаружение мошенничества</th>
      <td style="vertical-align: top;">
        <ul style="margin: 0; padding-left: 1.2em;">
          <li>Данные о поездках и платежах</li>
          <li>Исходные оценки риска</li>
          <li>Итоги проверок на мошенничество</li>
        </ul>
      </td>
      <td style="vertical-align: top;">
        <ul style="margin: 0; padding-left: 1.2em;">
          <li>Аналитический конвейер связывает итоги проверок с поездками или платежами по идентификаторам.</li>
          <li>Итоги проверок служат метками для обучения.</li>
          <li>BI показывает случаи, по которым проверка ещё идёт.</li>
        </ul>
      </td>
      <td style="vertical-align: top;">
        <ul style="margin: 0; padding-left: 1.2em;">
          <li>Итог проверки попадает в запись о другой поездке или платеже.</li>
          <li>Данные показывают мошенничество даже в случаях, когда есть только подозрение.</li>
        </ul>
      </td>
      <td style="vertical-align: top;">
        <strong>Реализовано.</strong>
        <ul style="margin: 0; padding-left: 1.2em;">
          <li>Fraud передаёт итоги проверок для обучения и отчётности.</li>
        </ul>
        <strong>К реализации.</strong>
        <ul style="margin: 0; padding-left: 1.2em;">
          <li>Конвейер проверяет идентификаторы поездок и платежей, когда сопоставляет записи с итогами проверок.</li>
          <li>Этот этап предшествует передаче обучающего набора в SageMaker AI.</li>
        </ul>
      </td>
    </tr>
  </tbody>
</table>
