# stock: Quake III Arena (ioquake3)

[Образ](../glossary.md#образ) `ghcr.io/doszoner/q3ded:20261112-d608e3a-stock`, [digest](../glossary.md#digest) `sha256:4461a135fde82ddf4d8695bd9362ed1981e9bc3b9092081ac77e8ca7f5582294`. Проверено запуском на этом [теге](../glossary.md#тег).

| | |
|---|---|
| [Каталог игры](../glossary.md#каталог-игры) | `demoq3` (`/opt/q3-data/demoq3`) |
| Пример | [examples/stock](../examples/stock/) |
| Браузерный вход | через [лобби](../glossary.md#лобби) dos.zone, проверено запуском с примером; после выхода кнопкой Exit слот освобождается сразу |
| Версия для лобби | `q3Version` 37 |

## Сочетания

Режим задаёт [ENV](../glossary.md#env) `MODE`. Лимиты матча [entrypoint](../glossary.md#entrypoint) берёт из ENV, и они перекрывают [`autoexec.cfg`](../glossary.md#autoexeccfg).

| `MODE` | `g_gametype` / тип в лобби | Карта по умолчанию | `MAXCLIENTS` по умолчанию | `PRESET` | `GAMEPLAY` | Лимиты по умолчанию | Откуда лимиты |
|---|---|---|---|---|---|---|---|
| `ffa` | 0 / 2 | q3dm17 | 8 | только `classic` | не задавать | `TIMELIMIT` 15, `FRAGLIMIT` 30 | из ENV через `+set` |
| `duel` | 1 / 1 | ztn3tourney1 | 2 | только `classic` | не задавать | 15, 30 | из ENV через `+set` |
| `tdm` | 3 / 3 | q3dm6 | 8 | только `classic` | не задавать | 15, 30 | из ENV через `+set`; кроме того, `g_teamAutoJoin 1`, `g_teamForceBalance 1` |
| `ctf` | 4 / 4 | q3ctf1 | 8 | только `classic` | не задавать | 15, 30, `CAPTURELIMIT` 8 | из ENV через `+set`; командные настройки те же, что у `tdm` |

Карты, значения по умолчанию и диапазоны в таблице даны по коду entrypoint, кроме того, что ниже названо проверенным запуском. `MAXCLIENTS` во всех режимах 2–26. Сервер стартует с карты из первой строки `maplist.txt`. Карта по умолчанию (или ENV `MAP`) нужна, только если `maplist.txt` не [смонтирован](../glossary.md#смонтированный-файл) совсем, и тогда [ротации](../glossary.md#ротация-карт) нет. Проверено запуском: без `MAP` сервер стартует на q3dm17 (`ffa`), с `MAP=q3dm6` на q3dm6, `rotation.cfg` не появляется; когда истекает `TIMELIMIT` (`TIMELIMIT=1`, один игрок), перезапускается та же карта (`nextmap` = `map_restart 0`).

[Отказы](../glossary.md#отказ-при-старте) на сочетания:

| ENV | Строка лога |
|---|---|
| `MODE=ca` и любой другой режим не из четырёх | `MODE must be ffa, duel, tdm or ctf` |
| `PRESET=instagib` и любой другой, кроме `classic` | `PRESET must be classic for stock` |
| `GAMEPLAY` непустой (если не задан или пуст, отказа нет) | `GAMEPLAY is CPMA-only` |
| `MAXCLIENTS=30` | `MAXCLIENTS must be <= 26` |
| `MODE=ctf` и карта не для CTF | `MODE=ctf requires a CTF-compatible packaged map: …` |

Подходит ли карта к режиму, stock проверяет только при `MODE=ctf`. `MODE=duel` с `q3ctf1` запускается.

Все ENV описаны в [env.md](../common/env.md), все отказы при старте собраны в [troubleshooting.md](../troubleshooting.md#отказы-при-старте).

## Ротация

Список карт задаёт `maplist.txt`. Entrypoint превращает его в `rotation.cfg`, цепочку команд `vstr`: `set rotate1 "map q3dm17 ; set nextmap vstr rotate2"` и так далее. Карта меняется по окончании матча, а после [рестарта контейнера](../glossary.md#рестарт-контейнера) сервер начинает с первой. Подробно в [рецепте ротации](../recipes/rotation.md).

## Свои карты

[PK3](../glossary.md#pk3) с картами монтируются в `/data/custom/demoq3`. Перед стартом сервер проверяет, что лежит в архиве. [Обычному клиенту](../glossary.md#обычный-клиент) он раздаёт карту по [UDP](../glossary.md#udp). С ENV `DOWNLOAD_URL` клиент может скачать PK3 по HTTPS вместо UDP.

Что допускается в PK3, какие нужны зависимости, как карта попадает к игроку и как настроить раздачу по HTTPS, описано в [рецепте](../recipes/stock-custom-maps.md).

## Обычный клиент

Обычному клиенту нужны все 138 PK3 образа из `/opt/q3-data/demoq3/`, их кладут в каталог `~/.q3a/demoq3/`. Как скопировать файлы и подключиться, описано в рецепте [«Файлы игры»](../recipes/game-files.md#что-положить).

## Переменные ioquake3

Переменные, которые образ не задаёт, описаны в документации ioquake3: [руководство администратора](https://ioquake3.org/help/sys-admin-guide/). Значения, которые задаёт образ, перечислены в [pinned.md](../common/pinned.md).

Свой `.cfg` для опытов можно положить и выполнить по рецепту [«Свои конфиги для опытов»](../recipes/own-configs.md).

## Чего образ не позволяет

| Что | Почему |
|---|---|
| боты | entrypoint задаёт `bot_enable 0` при каждом старте, ENV для ботов нет; [rcon](../glossary.md#rcon) `addbot` молча ничего не делает |
| включить голосование постоянно | entrypoint задаёт `g_allowVote 0`; rcon `g_allowVote 1` действует до рестарта |
| включить [разминку](../glossary.md#разминка) постоянно | entrypoint задаёт `g_doWarmup 0` при каждом старте; rcon `g_doWarmup 1` действует до рестарта |
| [`PRESET`](../glossary.md#preset) кроме `classic`, `GAMEPLAY` | пресеты и [физика](../glossary.md#физика-игры) VQ3/CPM есть только в [CPMA](../glossary.md#cpma) |
| режимы кроме `ffa`, `duel`, `tdm`, `ctf` | entrypoint знает только эти четыре |
| больше 26 игроков | это предел `MAXCLIENTS` в entrypoint |
| свой игровой модуль `qagame.qvm` | сервер отказывается стартовать: `saved-home game modules are not supported`; он работает только с модулем образа |
| продолжить ротацию после рестарта | `rotation.cfg` пишется заново, и сервер начинает с первой карты |

## Изменения

| Тег | Чем отличается от `20261112-d608e3a-stock` |
|---|---|
| `20261110-d608e3a-stock` | нет проверки `GAMEPLAY` и поля `gameplay=` в строке `Starting ioq3ded:`. Сервер, карты и манифест те же |
