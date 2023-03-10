# Об этой документации

Документация к PaymentSDK сделана в формате YFM, часть страниц пишутся вручую, а общий справочник по классам автогенерируется в проекте.

## Как быстро обновить
### Сборка
В `payment-sdk/android` вызвать:
* для Android
```
./gradlew generateDocumentation
```
* для iOS необходимо предварительно установить swift-doc
```
brew install swiftdocorg/formulae/swift-doc
```
после чего для обновления можно вызывать
```
./gradlew generateIOSDocumentation
```
Результат будет скопирован в `docs/payment-sdk`, можно проверить локально сборку выполнив в этой директории
```
ya make
```
Если всё хорошо - коммитим и открываем пуллреквест.
### Публикация
На все изменения в документации будет приходить робот в пулл-реквест и деплоить коммиты в тестинг. После мержа вся документация автоматически опубликуется в релиз.

Можно управлять публикацией вручную, для этого идем в директорию с [документацией](https://a.yandex-team.ru/arc_vcs/docs/payment-sdk?rev=trunk)
Должна быть кнопка где-то справа `Manage Docs`. Через неё можно катить разные версии в тестинг и прод.

## Внутреннее устройство

{% note alert %}

Этот раздел **только** для тех, кто хочет понять внутреннее устройство нашего механизма работы с документацией.
Если вы хотите просто её обновить, читайте предыдущий раздел.

{% endnote %}

### Про CI
Документация публикуется через новый CI, конфиг расположен [тут](https://a.yandex-team.ru/arc_vcs/docs/payment-sdk/a.yaml).

Все сборки документации и её деплой проходит в нашей группе Sandbox [PAYMENTSDK](https://sandbox.yandex-team.ru/admin/groups/PAYMENTSDK/general).

Для работы всего этого процесса был заведен робот https://staff.yandex-team.ru/robot-paymentsdk-doc , он же будет приходить со ссылками на опубликованную документацию в пулл-реквесты. Его токен для доступа лежит в [секретнице](https://yav.yandex-team.ru/secret/sec-01fy93x8q31643h61aw1zrj371)

Подробнее про деплой документации через CI можно почитать по этой [ссылке](https://docs.yandex-team.ru/docstools/deploy).

### Что это за YFM?
Модифицированный формат Markdown. Подробнее про систему документации вот [тут](https://docs.yandex-team.ru/docstools/)

### Как это работает на Android

На Android используется стандартная утилита Dokka, которая формирует Markdown странички по kdoc комментариям в коде. Настройки можно посмотреть в `payment-sdk/android/gradle/documentation.gradle`. Головная таска `generateDocumentation` в корневом `build.gradle`. 

#### Этап 1: Генерация GFM по комментариям
Самый удобный для нашего случая формат - GFM (Github Flavored Markdown), выбираем его.

Так как проект многомодульный, то используется режим MultiModule - во всех дочерние тасках, где нужна документация, конфигурируем `dokkaGfmPartial`, а в корневом вызываем `dokkaGfmMultiModule`.

#### Этап 2: Собираем включаемое оглавление
Теперь нужно сделать включаемый `toc.yaml`, через который мы будем подцеплять нашу автосгенерированную документацию по классам. Для этого сделана Gradle таска `generateTocYaml`, которая обходит дерево сгенерированных Markdown файлов и создает оглавление.

При этом мы выставляем `hidden` для неосновных страничек с методами, свойствами и прочим - это оставляет их доступными, но не захламляет оглавление.

#### Этап 3: Копирование и чуть-чуть костылей
У MultiModule есть некоторая проблема со ссылками на объекты, которые не имеют своих собственных страничек - это обнаружилось на `PaymentCompletion` в нашем коде. Так что сначала прогоняем маленькийс скрипт и подменяем ссылки на такое в `fixupDokkaIssues`.

После этого удаляем предыдущие результаты, если есть, и копируем новые. В процессе оказалось важно для верхнеуровневых страничек во включаемом оглавлении не пересекаться с верхнеуровневыми в основном, поэтому переименовываем при копировании `index.md` в `modules.md` - смотреть таску `copyReferenceIndex`.

### Как это работает на iOS

На iOS для генерации используется swift-doc - его необходимо поставить через brew. Головная таска генерации `generateIOSDocumentation` в корневом [build.gradle](https://a.yandex-team.ru/arc_vcs/mobile/payment-sdk/android/build.gradle). 

#### Этап 1: Генерация Markdown по комментариям.
Используем формат Common Markdown. Так как мультимодульности в swift-doc нет, то вручную запускаем на нужных директориях - таска `generateSwiftDoc`.

#### Этап 2: Собираем включаемое оглавление
По аналогии с Android формируем `toc.yaml` с оглавлением. Этим занимается таска `generateIOSTocYaml`. Одновременно с этим слегка подправляем головные `Home.md` для модулей регекспом чтобы ссылки были кликабельными.

#### Этап 3: Копируем всё вместе
Таска `copyIOSDocs` копирует всё из временной директории в расположение документации PaymentSDK.
