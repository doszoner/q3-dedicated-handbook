# Зашитые значения

Проверено запуском на `20261112-d608e3a-stock`, `20261112-c64186e-cpma`, `20261114-9e899840-osp`.

[Entrypoint](../glossary.md#entrypoint) передаёт [движку](../glossary.md#ioq3ded) эти переменные через `+set` в [командной строке](env.md#как-значение-env-попадает-в-игру). Движок применяет `+set` после `default.cfg`, [`q3config_server.cfg`](../glossary.md#q3config_servercfg) и [`autoexec.cfg`](../glossary.md#autoexeccfg), поэтому строка с той же переменной в `autoexec.cfg` проигрывает. Порядок целиком описан в [layers.md](layers.md).

В колонке «Как изменить» пометка «ENV» значит, что значение берётся из [переменных окружения контейнера](env.md) и меняется правкой `compose.yaml` и командой `docker compose up -d`, а «нельзя» значит, что значение зашито и ENV для него нет.

Значение, заданное через [rcon](../glossary.md#rcon) поверх зашитого, держится после [`map_restart`](../glossary.md#map_restart) и смены карты и теряется при [рестарте контейнера](../glossary.md#рестарт-контейнера), потому что entrypoint снова задаёт своё. То же происходит со значениями, которые вы задали через rcon поверх значений из [`cpma.cfg`](../glossary.md#конфиг-мода) у CPMA или после загрузки режима поверх его значений у OSP. Проверенные примеры для каждого мода собраны в таблицах [жизненного цикла](layers.md#жизненный-цикл-настройки).

Переменные, которые сервер сообщает клиентам, видны в [`getstatus`](../glossary.md#getstatus) или `serverinfo`. Остальные проверяются командой `<переменная>` в [консоли](../glossary.md#консоль-сервера) или rcon.

## Игра и правила

| Переменная | stock | CPMA | OSP | Как изменить | Проверка |
|---|---|---|---|---|---|
| [`sv_pure`](../glossary.md#sv_pure) | 1 | 0 | 1 | нельзя | rcon `sv_pure` |
| `sv_allowDownload` | 1 (entrypoint задаёт 0, затем 1; действует 1) | 0 | 0 на старте, [режим OSP](../glossary.md#режим-мода) ставит 1 | нельзя | `getstatus`: `sv_allowDownload` |
| `sv_fps` | 20 | 20 | 20 | нельзя | rcon `sv_fps` |
| `bot_enable` | 0 | 0 | 0 | нельзя | rcon `bot_enable`; [stock](../glossary.md#stock) и CPMA: rcon `addbot` молча ничего не делает |
| `g_allowVote` | 0 | 0 (`cpma.cfg`) | entrypoint не задаёт; после загрузки режима 1, голосование включено | stock, CPMA: нельзя; OSP: выключить можно через rcon, до рестарта | голосование: `Voting not allowed here.` |
| `g_doWarmup` | 0 | 0, но CPMA его не читает: [разминку](../glossary.md#разминка) задаёт параметр режима `warmup` | 0, но режим OSP ставит 1: разминка включена | насовсем нельзя; stock: rcon `g_doWarmup 1` до рестарта; OSP: rcon после загрузки режима, до рестарта; CPMA: [голосование](layers.md#разбор-строк-которые-не-применяются) | rcon `g_doWarmup`; ответ на rcon `map` содержит `Warmup:`, если разминка включена |
| `g_gametype` | ffa 0, duel 1, tdm 3, ctf 4 | duel 1, ffa 0, tdm 3, ctf 4, ca 5, ftag 6, ctfs 7 | duel 1, ffa 0, tdm 3, ctf 4, ca 5, ftag 3 | ENV `MODE` | `getstatus`: `g_gametype` |
| `sv_maxclients` | ENV `MAXCLIENTS` | ENV `MAXCLIENTS` | 14 из `osp.cfg` | ENV `MAXCLIENTS` (OSP: только 14) | `getstatus`: `sv_maxclients` |
| `timelimit`, `fraglimit` | ENV `TIMELIMIT`, `FRAGLIMIT` | только duel: 3 и 3. CPMA эти [переменные движка](../glossary.md#переменная-сервера) не читает, лимиты берёт из режима | не задаются; задаёт режим OSP | stock: ENV; CPMA: [судья](../glossary.md#судья) на время карты, см. [CPMA](../cpma/README.md#лимиты-матча-и-судья) | `getstatus` (у CPMA не отражает лимиты матча) |
| `capturelimit` | только ctf: ENV `CAPTURELIMIT` | не задаётся | не задаётся | stock ctf: ENV | `getstatus` |
| `g_teamAutoJoin`, `g_teamForceBalance` | tdm и ctf: 1 и 1 | не задаётся | не задаётся | нельзя | игрок попадает в команду сам |
| `sv_hostname` | ENV `HOSTNAME` | то же | то же | ENV | `getstatus`: `sv_hostname` |

## Каталоги и модули

| Переменная | stock | CPMA | OSP | Как изменить |
|---|---|---|---|---|
| [`fs_basepath`](../glossary.md#fs_basepath) | `/opt/q3-data` | то же | то же | нельзя; `Q3_DATA_DIR` только для отладки |
| [`fs_homepath`](../glossary.md#fs_homepath) | `/data/home` | то же | то же | нельзя; `Q3_HOME_DIR` только для отладки |
| [`fs_game`](../glossary.md#каталог-игры) | `demoq3` | `cpma` | `osp` | нельзя |
| `fs_custompath` | `/data/custom/demoq3` | не задаётся | не задаётся | ENV `Q3_CUSTOM_DIR` |
| `dedicated` | 2 | 2 | 2 | нельзя |
| `vm_game` | 2 | 2 | 2 | нельзя |
| `vm_cgame`, `vm_ui` | не задаётся | 2 | не задаётся | нельзя |

## Сеть

| Переменная | Значение (все моды) | Как изменить |
|---|---|---|
| `net_nativeUDP` | ENV `NATIVE_UDP` | ENV |
| `net_enabled` | 1, при `NATIVE_IPV6=1` значение 3 | ENV `NATIVE_IPV6` |
| `net_ip`, `net_ip6` | ENV `NATIVE_BIND`, `NATIVE_BIND6` | ENV |
| `net_port`, `net_port6` | ENV `PORT` | ENV |
| `net_socksEnabled` | 0 | нельзя |
| `sv_masterPublish` | ENV `PUBLISH_Q3_MASTER` | ENV |
| `sv_master1` … `sv_master5` | адреса из ENV `Q3_MASTERS`, оставшиеся переменные пусты | ENV |
| `net_peer_server` | ENV `NET_PEER_SERVER` | ENV |
| `net_server_name` | ENV `NET_SERVER_NAME` | ENV |
| `sv_dlURL`, `sv_dlRate` | stock: ENV `DOWNLOAD_URL`, `DOWNLOAD_RATE` | ENV |

## CPMA и OSP: режим

| Переменная | CPMA | OSP | Как изменить |
|---|---|---|---|
| `mode_start` | по `MODE` и [`PRESET`](../glossary.md#preset): `1v1`, `FFA`, `TDM`, `CTF`, `CA`, `FT`, `CTFS` или `DZIFFA`, `DZRFFA`, `DZBFFA`, `DZGFFA`, `DZITDM`, `DZICTF`, `DZIFT` | по `MODE`: duel 1, ffa 0, tdm 3, ctf 4, ca 5, ftag 6 | ENV `MODE`, `PRESET` |
| `server_gameplay` | `CPM` или `VQ3` | не задаётся | ENV `GAMEPLAY` |
| `map_cfgdir` | duel `cfg-pilot`, tdm `cfg-tdm`, остальные `cfg-maps` | не задаётся | ENV `MODE` |
| `map_rotate` | 1, если в `maplist.txt` больше одной карты, иначе 0 | задаёт режим: ffa 1, остальные 0 | CPMA: `maplist.txt` |
| `server_availmodes` | не задаётся | по `MODE`: duel 2, ffa 1, tdm 8, ctf 16, ca 32, ftag 64 | ENV `MODE` |

## Конфиги, которые entrypoint выполняет после `+set`

| Мод | Команда | Что задаёт файл |
|---|---|---|
| CPMA | `+exec cpma.cfg` | `server_useMapModes 0`, `map_restrict 1`, `g_allowVote 0`, `bot_enable 0`, `bot_minplayers 0`, `sv_fps 20`, `snaps 20`, `cl_maxpackets 125`, `com_maxfps 125`, `sv_pure 0`, `sv_allowDownload 0` |
| OSP | `+exec osp.cfg` | `server_promode 0`, `server_cq3 0`, `sv_maxclients 14`, `bot_minplayers 0`, `sv_pure 1` |
| stock, больше одной карты в `maplist.txt` | `+exec rotation.cfg +vstr rotate1` вместо `+map` | цепочка `rotate1`, `rotate2`, …; см. [stock](../stock/README.md#ротация) |

`cpma.cfg` и `osp.cfg` лежат в `/opt/q3-data/<мод>/` и проверяются по хешу. Свой файл с тем же именем в `/data/home/<мод>/` останавливает запуск, см. [container.md](container.md#каталоги).
