# Операции на бирже

## Создание ордера

На внутренней бирже GolosDEX можно покупать и продавать средства в GOLOS, GBG и токенах UIA. Чтобы продать или купить средства, нужно создать ордер.

**JavaScript**

```javascript
golos.config.set('websocket', 'wss://api.golos.today/ws')

const acc = 'snake' // имя аккаунта, который будет создавать ордер
const wif = '5JFZC7AtEe1wF2ce6vPAUxDeevzYkPgmtR14z9ZVgvCCtrFAaLw' // приватный active-ключ

const orderid = Math.floor(Date.now() / 1000) // у каждого ордера должен быть уникальный id, который необходимо сгенерировать. Здесь применяется один из способов

let expiration = new Date()
expiration.setHours(expiration.getHours() + 1)
expiration = expiration.toISOString().substr(0, 19) // получается формат вида 2022-01-18T11:33:00

try {
    const res = await golos.broadcast.limitOrderCreateAsync(wif, acc, orderid, '1.000 GOLOS', '0.010 GBG', false, expiration)
    console.log('order created')
} catch (err) {
    console.error('cannot create order', err)
}
```

**Python**

```python
#!/usr/bin/env python3
from datetime import datetime, timedelta
import logging
from golos.steem import Steem
from golos.transactionbuilder import TransactionBuilder
from golosbase import operations

acc = 'snake'
wif = '5JFZC7AtEe1wF2ce6vPAUxDeevzYkPgmtR14z9ZVgvCCtrFAaLw'

steem = Steem('wss://api.golos.today/ws')
print('Connected to GOLOS successfully!')

steem.commit.wallet.setKeys(wif)

orderid = int(datetime.now().timestamp())

expiration = datetime.utcnow() + timedelta(hours=1)
expiration = expiration.replace(microsecond=0).isoformat()

op = operations.LimitOrderCreate(
    owner=acc,
    orderid=orderid,
    amount_to_sell='1.000 GOLOS',
    min_to_receive='0.010 GBG',
    fill_or_kill=False,
    expiration=expiration
    )

try:
    steem.commit.finalizeOp([op], acc, 'active')
    print('order created')
except Exception as err:
    print('order not created')
    logging.error(err, exc_info=True)
```

**В этом примере:**

* 1.000 GOLOS продаем. Они спишутся с баланса в момент создания ордера
* 0.010 GBG покупаем за 1.000 GOLOS
* false означает, что не следует использовать Immediate-Or-Cancel, то есть если нет подходящего ордера для сделки, то ордер будет висеть до момента expiration. Если заменить false на true, то при создании ордера сработает IOC и если нет подходящего ордера, то созданный ордер сразу самоуничтожится.
* expiration - сколько времени висеть ордеру (в данном случае 1 час)

## Отмена ордера

Аккаунт может отменить созданный им ордер, если по нему еще не прошли сделки.

**JavaScript**

```javascript
const orderid = 13548 // а так неправильно: orderid = '13548'

try {
    const res = await golos.broadcast.limitOrderCancelAsync(wif, acc, orderid)
    console.log('order canceled')
} catch (err) {
    console.error('cannot cancel order', err)
}
```

**Python**

```python
orderid = 13548 # а так неправильно: orderid = '13548'

op = operations.LimitOrderCancel(
    owner=acc,
    orderid=orderid
)

try:
    steem.commit.finalizeOp([op], acc, 'active')
    print('order canceled')
except Exception as err:
    print('order not canceled')
    logging.error(err, exc_info=True)
```

## Балансы пользователя

**JavaScript**

```javascript
const res = await golos.api.getAccounts(['snake'])
if (!data[0]) {
    console.log('no such account')
} else {
    console.log(data[0].balance) // GOLOS
    console.log(data[0].sbd_balance) // GBG
}

// для UIA следует использовать get_accounts_balances

const balances = await golos.api.getAccountsBalances(['lex'])
const dogecoin = balances[0]['DOGECOIN']
if (dogecoin) {
    console.log(dogecoin.balance)
    console.log(dogecoin.tip_balance)
    console.log(dogecoin.market_balance)
} else {
    console.log('no DOGECOIN balance')
}
```

**Python**

```python
data = steem.get_accounts(['snake'])
if not len(data):
    print('no such account')
else:
    print(data[0]['balance']) # GOLOS
    print(data[0]['sbd_balance']) # GBG

# для UIA следует использовать get_accounts_balances

balances = steem.get_accounts_balances(['lex'])
data = None
try:
    data = balances[0'DOGECOIN']
except KeyError as err:
    print('no DOGECOIN balance')
if not data == None:
    print(data['balance'])
    print(data['tip_balance'])
    print(data['market_balance'])
```

## Создание бота\скрипта

В этом случае требуется Node.js, и его версия должна быть не ниже **11**, а рекомендуется - **16**.

#### Установка Node.js

В случае с Ubuntu выполните в приложении Терминал следующие команды, чтобы удалить существующий Node.js (если он имеется) и установить 16:

```
sudo apt-get remove nodejs
curl -fsSL https://deb.nodesource.com/setup_16.x | sudo -E bash -
sudo apt-get install -y nodejs
```

* Для других Linux см. [здесь](https://github.com/nodesource/distributions#table-of-contents)
* Для Windows - [здесь](https://nodejs.org/dist/v16.13.2/node-v16.13.2-x64.msi)
* Для 32-битных Windows - [здесь](https://nodejs.org/dist/v16.13.2/node-v16.13.2-x86.msi) (использовать только на компьютерах, где обычный вариант не работает)
* Для macOS - [здесь](https://nodejs.org/dist/v16.13.2/node-v16.13.2.pkg)

После того, как Node.js установлен, проверьте, что он доступен из Терминала.

1. Проверьте работу npm:

```
npm --version
```

Она должна выдать ответ наподобие "8.1.2", а не ошибку.

1. Проверьте работу самого Nodejs:

```
nodejs --version
```

А если эта команда выдает ошибку, то должна работать эта:

```
node --version
```

Если все команды выдают ошибку, то в случае Windows необходимо сделать следующее:

1. Найдите, по какому пути установлен nodejs. Обычно это `C:\Program Files\nodejs`, но возможно также `C:\Program Files (x86)\nodejs`
2. Открыть "Панель управления", выбрать "Система", нажать синюю надпись "Дополнительные параметры системы", нажать кнопку "Переменные среды", в списке "Переменные среды пользователя" выбрать переменную Path или PATH, нажать кнопку "Изменить", затем "Создать" и вставить туда путь из пункта 1. Затем нажмите "OK", снова "ОК", затем "Применить", и закройте панель управления.
3. Откройте новый PowerShell и снова проверьте команды.

#### Создание скрипта

После того, как Node.js будет корректно установлен, можно создать свой первый скрипт, который будет работать на вашем ПК и взаимодействовать с Golos.

1. Создайте папку hellogolos
2. Откройте эту папку в терминале. В случае с Windows, сначала просто откройте папку, затем, щелкните "Файл", и затем "Запустить PowerShell".
3. Выполните команду

```
npm install golos-lib-js
```

1. Создайте в папке скрипт index\*\*.mjs\*\* со следующим кодом:

```js
import golos from 'golos-lib-js'

async function main() {
    const dgp = await golos.api.getDynamicGlobalProperties()
    console.log(dgp.head_block_number)

    process.exit(0)
}

main()
```

А если используете Node.js старше 14, то переименуйте файл в index\*\*.js\*\* и вместо import используйте

```js
const golos = require('golos-lib-js')
```

1. Запустите с помощью:

```
node index.mjs
```

## Создание бота\скрипта

Необходим Python версии не ниже 3.6.1, но не 4.0.0 и выше.

#### Установка Python и инструментов для сборки

Чтобы установить Python на **Linux** и не приходилось собирать его из исходного кода, добавьте этот репозиторий:

```
sudo add-apt-repository ppa:deadsnakes/ppa
```

Затем используйте эти команды:

```
sudo apt-get update
sudo apt-get install python3.7 python3.7-dev python3.7-venv
```

(Пакет **-dev** нужен для сборки golos-lib-python. А пакет **-venv** для установки pip.)\
Установите pip:

```
python3.7 -m ensurepip
```

Готово.

**Windows**

Если у вас Windows, используйте [эту](https://www.python.org/downloads/windows/) страницу. Найдите там самую новую версию, которая имеет ссылку типа "Download Windows installer (64-bit)". В случае, если Windows 32-битный и этот установщик не работает, используйте "Download Windows installer (32-bit)".\
**Важно:** при установке обязательно отметьте флажок "Add Python to PATH", чтобы Python был доступен из PowerShell (терминала). Если вы установили Python без этого флажка, переустановите Python.

Затем необходимо установить Microsoft Visual Studio не ниже 2015, а рекомендуется 2019 (https://visualstudio.microsoft.com/visual-cpp-build-tools/ и выбрать пункт "Классические приложения C++", остальное по умолчанию). Это необходимо для работы библиотеки python-golos.

Кроме того, необходимо установить OpenSSL.

**macOS**

Используйте [эту](https://www.python.org/downloads/macos/) страницу, а затем выполните [эти](https://github.com/golos-blockchain/lib-python#homebrew-build-prereqs) инструкции.

#### Создание скрипта

1. Создайте папку hellogolospy
2. Установите библиотеку golos-lib-python:

```
python3.7 -m pip install golos-lib-python
```

или (Windows)

```
python -m pip install golos-lib-python
```

Библиотеки устанавливаются для всей системы (в site-packages), а не в конкретную папку.\
3\. Создайте скрипт:

```python
#!/usr/bin/env python3
from golos.steem import Steem

steem = Steem('wss://api.golos.today/ws')
print('Connected to GOLOS successfully!')

dgp = steem.get_dynamic_global_properties()
print(dgp['head_block_number'])
```
