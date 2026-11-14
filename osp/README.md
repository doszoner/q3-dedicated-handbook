# OSP

[Образ](../glossary.md#образ) `ghcr.io/doszoner/q3ded:20261114-9e899840-osp`, [digest](../glossary.md#digest) `sha256:ba1c045a2cdac33bffd53279c3dcf8105115fe1ba50a64c6e0ea4a3dd95377c3`. Проверено запуском на этом [теге](../glossary.md#тег).

| | |
|---|---|
| Мод | OSP 1.03a, каталог `osp` (`/opt/q3-data/osp`), базовые файлы в `/opt/q3-data/baseq3` |
| Пример | [examples/osp](../examples/osp/) |
| Браузерный вход | проверен запуском через [лобби](../glossary.md#лобби) [стенда](../glossary.md#стенд) разработчиков DosZone. В лобби dos.zone [комната](../glossary.md#комната) видна с подписью «ioq3 FFA», вход оттуда не проверялся. Подробнее [ниже](#браузерный-вход) |
| Версия для лобби | `q3Version` 38 (у [stock](../glossary.md#stock) и [CPMA](../glossary.md#cpma) 37) |
| Лимит игроков | всегда 14 |
| Голосование и [разминка](../glossary.md#разминка) | включены по умолчанию: после загрузки режима `g_allowVote 1`, `g_doWarmup 1` |

## Сочетания

Режимов шесть. У всех `MAXCLIENTS` 14, [`PRESET`](../glossary.md#preset) только `classic`, `GAMEPLAY` не читается, лимиты задаёт режим.

| `MODE` | Режим OSP в логе | `g_gametype` / тип в лобби | `mode_start` / `server_availmodes` | Карта по умолчанию | Лимиты из режима |
|---|---|---|---|---|---|
| `ffa` | `Free-For-All` | 0 / 2 | 0 / 1 | ospdm1 | `timelimit` 20, `fraglimit` 30, [ротация](../glossary.md#ротация-карт) `map_rotate 1` |
| `duel` | `1v1` | 1 / 1 | 1 / 2 | ospdm4 | `timelimit` 15, `fraglimit` 0 |
| `tdm` | `TeamplayDM` | 3 / 3 | 3 / 8 | ospdm1 | `timelimit` 20, `fraglimit` 0 |
| `ctf` | `CTF` | 4 / 4 | 4 / 16 | ospctf1 | `timelimit` 20, `capturelimit` 0 |
| `ca` | `ClanArena` | 5 / 5 | 5 / 32 | ospca1 | `roundlimit` 7, `fraglimit` 7, `timelimit` 0 |
| `ftag` | `FreezeTag TDM` | **3** / 6 | 6 / 64 | pro-q3dm6 | `timelimit` 20; заморозку включает `server_freezetag 1` |

[Entrypoint](../glossary.md#entrypoint) задаёт `server_availmodes` равным `1 << mode_start`, то есть в маске разрешённых режимов стоит только выбранный. Проверено запуском: при `MODE=ffa` значение `server_availmodes 1`, при `MODE=tdm` значение `8`; смену режима голосованием не пробовали. Freeze Tag в OSP устроен как командный бой (deathmatch, `g_gametype 3`) с `server_freezetag 1`, а лобби показывает его как тип 6.

Режим готов, когда в логе появляется эта строка, примерно через секунду после `registration confirmed`:

```
Default mode "Free-For-All" enabled.
```

## Сброс при инициализации режима

После этой строки режим задаёт свои значения и перезапускает карту (`Gametype changed, clearing session data`). Всё, что задаёт режим, перекрывает [`autoexec.cfg`](../glossary.md#autoexeccfg). После старта у OSP две фазы:

| Фаза | Когда | Что действует |
|---|---|---|
| A | от первого [`InitGame`](../glossary.md#initgame), около 1,8 с, до строки `Default mode "<режим>" enabled.` | значения из `autoexec.cfg`, кроме всего, что entrypoint задаёт через `+set` ([pinned.md](../common/pinned.md)); из проверенных строк это `sv_allowDownload 0`, `g_doWarmup 0`, `bot_enable 0` |
| B | после `Default mode "<режим>" enabled.` и второго `InitGame`, через 0,06 с | режим задал `team_allcaptain 1`, `match_readypercent 100`, `sv_allowDownload 1`, `g_allowVote 1`, `g_doWarmup 1` и `timelimit` режима (`tdm` 20, `duel` 15). `g_teamAutoJoin`, `g_inactivity` и свои переменные из `autoexec.cfg` (`server_gameplay`, `warmup`) остаются; `bot_enable` остаётся 0 |

Значения смотрите только в фазе B. [`map_restart`](../glossary.md#map_restart) и смена карты режим заново не загружают: за 15 секунд после них строки загрузки режима нет. Проверено запуском в `tdm` и `duel`. У stock и CPMA фаза одна.

Что задаёт режим, кроме лимитов:

| Переменная | Значение |
|---|---|
| `sv_allowdownload` | 1, во всех шести режимах |
| `sv_floodprotect` | 0 |
| `match_readypercent` | 100 в `duel`, `tdm`, `ctf`, `ca`, `ftag` |
| `g_warmup` | 0 в `duel`, `tdm`, `ctf`, `ftag`; 60 в `ca` |
| `map_rotate` | 1 только в `ffa` |
| `team_maxplayers` | 4 в `tdm`, 5 в `ftag` |

Из `autoexec.cfg` нельзя задать `team_allcaptain`, `match_readypercent`, `g_allowVote`, `g_doWarmup`, `sv_allowDownload`, `timelimit`. После загрузки режима у них значение режима, что бы ни стояло в файле (проверено запуском в `tdm` и `duel`).

Что из заданного в `autoexec.cfg` и через [rcon](../glossary.md#rcon) сохраняется после `map_restart`, смены карты, [рестарта](../glossary.md#рестарт-контейнера) и [пересоздания контейнера](../glossary.md#пересоздание-контейнера), показано в [таблице жизненного цикла OSP](../common/layers.md#osp). Разбор каждой строки, которая не применяется, есть в [layers.md](../common/layers.md#разбор-строк-которые-не-применяются).

## Что принимается, отвергается, игнорируется

| ENV или файл | Как ведёт себя образ OSP |
|---|---|
| `MODE` | принимается: одно из шести, иначе `OSP MODE must be duel, ffa, tdm, ctf, ca or ftag`. `MODE` можно сменить на прежних [томах](../glossary.md#том) |
| `MAP` | принимается: карта из [`/app/maps.txt`](../glossary.md#каталог-карт) с вашим `MODE` |
| `HOSTNAME`, `NET_SERVER_NAME`, `NET_PEER_SERVER`, `NATIVE_UDP`, `PORT` | принимаются так же, как у других образов: [env.md](../common/env.md) |
| `MAXCLIENTS` | только пустой или `14`, иначе `OSP requires MAXCLIENTS=14`. 14 задаёт [`osp.cfg`](../glossary.md#конфиг-мода), сам [ENV](../glossary.md#env) на сервер не передаётся |
| `TIMELIMIT`, `FRAGLIMIT`, `CAPTURELIMIT` | отвергаются: `OSP uses native limits` |
| `PRESET` | только `classic`, иначе `PRESET must be classic for stock` |
| `DOWNLOAD_URL`, `DOWNLOAD_RATE` | отвергаются: `DOWNLOAD_URL/DOWNLOAD_RATE are stock-only` |
| `GAMEPLAY` | молча игнорируется: с `GAMEPLAY=vq3` сервер запустился, в логе об этом ни строки, переменной `server_gameplay` в OSP нет |
| `/data/maplist.txt` | молча игнорируется: сервер стартует на карте `MAP` или карте по умолчанию. Проверено запуском с неверным списком (`nosuchmap`, две карты на строке): сервер запустился, в логе о списке ни строки, OSP грузит свой список `Loading maplist from "cfg-maps/1v1maps.txt"` |
| `autoexec.cfg` в `/opt/q3-data/osp/` | читается. То, что задаёт режим, перекрывается, остальное сохраняется: `g_inactivity` из примера действует и после `Default mode … enabled.` |

В образе OSP часть сообщений об ошибках называет stock или CPMA. Это общий текст, и он относится к OSP тоже. Две такие строки проверены запуском:

```
Conflicting CPMA saved content: /data/home/osp/osp.cfg
PRESET must be classic for stock
```

Первая появляется, если в томе home лежит свой `osp.cfg`, вторая, если `PRESET` не `classic`. Полный список [отказов](../glossary.md#отказ-при-старте) собран в [troubleshooting.md](../troubleshooting.md#отказы-при-старте).

В логе нормального старта OSP есть строки, похожие на ошибки; они перечислены в [troubleshooting.md](../troubleshooting.md#строки-которые-выглядят-как-ошибка-но-нормальны).

## Браузерный вход

| Лобби | Комната видна | Вход |
|---|---|---|
| стенд разработчиков DosZone ([рецепт](../recipes/stand-to-dozone.md)) | да, карточка «OSP 1.03A · FFA» | **проверен запуском**: игрок вошёл (`ClientBegin`), клиент OSP загрузил свой игровой интерфейс (HUD) |
| dos.zone | да, но карточка подписана «ioq3 FFA» | **не проверялся** |

Лобби dos.zone показывает комнату OSP как обычную ioquake3. По коду лобби кнопка Join запускает обычный браузерный клиент ioquake3, а не клиент OSP. Не обещайте игрокам браузерный вход в OSP через dos.zone.

Когда браузерный игрок выходит кнопкой Exit, его слот остаётся занят около 197 секунд, пока сервер не отключит его по `timed out`.

## Обычный клиент

[Обычному клиенту](../glossary.md#обычный-клиент) нужны все 14 [PK3](../glossary.md#pk3) образа, 5 из `/opt/q3-data/baseq3/` и 9 из `/opt/q3-data/osp/`, все в одном каталоге `~/.q3a/osp/`. Как скопировать файлы и подключиться (клиент обязательно запускается с `fs_game osp`), описано в рецепте [«Файлы игры»](../recipes/game-files.md#что-положить).

## Переменные OSP

[Переменные сервера](../glossary.md#переменная-сервера) OSP описаны в документации авторов: файл `osp/Docs/osp-q3-Variables.txt` (OSP 1.03, 23 февраля 2003) в архиве [osp-Quake3-1.03a_full.zip](https://osp.dget.cc/orangesmoothie/downloads/osp-Quake3-1.03a_full.zip). Из этого архива в образе пять PK3 (`zz-osp-pak0`…`pak3`, `zz-osp-server3a`), а [игровой модуль](../glossary.md#игровой-модуль) взят из выпуска [1.03a-secured](https://github.com/JKornev/xq3osp/releases/tag/1.03a-secured). Там же лежат `osp-q3-Commands.txt` (команды, судья) и `osp-q3-ModeInfo.txt` (режимы). В образ эти файлы не входят.

Документация написана для исходного OSP 1.03a. Какие переменные есть на вашем сервере и какие у них значения, покажет `cvarlist` в [консоли](../recipes/console.md). Значения, которые задаёт образ, перечислены в [pinned.md](../common/pinned.md).

Свой `.cfg` для опытов можно положить и выполнить по рецепту [«Свои конфиги для опытов»](../recipes/own-configs.md).

## Чего образ не позволяет

| Что | Почему |
|---|---|
| ротация из своего списка карт | `maplist.txt` не читается; `map_rotate 1` режим ставит только в `ffa` |
| больше или меньше 14 игроков | entrypoint принимает только 14, число задаёт `osp.cfg` образа |
| свои лимиты матча | entrypoint отказывает на `TIMELIMIT`, `FRAGLIMIT`, `CAPTURELIMIT`: лимиты задаёт [режим OSP](../glossary.md#режим-мода) |
| несколько режимов на одном сервере | entrypoint ставит в `server_availmodes` только выбранный режим (проверено запуском значение маски; смену режима не пробовали) |
| пресеты, [физика](../glossary.md#физика-игры) CPM | только `classic`; `GAMEPLAY` не читается |
| `team_allcaptain`, `match_readypercent`, `g_allowVote`, `g_doWarmup`, `sv_allowDownload`, `timelimit` из `autoexec.cfg` | режим OSP задаёт их при загрузке и перекрывает файл. rcon после `Default mode … enabled.` меняет их до рестарта |
| выключить голосование и разминку постоянно | entrypoint `g_allowVote` не задаёт, а режим включает голосование и разминку при каждом старте |
| свои настройки режима | файлы `osp.cfg`, `dz-osp.cfg`, PK3 в `/data/home` останавливают запуск (`Conflicting CPMA saved content`), изменённый файл в `/opt/q3-data` тоже (`CPMA packaged data hash verification failed`) |
| свои карты | сервер играет только карты из `/app/maps.txt` |
| браузерный вход через dos.zone | лобби dos.zone показывает комнату OSP как обычную ioquake3 |
