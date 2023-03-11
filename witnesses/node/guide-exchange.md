# Настройка ноды для бирж

## Использование готового cli\_wallet

Скачиваем cli\_wallet и устанавливаем права на файл:

```
wget https://files.golos.app/cli_wallet && chmod +x cli_wallet
```

Запускаем cli\_wallet (список альтернативных публичных [API-нод](https://wallet.golos.id/nodes)):

```
./cli_wallet -s wss://api.golos.id/ws --rpc-http-endpoint 127.0.0.1:8094 --rpc-http-allowip 127.0.0.1
```

Все **параметры запуска** cli\_wallet можно посмотреть командой`./cli_wallet --help`

В примере выше заданы:&#x20;

`--rpc-http-endpoint 127.0.0.1:8094`\
`Endpoint for wallet HTTP RPC to listen on`

`--rpc-http-allowip 127.0.0.1`\
`Allows only specified IPs to connect to the HTTP endpoint`

Возможно вместо запуска cli\_wallet с помощью screen будет удобно использовать режим демона, добавив опцию:

`-d [ --daemon ]`\
`Run the wallet in daemon mode`

Устанавливаем пароль на кошелёк, разблокируем его, импортируем приватный активный ключ для осущестления переводов.

```
set_password 123456

unlock 123456

import_key 5JX..........
```

Примеры команд к cli\_wallet [описаны ниже](guide-exchange.md#primery-komand-k-cli\_wallet-cherez-curl).

## Самостоятельная сборка cli\_wallet

Собрать cli\_wallet можно и с исходного кода за 5 начальных шагов [этой инструкции](../../developers/hardforks/hf18\_instruction.md#razdel\_4-iznachalnaya-ustanovka-blokcheina).

Подключиться к cli\_wallet (список альтернативных публичных [API-нод](https://wallet.golos.id/nodes)):

```
/usr/local/bin/cli_wallet \
  --wallet="/var/lib/golosd/wallet.json" \
  --server-rpc-endpoint="wss://api.golos.id/ws" \
  --rpc-http-endpoint="127.0.0.1:8094" \
  --rpc-http-allowip="127.0.0.1"
```

## Запуск ноды блокчейна с docker-образа

Устанавливаем [Docker](https://wiki.golos.id/witnesses/node/guide#ustanavlivaem-docker) (если его ещё нет).

[Скачиваем файл](https://wiki.golos.id/witnesses/node/guide#ustanavlivaem-nodu) цепочки блоков (без него синхронизация от seed-нод блокчейна занимает более суток).

Добавляем актуальный файл конфигурации ноды (предварительно поменяв аккаунт отслеживания`track-account` и срок хранения истории `history-blocks`, 864000 блоков x 3 секунды = месяц).

```
echo 'webserver-thread-pool-size = 8
webserver-http-endpoint = 0.0.0.0:8090
webserver-ws-endpoint = 0.0.0.0:8091
read-wait-micro = 500000
max-read-wait-retries = 2
write-wait-micro = 500000
max-write-wait-retries = 3
single-write-thread = true
enable-plugins-on-push-transaction = false
shared-file-size = 2G
block-num-check-free-size = 1200
plugin = chain p2p json_rpc webserver network_broadcast_api database_api operation_history account_history account_by_key
history-start-block = 42000000
history-blocks = 864000
track-account = rudex
clear-votes-before-block = 4294967295
store-account-metadata = false
store-comment-extras = false
skip-virtual-ops = true
enable-stale-production = false
mining-threads = 0
[log.console_appender.stderr]
stream=std_error
[log.file_appender.p2p]
filename=logs/p2p/p2p.log
[logger.default]
level=debug
appenders=stderr
[logger.p2p]
level=none
appenders=stderr' | sudo tee -a ~/config.ini
```

Запускаем контейнер:

```
sudo docker run -d \
    -p 127.0.0.1:8090:8090 \
    -p 127.0.0.1:8091:8091 \
    -p 127.0.0.1:8094:8094 \
    -v ~/config.ini:/etc/golosd/config.ini \
    -v ~/blockchain:/var/lib/golosd/blockchain \
    -v ~/wallet:/golosd \
    --name golosd golosblockchain/golos:latest
```

Начнётся загрузка образа ноды и реплей (наполнение данных `shared_memory.bin` из файла цепочки блоков), который будет продолжаться несколько часов в зависимости от производительности сервера.

Посмотреть логи командой:

```
sudo docker logs -f --tail 50 golosd
```

Запуск приложения cli\_wallet внутри контейнера ноды:

```
sudo docker exec -ti -d golosd cli_wallet \
  --wallet="/golosd/wallet.json" \
  --server-rpc-endpoint="ws://localhost:8091" \
  --rpc-http-endpoint="0.0.0.0:8094" \
  --rpc-http-allowip="172.17.0.1"
```

## Примеры команд к cli\_wallet через curl

Установка на кошелёк пароля и разблокировка:

```
curl --data '{"jsonrpc": "2.0", "method": "set_password", "params": ["123456"], "id": 1}' http://127.0.0.1:8094
```

```
curl --data '{"jsonrpc": "2.0", "method": "unlock", "params": ["123456"], "id": 1}' http://127.0.0.1:8094
```

Импортирование приватного активного ключа в кошелёк:

```
curl --data '{"jsonrpc": "2.0", "method": "import_key", "params": ["5JVFFWRLwz6JoP9kguuRFfytToGU6cLgBVTL9t6NB3D3BQLbUBS"], "id": 1}' http://127.0.0.1:8094
```

Список добавленных в кошелёк аккаунтов:

```
curl --data '{"jsonrpc": "2.0", "method": "list_my_accounts", "params": [], "id": 1}' http://127.0.0.1:8094
```

Получение информации об аккаунте:

```
curl --data '{"jsonrpc": "2.0", "method": "get_account", "params": ["rudex"], "id": 1}' http://127.0.0.1:8094
```

Перевод/трансфер токенов:

```
curl --data '{"jsonrpc": "2.0", "method": "transfer", "params": ["rudex","test","1.000 GOLOS","",true], "id": 1}' http://127.0.0.1:8094
```

Запрос истории последних 50 трансферов где получателем был аккаунт (иные варианты фильтра истории [описаны тут](../../developers/hardforks/sf18.4\_release.md#filtraciya-zaprashivaemoi-informacii-ob-operaciyakh-iz-istorii-akkaunta)):

```
curl --data '{"jsonrpc": "2.0", "method": "filter_account_history", "params": ["rudex",-1,50,{"direction":"receiver","select_ops":["transfer_operation"]}], "id": 1}' http://127.0.0.1:8094
```

Получение информации об операциях в блоке:

```
curl --data '{"jsonrpc": "2.0", "method": "get_block", "params": ["30000000"], "id": 1}' http://127.0.0.1:8094
```

Получение операций из блока вместе с виртуальными и trx\_id:

```
curl --data '{"jsonrpc": "2.0", "method": "get_ops_in_block", "params": ["30000000","false"], "id": 1}' http://127.0.0.1:8094
```

Описание команд к cli\_wallet также есть [здесь](../../developers/api/cli-wallet.md) или можно сформировать формат пользуясь сервисом [gapi.golos.today](https://gapi.golos.today/api/)
