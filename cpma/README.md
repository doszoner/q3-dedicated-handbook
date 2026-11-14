# CPMA

[Образ](../glossary.md#образ) `ghcr.io/doszoner/q3ded:20261112-c64186e-cpma`, [digest](../glossary.md#digest) `sha256:09f9c028b925395296a4b7f0ee84d08f43268c6ec99726c797e675213f0f58d3`. Проверено запуском на этом [теге](../glossary.md#тег).

| | |
|---|---|
| Мод | CPMA 1.53, каталог `cpma` (`/opt/q3-data/cpma`), базовые файлы в `/opt/q3-data/baseq3` |
| Пример | [examples/cpma](../examples/cpma/) |
| Браузерный вход | через [лобби](../glossary.md#лобби) dos.zone, проверено запуском с примером; после выхода кнопкой Exit слот освобождается сразу |
| Версия для лобби | `q3Version` 37 |
| Обязательный [ENV](../glossary.md#env) | `NET_PEER_SERVER: wss://net.js-dos.com:444`; без него [отказ](../glossary.md#отказ-при-старте) `CPMA requires the approved native NET_PEER_SERVER` |

## Как CPMA выбирает режим

| Что | Кто задаёт | Что делает |
|---|---|---|
| [`mode_start`](../glossary.md#режим-мода) | [entrypoint](../glossary.md#entrypoint) из `MODE` и [`PRESET`](../glossary.md#preset) | режим CPMA, с которым стартует сервер: `1v1`, `FFA`, `TDM`, `CTF`, `CA`, `FT`, `CTFS` или пресет `DZ…` |
| `modes/<режим>.cfg` | образ, `/opt/q3-data/cpma/modes/` | настройки пресетов: `DZIFFA` ([instagib](../glossary.md#instagib) FFA), `DZRFFA` (только ракетница), `DZBFFA` (только BFG), `DZGFFA` (только перчатка), `DZITDM`, `DZICTF`, `DZIFT` (instagib); все семь проверены запуском по строке `Custom Mode:` в логе и значению `mode_current` |
| `map_cfgdir` | entrypoint из `MODE` | каталог списка карт: `cfg-pilot` (duel), `cfg-tdm` (tdm), `cfg-maps` (остальные) |
| список карт | entrypoint из `maplist.txt` при каждом старте | например `cfg-maps/ffamaps.txt`; списки других режимов entrypoint удаляет |
| `server_gameplay` | entrypoint из `GAMEPLAY` | [физика](../glossary.md#физика-игры): `CPM` или `VQ3` |
| [`cpma.cfg`](../glossary.md#конфиг-мода) | образ; entrypoint выполняет его после всех `+set` | `g_allowVote 0`, `bot_enable 0`, `sv_allowDownload 0`, `sv_pure 0`, `server_useMapModes 0` и другие, полный список в [pinned.md](../common/pinned.md) |

Что выбрано, видно в логе: `Standard Mode: FFA` (с цветовыми кодами `^3`, `^5`), затем `Loading maplist from cfg-maps/ffamaps.txt`.

`MODE` можно сменить на прежних [томах](../glossary.md#том), сбрасывать том не нужно: entrypoint удаляет список старого режима и пишет новый.

## Сочетания

| `MODE` | `g_gametype` / тип в лобби | `mode_start` | Список карт | Карта по умолчанию | `MAXCLIENTS` | `PRESET` | `GAMEPLAY` | Лимиты |
|---|---|---|---|---|---|---|---|---|
| `duel` (по умолчанию) | 1 / 1 | `1v1` | `cfg-pilot/1v1maps.txt` | cpm1a | только 20 | `classic` | `cpm`, `vq3` | только `TIMELIMIT` 3, `FRAGLIMIT` 3; `CAPTURELIMIT` принимается, но в игру не передаётся |
| `ffa` | 0 / 2 | `FFA` | `cfg-maps/ffamaps.txt` | cpm4 | 2–26, по умолчанию 20 | `classic`, `instagib`, `rocket-only`, `bfg-only`, `gauntlet-only` | `cpm`, `vq3` | из режима CPMA; ENV лимитов не задавать |
| `tdm` | 3 / 3 | `TDM` | `cfg-tdm/teammaps.txt` | cpm21 | 2–26, 20 | `classic`, `instagib` | `cpm`, `vq3` | из режима |
| `ctf` | 4 / 4 | `CTF` | `cfg-maps/ctfmaps.txt` | cpmctf3 | 2–26, 20 | `classic`, `instagib` | `cpm`, `vq3` | из режима |
| `ca` | 5 / 5 | `CA` | `cfg-maps/camaps.txt` | cpm4 | 2–26, 20 | `classic` | `cpm`, `vq3` | из режима |
| `ftag` | 6 / 6 | `FT` | `cfg-maps/teammaps.txt` | cpm21 | 2–26, 20 | `classic`, `instagib` | `cpm`, `vq3` | из режима |
| `ctfs` | 7 / 7 | `CTFS` | `cfg-maps/ctfmaps.txt` | cpmctf3 | 2–26, 20 | `classic` | `cpm`, `vq3` | из режима |

Карты по умолчанию и диапазоны `MAXCLIENTS` в таблице даны по коду entrypoint; сочетания `PRESET` проверены запуском (см. выше). Режимы `ca` и `ctfs` запуском не проверялись.

Каждая карта из `maplist.txt` должна быть в [`/app/maps.txt`](../glossary.md#каталог-карт), и в её колонке режимов должен стоять ваш `MODE`. Если `MODE` не задан, режим `duel`.

Отказы на сочетания (проверено запуском):

| ENV | Строка лога |
|---|---|
| `MODE=tdm PRESET=rocket-only` | `PRESET must be classic, instagib (MODE=ffa\|tdm\|ctf\|ftag) or rocket-only\|bfg-only\|gauntlet-only (MODE=ffa)` |
| `MODE=ffa TIMELIMIT=5` | `CPMA ffa uses native mode limits; TIMELIMIT/FRAGLIMIT/CAPTURELIMIT overrides are not supported` |
| `MODE=duel MAXCLIENTS=8` | `CPMA duel allows only MAXCLIENTS=20, TIMELIMIT=3, FRAGLIMIT=3` |
| `GAMEPLAY=foo` | `GAMEPLAY must be cpm or vq3` |
| `DOWNLOAD_URL=…` | `DOWNLOAD_URL/DOWNLOAD_RATE are stock-only` |
| `MODE=ffa`, в `maplist.txt` карта cpm1a (она подходит только к duel и tdm) | `maplist.txt map cpm1a must be a ffa map in the packaged catalog (see maps.txt)` |
| `MODE` не задан, в `maplist.txt` карта cpm4 | `maplist.txt map cpm4 must be a duel map in the packaged catalog (see maps.txt)` |

## Начало матча

[Разминку](../glossary.md#разминка) задаёт режим CPMA, а не переменная: `warmup` входит в параметры режима, а `g_doWarmup` CPMA не читает.

| Режим | Разминка | Как начинается матч |
|---|---|---|
| `ffa` | нет, в логе `Warmup: DISABLED` | один игрок видит «Waiting for Players». Когда игроков двое, матч начинается сам через 5 секунд после входа второго; `\ready` не нужен, на него сервер отвечает `Match is already in progress!`. `match_readypercent` на это не влияет: проверено при `51` (по умолчанию) и `100` |
| `tdm` | 10 секунд | нужен `\ready`. При `match_readypercent 51` (значение по умолчанию), если готов 1 игрок из 2, матч не начинается; когда готовы 2 из 2, он начинается после 10 секунд отсчёта |

Другие режимы не проверялись.

Строку `Game_Start: <режим> in arena 0` CPMA пишет в лог при каждой загрузке карты, в том числе во время разминки. Сама по себе она не значит, что матч идёт.

В `tdm` игрок входит наблюдателем и сам выбирает команду: `\team r` или `\team b`. `g_teamAutoJoin 1` в [`autoexec.cfg`](../glossary.md#autoexeccfg) этого не меняет: значение переменной сохраняется, но на проверке оба игрока вошли наблюдателями.

Выключить разминку до конца карты можно так: [rcon](../glossary.md#rcon) `g_allowVote 1`, затем игрок голосует `\callvote warmup 0`. Проверено запуском в `tdm`: `Vote Passed, Arena 0: Warmup 0`, затем `Warmup: DISABLED`, через 5 секунд `Game_Start: TDM`. После rcon `map` режим возвращает свою разминку. Выключить разминку постоянно образ не позволяет: свой файл режима в `/data/home/cpma/modes/` останавливает запуск, а файл, [смонтированный](../glossary.md#смонтированный-файл) в `/opt/q3-data/cpma/modes/`, rcon `mode <имя>` не загрузил, см. [свой файл режима](../recipes/own-configs.md#свой-файл-режима-cpma).

## Лимиты матча и судья

CPMA не читает [переменные движка](../glossary.md#переменная-сервера) `timelimit` и `fraglimit`: ни строка в `autoexec.cfg`, ни rcon лимит матча не меняют. Лимиты задаёт режим CPMA. `timelimit` и `fraglimit` в [`getstatus`](../glossary.md#getstatus) лимиты матча не показывают.

Лимит можно поменять через судью CPMA, его настройки действуют до конца карты. Проверено запуском в `ffa`, в таком порядке:

1. Через rcon: `ref_password CHANGE_ME` (entrypoint и `cpma.cfg` его не задают, по умолчанию судьи нет).
2. В консоли игры: `\ref CHANGE_ME`, ответ `Type: r_help for a list of referee commands.`
3. Там же: `\callvote overtime 2`. В `ffa` по умолчанию включён овертайм «внезапная смерть» (`Overtime Mode: Sudden Death`): при ничьей матч по времени не кончается. `overtime 2` разрешает ничью.
4. Там же: `\callvote timelimit 1`. В логе видно `Referee Setting, Arena 0: Timelimit 1`. Настройка действует сразу, голосование игроков не нужно, и это работает даже при `g_allowVote 0`.

На следующей карте режим снова объявил `Sudden Death`: настройки судьи на новую карту не перешли. В проверке лимит поставили меньше уже прошедшего времени матча, и матч кончился сразу; полный отсчёт лимита с начала матча не проверялся.

## Ротация и простой

[Ротацию](../glossary.md#ротация-карт) ведёт CPMA по списку, который entrypoint при каждом старте пишет из `maplist.txt`. Ручная правка списка в томе теряется при старте. Без игроков через 10 минут после последней загрузки карты CPMA сбрасывает режим и возвращается к первой карте списка. Как настроить список, строки лога и что проверено, описано в [рецепте ротации](../recipes/rotation.md#cpma).

## Физика VQ3

Физику [VQ3](../glossary.md#vq3) включает `GAMEPLAY: "vq3"`, шаги описаны в [рецепте](../recipes/cpma-vq3.md).

## Файлы в томе home

CPMA не стартует, если в томе home (`/data/home`) лежит файл, который мог бы скрыть файл образа, например свой файл режима:

```
Conflicting CPMA saved content: /data/home/cpma/modes/PUBCTF.cfg
```

Какие файлы вызывают отказ и как убрать такой файл, описано в [рецепте](../recipes/edit-env-and-files.md#conflicting-cpma-saved-content). Свой `autoexec.cfg` в томе home отказа не вызывает, но скрывает смонтированный: [layers.md](../common/layers.md#какой-файл-читается).

## Обычный клиент

[Обычному клиенту](../glossary.md#обычный-клиент) нужны все 150 [PK3](../glossary.md#pk3) образа, 148 из `/opt/q3-data/baseq3/` и 2 из `/opt/q3-data/cpma/`, все в одном каталоге `~/.q3a/cpma/`. Как скопировать файлы и подключиться, описано в рецепте [«Файлы игры»](../recipes/game-files.md#что-положить).

## Переменные CPMA

Переменные мода, которые образ не задаёт, описаны в [документации CPMA](https://playmorepromode.com/docs).

Свой `.cfg` для опытов можно положить и выполнить по рецепту [«Свои конфиги для опытов»](../recipes/own-configs.md).

## Чего образ не позволяет

| Что | Почему |
|---|---|
| CTF (и любой режим) со своими настройками режима, например без турнирных правил | настройки режима берутся из `mode_start` и файлов образа. Свой файл в `/data/home/cpma/modes/` останавливает запуск (`Conflicting CPMA saved content`), изменённый `cpma.cfg` или изменённый файл в `/opt/q3-data` тоже (`CPMA packaged data hash verification failed`) |
| свои лимиты матча в `ffa`, `tdm`, `ctf`, `ca`, `ftag`, `ctfs` постоянно | entrypoint отказывает на `TIMELIMIT`, `FRAGLIMIT`, `CAPTURELIMIT`; `timelimit` и `fraglimit` CPMA не читает; лимиты задаёт режим CPMA. До конца карты лимит меняет [судья](#лимиты-матча-и-судья) (проверено запуском в `ffa`) |
| изменить число игроков или лимиты матча в `duel` | entrypoint разрешает только `MAXCLIENTS=20`, `TIMELIMIT=3`, `FRAGLIMIT=3` |
| свои карты | сервер играет только карты из `/app/maps.txt`; свой PK3 в томе home вызывает отказ |
| пресеты кроме перечисленных в таблице | entrypoint знает только `classic`, `instagib`, `rocket-only`, `bfg-only`, `gauntlet-only` |
| голосование, боты | `cpma.cfg` задаёт `g_allowVote 0`, `bot_enable 0`, подменить его нельзя. rcon `addbot` молча ничего не делает. Голосование можно включить через rcon `g_allowVote 1`, оно действует до [рестарта](../glossary.md#рестарт-контейнера) |
| постоянно изменить разминку | разминку задаёт режим CPMA; голосование `\callvote warmup` действует до смены карты |
| игроки сами попадают в команду | `g_teamAutoJoin 1` в `tdm` не действует |
| раздача файлов клиентам | `cpma.cfg` задаёт `sv_allowDownload 0` |
| запуск без `NET_PEER_SERVER` | entrypoint требует одобренный адрес |
