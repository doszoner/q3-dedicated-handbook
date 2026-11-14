# Первый запуск

По этой странице сервер из готового примера поднимается за 15 минут. Шаги проверены запуском примеров [stock](glossary.md#stock), [CPMA](glossary.md#cpma) и [OSP](glossary.md#osp) в пустом каталоге на новых [томах](glossary.md#том), на Ubuntu 24.04 x86-64.

## 1. Хост

| | |
|---|---|
| ОС | Linux x86-64 |
| Docker | проверено на 29.2.1 |
| Docker Compose | проверено на v5.0.2 (`docker compose version`) |
| Исходящие соединения | HTTPS для публикации комнаты и [WSS](glossary.md#wss) к `net.js-dos.com:444`; адрес публикации указан в [таблице соединений](common/network.md#соединения) |
| Входящие соединения | [UDP-порт](glossary.md#udp) сервера; он нужен только [обычным клиентам](glossary.md#обычный-клиент) Quake III |

[Браузерным игрокам](glossary.md#браузерный-клиент) входящие порты не нужны. Все соединения описаны на странице [сеть](common/network.md).

## 2. Скачать пример

```sh
git clone https://github.com/doszoner/q3-dedicated-handbook
cd q3-dedicated-handbook/examples/stock      # или cpma, osp
```

В каталоге лежат `compose.yaml` и [`autoexec.cfg`](glossary.md#autoexeccfg), у stock и CPMA ещё `maplist.txt`. [Образ](glossary.md#образ) в `compose.yaml` указан по [digest](README.md#образы), а его [тег](glossary.md#тег) записан в комментарии строкой выше.

## 3. Задать имя комнаты

В `compose.yaml`:

| Строка | Что вписать |
|---|---|
| `NET_SERVER_NAME: room.q3ded.yourserver-stock-ffa` | замените `yourserver` своим именем из латиницы, цифр и знаков `. _ -`. Имя должно быть уникальным. Не заканчивайте его на `p`: почему, сказано [в описании этой переменной](common/env.md#net_server_name) |
| `HOSTNAME: "Q3 FFA"` | название сервера в [лобби](glossary.md#лобби) и в списках серверов |

Если на [хосте](glossary.md#хост) уже есть сервер, второму серверу нужны свои значения:

| Строка | Почему |
|---|---|
| `name:` в начале `compose.yaml`, например `q3ded-stock-2` | по `name:` Compose находит [контейнер](glossary.md#контейнер) и тома проекта. Если `name:` совпадает, `docker compose up -d` во второй копии [пересоздаст](glossary.md#пересоздание-контейнера) первый сервер и заберёт его тома ([как это проверяли](common/env.md#net_server_name)) |
| `PORT` и обе стороны `ports:`, везде одно и то же свободное число | у каждого сервера свой UDP-порт |
| `NET_SERVER_NAME` | у каждой [комнаты](glossary.md#комната) свой [alias](glossary.md#alias) |

Копию примера для второго сервера кладите в отдельный каталог и сразу меняйте в ней `name:`.

## 4. Открыть порт

Для обычных клиентов откройте входящий UDP-порт 27960 (или ваш `PORT`) в сетевом экране (firewall) хоста и у провайдера. Если firewall хоста отбрасывает трафик, который Docker пересылает в контейнер, проброс `ports:` снаружи не работает. Что тогда делать, сказано в разделе [Обычный клиент не видит сервер](common/network.md#обычный-клиент-не-видит-сервер).

## 5. Запустить

```sh
docker compose up -d
docker compose logs -f q3ded
```

## 6. Признаки готовности

| Что проверяем | Признак |
|---|---|
| сервер стартовал с вашими настройками | `Starting ioq3ded: mode=ffa preset=classic … alias=room.q3ded.yourserver-stock-ffa` |
| карта загружена | `Server: q3dm6` |
| режим загружен (только OSP) | `Default mode "Free-For-All" enabled.` |
| сервер зарегистрирован на [peer-сервере](glossary.md#peer-сервер) | `Alias "room.q3ded.yourserver-stock-ffa" registration confirmed for peer <число>` |
| комната в лобби | карточка с вашим `HOSTNAME` в лобби [dos.zone/mp/?lobby=q3](https://dos.zone/mp/index.html?lobby=q3) |
| браузерный игрок вошёл (stock, CPMA) | нажмите `Join` на карточке, ещё раз `Join` в открывшемся окне, затем `Start playing`; в логе видна строка `ClientBegin: 0` |
| обычный клиент видит сервер | запрос [`getstatus`](glossary.md#getstatus) с другой машины получает ответ; команда есть в разделе [Как проверить](common/network.md#как-проверить) |

Строка `registration confirmed` значит только, что сервер зарегистрировался на peer-сервере: то, что комната есть в лобби, она не доказывает. Что доказывает и чего не доказывает каждый признак, сказано в таблице [состояний сервера](common/network.md#состояния-сервера-и-их-признаки).

Если каких-то строк нет или контейнер остановился, откройте страницу [Сервер не работает: где смотреть](troubleshooting.md).

## Дальше

| Задача | Где |
|---|---|
| поменять настройку и применить | [правка ENV и файлов](recipes/edit-env-and-files.md): `docker compose up -d --force-recreate q3ded` |
| включить rcon | [rcon](recipes/rcon.md) |
| задать список карт | [ротация](recipes/rotation.md) |
| что в моде можно настроить и чего нельзя | [stock](stock/README.md), [CPMA](cpma/README.md), [OSP](osp/README.md) |
| перенести сервер со стенда разработчиков DosZone на dos.zone | [переход со стенда на dos.zone](recipes/stand-to-dozone.md) |
| обновить образ | [обновление тега](recipes/update-tag.md) |
