# О PaymentSDK

PaymentSDK - это библиотека для работы с платежами и банковскими картами в iOS/Android.

Есть два основных варианта интеграции:
1. Диалог - PaymentSDK запускается как `Activity`/`ViewController` и полностью содержит в себе всю логику по выбранному сценарию. Подходит, когда нужно быстро интегрировать оплату в приложение. Чтобы узнать больше, ознакомьтесь с [How To Dev](paymentsdk/index.md)
1. Компоненты - сервис сам решает, какие компоненты PaymentSDK ему нужны и может более гибко настраивать сценарий работы. Готовые UI-компоненты при этом помогут сохранить единый внешний вид с другими приложениями Яндекса. Представлены отдельными gradle-модулями в Android и сабспеками в iOS. Посмотрите статьи по быстрому старту для [Android](android/quick.md) и [iOS](ios/quick.md).
