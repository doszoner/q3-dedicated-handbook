# Обновление тега с сохранением томов

[Мод](../glossary.md#мод): все. Проверено запуском на [stock](../glossary.md#stock), переход с `20261110-d608e3a-stock` на `20261112-d608e3a-stock`, пример [examples/stock](../examples/stock/).

## Шаги

1. Посмотрите, чем новый тег отличается от прежнего. Для stock отличия перечислены в разделе [«Изменения»](../stock/README.md#изменения).
2. В `compose.yaml` замените строку `image:` на строку с [digest](../README.md#образы) нового образа и обновите тег в комментарии. Строку `name:` не меняйте: по ней Compose находит прежние [тома](../glossary.md#том).

   ```yaml
   # Тег 20261112-d608e3a-stock
   image: ghcr.io/doszoner/q3ded@sha256:4461a135fde82ddf4d8695bd9362ed1981e9bc3b9092081ac77e8ca7f5582294
   ```

3. Примените:

   ```sh
   docker compose up -d --force-recreate q3ded
   ```

   `docker compose down -v` не нужен: эта команда удаляет тома.

## Результат

| Что | Признак |
|---|---|
| работает новый образ | `docker inspect $(docker compose ps -q q3ded) --format '{{.Config.Image}}'` показывает строку `image:` с новым digest; при переходе на `20261112` в строке `Starting ioq3ded:` появилось поле `gameplay=` |
| тома прежние | `docker volume inspect` показывает прежнюю дату создания |
| настройки из тома `home` на месте | в логе `execing q3config_server.cfg`; значение, заданное через [rcon](../glossary.md#rcon) `seta` на старом теге, действует на новом |

Что ещё сохраняется при [пересоздании](../glossary.md#пересоздание-контейнера), описано в разделе [жизненный цикл](../common/layers.md#жизненный-цикл-настройки).
