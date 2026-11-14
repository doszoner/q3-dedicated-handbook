# Консоль сервера

[Мод](../glossary.md#мод): все. Проверено запуском на `20261112-d608e3a-stock`.

У сервиса в `compose.yaml` должны быть `stdin_open: true` и `tty: true`. В [примерах](../examples/) они уже есть.

## Шаги

1. Подключитесь из каталога с `compose.yaml`:

   ```sh
   docker attach $(docker compose ps -q q3ded)
   ```

2. Введите команду, например `status`. Ответ печатается в консоль:

   ```
   ] map: q3dm17
   ] cl score ping name            address                                 rate
   ```

3. Отключитесь, не останавливая сервер: **Ctrl-P, затем Ctrl-Q**.

Docker передаёт Ctrl-C процессу в контейнере, поэтому для выхода используйте только Ctrl-P Ctrl-Q.

## Результат

- `docker attach` печатает `read escape sequence` и выходит с кодом 1. После Ctrl-P Ctrl-Q это нормальный выход. Код 1 без строки `read escape sequence` означает настоящую ошибку.
- Сервер работает дальше: `docker compose ps` показывает тот же `Up`, время старта не изменилось, запрос [`getstatus`](../glossary.md#getstatus) получает ответ (команда в разделе [«Как проверить»](../common/network.md#как-проверить)).

Лог можно посмотреть и без консоли: `docker compose logs --tail 100 q3ded`.
