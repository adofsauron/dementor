# Core API

Представляет собой API для работы с оплатами и привязками без каких либо UI зависимостей - вы можете встраивать его в своё приложение и стыковать с собственным интерфейсом как угодно.
Однако, если вы хотите уметь привязывать карты, а также осуществлять оплату карточками со вводом CVV, то вам придется использовать также Prebuilt UI компоненты, чтобы не заниматься самому передачей чувствительных данных банковских карт. Подробнее о них в соответствующем разделе.

## Подключение
Для подключения добавьте зависимость от модуля `core`
```
implementation 'com.yandex.paymentsdk:core:$versions.paymentsdk'
```