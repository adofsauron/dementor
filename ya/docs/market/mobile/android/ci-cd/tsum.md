# ЦУМ

ЦУМ позволяет делать несколько вещей. Одна из которых - запускать пайплайны.
Пайплайны служат визуализацией процесса CI/CD как релизов, так и фичей разработчиков.

## Проект

Попасть в проект мобилок в ЦУМ можно по [ссылке](https://tsum.yandex-team.ru/pipe/projects/mobile-blue)

![alt text](https://jing.yandex-team.ru/files/apopsuenko/2021-10-27T08:07:31Z.4abb729.png)

## Пайплайны

### Управление пайплайнами

В этом пункте можно изменить уже созданные пайплайны, создать новую версию, опубликовать и т.д.

![alt text](https://jing.yandex-team.ru/files/apopsuenko/2021-10-27T08:17:57Z.31b494c.png)

![alt text](https://jing.yandex-team.ru/files/apopsuenko/2021-10-27T08:25:10Z.bf3f211.png)

![alt text](https://jing.yandex-team.ru/files/apopsuenko/2021-10-27T08:27:37Z.e0466ff.png)

Конечно можно редактировать и джобы

![alt text](https://jing.yandex-team.ru/files/apopsuenko/2021-10-27T08:28:29Z.97f5309.png)

Настройки:
- `Adapter` - джоба станет круглой, это просто место приёма результатов работы от нескольких джоб
- `Has manual trigger` - чтобы джоба не запускалась автоматически, а только по нажатию юзера

Остальные настройки нам не очень нужны


### Создание пайплайна

Тут можно создать свой пайплайн,собрать как конструктор из имеющихся джоб

## Мультитестовые среды

### Управление средами

Польза пункта только в том, что можно посмотреть свои среды

![alt text](https://jing.yandex-team.ru/files/apopsuenko/2021-10-27T08:32:59Z.688f1d5.png)

### Создание среды

При создании среды важно не ошибиться в выборе пайплайна и ввести ключ тикета заглавными буквами

![alt text](https://jing.yandex-team.ru/files/apopsuenko/2021-10-27T08:37:26Z.e13d87a.png)

## Релизы

### Активные Релизы

Здесь висят незавершённые релизные пайплайны

### Завершённые Релизы

Здесь висят завершённые релизные пайплайны

## Запуск релиза

Важно выбрать нужный пайплайн и ввести необходимые данные

## Android Pipeline

### Этап подготовки

![alt text](https://jing.yandex-team.ru/files/apopsuenko/2021-10-27T08:54:03Z.ed91c77.png)

Что происходит:
1. Ищется репа с кодом
2. Ищется ветка для задачи (которую указали при запуске пайпа)
3. Подливается `develop` в ветку
4. После подливания `develop` берётся и запоминается последний коммит на ветке (для того, чтобы понимать, на каком коммите прогонлись сборки)
5. Нижние кубики нужны для работы [параллельного flow](https://docs.yandex-team.ru/market-mobile/android/git-flow-guide#parallel-epic-pipeline)

{% note alert %}

После подливания кода в ветку, перезапускайте пайплайн с самого начала.
Это нужно для того, чтобы сборки прогнались на последнем коммите и вы могли вмержиться в `develop` с пройденными проверками

{% endnote %}

### Сборка для QA

![alt text](https://jing.yandex-team.ru/files/apopsuenko/2021-10-27T08:58:57Z.3e5be5b.png)

На данном этапе:
1. Собирается сборка для QA
2. Эпик и подзадачи переводятся в `Можно тестировать`
3. Можно запустить сборку не `qaRelease`, а `baseRelease` версии, если нужно проверить специфичные штуки

### Остальные сборки

![alt text](https://jing.yandex-team.ru/files/apopsuenko/2021-10-27T09:01:39Z.13da6bf.png)

На данном этапе прогоняются другие сборки:
- сборка перфтестов
- прогон статических анализаторов
- собираются и прогоняются автотесты (отдельным кубиком, чтобы не пересобирать тесты, если вдруг упал прогон и его нужно перезапустить)

{% note warning %}

Ссылка на `Allure`-отчёт не работает.
Чтобы посмотреть отчёт, нужно пройти на [TeamCity](https://docs.yandex-team.ru/market-mobile/android/ci-cd/teamcity#sborka) и дальше на `Sandbox`

{% endnote %}

### Проверки тикета и код-ревью

![alt text](https://jing.yandex-team.ru/files/apopsuenko/2021-10-27T09:07:51Z.80c911c.png)

Проверяется, что:
1. Pull Request пройден
2. Заполнены поля в тикете, которые подсказывают, как тестировать и что затронуто в рамках фичи
3. Проверка тикетов и коммитов, что нет тикетов без коммитов и нет лишних коммитов

### Завершающая стадия

![alt text](https://jing.yandex-team.ru/files/apopsuenko/2021-10-27T09:11:51Z.07b5ff2.png)

Происходит:
1. Мерж в develop (когда всё собранои фича протестированна)
2. Перевод в статус `Ждёт релиза`
3. Удаление веток
4. Освобождение среды

## Возможные проблемы

1. Не находится тикет - возможно при создании среды тикет указали полной ссылкой или в нижнем регистре

## Как написать свою джобу

Код на `Java`, репа - Аркадия, так что изи.
[Дока](https://wiki.yandex-team.ru/market/development/infra/tsum/) по разворачиванию и принципам работы ЦУМа
