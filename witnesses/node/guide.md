# Гайд для witness/seed ноды

Рекомендованные (минимальные) системные требования:

* 4 Гб оперативной памяти и 80 Гб SSD накопителя
* Linux-система, напр. Ubuntu 18.04/20.04 + стабильный интернет&#x20;

## Устанавливаем Docker

{% code overflow="wrap" %}
```
sudo apt-get update && 
sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```
{% endcode %}

{% code overflow="wrap" %}
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```
{% endcode %}

{% code overflow="wrap" %}
```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
{% endcode %}

{% code overflow="wrap" %}
```
sudo apt-get update && 
sudo apt-get install docker-ce docker-ce-cli containerd.io -y
```
{% endcode %}

## Устанавливаем ноду

Скачиваем файл цепочки блоков **only block\_log** (без него синхронизация от сети seed-нод занимает около 2-3 суток), либо полный **backup witness node** (с ним запуск займёт около часа).

{% tabs %}
{% tab title="Германия 1" %}
{% code title="only block_log" overflow="wrap" %}
```
rsync --progress -e 'ssh -p23' --recursive u379169-sub1@u379169-sub1.your-storagebox.de:block_log ~/home/blockchain/
```
{% endcode %}

{% code title="backup witness node" overflow="wrap" %}
```
rsync --progress -e 'ssh -p23' --recursive u379169-sub1@u379169-sub1.your-storagebox.de: ~/home/blockchain/
```
{% endcode %}

{% code title="password" %}
```
veAujuuVZtivnUj7
```
{% endcode %}
{% endtab %}

{% tab title="Финляндия 1" %}
{% code title="only block_log" overflow="wrap" %}
```
rsync --progress -e 'ssh -p23' --recursive u379085-sub1@u379085-sub1.your-storagebox.de:block_log ~/home/blockchain/
```
{% endcode %}

{% code title="backup witness node" overflow="wrap" %}
```
rsync --progress -e 'ssh -p23' --recursive u379085-sub1@u379085-sub1.your-storagebox.de: ~/home/blockchain/
```
{% endcode %}

{% code title="password" %}
```
dg8NrzpYZ3gMiU7t
```
{% endcode %}
{% endtab %}

{% tab title="Германия 2" %}
{% code title="only block_log" overflow="wrap" %}
```
rsync --progress -e 'ssh -p23' --recursive u379178-sub1@u379178-sub1.your-storagebox.de:block_log ~/home/blockchain/
```
{% endcode %}

{% code title="backup witness node" overflow="wrap" %}
```
rsync --progress -e 'ssh -p23' --recursive u379178-sub1@u379178-sub1.your-storagebox.de: ~/home/blockchain/
```
{% endcode %}

{% code title="password" %}
```
Z8iSXLhyXsCBGqeL
```
{% endcode %}
{% endtab %}

{% tab title="Финляндия 2" %}
{% code title="only block_log" overflow="wrap" %}
```
rsync --progress -e 'ssh -p23' --recursive u378847-sub1@u378847-sub1.your-storagebox.de:block_log ~/home/blockchain/
```
{% endcode %}

{% code title="backup witness node" overflow="wrap" %}
```
rsync --progress -e 'ssh -p23' --recursive u378847-sub1@u378847-sub1.your-storagebox.de: ~/home/blockchain/
```
{% endcode %}

{% code title="password" %}
```
db3YNrkPtrdyF6id
```
{% endcode %}
{% endtab %}
{% endtabs %}

### **Генерируем ключи**

Для облегчения получения ключей, вместо запуска ноды без них, использования команды `suggest_brain_key` в cli-wallet для генерирования, правки конфига и перезапуска ноды, можно сразу сгенерировать их [здесь](https://control.viz.world/tools/keys/), или на [раз](https://cyberway.ropox.app/cyberway/keygen) / [два](https://gapi.golos.today/utils/keys).

![](https://lh6.googleusercontent.com/CiBEBEORRNyJRZDLDmgrPpqZ8k0yG6NR0doq88h26YaRpH5ioh-eOcFFT-ztCMgVA9u2PAAYnnBBOJ8wkKo10N2NYRPC7e5H3EZrdZiZOIQw\_Az1lmUl6Tlut17nbMc5AroXUR5g)

Только в Public-key (публичном ключе) заменить три начальных символа `VIZ` на `GLS`. Этот ключ нам понадобится для объявления себя делегатом позднее, а Private-key (приватный ключ) уже на следующем шаге.

### **Загружаем конфиг**

Предварительно заменив значения `witness` и `private-key` на свои. \
В качестве `witness` запишем **логин без @** от своего аккаунта на Голосе**,** `private-key` тот что сгенерировали на шаге выше.

```
mkdir ~/config && echo 'p2p-endpoint = 0.0.0.0:4243
webserver-thread-pool-size = 4
webserver-http-endpoint = 0.0.0.0:8090
webserver-ws-endpoint = 0.0.0.0:8091
read-wait-micro = 500000
max-read-wait-retries = 2
write-wait-micro = 500000
max-write-wait-retries = 3
single-write-thread = true
enable-plugins-on-push-transaction = false
block-num-check-free-size = 7200
plugin = chain p2p json_rpc webserver network_broadcast_api witness database_api witness_api
clear-votes-before-block = 4294967295
store-account-metadata = false
store-memo-in-savings-withdraws = false
store-comment-extras = false
skip-virtual-ops = true
enable-stale-production = false
required-participation = 33
witness = "ЛОГИН-ДЕЛЕГАТА"
private-key = ПРИВАТНЫЙ-КЛЮЧ-НОДЫ
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

### Использование Docker-Compose

{% hint style="info" %}
Для тех кто разбирается, обратите внимание на [этот пост](https://golos.id/ru--golos/@lex/variant-zapuska-nody-s-docker-compose).
{% endhint %}

### **Запускаем контейнер**

```
sudo docker run -it \
    -p 4243:4243 \
    -v ~/config/config.ini:/etc/golosd/config.ini \
    -v ~/home/blockchain:/var/lib/golosd/blockchain \
    -v ~/wallet:/golosd \
    --log-opt max-size=500m \
    --name golosd golosblockchain/golos:latest
```

Начнётся загрузка образа ноды и реплей (наполнение файла оперативных данных `shared_memory.bin` из блоков), который будет продолжаться от пары часов до суток (в зависимости от производительности вашего сервера).&#x20;

Можно оставить окно терминала или закрыть его, подключившись позднее и зайдя в логи ноды командой:

```
sudo docker logs -f --tail 50 golosd
```

С появлением в логах ноды подобных сообщений о получении блоков, окно терминала можно закрыть (всё в порядке).

![](https://lh4.googleusercontent.com/xfrNK9kKgadic6F7mAiZ3notwNTdG1IxzMwWWtybsdox6VtV9HrXU5LLkQyLBWHPb-GLgKcv39spoG9Heavv4yTQItEQRyc01aJjCi21BfGorCxs6aGyPyYTxgkzea\_8iv6QAFzd)

### Телеграм-бот о пропуске блоков

Для отслеживания работы делегатских нод и пропуска ими блоков можно использовать бот [@golos\_witness\_monitor\_bot](https://t.me/golos\_witness\_monitor\_bot)

## **Работа с cli-wallet**

{% hint style="info" %}
Эти действия нужны лишь для первого запуска ноды.
{% endhint %}

Заходим в cli\_wallet ноды.&#x20;

```
sudo docker exec -it golosd cli_wallet \
    -w /golosd/wallet.json \
    -s ws://localhost:8091
```

Добавляем свой пароль к cli\_wallet (запишите его и сохраните):

```
set_password 123456789
```

Разблокируем доступ:

```
unlock 123456789
```

Импортируем в cli\_wallet наш приватный активный ключ аккаунта-делегата (ключ, который смотреть тут [https://golos.id/@lex/permissions](https://golos.id/@lex/permissions), начинается с цифры 5)**.**

```
import_key 5Js............
```

Объявляем себя делегатом, с публичным ключом GLS (который мы сгенерировали ранее в паре к приватному ключу для конфига ноды)**.**

Нужно заменить логин, ссылку на пост/аккаунт делегата + публичный ключ.

```
update_witness "ЛОГИН" "https://golos.id/@ЛОГИН" GLS............. true
```

Публикуем свой первый прайс-фид, заменив на свой логин-делегата.

```
publish_feed ЛОГИН {"quote":"1.000 GOLOS", "base":"0.500 GBG"} true
```

Для выхода из cli\_wallet вводим команду:

```
quit
```

## **Публикация прайсфидов**

Запускаем в докере ещё один контейнер, подробнее об этом скрипте можно прочитать [здесь](https://wiki.golos.id/witnesses/price-feed).

```
sudo docker run -it \
    -e NODE=ws://localhost:8091 \
    -e WITNESS=ЛОГИН-ДЕЛЕГАТА \
    -e KEY=ПРИВАТНЫЙ-АКТИВНЫЙ-КЛЮЧ \
    --log-opt max-size=500m \
    --name feed --net=container:golosd vvk123/golos-witness-tools ./update_price_feed.py --monitor
```

\* нужно заменить логин и приватный активный ключ аккаунта-делегата на свои

Пример скриншота после выполнения команды:

![](https://lh6.googleusercontent.com/zmlDvPxwhoHPAvQwz4MzO8esAFGcO26\_fWM1Mvb\_5eMQxavdQb6HIBwDYuPEOo\_zQXfIbRup0SvM\_v150D4mSJ6UCwHcO6LCS2h\_ZOWnSoY5D9YrjVRyL2urxo22qWxvxGPCnx7N)

После появления логов с калькуляцией курса GBG, закрываем окно терминала.

## **Изменения в конфиге**

Заходим в конфиг командой**:**

```
sudo nano ~/config/config.ini
```

Вносим нужные правки, нажимаем `Ctrl+O`, подтверждаем `Enter`, выходим `Ctrl+X`**.**

Перезапускаем контейнер ноды.

```
sudo docker restart golosd
```

## **Делегатские параметры**

Заходим на [https://golos.id/\~witnesses](https://golos.id/\~witnesses) и напротив своего делегата в столбце “Параметры” нажимаем на значок настроек **(**описание каждого параметра возникает при наведении мышкой).

[Подробнее](https://wiki.golos.id/witnesses/median-props) о значении медианных параметров. Изменить параметры можно и через [cli\_wallet](guide.md#rabota-s-cli-wallet) ноды, заменив логин и выполнив команду.\
\
Параметры в порядке появления в хардфорках блокчейна:

{% tabs %}
{% tab title="16" %}
{% code overflow="wrap" %}
```
update_chain_properties ЛОГИН {"account_creation_fee":"1.000 GOLOS", "maximum_block_size":65536, "sbd_interest_rate":0, "create_account_min_golos_fee":"0.100 GOLOS", "create_account_min_delegation":"1.000 GOLOS", "create_account_delegation_time":2592000, "min_delegation":"1.000 GOLOS"} true
```
{% endcode %}
{% endtab %}

{% tab title="19" %}
{% code overflow="wrap" %}
```
update_chain_properties ЛОГИН {"max_referral_interest_rate":1000, "max_referral_term_sec":15552000, "min_referral_break_fee":"1.000 GOLOS", "max_referral_break_fee":"100.000 GOLOS", "posts_window":3, "posts_per_window":1, "comments_window":200, "comments_per_window":10, "votes_window":15, "votes_per_window":5, "max_delegated_vesting_interest_rate":8000, "custom_ops_bandwidth_multiplier":10, "min_curation_percent":7500, "max_curation_percent":7500, "curation_reward_curve":"square_root"} true
```
{% endcode %}
{% endtab %}

{% tab title="22" %}
{% code overflow="wrap" %}
```
update_chain_properties ЛОГИН {"worker_request_creation_fee":"100.000 GBG", "worker_request_approve_min_percent":1500, "sbd_debt_convert_rate":100, "vote_regeneration_per_day":10, "witness_skipping_reset_time":21600, "witness_idleness_time":7776000, "account_idleness_time":15552000} true
```
{% endcode %}
{% endtab %}

{% tab title="23" %}
{% code overflow="wrap" %}
```
update_chain_properties ЛОГИН {"min_invite_balance":"10.000 GOLOS"} true
```
{% endcode %}
{% endtab %}

{% tab title="24" %}
{% code overflow="wrap" %}
```
update_chain_properties ЛОГИН {"asset_creation_fee":"500.000 GBG", "invite_transfer_interval_sec":60} true
```
{% endcode %}
{% endtab %}

{% tab title="26" %}
{% code overflow="wrap" %}
```
update_chain_properties ЛОГИН {"worker_emission_percent":100, "vesting_of_remain_percent":8000, "convert_fee_percent":500, "min_golos_power_to_curate":"1000.000 GBG"} true
```
{% endcode %}
{% endtab %}

{% tab title="27" %}
{% code overflow="wrap" %}
```
update_chain_properties ЛОГИН {"unwanted_operation_cost":"100.000 GOLOS", "unlimit_operation_cost":"10.000 GOLOS"} true
```
{% endcode %}
{% endtab %}

{% tab title="28" %}
{% code overflow="wrap" %}
```
update_chain_properties ЛОГИН {"min_golos_power_to_emission":"2000.000 GBG"} true
```
{% endcode %}
{% endtab %}
{% endtabs %}

## **Обновление ноды**

Ставим “пустой ключ” для ноды чтобы приостановить подпись блоков через параметры на странице [https://golos.id/@lex/witness](https://golos.id/@lex/witness) (заменив на свой логин).\
\
Или через [cli\_wallet](guide.md#rabota-s-cli-wallet) ноды командой

```
update_witness "ЛОГИН" "https://golos.id" GLS1111111111111111111111111111111114T1Anm true
```

Останавливаем докер-контейнер ноды и удаляем его

```
sudo docker stop golosd && sudo docker rm golosd
```

{% hint style="info" %}
Если в анонсе обновления было указано что нужен реплей, удаляем файл shared\_memory.bin
{% endhint %}

```
sudo rm ~/home/blockchain/shared_memory.bin
```

Останавливаем докер-контейнер скрипта прайсфида и удаляем его

```
sudo docker stop feed && sudo docker rm feed
```

Удаляем образы для ноды и скрипта прайсфида

```
sudo docker rmi golosblockchain/golos && sudo docker rmi vvk123/golos-witness-tools
```

Запускаем контейнер с новой версией

```
sudo docker run -it \
    -p 4243:4243 \
    -v ~/config/config.ini:/etc/golosd/config.ini \
    -v ~/home/blockchain:/var/lib/golosd/blockchain \
    -v ~/wallet:/golosd \
    --log-opt max-size=500m \
    --name golosd golosblockchain/golos:latest
```

После появления логов вида `handle_block "Got 0 transactions on block 34563842 by ..."`, закрываем окно терминала.&#x20;

Возвращаем публичный ключ ноды GLS.............. который ранее сбрасывали через параметры на странице [https://golos.id/@lex/witness](https://golos.id/@lex/witness)\
\
или через [cli\_wallet](guide.md#rabota-s-cli-wallet), заменив в команде ниже логин, ссылку на пост/аккаунт делегата + публичный ключ на свои:

```
update_witness "ЛОГИН" "https://golos.id/@ЛОГИН" GLS................. true
```

Возвращаем докер-контейнер скрипта публикации прайсфида (заменив логин и приватный активный ключ аккаунта-делегата на свои)

```
sudo docker run -it \
    -e NODE=ws://localhost:8091 \
    -e WITNESS=ЛОГИН-ДЕЛЕГАТА \
    -e KEY=ПРИВАТНЫЙ-АКТИВНЫЙ-КЛЮЧ \
    --log-opt max-size=500m \
    --name feed --net=container:golosd vvk123/golos-witness-tools ./update_price_feed.py --monitor
```

После появления логов с калькуляцией курса GBG, закрываем окно терминала.

## Есть вопросы?

Можно уточнить в чате делегатов [https://t.me/golos\_witnesses](https://t.me/golos\_witnesses)
