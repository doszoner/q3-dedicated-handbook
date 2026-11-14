# rcon

[Мод](../glossary.md#мод): все. Проверено запуском на `20261112-d608e3a-stock`.

rcon работает по [UDP](../glossary.md#udp), поэтому нужны `NATIVE_UDP: "1"` и доступный `PORT`. При `NATIVE_UDP=0` UDP-сокета нет, и rcon не отвечает.

## Включить

1. В [`autoexec.cfg`](../glossary.md#autoexeccfg) уберите `//` перед строкой пароля и замените `CHANGE_ME` своим паролем:

   ```
   set rconPassword "CHANGE_ME"
   ```

   Переменная называется `rconPassword`, а не `rcon_password`.
2. Примените: `docker compose up -d --force-recreate q3ded`.

Пароль не попадает в [`q3config_server.cfg`](../glossary.md#q3config_servercfg) и хранится только в вашем `autoexec.cfg`.

## Пользоваться

На [обычном клиенте](../glossary.md#обычный-клиент) Quake III, подключённом к серверу, введите в консоли:

```
/rconPassword CHANGE_ME
/rcon status
```

Ответ печатается в консоли клиента. Клиенту, который не подключён к этому серверу, нужен ещё `/rconAddress SERVER_IP:27960`; так не проверялось.

Без клиента команду можно отправить с любой машины, откуда до сервера доходит UDP. На проверке такой же запрос отправлял скрипт с [хоста](../glossary.md#хост), а эта однострочная команда отдельно не запускалась:

```sh
python3 -c 'import socket,sys; s=socket.socket(2,2); s.settimeout(3); s.sendto(b"\xff"*4+b"rcon "+" ".join(sys.argv[3:]).encode(),(sys.argv[1],int(sys.argv[2]))); print(s.recv(65535)[4:].decode("latin-1"))' SERVER_IP 27960 CHANGE_ME status
```

## Результат

| Случай | Ответ сервера | Строка в логе сервера |
|---|---|---|
| верный пароль | вывод команды, например `map: q3dm17` и таблица игроков; в консоли клиента то же самое | `Rcon from <адрес>: status` |
| неверный пароль | `Bad rconpassword.`, в том числе в консоли клиента | `Bad rcon from <адрес>: status` |
| пароль пустой или строка закомментирована | `No rconpassword set on the server.` на любой пароль | в проверке не записана |

Что из заданного через rcon сохраняется после [рестарта](../glossary.md#рестарт-контейнера), описано в [layers.md](../common/layers.md#жизненный-цикл-настройки).
