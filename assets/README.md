# Конфиги модов из образов

В этой папке лежат текстовые конфиги [модов](../glossary.md#мод) CPMA и OSP в том виде, в каком их содержит [образ](../glossary.md#образ): байты и окончания строк те же. Файлы даны для чтения, чтобы видеть настройки образа, не открывая [контейнер](../glossary.md#контейнер). Здесь только отдельные текстовые файлы из `/opt/q3-data/`, содержимое [PK3](../glossary.md#pk3) не извлечено. Действующие значения зависят ещё от [ENV](../glossary.md#env), файлов в `/data/home` и режима мода, см. [порядок настроек](../common/layers.md). Свои настройки кладут отдельным файлом по рецепту [«Свои конфиги для опытов»](../recipes/own-configs.md).

| Мод | [Тег](../glossary.md#тег) образа | Путь в контейнере | Папка здесь |
|---|---|---|---|
| CPMA | `20261112-c64186e-cpma` | `/opt/q3-data/cpma/` | [cpma/](cpma/) |
| OSP | `20261114-9e899840-osp` | `/opt/q3-data/osp/` | [osp/](osp/) |
| stock | `20261112-d608e3a-stock` | `/opt/q3-data/baseq3/` | нет: в `/opt/q3-data/` нет отдельных текстовых конфигов вне PK3 |

В `/opt/q3-data/` образа stock, кроме PK3, лежат только [игровые модули](../glossary.md#игровой-модуль) `baseq3/vm/cgame.qvm`, `baseq3/vm/qagame.qvm` и `baseq3/vm/ui.qvm`. PK3 CPMA и OSP сюда тоже не скопированы.

У OSP файлы `filters.txt` и `locs/` совпадают с архивом OSP 1.03a, у CPMA файлы `itemtimers.txt` и `mapmodes.txt` в `cfg-pilot/` и `cfg-tdm/` совпадают с архивом CPMA 1.53. Эти файлы принадлежат авторам модов. Остальные файлы составлены или изменены для образа. Всё здесь приведено для справки.

## Извлечь заново

Команды выполняют из корня справочника. `docker create` создаёт контейнер, не запуская сервер, `docker cp` копирует из него файлы, `docker rm` удаляет контейнер.

CPMA:

```sh
id=$(docker create ghcr.io/doszoner/q3ded:20261112-c64186e-cpma)
mkdir -p assets/cpma
for p in cpma.cfg modes cfg-maps cfg-pilot cfg-tdm; do
  docker cp "$id:/opt/q3-data/cpma/$p" assets/cpma/
done
docker rm "$id"
```

OSP:

```sh
id=$(docker create ghcr.io/doszoner/q3ded:20261114-9e899840-osp)
mkdir -p assets/osp
for p in osp.cfg filters.txt cfg-maps locs; do
  docker cp "$id:/opt/q3-data/osp/$p" assets/osp/
done
docker rm "$id"
```

stock: показать содержимое `/opt/q3-data/`, исключив имена PK3:

```sh
id=$(docker create ghcr.io/doszoner/q3ded:20261112-d608e3a-stock)
docker export "$id" | tar -t | grep '^opt/q3-data/' | grep -v '\.pk3$'
docker rm "$id"
```
