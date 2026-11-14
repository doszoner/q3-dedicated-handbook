# Справочник оператора: выделенные серверы Quake III DosZone

Справочник описывает [образы](glossary.md#образ) `ghcr.io/doszoner/q3ded` для обычного Quake III (ioquake3), [CPMA](glossary.md#cpma) и [OSP](glossary.md#osp). Здесь сказано, где что лежит в [контейнере](glossary.md#контейнер), почему строка из [`autoexec.cfg`](glossary.md#autoexeccfg) не применилась и где задать это значение, если его вообще можно задать.

Утверждения о поведении сервера проверены запуском образа названного [тега](glossary.md#тег). Что не проверялось, сказано прямо в тексте.

Пометка «по коду entrypoint» стоит у справочных значений, которые проверяет сам [entrypoint](glossary.md#entrypoint) образа: значения по умолчанию, допустимые значения, разница между незаданной и пустой переменной, тексты и причины [отказов](glossary.md#отказ-при-старте), список файлов, из-за которых сервер отказывается стартовать. Такие значения взяты из кода entrypoint того же образа. Запуском они проверены только там, где это сказано рядом.

## Куда идти

| Нужно | Страница |
|---|---|
| Поднять сервер по примеру | [Первый запуск](start.md) |
| Сервер не стартует, комнаты нет, строка не действует | [Сервер не работает: где смотреть](troubleshooting.md) |
| Как устроен контейнер, в каком порядке применяются настройки, [ENV](glossary.md#env), сеть | [Общая часть](common/layers.md) |
| Обычный Quake III (ioquake3) | [stock](stock/README.md) |
| CPMA | [CPMA](cpma/README.md) |
| OSP | [OSP](osp/README.md) |
| Пошаговые рецепты: простой игроков, [ротация карт](glossary.md#ротация-карт), [rcon](glossary.md#rcon), обновление образа | [Рецепты](recipes/README.md) |
| Что значит слово: entrypoint, digest, archive-переменная и другие | [Термины и определения](glossary.md) |

Страницы общей части:

| Страница | О чём |
|---|---|
| [common/layers.md](common/layers.md) | в каком порядке применяются настройки, какой файл читается, когда настройка сохраняется и когда сбрасывается, почему строки настроек не применяются |
| [common/container.md](common/container.md) | из каких частей состоит контейнер, каталоги и тома, файлы, которые пишет сервер |
| [common/env.md](common/env.md) | все ENV: общая таблица и подробные карточки |
| [common/pinned.md](common/pinned.md) | переменные, которые задаёт entrypoint, их значения у каждого мода и можно ли их изменить |
| [common/network.md](common/network.md) | соединения, порты, firewall, мастер-серверы, IPv6 |

## Образы

Что значат [тег](glossary.md#тег) и [digest](glossary.md#digest), сказано на странице терминов.

| Мод | Тег | Digest |
|---|---|---|
| stock | `20261112-d608e3a-stock` | `sha256:4461a135fde82ddf4d8695bd9362ed1981e9bc3b9092081ac77e8ca7f5582294` |
| CPMA | `20261112-c64186e-cpma` | `sha256:09f9c028b925395296a4b7f0ee84d08f43268c6ec99726c797e675213f0f58d3` |
| OSP | `20261114-9e899840-osp` | `sha256:ba1c045a2cdac33bffd53279c3dcf8105115fe1ba50a64c6e0ea4a3dd95377c3` |

Готовые `compose.yaml` для каждого мода лежат в [examples/](examples/).

## Исходники и документация

Ниже указаны опубликованная основа движка [образов](#образы), выпуск исходников OSP и оригинальная документация компонентов. Конфиги модов из образов лежат в [assets/](assets/README.md).

| Компонент | Исходники | Документация |
|---|---|---|
| Движок [`ioq3ded`](glossary.md#ioq3ded), все три образа | [jdarpinian/ioq3 `7801abd5`](https://github.com/jdarpinian/ioq3/tree/7801abd5ac510a889087ad62473c54dbe509e106), версия ioquake3 для браузера, ответвлённая от [ioquake/ioq3 `6f5c0389`](https://github.com/ioquake/ioq3/tree/6f5c0389477c6c65c04be12ea2eb7a33a79e659f) | [руководство администратора](https://ioquake3.org/help/sys-admin-guide/), [переменные и команды, которые ioquake3 добавил к Quake III](https://github.com/ioquake/ioq3/blob/6f5c0389477c6c65c04be12ea2eb7a33a79e659f/README.md#console) |
| OSP | [JKornev/xq3osp `1.03a-secured`](https://github.com/JKornev/xq3osp/tree/1.03a-secured), восстановленные исходники OSP | каталог `osp/Docs/` в архиве [osp-Quake3-1.03a_full.zip](https://osp.dget.cc/orangesmoothie/downloads/osp-Quake3-1.03a_full.zip), см. [Переменные OSP](osp/README.md#переменные-osp) |
| CPMA 1.53 | не опубликованы | [документация CPMA](https://playmorepromode.com/docs), [настройки сервера](https://playmorepromode.com/guides/cpma-server-settings), [изменения в 1.53](https://playmorepromode.com/docs/cpma-153-changelog) |
| Docker Compose | [docker/compose v5.0.2](https://github.com/docker/compose/releases/tag/v5.0.2), версия, на которой проверен справочник | [документация Docker Compose](https://docs.docker.com/compose/) |

Движок образов собран из дерева jdarpinian/ioq3 `7801abd5` со своими правками для игры в браузере. Эти правки не опубликованы. По документации сборки [игровые модули](glossary.md#игровой-модуль) stock собираются из того же дерева, что и движок образа stock.

В каталоге `osp/Docs/` архива OSP лежат `README.txt`, `CPM-info.txt`, `osp-q3-ClientReadme.txt`, `osp-q3-Commands.txt`, `osp-q3-CQ3Info.txt`, `osp-q3-DecalEditing.txt`, `osp-q3-FreezeTag.txt`, `osp-q3-History.txt`, `osp-q3-Logging.txt`, `osp-q3-MapConfigInfo.txt`, `osp-q3-ModeInfo.txt`, `osp-q3-PlayerFilters.txt`, `osp-q3-ServerGfx.txt`, `osp-q3-Variables.txt` и `osp-q3-Viewcam.txt`, а переводы части файлов лежат в `CHN/`, `ESP/` и `RUS/`.

Если вы нашли ошибку или рецепт не сработал, прочитайте, [как предложить исправление](CONTRIBUTING.md).
