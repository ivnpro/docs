# Тестирование производительности

Чтобы оценить производительность сервиса синтеза речи вашей инсталляции {{ sk-hybrid-name }}, используйте [контейнер с утилитой тестирования](#test-order-synthesis). Эта утилита выполняет синтетические тесты на основе подготовленного набора файлов с текстами.

## Тесты и ожидаемые результаты {#test-synthesis-description}

{% note info %}

Результаты тестирования зависят от конфигурации вашего оборудования и требований к скорости ответа системы.

{% endnote %}

Сервис {{ speechkit-name }} выполняет синтез речи в режиме <q>запрос-ответ</q>. Утилита тестирования отправляет в сервис запросы, имитируя реальную нагрузку на сервис. Интенсивность запросов задается в параметре `RPS` (requests per second, количество запросов в секунду) в [настройках утилиты](#test-order-synthesis). В лог контейнера записываются метрики ответов сервиса:

* Процессные метрики:
   * Seconds per second (SPS) – количество секунд синтезированного текста, которое генерируется за секунду работы.
   * Длина итогового аудио.
   * Задержка ответа.
* Итоговые метрики:
   * `q99` — квантиль уровня `0.99` времени ответа сервиса на запрос (время синтеза фразы в 99% случаев меньше или равно величине квантиля):
      * на синтез фраз, длительность которых менее 6 секунд;
      * на синтез фраз, длительность которых более 6 секунд;
      * на синтез фраз любой длительности.

Пример фрагмента лога тестирования со значениями процессных метрик:

```text
INFO: 2021-09-22 15:07:00.882 +0000 load_test.cpp:89 Sample=но боязнь и одновременно надежды на чудо останавливали меня от этого сообщения ., duration=4.305375s, latency=0.371161s
INFO: 2021-09-22 15:07:00.882 +0000 load_test.cpp:92 Passed time: 4.384470733, seconds generated=49.561, sps=11.17579835, avgAudioLength=5506
```

Где:

* `duration` — длина итогового аудио.
* `latency` — задержка ответа.
* `sps` — SPS.

Пример фрагмента лога тестирования со значением квантиля `q99` по фразам длительностью менее 6 секунд:

```text
INFO: 2021-09-22 15:07:00.059 +0000 load_test.cpp:260 Audio < 6 seconds latencies
INFO: 2021-09-22 15:07:00.059 +0000 load_test.cpp:262 q=0.99: 390ms
```

Пример фрагмента лога тестирования со значением квантиля `q99` по фразам длительностью более 6 секунд:

```text
INFO: 2021-09-22 15:06:59.734 +0000 load_test.cpp:260 Audio >= 6 seconds latencies
INFO: 2021-09-22 15:06:59.734 +0000 load_test.cpp:262 q=0.99: 505ms
```

Пример фрагмента лога тестирования со значениями квантиля `q99` по фразам любой длительности:

```text
INFO: 2021-09-22 15:07:00.059 +0000 load_test.cpp:260 Utterance latency
INFO: 2021-09-22 15:07:00.059 +0000 load_test.cpp:262 q=0.99: 505ms
```

Контейнеры {{ sk-hybrid-name }} рассчитаны на то, чтобы обеспечивать [указанные значения метрики SPS для двух рекомендованных конфигураций сервера](../system-requirements.md#hardware). При таких значениях SPS величина квантиля `q99` не превышает 500-600 мс. Увеличивайте `RPS` до тех пор, пока значения метрик SPS и `q99` не выйдут за эти пределы. Результаты тестирования помогут оценить  производительность вашей инсталляции и то, насколько увеличение количества запросов в секунду повышает SPS и влияет на задержку ответа сервиса.

## Порядок проведения тестирования {#test-order-synthesis}

1. Скачайте контейнер с тестами:

    ```bash
    docker pull cr.yandex/${REGISTRY_ID}/tts-tools
    ```

2. Запустите контейнер `tts-tools`:

    ```bash
    docker run --network=host \
       --env ENVOY_HOST=0.0.0.0 \
       --env ENVOY_TTS_PORT=9080 \
       --env RPS=1 \
       tts-tools
    ```

   Где:

    * `ENVOY_HOST` — IP-адрес сервиса синтеза. Если тесты запускаются на том же сервере, что и сервис синтеза, укажите значение `0.0.0.0`.
    * `ENVOY_TTS_PORT` — порт сервиса синтеза (по умолчанию `9080`).
    * `RPS` — количество запросов синтеза в секунду.

3. Результаты теста будут доступны в логах контейнера:

    ```bash
    docker logs tts-tools
    ```