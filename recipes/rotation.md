# Ротация карт

[Мод](../glossary.md#мод): [stock](../glossary.md#stock), [CPMA](../glossary.md#cpma). OSP `maplist.txt` не читает, см. [ограничения OSP](../osp/README.md#чего-образ-не-позволяет).

Список карт лежит в `maplist.txt` рядом с `compose.yaml`, по одной карте на строку. Список уже [смонтирован](../glossary.md#смонтированный-файл) в примерах [stock](../examples/stock/) и [CPMA](../examples/cpma/). С карты из первой строки сервер стартует. `MAP` в [ENV](../glossary.md#env) не задавайте: если он отличается от первой строки, сервер не стартует.

## stock

Проверено запуском на `20261112-d608e3a-stock`.

1. Впишите карты в `maplist.txt`:

   ```
   q3dm17
   q3dm6
   ```

2. Примените: `docker compose up -d --force-recreate q3ded`.

[Entrypoint](../glossary.md#entrypoint) пишет из списка `/data/home/demoq3/rotation.cfg` и стартует с `+exec rotation.cfg +vstr rotate1`:

```
set rotate1 "map q3dm17 ; set nextmap vstr rotate2"
set rotate2 "map q3dm6 ; set nextmap vstr rotate1"
```

Результат: матч кончается по лимиту, после перерыва (около 16 секунд) грузится следующая карта, а после последней снова первая.

```
Exit: Timelimit hit.
Server: q3dm6
```

| Ситуация | Что происходит |
|---|---|
| игроков нет | ротация идёт по `TIMELIMIT` |
| игрок на сервере | перерыв после матча ждёт, пока игрок нажмёт атаку |
| [`docker compose restart`](../glossary.md#рестарт-контейнера) | сервер снова начинает с первой карты |
| одна карта в списке | `rotation.cfg` не выполняется, по окончании матча карта перезапускается (`nextmap` = `map_restart 0`) |
| ручная правка `rotation.cfg` | при списке длиннее одной карты теряется: файл переписывается при каждом старте; при одной карте старый файл остаётся, но не выполняется |
| [rcon](../glossary.md#rcon) `timelimit 1` | действует и на следующих картах, теряется при рестарте |

Для stock подходят карты из [`/app/maps.txt`](../glossary.md#каталог-карт) или из [своих PK3](stock-custom-maps.md). Для `MODE=ctf` все карты должны быть картами для CTF, иначе [отказ](../glossary.md#отказ-при-старте); для остальных режимов stock не проверяет, подходит ли карта к режиму.

## CPMA

Проверено запуском на `20261112-c64186e-cpma`.

Каждая карта списка должна подходить к `MODE` по каталогу `/app/maps.txt`. Иначе отказ, например:

```
maplist.txt map cpm1a must be a ffa map in the packaged catalog (see maps.txt)
```

Ротацию ведёт сам CPMA. Entrypoint переписывает из `maplist.txt` список выбранного режима при каждом старте, например `/data/home/cpma/cfg-maps/ffamaps.txt`, и мод пишет в лог:

```
Loading maplist from cfg-maps/ffamaps.txt
```

Ручная правка этого списка теряется при старте. Меняйте `maplist.txt` и применяйте правку: `docker compose up -d --force-recreate q3ded`.

Без игроков через 10 минут после последней загрузки карты CPMA сбрасывает режим и возвращается к первой карте списка:

```
Idle for 10 minutes: resetting back to defaults
Server: cpm4
```

Результат в `ffa` со списком `cpm4`, `cpm15`: матч кончился по лимиту, после перерыва загрузилась вторая карта списка.

```
Exit: Timelimit hit in arena 0
Server: cpm15
Loading maplist from cfg-maps/ffamaps.txt
```

| Ситуация | Что происходит |
|---|---|
| лимит матча | задаёт [режим CPMA](../glossary.md#режим-мода); `timelimit` в [`autoexec.cfg`](../glossary.md#autoexeccfg) и rcon не действуют. На время карты лимит меняет [судья](../cpma/README.md#лимиты-матча-и-судья) |
| ничья в `ffa` | по умолчанию включён овертайм «внезапная смерть», и по времени матч не кончается |
| игроки на сервере | перерыв после матча сам не кончился за 2 минуты; карта сменилась через 2 секунды после того, как атаку нажал второй из двух игроков. Переход без игроков не проверялся |
| `docker compose restart` | сервер снова начинает с первой карты: `Server: cpm4` |
| ручная правка `cfg-maps/ffamaps.txt` | теряется при старте |

## Отказы

Все отказы происходят до запуска [агента комнаты](../glossary.md#агент-комнаты): контейнер останавливается с кодом 1, комнаты нет. Полный список отказов собран в [troubleshooting.md](../troubleshooting.md#отказы-при-старте).

| Ошибка в списке | Строка лога |
|---|---|
| только комментарии и пустые строки | `maplist.txt lists no maps` |
| две карты на строке | `maplist.txt: one map per line: q3dm17 q3dm6` |
| нет такой карты, stock | `maplist.txt map nosuchmap is neither a packaged map (see maps.txt) nor in a PK3 in /data/custom/demoq3` |
| нет такой карты, CPMA | `maplist.txt map nosuchmap must be a packaged map (see maps.txt)` |
| `MAP` не равен первой строке | `MAP=q3dm6 differs from the first map of maplist.txt (q3dm17): remove MAP or make them equal` |
