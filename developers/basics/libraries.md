# Библиотеки для работы

Наличие библиотеки для конкретного языка программирования зависит от наличия заготовок для работы с криптографией, большими числами и транспортными протоколами (http/ws). Так как GOLOS исторически эволюционировал из Graphene, многие библиотеки для кодовой базы Steem нередко подходят для Golos. Отличительной особенностью является формат общения с нодой (структура json-rpc), порядок и наименование параметров при описании операции, формат сложных данных в бинарном виде.

Каждый разработчик может поднять свою ноду для взаимодействия с GOLOS, но для начинающих разбираться существуют [публичные ноды](https://wallet.golos.id/nodes).

Ниже перечислены основные библиотеки, которые поддерживают большинство API запросов к ноде и формирование транзакций.

## JavaScript

Фаворит для разработки приложений библиотека [golos-lib-js](https://github.com/golos-blockchain/libs/tree/master/golos-lib-js). В нем есть поддержка всего что нужно как для серверного (nodejs), так и для пользовательского (js в браузерах) взаимодействия с Golos:

* Создание и кодирование ключей;
* API-запросы;
* Формирование транзакций;
* Упрощенный конструктор транзакций для операций;
* Функции обратного вызова для запросов;

[Документация](https://github.com/golos-blockchain/libs/tree/master/golos-lib-js/docs) golos-lib-js доступна на GitHub. Также примеры для часто используемых операций смотрите в [разделе Примеры кода](code-examples.md).

## Python

Библиотека [https://github.com/golos-blockchain/lib-python](https://github.com/golos-blockchain/lib-python), наиболее **актуальная** **на текущий момент** и поддерживает последние изменения в блокчейне (26 ХФ).\
[https://pypi.org/project/golos-lib-python/](https://pypi.org/project/golos-lib-python/)\
\
За её основу была взята[ golos-python](https://github.com/bitfag/golos-python) от[ @vvk](https://golos.id/@vvk) (актуальна до 23 ХФ).

[Библиотека golos-python](https://github.com/Privex/golos-python) от [@someguy](https://golos.id/@someguy123)/[@ksantoprotein](https://golos.id/@ksantoprotein) поддерживает как API запросы, так и формирование транзакций. [Документация](https://golos-python.readthedocs.io/en/latest/index.html) к ней на английском языке.

## PHP

Сложность разработки поддержки на PHP в том, что нет стандартных библиотек для работы с криптографией. Поэтому необходим полный доступ к серверу, чтобы собрать secp256k1 для PHP и включить поддержку [GMP](https://ru.wikipedia.org/wiki/GNU\_Multi-Precision\_Library). Это накладывает определенные ограничения на разработчиков (требует опыт в администрировании).

Несмотря на это, существует [библиотека php-graphene-node-client](https://github.com/t3ran13/php-graphene-node-client/) с поддержкой GOLOS от [@t3ran13](https://golos.id/@t3ran13), установка возможна через Docker.

## GO

[Библиотека golos-lib-go](https://github.com/golos-blockchain/golos-lib-go) от [@asuleymanov](https://golos.id/@asuleymanov) также подходит для API запросов и изучения формирования транзакций.

## Другое

Если вы не нашли требуемый язык программирования, то можно обратить внимание на существующие библиотеки для Steem и EOS. Чтобы модифицировать их и получить поддержку Golos достаточно проверить формат json-rpc запросов, поменять chain\_id (в GOLOS он равен `782a3039b478c839e4cb0c941ff4eaeb7df40bdd68bd441afd444b9da763de12` — это префикс для подписи сырых транзакций) и настроить конструктор операций.

* [C# Ditch](https://github.com/Chainers/Ditch) — быстрая и простая библиотека на C# использующая .NET стандарта 2.0;
* [Elixir API wrapper](https://github.com/metachaos-systems/steemex) — библиотека на Elixir для API-запросов;
* [Swift Steem](https://github.com/steemit/swift-steem) — библиотека на Swift.
