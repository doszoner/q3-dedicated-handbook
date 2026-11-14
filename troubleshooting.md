# Сервер не работает: где смотреть

Проверено на `20261112-d608e3a-stock`, `20261112-c64186e-cpma`, `20261114-9e899840-osp`.

## 1. Состояние и лог

```sh
docker compose ps -a                       # running или Exited (<код>)
docker compose logs --tail 100 q3ded    # последние строки лога
```

| Что видно | Что значит | Куда дальше |
|---|---|---|
| `Exited (1)`, последняя строка лога: фраза на английском | [entrypoint](glossary.md#entrypoint) остановился, не запустив сервер, [комнаты](glossary.md#комната) нет | [таблица отказов](#отказы-при-старте) |
| `Exited (3)` | [движок](glossary.md#ioq3ded) не смог занять порт или на [stock](glossary.md#stock) отверг вашу собственную карту | [занятый порт](common/network.md#занятый-порт), [свои карты stock](recipes/stock-custom-maps.md) |
| `running`, но в логе нет `registration confirmed` | сервер не зарегистрировался на [peer-сервере](glossary.md#peer-сервер) | [сеть](common/network.md) |
| `registration confirmed` есть, но комнаты нет в [лобби](glossary.md#лобби) | регистрация на peer-сервере прошла, но публикация комнаты [агентом](glossary.md#агент-комнаты) не подтверждена; при `RUN_ROOM_AGENT=0` комнаты не будет | [сеть](common/network.md#состояния-сервера-и-их-признаки), [`RUN_ROOM_AGENT`](common/env.md#run_room_agent), [`NET_SERVER_NAME`](common/env.md#net_server_name) |
| комната есть, но [обычный клиент](glossary.md#обычный-клиент) сервер не видит | [UDP](glossary.md#udp)-пакеты не доходят до сервера | [обычный клиент не видит сервер](common/network.md#обычный-клиент-не-видит-сервер) |
| сервер работает, но строка из [`autoexec.cfg`](glossary.md#autoexeccfg) не действует | значение перекрыто другой настройкой или файл не читается | [почему строка не применилась](#почему-строка-не-применилась) |

Признаки каждого состояния сервера собраны в [common/network.md](common/network.md#состояния-сервера-и-их-признаки).

## 2. Консоль сервера

Как подключиться к консоли и отключиться от неё, не остановив сервер, описано в рецепте [консоль сервера](recipes/console.md). В консоли и через [rcon](glossary.md#rcon) полезны такие команды:

| Команда | Что показывает |
|---|---|
| `status` | карта, игроки, их адреса и пинг |
| `serverinfo` | переменные, которые сервер сообщает клиентам |
| `<переменная>` | текущее значение, например `"g_inactivity" is:"0^7", the default` |
| `cvarlist <префикс>` | список переменных с их флагами. Флаг `A` значит [archive](glossary.md#archive-переменная) |

Если в ответ на имя переменной сервер пишет `broadcast: print "server: <имя>"`, такой переменной на сервере нет: выделенный сервер превращает неизвестную команду в сообщение чата.

Как подавать эти команды через rcon с обычного клиента, описано в рецепте [rcon](recipes/rcon.md).

## Почему строка не применилась

Примеры строк, которые не применяются, разобраны в [layers.md](common/layers.md#разбор-строк-которые-не-применяются). Для любой другой строки проверьте по порядку:

1. **Узнать значение.** В консоли или через rcon введите имя переменной. Ответ `server: <имя>` значит, что такой переменной в этом [моде](glossary.md#мод) нет.
2. **Проверить, не задаёт ли её entrypoint.** Найдите имя в [pinned.md](common/pinned.md). Эти значения entrypoint задаёт после `autoexec.cfg`, поэтому они всегда побеждают.
3. **Найти, какой файл читается.** Если в `/data/home/<мод>/` лежит свой `autoexec.cfg`, то файл, [смонтированный](glossary.md#смонтированный-файл) в `/opt/q3-data/<мод>/`, не читается вовсе. В каком порядке сервер ищет этот файл, сказано в [layers.md](common/layers.md#какой-файл-читается).
4. **Проверить после загрузки карты.** [CPMA](glossary.md#cpma) и [OSP](glossary.md#osp) задают свои значения, когда на карте загружается режим. У OSP смотрите значение только после того, как в логе появились строка `Default mode "<режим>" enabled.` и следующий за ней [`InitGame`](glossary.md#initgame). До этой строки, около 1,8 с, ещё действуют значения `autoexec.cfg`.

## Отказы при старте

Каждую строку из таблиц ниже entrypoint печатает последней строкой лога и выходит с кодом 1, не запустив сервер. Почти все эти строки появляются ещё до запуска агента комнаты. Две строки, `rtc.json not ready after <N>s` и `dedicated_room_agent exited before rtc.json was ready`, появляются позже: entrypoint уже запустил агент и ждёт от него [`rtc.json`](glossary.md#rtcjson), а при отказе останавливает агент. Текст строк приведён дословно, на месте `<…>` будет ваше значение. В [образе](glossary.md#образ) OSP часть сообщений называет stock или CPMA: это общий для образов текст, и он относится и к OSP.

Строки, отмеченные ✓, проверены запуском. Для остальных строк текст отказа, причина и рекомендуемое действие приведены по коду entrypoint. Запуском они не проверялись.

### Комната, агент, rtc.json

| Строка лога | Моды | Причина | Что делать |
|---|---|---|---|
| ✓ `RUN_ROOM_AGENT=0 requires a non-empty rtc.json in /data/run` | все | `RUN_ROOM_AGENT=0`, а в [томе](glossary.md#том) `/data/run` нет готового `rtc.json` | вернуть `RUN_ROOM_AGENT=1`; см. [env.md](common/env.md#run_room_agent) |
| ✓ `rtc.json not ready after 5s` (число в строке равно вашему `RTC_WAIT_SECS`; на stock получено при `RTC_WAIT_SECS=5`) | все | агент не записал `rtc.json` за `RTC_WAIT_SECS` секунд (по умолчанию 30); на stock получено с недоступным `CLOUD_BASE_URL` | проверить исходящий доступ к адресу публикации из [`CLOUD_BASE_URL`](common/env.md#cloud_base_url), если переменная не задана, это `https://cloud.js-dos.com`; см. [сеть](common/network.md) |
| ✓ `dedicated_room_agent exited before rtc.json was ready` | все | агент завершился до того, как файл был готов; на stock строка получена, когда агент завершился во время ожидания. При запуске с `NET_PEER_SERVER=wss://127.0.0.1:1` агент не завершился, и ioq3ded запустился | смотреть строки агента выше в логе |
| `RUN_ROOM_AGENT must be 0 or 1` | все | задано другое значение | `0` или `1` |
| `dedicated_room_agent is not executable`, `dedicated_room_agent.toml is not readable` | все | повреждён каталог `/app` | взять образ заново по [digest](README.md#образы) |

### Каталоги и файлы

| Строка лога | Моды | Причина | Что делать |
|---|---|---|---|
| ✓ `RUN_DIR and HOME_DIR must be writable` | все | `/data/run` или `/data/home` смонтирован только для чтения (`:ro`) или не принадлежит uid 65532 | именованный том или каталог с владельцем 65532; см. [container.md](common/container.md#каталоги) |
| `Required directory missing: /data/run` (или другой каталог) | все | каталога нет в [контейнере](glossary.md#контейнер) | не переопределять `Q3_RUN_DIR`, `Q3_HOME_DIR`, `Q3_DATA_DIR` |
| ✓ `Conflicting CPMA saved content: /data/home/cpma/modes/<файл>`; у OSP, например, `Conflicting CPMA saved content: /data/home/osp/osp.cfg` | CPMA, OSP | в `/data/home` лежит файл, который мог бы скрыть файл образа | удалить ровно названный файл: [рецепт](recipes/edit-env-and-files.md#conflicting-cpma-saved-content) |
| ✓ `CPMA packaged data hash verification failed` | CPMA, OSP | файл в `/opt/q3-data` не совпадает с [манифестом образа](glossary.md#манифест-образа) | убрать монтирование, которое закрывает файлы образа |
| ✓ `conflicting /data/home/demoq3/vm/qagame.qvm: saved-home game modules are not supported; remove it to use the packaged module` | stock | в томе home лежит свой `qagame.qvm` | удалить этот файл |
| ✓ `/data/maplist.txt is not a file: is maplist.txt missing next to compose.yaml?` | stock, CPMA | на [хосте](glossary.md#хост) нет `maplist.txt`, и Docker создал на его месте каталог | [container.md](common/container.md#bind-монтирование-отсутствующего-файла) |
| `Custom directory missing or unreadable: /data/custom/demoq3` | stock | `Q3_CUSTOM_DIR` указывает не туда | вернуть значение по умолчанию или смонтировать каталог |
| `ioq3ded.x86_64 is not executable in /app`, `runtime-manifest.txt missing or lacks browser_version`, `packaged map catalog missing: /app/maps.txt` | все | повреждён каталог `/app` или переопределён `Q3_APP_DIR` | не задавать `Q3_APP_DIR`; взять образ заново |
| `RELEASE_PROFILE conflicts with package`, `Unknown packaged profile: <имя>`, `CPMA data requires a CPMA package profile` | все / stock | профиль образа менять нельзя | убрать `RELEASE_PROFILE` |

### Режим, пресет, физика

| Строка лога | Моды | Что делать |
|---|---|---|
| ✓ `MODE must be ffa, duel, tdm or ctf` | stock | одно из четырёх значений |
| `MODE must be duel, ffa, tdm, ctf, ca, ftag or ctfs` | CPMA | одно из семи значений |
| ✓ `OSP MODE must be duel, ffa, tdm, ctf, ca or ftag` | OSP | одно из шести значений |
| ✓ `PRESET must be classic for stock` | stock, OSP | убрать [`PRESET`](glossary.md#preset) или задать `classic` |
| ✓ `PRESET must be classic, instagib (MODE=ffa\|tdm\|ctf\|ftag) or rocket-only\|bfg-only\|gauntlet-only (MODE=ffa)` | CPMA | см. [таблицу сочетаний CPMA](cpma/README.md#сочетания) |
| `packaged preset mode missing: cpma/modes/<имя>.cfg` | CPMA | образ повреждён, взять его заново |
| ✓ `GAMEPLAY is CPMA-only` | stock | убрать `GAMEPLAY` |
| ✓ `GAMEPLAY must be cpm or vq3` | CPMA | `cpm` или `vq3` |

### Карты и maplist.txt

| Строка лога | Моды | Что делать |
|---|---|---|
| ✓ `maplist.txt lists no maps` | stock, CPMA | указать хотя бы одну карту; `#` отмечает комментарий |
| ✓ `maplist.txt: one map per line: q3dm17 q3dm6` | stock, CPMA | писать одну карту на строку |
| ✓ `MAP=q3dm6 differs from the first map of maplist.txt (q3dm17): remove MAP or make them equal` | stock, CPMA | убрать `MAP` |
| ✓ `maplist.txt map nosuchmap is neither a packaged map (see maps.txt) nor in a PK3 in /data/custom/demoq3` | stock | карта из [`/app/maps.txt`](glossary.md#каталог-карт) или [PK3](glossary.md#pk3) с картой в `/data/custom/demoq3` |
| ✓ `MODE=ctf requires a CTF-compatible packaged map: 13vast Facer2d q3ctf1 …` | stock | карта, у которой в `/app/maps.txt` указан `ctf` |
| ✓ `maplist.txt map nosuchmap must be a packaged map (see maps.txt)` | CPMA | только карты из `/app/maps.txt` |
| ✓ `maplist.txt map cpm1a must be a ffa map in the packaged catalog (see maps.txt)` | CPMA | карта, у которой в `/app/maps.txt` есть ваш `MODE` |
| ✓ `MAP must be a packaged map (see maps.txt)`, `MAP must be a duel map in the packaged catalog (see maps.txt)` (на месте режима будет ваш `MODE`: `ffa`, `tdm`…) | OSP | карта из `/app/maps.txt` с вашим `MODE` |
| `MAP must match [A-Za-z0-9_-]{1,64}` | все | латиница, цифры, `_` и `-` |

Stock проверяет, подходит ли карта к режиму, только при `MODE=ctf`: например, `MODE=duel` с картой `q3ctf1` запускается.

Если у CPMA не задан `MODE`, режим будет `duel`, и карта не для дуэли даёт отказ: `maplist.txt map cpm4 must be a duel map in the packaged catalog (see maps.txt)`.

### Числа и лимиты

| Строка лога | Моды | Что делать |
|---|---|---|
| ✓ `FRAGLIMIT must be a non-negative integer` | stock | целое число; отказ получен на числе из 11 цифр |
| ✓ `CAPTURELIMIT must be a non-negative integer` | stock, CPMA (`duel`) | целое число; отказ получен на stock (число из 11 цифр) и на CPMA (`abc`) |
| `MAXCLIENTS must be a non-negative integer` | все | целое число; то же сообщение выдаётся для `PORT`, `RTC_WAIT_SECS`, а на stock ещё для `TIMELIMIT`, `DOWNLOAD_RATE` |
| ✓ `TIMELIMIT must be <= 35791` | stock | не больше 35791 |
| ✓ `FRAGLIMIT must be <= 2147483647` | stock | не больше 2147483647 |
| `CAPTURELIMIT must be <= 2147483647` | stock, CPMA (`duel`) | не больше 2147483647 |
| ✓ `RTC_WAIT_SECS must be <= 30` | все | не больше 30; отказ получен на stock |
| ✓ `MAXCLIENTS must be <= 26` | все | не больше 26 |
| `MAXCLIENTS must be 2-26` | stock, CPMA | 2–26 |
| `PORT must be 1-65535` | все | 1–65535 |
| ✓ `OSP requires MAXCLIENTS=14` | OSP | убрать `MAXCLIENTS` или задать `14` |
| ✓ `OSP uses native limits` | OSP | убрать `TIMELIMIT`, `FRAGLIMIT`, `CAPTURELIMIT` |
| ✓ `CPMA duel allows only MAXCLIENTS=20, TIMELIMIT=3, FRAGLIMIT=3` | CPMA | убрать эти [ENV](glossary.md#env) |
| ✓ `CPMA ffa uses native mode limits; TIMELIMIT/FRAGLIMIT/CAPTURELIMIT overrides are not supported` | CPMA | убрать лимиты; см. [сочетания CPMA](cpma/README.md#сочетания) |

### Сеть и адреса

| Строка лога | Моды | Что делать |
|---|---|---|
| `CPMA requires the approved native NET_PEER_SERVER` | CPMA | задать `NET_PEER_SERVER: wss://net.js-dos.com:444` |
| `NET_PEER_SERVER must be empty or ws(s)://host[:port]` | все | `wss://host[:port]` без пути |
| `CLOUD_BASE_URL must be empty or http(s)://host[:port]` | все | для dos.zone убрать эту переменную |
| `NET_SERVER_NAME is required: set a unique room.q3ded.<suffix> alias` | все | задать свой [alias](glossary.md#alias) |
| `NET_SERVER_NAME must match room.q3ded.<1+ chars of A-Za-z0-9._->, <=117 chars total` | все | латиница, цифры, `. _ -`, не длиннее 117 |
| `NET_SERVER_NAME must have a non-empty suffix after room.q3ded.` | все | дописать суффикс после `room.q3ded.` |
| ✓ `NATIVE_UDP must be 0 or 1`, `PUBLISH_Q3_MASTER must be 0 or 1` | все | `0` или `1`; на stock отказ получен с пустым значением |
| `NATIVE_IPV6 must be 0 or 1` | все | `0` или `1`; пустое значение тоже даёт отказ |
| `NATIVE_IPV6=1 requires NATIVE_UDP=1`, `PUBLISH_Q3_MASTER=1 requires NATIVE_UDP=1` | все | включить `NATIVE_UDP` |
| ✓ `NATIVE_BIND must be a local IPv4 literal` | все | IPv4-адрес или убрать |
| `NATIVE_BIND6 must be a local IPv6 literal with optional scope, not IPv4-mapped` | все | IPv6-адрес или убрать |
| ✓ `Q3_MASTERS accepts at most five endpoints`, `Q3_MASTERS must not be empty when publication is enabled` | все | 1–5 адресов `host[:port]`; отказы получены на stock |
| `Invalid Q3_MASTERS endpoint (use host[:port] or [IPv6][:port])` | все | `host[:port]` или `[IPv6][:port]` |
| ✓ `DOWNLOAD_URL/DOWNLOAD_RATE are stock-only` | CPMA, OSP | убрать |
| `DOWNLOAD_RATE must be 1-100000 KiB/s` | stock | 1–100000 |
| `DOWNLOAD_URL must be an HTTPS base without credentials, query, fragment or unsafe path`, `DOWNLOAD_URL contains dot path segments`, `DOWNLOAD_URL port must be 1-65535` | stock | `https://host[:port]/путь` |

### Длина значений и командной строки

| Строка лога | Моды | Что делать |
|---|---|---|
| `Invalid value for HOSTNAME: use at most 509 bytes, without quotes or line breaks` | все | без `"` и переводов строки, до 509 байт; то же ограничение действует для любого ENV, который entrypoint передаёт серверу через `+set` |
| `Native dedicated startup command exceeds engine limits`, `Native dedicated startup arguments exceed engine limits` | все | укоротить `HOSTNAME`, `Q3_MASTERS` |
| `OSP startup exceeds 31 commands` | OSP | ENV на это не влияют; сообщите об ошибке через [issue](CONTRIBUTING.md) |

## Строки, которые выглядят как ошибка, но нормальны

| Строка | Где |
|---|---|
| `ERROR: Could not open player filter file "filters.txt"` | CPMA, каждый старт |
| `WARNING: baseq3/pak0.pk3 is present but its checksum (927214862) is not correct` | OSP |
| `ERROR: Could not open maps list file "maps.txt"` | OSP |
| `ERROR: Could not open decal file "decals/decal-<карта>.txt"` | OSP |
| `can't stat status file "./status.json" … considering server dead`, затем `status file is fresh again — server recovered` | агент комнаты, при старте |
| `couldn't exec q3config_server.cfg`, `couldn't exec autoexec.cfg` | первый старт на новых томах или без `autoexec.cfg` |
| `sv_maxclients will be changed upon restarting.` | OSP, после `execing osp.cfg` |
