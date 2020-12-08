# Гайд для witness/seed ноды

Рекомендованные минимальные системные требования:

* 8 Гб оперативной памяти и 80 Гб SSD накопителя
* Linux-система, напр. Ubuntu 16.04/18.04 + стабильный доступ к интернету 

## Устанавливаем Docker

```text
sudo apt-get update
```

```text
sudo apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
```

```text
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

```text
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```

```text
sudo apt-get update
```

```text
sudo apt-get install docker-ce -y
```

## Устанавливаем ноду

Скачиваем файл цепочки блоков \(без него синхронизация от seed-нод занимает около 2 суток\), либо полный бэкап \(с ним запуск займёт менее часа\).  
  
Только цепочка блоков \(реплей займёт несколько часов\):

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
```text
wget -P ~/home/blockchain --user=u223265-sub1 --password=tXjXAmNBcu8PmmbQ https://u223265-sub1.your-storagebox.de/block_log.index https://u223265-sub1.your-storagebox.de/block_log

```
{% endtab %}

{% tab title="Финляндия 2" %}
```text
wget -P ~/home/blockchain --user=u233417-sub1 --password=xCbthClwoWSVGIt1 https://u233417-sub1.your-storagebox.de/block_log.index https://u233417-sub1.your-storagebox.de/block_log

```
{% endtab %}
{% endtabs %}

Полный бэкап \(реплей не требуется, менее часа\):

{% tabs %}
{% tab title="Германия 1" %}
```
wget -P ~/home/blockchain --user=u237308-sub1 --password=3oOk8579Ff8ceKdy https://u237308-sub1.your-storagebox.de/block_log.index https://u237308-sub1.your-storagebox.de/block_log https://u237308-sub1.your-storagebox.de/shared_memory.bin

```
{% endtab %}

{% tab title="Финляндия 1" %}
```
wget -P ~/home/blockchain --user=u245960-sub1 --password=7USy9jS9GS2Yka3c https://u245960-sub1.your-storagebox.de/block_log.index https://u245960-sub1.your-storagebox.de/block_log https://u245960-sub1.your-storagebox.de/shared_memory.bin

```
{% endtab %}

{% tab title="Германия 2" %}
```
wget -P ~/home/blockchain --user=u223265-sub1 --password=tXjXAmNBcu8PmmbQ https://u223265-sub1.your-storagebox.de/block_log.index https://u223265-sub1.your-storagebox.de/block_log https://u223265-sub1.your-storagebox.de/shared_memory.bin

```
{% endtab %}

{% tab title="Финляндия 2" %}
```
wget -P ~/home/blockchain --user=u233417-sub1 --password=xCbthClwoWSVGIt1 https://u233417-sub1.your-storagebox.de/block_log.index https://u233417-sub1.your-storagebox.de/block_log https://u233417-sub1.your-storagebox.de/shared_memory.bin

```
{% endtab %}
{% endtabs %}

### **Генерируем ключи**

Для облегчения получения ключей, вместо запуска ноды без них, использования команды `suggest_brain_key` в cli-wallet для генерирования, правки конфига и перезапуска ноды, можно сразу сгенерировать их [здесь](https://control.viz.world/tools/keys/), либо [здесь](https://cyberway.ropox.app/cyberway/keygen).

![](https://lh6.googleusercontent.com/CiBEBEORRNyJRZDLDmgrPpqZ8k0yG6NR0doq88h26YaRpH5ioh-eOcFFT-ztCMgVA9u2PAAYnnBBOJ8wkKo10N2NYRPC7e5H3EZrdZiZOIQw_Az1lmUl6Tlut17nbMc5AroXUR5g)

Только в Public-key \(публичном ключе\) заменить три начальных символа `VIZ` на `GLS`. Этот ключ нам понадобится для объявления себя делегатом позднее, а Private-key \(приватный ключ\) уже на следующем шаге.

### **Загружаем конфиг**

Предварительно заменив значения `witness` и `private-key` на свои.   
****В качестве `witness` запишем **логин без @** от своего аккаунта на Голосе**,** `private-key` тот что сгенерировали на шаге выше.

```text
mkdir ~/config && echo 'p2p-endpoint = 0.0.0.0:4243
webserver-thread-pool-size = 1
webserver-http-endpoint = 0.0.0.0:8090
webserver-ws-endpoint = 0.0.0.0:8091
read-wait-micro = 500000
max-read-wait-retries = 2
write-wait-micro = 500000
max-write-wait-retries = 3
single-write-thread = true
enable-plugins-on-push-transaction = false
shared-file-size = 2G
min-free-shared-file-size = 500M
inc-shared-file-size = 2G
block-num-check-free-size = 1000
plugin = chain p2p json_rpc webserver network_broadcast_api witness database_api witness_api
clear-votes-before-block = 4294967295
store-account-metadata = false
store-memo-in-savings-withdraws = false
replay-if-corrupted = true
skip-virtual-ops = true
enable-stale-production = false
witness = "ЛОГИН-ДЕЛЕГАТА"
private-key = ПРИВАТНЫЙ-КЛЮЧ-НОДЫ
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
appenders=stderr' | sudo tee -a ~/config/config.ini
```

### **Запускаем контейнер**

```text
sudo docker run -it \
    -p 4243:4243 \
    -v ~/config/config.ini:/etc/golosd/config.ini \
    -v ~/home/blockchain:/var/lib/golosd/blockchain \
    -v ~/wallet:/golosd \
    --name golosd golosblockchain/golos:latest
```

Начнётся загрузка образа ноды и реплей \(наполнение файла оперативных данных `shared_memory.bin` из блоков\), который будет продолжаться от пары часов до суток \(в зависимости от производительности вашего сервера\). 

Можно оставить окно терминала или закрыть его, подключившись позднее и зайдя в логи ноды командой:

```text
sudo docker logs -f --tail 50 golosd
```

С появлением в логах ноды подобных сообщений о получении блоков, окно терминала можно закрыть \(всё в порядке\).

![](https://lh4.googleusercontent.com/xfrNK9kKgadic6F7mAiZ3notwNTdG1IxzMwWWtybsdox6VtV9HrXU5LLkQyLBWHPb-GLgKcv39spoG9Heavv4yTQItEQRyc01aJjCi21BfGorCxs6aGyPyYTxgkzea_8iv6QAFzd)

### Телеграм-бот о пропуске блоков

Для отслеживания работы делегатских нод и пропуска ими блоков можно использовать бот [@golos\_witness\_monitor\_bot](https://t.me/golos_witness_monitor_bot)

## **Работа с cli-wallet**

{% hint style="info" %}
Эти действия нужны лишь для первого запуска ноды.
{% endhint %}

Заходим в cli\_wallet ноды. 

```text
sudo docker exec -it golosd cli_wallet \
    -w /golosd/wallet.json \
    -s ws://localhost:8091
```

Добавляем свой пароль к cli\_wallet \(запишите его и сохраните\):

```text
set_password 123456789
```

Разблокируем доступ:

```text
unlock 123456789
```

Импортируем в cli\_wallet наш приватный активный ключ аккаунта-делегата \(ключ, который смотреть тут [https://golos.id/@lex/permissions](https://golos.id/@lex/permissions), начинается с цифры 5\)**.**

```text
import_key 5Js............
```

Объявляем себя делегатом, с публичным ключом GLS \(который мы сгенерировали ранее в паре к приватному ключу для конфига ноды\)**.**

Нужно заменить логин, ссылку на пост/аккаунт делегата + публичный ключ.

```text
update_witness "ЛОГИН" "https://golos.id/@ЛОГИН" GLS............. true
```

Публикуем свой первый прайс-фид, заменив на свой логин-делегата.

```text
publish_feed ЛОГИН {"quote":"1.000 GOLOS", "base":"0.500 GBG"} true
```

Для выхода из cli\_wallet вводим команду:

```text
quit
```

## **Публикация прайсфидов**

Запускаем в докере ещё один контейнер, подробнее об этом удобном скрипте можно прочитать [здесь](https://wiki.golos.id/witnesses/price-feed).

```text
sudo docker run -it \
    -e NODE=ws://localhost:8091 \
    -e WITNESS=ЛОГИН-ДЕЛЕГАТА \
    -e KEY=ПРИВАТНЫЙ-АКТИВНЫЙ-КЛЮЧ \
    --name feed --net=container:golosd vvk123/golos-witness-tools ./update_price_feed.py --monitor
```

\* нужно заменить логин и приватный активный ключ аккаунта-делегата на свои

Пример скриншота после выполнения команды:

![](https://lh6.googleusercontent.com/zmlDvPxwhoHPAvQwz4MzO8esAFGcO26_fWM1Mvb_5eMQxavdQb6HIBwDYuPEOo_zQXfIbRup0SvM_v150D4mSJ6UCwHcO6LCS2h_ZOWnSoY5D9YrjVRyL2urxo22qWxvxGPCnx7N)

После появления логов с калькуляцией курса GBG, закрываем окно терминала.

## **Изменения в конфиге**

Заходим в конфиг командой**:**

```text
sudo nano ~/config/config.ini
```

Вносим нужные правки, нажимаем `Ctrl+O`, подтверждаем `Enter`, выходим `Ctrl+X`**.**

Перезапускаем контейнер ноды.

```text
sudo docker restart golosd
```

## **Делегатские параметры**

Заходим на [https://golos.id/~witnesses](https://golos.id/~witnesses) и напротив своего делегата в столбце “Параметры” нажимаем на значок настроек **\(**описание каждого параметра возникает при наведении мышкой\).

[Подробнее](https://wiki.golos.id/witnesses/median-props) о значении медианных параметров. Изменить параметры можно и через [cli\_wallet](guide.md#rabota-s-cli-wallet) ноды, заменив логин и выполнив команду:

{% tabs %}
{% tab title="Базовые" %}
```text
update_chain_properties ЛОГИН {"account_creation_fee":"1.000 GOLOS", "maximum_block_size":65536, "sbd_interest_rate":0, "create_account_min_golos_fee":"0.100 GOLOS", "create_account_min_delegation":"1.000 GOLOS", "create_account_delegation_time":2592000, "min_delegation":"1.000 GOLOS"} true

```
{% endtab %}

{% tab title="19 ХФ" %}
```text
update_chain_properties ЛОГИН {"max_referral_interest_rate":1000, "max_referral_term_sec":15552000, "min_referral_break_fee":"1.000 GOLOS", "max_referral_break_fee":"100.000 GOLOS", "posts_window":3, "posts_per_window":1, "comments_window":200, "comments_per_window":10, "votes_window":15, "votes_per_window":5, "auction_window_size":0, "max_delegated_vesting_interest_rate":8000, "custom_ops_bandwidth_multiplier":10, "min_curation_percent":7500, "max_curation_percent":7500, "curation_reward_curve":"square_root", "allow_distribute_auction_reward":true, "allow_return_auction_reward_to_fund":true} true

```
{% endtab %}

{% tab title="22 ХФ" %}
```
update_chain_properties ЛОГИН {"worker_reward_percent":1000, "witness_reward_percent":1500, "vesting_reward_percent":6500, "worker_request_creation_fee":"100.000 GBG", "worker_request_approve_min_percent":1500, "sbd_debt_convert_rate":100, "vote_regeneration_per_day":10, "witness_skipping_reset_time":21600, "witness_idleness_time":7776000, "account_idleness_time":15552000} true

```
{% endtab %}

{% tab title="23 ХФ" %}
```
update_chain_properties ЛОГИН {"claim_idleness_time":86400, "min_invite_balance":"10.000 GOLOS"} true

```
{% endtab %}

{% tab title="24 ХФ" %}
```
update_chain_properties ЛОГИН {"asset_creation_fee":"2000.000 GBG", "invite_transfer_interval_sec":60} true

```
{% endtab %}
{% endtabs %}

## **Обновление ноды**

Ставим “пустой ключ” для ноды чтобы приостановить подпись блоков через параметры на странице [https://golos.id/@lex/witness](https://golos.id/@lex/witness) \(заменив на свой логин\).  
  
****Или через [cli\_wallet](guide.md#rabota-s-cli-wallet) ноды командой

```text
update_witness "ЛОГИН" "https://golos.id" GLS1111111111111111111111111111111114T1Anm true
```

Останавливаем докер-контейнер ноды и удаляем его

```text
sudo docker stop golosd && sudo docker rm golosd
```

{% hint style="info" %}
Если в анонсе обновления было указано что нужен реплей, удаляем файл shared\_memory.bin
{% endhint %}

```text
sudo rm ~/home/blockchain/shared_memory.bin
```

Останавливаем докер-контейнер скрипта прайсфида и удаляем его

```text
sudo docker stop feed && sudo docker rm feed
```

Удаляем образы для ноды и скрипта прайсфида

```text
sudo docker rmi golosblockchain/golos && sudo docker rmi vvk123/golos-witness-tools
```

Запускаем контейнер с новой версией

```text
sudo docker run -it \
    -p 4243:4243 \
    -v ~/config/config.ini:/etc/golosd/config.ini \
    -v ~/home/blockchain:/var/lib/golosd/blockchain \
    -v ~/wallet:/golosd \
    --name golosd golosblockchain/golos:latest
```

После появления логов вида `handle_block "Got 0 transactions on block 34563842 by ..."`, закрываем окно терминала. 

Возвращаем публичный ключ ноды GLS.............. который ранее сбрасывали через параметры на странице [https://golos.id/@lex/witness](https://golos.id/@lex/witness)  
  
или через [cli\_wallet](guide.md#rabota-s-cli-wallet), заменив в команде ниже логин, ссылку на пост/аккаунт делегата + публичный ключ на свои:

```text
update_witness "ЛОГИН" "https://golos.id/@ЛОГИН" GLS................. true
```

Возвращаем докер-контейнер скрипта публикации прайсфида \(заменив логин и приватный активный ключ аккаунта-делегата на свои\)

```text
sudo docker run -it \
    -e NODE=ws://localhost:8091 \
    -e WITNESS=ЛОГИН-ДЕЛЕГАТА \
    -e KEY=ПРИВАТНЫЙ-АКТИВНЫЙ-КЛЮЧ \
    --name feed --net=container:golosd vvk123/golos-witness-tools ./update_price_feed.py --monitor
```

После появления логов с калькуляцией курса GBG, закрываем окно терминала.

## Есть вопросы?

Можно уточнить в чате делегатов [https://t.me/golos\_witnesses](https://t.me/golos_witnesses)

