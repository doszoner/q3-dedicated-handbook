# Правка переменных окружения (ENV) и файлов

[Мод](../glossary.md#мод): все. Проверено запуском на `20261112-d608e3a-stock`, `20261112-c64186e-cpma`, `20261114-9e899840-osp` с [примерами](../examples/): в [`autoexec.cfg`](../glossary.md#autoexeccfg) включили [rcon](../glossary.md#rcon) и изменили `g_inactivity`, в `compose.yaml` изменили `MAXCLIENTS` (stock, CPMA) или `MODE` (OSP, с `ffa` на `tdm` на прежних [томах](../glossary.md#том)).

## Одна команда на всё

После правки `compose.yaml`, `autoexec.cfg` или `maplist.txt`:

```sh
docker compose up -d --force-recreate q3ded
```

[Контейнер](../glossary.md#контейнер) пересоздаётся, а тома `/data/run` и `/data/home` остаются прежними.

## Почему не что-то другое

| Команда | Правка ENV | Правка смонтированного файла |
|---|---|---|
| `docker compose restart` | **не применяется**: контейнер стартует со старым [ENV](../glossary.md#env) | применяется, и правка на месте, и замена файла редактором |
| `docker compose up -d` | применяется: Compose видит изменение и пересоздаёт контейнер | **не применяется**: Compose не видит изменений и контейнер не трогает |
| `docker compose up -d --force-recreate q3ded` | применяется | применяется |

Если редактор сохранил файл заменой (так работает и `sed -i`), `docker compose exec q3ded cat …` покажет в работающем контейнере прежний текст; почему так, объяснено в [примечаниях](../common/layers.md#примечания). `restart` проверен запуском на stock, пересоздание с `sed -i` проверено на всех трёх примерах.

Если правили только файлы, хватит `docker compose restart`. Если правили ENV или не уверены, выполните `docker compose up -d --force-recreate q3ded`: эта команда покрывает оба случая.

## Проверить, что применилось

| Что меняли | Как проверить |
|---|---|
| `HOSTNAME`, `MAXCLIENTS`, `MODE`, лимиты | [`getstatus`](../glossary.md#getstatus): `sv_hostname`, `sv_maxclients`, `g_gametype`, `timelimit` |
| строку в `autoexec.cfg` | ввести имя переменной в rcon или [консоли](../glossary.md#консоль-сервера), в ответ придёт новое значение |
| `maplist.txt` | в логе `Server: <первая карта списка>` |

Тома остались прежними, если `docker volume ls` показывает те же тома, а в `/data/home/<мод>/` лежит [`q3config_server.cfg`](../glossary.md#q3config_servercfg) от прошлых запусков.

Что из этого сохраняется после каждого события, описано в [layers.md](../common/layers.md#жизненный-цикл-настройки).

## Conflicting CPMA saved content

CPMA и OSP не стартуют, если в `/data/home` лежит файл, который мог бы скрыть файл [образа](../glossary.md#образ):

```
Conflicting CPMA saved content: /data/home/cpma/modes/<файл>
```

Удалите ровно этот файл, а не весь том. Путь возьмите из строки лога; пример для `PUBCTF.cfg`:

```sh
docker compose run --rm --no-deps --entrypoint rm q3ded /data/home/cpma/modes/PUBCTF.cfg
docker compose up -d --force-recreate q3ded
```

Всё остальное в томе, включая `q3config_server.cfg`, сохраняется. Пустой каталог `modes/` [отказа](../glossary.md#отказ-при-старте) не вызывает. Проверено запуском на `20261112-c64186e-cpma`: в томе лежал файл `modes/PUBCTF.cfg`, удалили только его, после `up -d` сервер запустился, в логе `registration confirmed`.

Какие файлы вызывают отказ, по коду [entrypoint](../glossary.md#entrypoint): любая символическая ссылка; `*.pk3`, `cgame.qvm`, `qagame.qvm`, `ui.qvm`, [`cpma.cfg`](../glossary.md#конфиг-мода) в любом месте `/data/home`; любой файл в `cfg-maps/`, `cfg-pilot/`, `cfg-tdm/`, `modes/`. У OSP к ним добавляются `osp.cfg` и `dz-osp.cfg`. Списки `*maps.txt` CPMA entrypoint удаляет и пишет сам, они отказа не вызывают. Проверено запуском: `modes/PUBCTF.cfg` и `vm/ui.qvm` у CPMA (отказ `Conflicting CPMA saved content: /data/home/cpma/vm/ui.qvm`) и `osp.cfg` у OSP.
