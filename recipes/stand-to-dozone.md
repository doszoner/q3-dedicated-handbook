# Переход со стенда на dos.zone

[Мод](../glossary.md#мод): все. Проверено запуском на `20261112-d608e3a-stock`, пример [examples/stock](../examples/stock/).

## Стенд

У [стенда разработчиков](../glossary.md#стенд) свои [лобби](../glossary.md#лобби) и [peer-сервер](../glossary.md#peer-сервер); их связь с [агентом комнаты](../glossary.md#агент-комнаты) показана на [схеме контейнера](../common/container.md#компоненты).

| | Стенд | dos.zone |
|---|---|---|
| Лобби | `https://188.225.33.164/?lobby=q3` | `https://dos.zone/mp/index.html?lobby=q3` |
| `NET_PEER_SERVER` | `wss://188.225.33.164:444` | `wss://net.js-dos.com:444` |
| `CLOUD_BASE_URL` | `https://188.225.33.164:8443` | не задавать |

## Шаги

1. В `compose.yaml`, в `environment:`:
   - задайте `NET_PEER_SERVER: wss://net.js-dos.com:444`;
   - удалите строку `CLOUD_BASE_URL`.
2. Примените: `docker compose up -d --force-recreate q3ded`.

Чистить [тома](../glossary.md#том) не нужно. Настройки агента комнаты [entrypoint](../glossary.md#entrypoint) пишет заново при каждом старте, и от стенда в `/data/run` ничего не остаётся.

## Результат

| Что | Признак |
|---|---|
| [комната](../glossary.md#комната) зарегистрирована | `Alias "room.q3ded.yourserver-…" registration confirmed for peer <число>` |
| в настройках агента указан адрес dos.zone | `docker compose exec q3ded grep base_url /data/run/dedicated_room_agent.runtime.toml` выводит `https://cloud.js-dos.com` |
| комната на dos.zone | карточка есть в [лобби](https://dos.zone/mp/index.html?lobby=q3) |
| комната ушла со стенда | карточки нет в лобби стенда после обновления страницы; в проверке через 16 секунд после переключения записи о сервере на стенде уже не было, время исчезновения карточки не измерялось |
| тома прежние | `docker volume inspect` показывает прежнюю дату создания; в логе `execing q3config_server.cfg` |

Для [OSP](../glossary.md#osp) карточка на dos.zone подписана «ioq3 FFA», и браузерный вход оттуда не проверялся: см. [OSP](../osp/README.md#браузерный-вход).
