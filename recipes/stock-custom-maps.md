# Свои карты stock

[Мод](../glossary.md#мод): [stock](../glossary.md#stock). Проверено запуском на `20261112-d608e3a-stock`.

[CPMA](../glossary.md#cpma) и [OSP](../glossary.md#osp) играют только карты из [каталога образа `/app/maps.txt`](../glossary.md#каталог-карт), свои карты они не принимают.

Исходная конфигурация показана в [примере stock](../examples/stock/).

## Шаги

1. Положите PK3 с картой в каталог `custom/` рядом с `compose.yaml`.
2. Смонтируйте его в `volumes:` сервиса:

   ```yaml
   - ./custom:/data/custom/demoq3:ro
   ```

3. Добавьте карту в `maplist.txt` по имени `.bsp` внутри PK3 (для `maps/phlxydm1.bsp` это `phlxydm1`).
4. Примените: `docker compose up -d --force-recreate q3ded`.

[Entrypoint](../glossary.md#entrypoint) ищет строку `maps/<имя>.bsp` в PK3 из `/data/custom/demoq3`. Если не нашёл, сервер не стартует:

```
maplist.txt map <имя> is neither a packaged map (see maps.txt) nor in a PK3 in /data/custom/demoq3
```

## Что допускается внутри PK3

[Движок](../glossary.md#ioq3ded) при старте проверяет каждый PK3 в `/data/custom/demoq3`. Разрешены только каталоги верхнего уровня `maps/`, `textures/`, `scripts/`, `models/`, `sound/`, `music/`, `levelshots/`, `env/`, а файлы в архиве могут храниться только без сжатия (stored) или со сжатием deflate. Один неподходящий PK3 останавливает весь сервер с кодом 3:

```
Custom map preflight: /data/custom/demoq3/map_cpm28.pk3: unsafe ZIP entry path/type or unsupported compression
```

Частая причина: каталог `sounds/` вместо `sound/`. Строка лога называет файл. Уберите его из `custom/` или пересоберите архив.

Нужны ли карте файлы из других PK3, сервер не проверяет. Как ведёт себя карта, которой не хватает текстур из чужого PK3, не проверялось.

## Как карта попадает к игроку

| Игрок | Что происходит |
|---|---|
| [обычный клиент](../glossary.md#обычный-клиент) с `cl_allowDownload 1`, `DOWNLOAD_URL` пуст | скачивает PK3 с сервера по [UDP](../glossary.md#udp) при подключении и входит. В логе сервера `clientDownload: 0 : beginning "demoq3/phlxydm1.pk3"`, затем `completed`. Скорость задаёт `DOWNLOAD_RATE`, по умолчанию 100 КиБ/с |
| обычный клиент с `cl_allowDownload 0` | без PK3 не входит: `ERROR: Couldn't load maps/phlxydm1.bsp`. Положите PK3 в `demoq3/` клиента сами |
| клиент ioquake3 с `cl_allowDownload 1`, на сервере задан `DOWNLOAD_URL`, PK3 лежит по этому адресу | скачал PK3 по HTTPS с `<DOWNLOAD_URL>/demoq3/<имя>.pk3`, переподключился и вошёл; строки `clientDownload` в логе сервера нет. При ошибке сертификата клиент отключился с `Download Error`, загрузки по UDP не было. Проверено клиентом ioquake3 `1.36_GIT_8d2c2b4` на Linux, [шаги ниже](#раздача-по-https) |
| [браузерный игрок](../glossary.md#браузерный-клиент) | не проверялось |

`sv_allowDownload` у stock равен 1: загрузка с сервера включена.

### Раздача по HTTPS

1. Положите тот же PK3 на свой HTTPS-сервер по адресу `<DOWNLOAD_URL>/demoq3/<имя>.pk3`.
2. Задайте в `environment:` `DOWNLOAD_URL: "https://host[:port]/путь"`.
3. Примените: `docker compose up -d --force-recreate q3ded`.

Сертификату HTTPS-сервера должен доверять механизм TLS библиотеки libcurl в клиенте. На проверке переменная `SSL_CERT_FILE` у клиента доверия не дала; загрузка прошла, когда сертификат удостоверяющего центра (CA) был в наборе, который использует libcurl клиента.
