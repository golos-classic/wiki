# Настройка для API-ноды

Публичные API-ноды важная составляющая для блокчейна, особенно когда есть заинтересованность в развитии приложений (сервисов, игр, ботов и пр.), которые часто повышают ценность всего проекта.

Ниже описан вариант установки API-ноды (с хранением истории операций за неделю). Для такой, оптимальный вариант - сервер с 16 Гб оперативной памяти и 100 Гб SSD накопителя, Ubuntu 18.04/20.04.

## Устанавливаем ноду

Устанавливаем [Docker](https://wiki.golos.id/witnesses/node/guide#ustanavlivaem-docker) (если его ещё нет).

Скачиваем большую часть блоков напрямую с сервера (чтобы не тратить 2 суток на их получение и лишнюю нагрузку делегатских seed-нод).

{% tabs %}
{% tab title="Германия 1" %}
```
wget -P ~/home/blockchain --user=u237308-sub1 --password=3oOk8579Ff8ceKdy https://u237308-sub1.your-storagebox.de/block_log.index https://u237308-sub1.your-storagebox.de/block_log
```
{% endtab %}

{% tab title="Финляндия 1" %}
```
wget -P ~/home/blockchain --user=u245960-sub1 --password=7USy9jS9GS2Yka3c https://u245960-sub1.your-storagebox.de/block_log.index https://u245960-sub1.your-storagebox.de/block_log
```
{% endtab %}

{% tab title="Германия 2" %}
```
wget -P ~/home/blockchain --user=u223265-sub1 --password=tXjXAmNBcu8PmmbQ https://u223265-sub1.your-storagebox.de/block_log.index https://u223265-sub1.your-storagebox.de/block_log
```
{% endtab %}

{% tab title="Финляндия 2" %}
```
wget -P ~/home/blockchain --user=u233417-sub1 --password=xCbthClwoWSVGIt1 https://u233417-sub1.your-storagebox.de/block_log.index https://u233417-sub1.your-storagebox.de/block_log
```
{% endtab %}
{% endtabs %}

Добавляем конфиг ноды (указанные в нём `202800` блоков = неделя). Какие плагины нужны для ваших целей, можно уточнить в чате делегатов [https://t.me/golos\_witnesses](https://t.me/golos\_witnesses)

```
mkdir ~/config && echo 'webserver-thread-pool-size = 8
webserver-http-endpoint = 0.0.0.0:8090
webserver-ws-endpoint = 0.0.0.0:8091
read-wait-micro = 500000
max-read-wait-retries = 2
write-wait-micro = 500000
max-write-wait-retries = 3
single-write-thread = true
enable-plugins-on-push-transaction = false
block-num-check-free-size = 1200
plugin = chain p2p json_rpc webserver network_broadcast_api witness database_api witness_api
plugin = social_network follow tags operation_history account_history market_history
plugin = account_by_key worker_api private_message account_notes event_plugin account_relations
clear-votes-before-block = 4294967295
history-start-block = 38000000
history-blocks = 202800
history-blacklist-ops = producer_reward
history-blacklist-ops = pow2
store-evaluator-events = true
event-blocks = 28800
comment-title-depth = 202800
comment-body-depth = 202800
comment-json-metadata-depth = 202800
follow-max-feed-size = 100
skip-virtual-ops = false
enable-stale-production = false
[log.console_appender.stderr]
stream=std_error
[log.file_appender.p2p]
filename=logs/p2p/p2p.log
[logger.default]
level=debug
appenders=stderr
[logger.p2p]
level=none
appenders=stderr' | sudo tee -a ~/config/config.ini
```

Запускаем ноду в докер-контейнере.

```
sudo docker run -it \
    -p 127.0.0.1:8090:8090 \
    -p 127.0.0.1:8091:8091 \
    -v ~/config/config.ini:/etc/golosd/config.ini \
    -v ~/home/blockchain:/var/lib/golosd/blockchain \
    --log-opt max-size=500m \
    --name golosd golosblockchain/golos:api-node
```

После загрузки докер-образа и реплея (который занимает несколько часов), с получением логов вида `handle_block "Got 0 transactions on block 35071930 by ..."` нода готова к работе.

## Устанавливаем Nginx

```
sudo apt-add-repository ppa:nginx/stable -y
```

```
sudo apt-get update
```

```
sudo apt-get install nginx -y
```

Добавляем файл для своих настроек Nginx.

```
sudo nano /etc/nginx/sites-enabled/node.conf
```

Копируем в него правила, предварительно заменив адрес `server_name` на свой субдомен/домен (не забыв привязать его в настройках DNS к нашему IP сервера). Бесплатные домены можно зарегистрировать напр. [здесь](http://www.freenom.com/ru/freeandpaiddomains.html).

```
server {
listen 80;
server_name test.lexai.host;
location / {
add_header 'Access-Control-Allow-Origin' '*';
add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
add_header 'Access-Control-Allow-Headers' 'DNT,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Content-Range,Range';
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_pass http://127.0.0.1:8090;
}
location /ws {
proxy_http_version 1.1;
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection "upgrade";
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_pass http://127.0.0.1:8091;
proxy_read_timeout 3600;
}
}
```

Сохраняем изменения `Ctrl+O`, подтверждаем `Enter`, выходим `Ctrl+X`.

## Устанавливаем Certbot

```
sudo snap install core
```

```
sudo snap refresh core
```

```
sudo snap install --classic certbot
```

```
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

После следующей команды потребуется ввести:

1. E-mail, на который будут отправляться уведомления о необходимости продления сертификата;&#x20;
2. Согласиться с правилами сервиса введя `A и Enter`;
3. Отказаться от рассылки `N и Enter`;
4. Подтвердить добавление сертификатов к указанным доменам вводом `Enter`;
5. Отказаться от редиректа, введя `1 и Enter`.

```
sudo certbot --nginx
```

Будут добавлены настройки в файл `node.conf`, которые можно перепроверить командой ниже и найти строки с пометкой `# managed by Certbot` в конце файла.

```
sudo nano /etc/nginx/sites-enabled/node.conf
```

Выходим из файла `Ctrl+X`.

Перезапускаем Nginx.

```
service nginx restart
```

Проверяем статус Nginx.

```
sudo systemctl status nginx.service
```

Мы запустили публичную API-ноду, к которой можно подключаться как по адресу `https://test.lexai.host` (RPC) так и `wss://test.lexai.host/ws` (WebSockets).\
\
Кроме того, если на API-ноду ожидается большое количество запросов, советуем обратить внимание на [сервис Jussi](https://golos.id/ru--golos/@lex/kesh-sloi-jussi-dlya-tekh-kto-zapustil-svoi-api-nody), который позволяет перед нодой настроить кеш-слой на базе Redis и Nginx.

При получении письма на e-mail о необходимости обновить сертификат (раз в 90 дней), это можно сделать командой:

```
sudo certbot renew
```

## Есть вопросы?

Можно уточнить в чате делегатов [https://t.me/golos\_witnesses](https://t.me/golos\_witnesses)
