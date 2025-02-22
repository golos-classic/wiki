# HF26: Новые возможности

## **Конвертация токенов GOLOS в GBG**

Расширен функционал внутренней конвертации токенов, стал возможен обмен GOLOS на GBG по среднему 3.5 дневному курсу делегатских котировок. Доработан интерфейс конвертации в обе стороны, с отображением примерной суммы (по текущей медиане) и размера комиссии. [Подробнее](https://golos.id/ru--golos/@lex/osnovnye-izmeneniya-26khf-uzhe-v-testovoi-seti).

Добавлен делегатский параметр процента комиссии по конвертации `convert_fee_percent`, 5% по умолчанию.

## **Доработка параметров распределения эмиссии**

В целях исключения манипуляций с медианой и возникновения ошибок, в параметрах исключена «взаимозависимость» и они заменены на `worker_emission_percent` и `vesting_of_remain_percent`

Где `worker_emission_percent` процент эмиссии, поступающий на наполнение фонда воркеров, а `vesting_of_remain_percent` процент распределения оставшейся эмиссии на пул вестинга и общий пул.

Несменяемое годами значение 15% на вознаграждения делегатов вернулось из параметров в конфиг, `worker_emission_percent` 1%, `vesting_of_remain_percent` 80%. Что означает, 80% от оставшихся 84% эмиссии (67.2% эмиссии) пойдёт пул вестинга/СГ, 20% (16.8% эмиссии) в общий пул.

## **Минимум СГ для получения кураторских наград**

Добавлен делегатский параметр минимальной суммы СГ, с которой пользователь начинает получать процент за курирование контента. `min_golos_power_to_curate`, 1000 GOLOS по умолчанию.

## **Влияние на репутацию пользователей из профиля**

Как и ранее, для снижения репутации нужно иметь репутацию выше, операции повышения/понижения расходуют батарейку апвоутов, но не влияют на распределение общего пула, не имеют ограничений в 7 дней и не могут быть отменены.

Была доработана `vote_operation`, без указания `permlink` в операции. В интерфейсе веб-клиента добавлена страница со списком аккаунтов, ушедших в отрицательную репутацию за поcледнее время (виртуальная операция `minus_reputation_operation`). [Подробнее](https://golos.id/ru--golos/@lex/osnovnye-izmeneniya-26khf-uzhe-v-testovoi-seti).

## **Параметризируемые лимиты при отриц. репутации**

Проведен перенос репутации из плагина и добавлены делегатские параметры на постинг-активность аккаунтов с отрицательной репутацией.

По умолчанию за сутки (1440 в минутах) - 3 поста/комментария/апвоута.

`negrep_posting_window: 1440`\
`negrep_posting_per_window: 3`

Кроме того, доработаны имеющиеся «антиспам» параметры:

```
"posts_window": 32767,
"posts_per_window": 4,
"comments_window": 32767,
"comments_per_window": 80,
"votes_window": 32767,
"votes_per_window": 80,
```

Значения в секундах будут пересчитаны на минуты. Делегаты смогут установить 12 часов, сутки, иные значения (не упираясь в текущий потолок 32767 секунд \~9 часов).

## **Добавлен** event-плагин

Что позволит получать события с виртуальными операциями для развития функционала веб-клиентов, альтернативы получения/стриминга блоков и пр. \
\
Для настройки, добавить к списку плагинов в конфиге ноды `event_plugin` и

```
store-evaluator-events = true
event-blocks = 600
```

_Информация будет дополнена._

**Также**, были восстановлены и добавлены тесты для нод, для локализации ошибок доработана запись стектрейса в логах докера ноды. В случае необходимости вывода строк сборку запускать в режиме Debug (добавив к `docker build` параметр `--build-arg TYPE=Debug).`

Исправлена ошибка с отменой всех ордеров (в том числе и по UIA токенам) на внутренней бирже при автоконвертации GBG.

Доработано начисление процентов держателям GBG в случае активного параметра `sbd_interest_rate`, только на сейф-балансе.
