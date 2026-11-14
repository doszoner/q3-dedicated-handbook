# Физика VQ3 в CPMA

[Мод](../glossary.md#мод): [CPMA](../glossary.md#cpma), `20261112-c64186e-cpma`. Проверено запуском в `MODE=ffa`: сервер с `GAMEPLAY=vq3` запускается, и на нём можно играть (результаты ниже), по умолчанию используется физика `CPM`, строка `set server_gameplay VQ3` в [`autoexec.cfg`](../glossary.md#autoexeccfg) не перекрывает [ENV](../glossary.md#env), а `GAMEPLAY=foo` вызывает [отказ](../glossary.md#отказ-при-старте).

Физику задаёт ENV `GAMEPLAY`, а не строка в `autoexec.cfg`: [entrypoint](../glossary.md#entrypoint) ставит `server_gameplay` через `+set` в [командной строке сервера](../common/env.md#как-значение-env-попадает-в-игру), и это перекрывает `autoexec.cfg`.

## Шаги

1. В `compose.yaml`, в `environment:`:

   ```yaml
   GAMEPLAY: "vq3"
   ```

2. Примените: `docker compose up -d --force-recreate q3ded`. [`docker compose restart`](../glossary.md#рестарт-контейнера) изменённый ENV не применяет.

## Результат

| Где | Что видно |
|---|---|
| лог | `Starting ioq3ded: … gameplay=VQ3 …`; по умолчанию там `gameplay=CPM` |
| ответ на запрос [`getstatus`](../glossary.md#getstatus) | `server_gameplay VQ3`; по умолчанию там `server_gameplay CPM` |
| после команд [`map_restart`](../glossary.md#map_restart) и `map cpm15`, отправленных через [rcon](../glossary.md#rcon) | по-прежнему `server_gameplay VQ3` |

В игре, на карте `cpm4` с одним игроком и надписью `Waiting for Players` на экране, физики различаются так:

| Что | CPM | VQ3 |
|---|---|---|
| здоровье сразу после появления | ровно `100` | больше 100 и убывает: на проверке `118`, затем `113`, `110`, `109` |
| патроны после появления | `50/10/5/10/50/5/50/5` | `100/10/10/10/100/10/50/20` |
| смена оружия, приблизительно, по кадрам записи экрана | около 0,2 с | около 0,6 с |

Быстрее всего проверить по числу здоровья на одном снимке экрана сразу после входа. Таблица счёта (Tab) физику не показывает.

Какие значения `GAMEPLAY` допустимы, какое из них действует по умолчанию и как переменную воспринимают [stock](../glossary.md#stock) и [OSP](../glossary.md#osp), описано в [карточке переменной](../common/env.md#gameplay).
