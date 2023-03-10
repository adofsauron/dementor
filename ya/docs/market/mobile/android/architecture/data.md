# Слой данных

Содержит логику по взаимодействию с источниками данных:
- бекенд
- БД
- преференсы
- файлы
- система
- кеш в памяти

## Репозиторий

Точкой входа в `data` слой являются репозитории. Именно в них ходят юзкейсы из `domain` слоя.
Уже репозиторий решает, откуда и как взять данные.

Ответственность репозитория:
- выбирать источник данных
- выбирать поток (`Thread`)
- отдавать данные для маппинга

## Модели data слоя

1. [Data Transfer Object (Dto)](https://nda.ya.ru/t/NpSXu4iR3yMqci) - модель, которая приходит от сервера

{% note warning %}

Наши `Dto` помечаются интерфейсом `Serializable`, так как кеширование запросов работает через сериализацию.
[Простыми словами](https://javarush.ru/groups/posts/2022-serializacija-i-deserializacija-v-java) о том, как работает сериализация

{% endnote %}

2. `RequestDto` - модель, которую отправляем на сервер

3. У нас не сильно используется БД, поэтому чего-то по типу `DBO` у нас нет

## Мапперы

Мапперы выполняют преобразование. Как правило это преобразование `Dto` в доменные модели. Или наоборот, для отправки на сервер.

Желательно, чтобы мапперы обрабатывали ошибки.
Например, при маппинге списка `Dto`, если один элемент пришёл кривой, то игнорировать его, залогировав ошибку, но смаппить остальные элементы, чтобы показать пользователю хоть что-то.

{% note info %}

Обычно мапперы помечаются аннотацией `@Reusable`. Так как они не имеют состояния и можно использовать один и тот же объект из разных потоков, чтобы не создавать лишние аллокации

{% endnote %}

## DataStore

Общее название для места хранения данных.

Датасторы могут:
- имплементировать логику обращения к сети, чтобы не перегружать репозиторий
- обращаться к БД
- быть `Singleton` и хранить кеш в оперативной памяти

В зависимости от места хранения датасторы имеют различные суффиксы:
- в сети - `NetDataStore`
- в памяти - `CacheDataStore`
- в преференцах - `PrefsDataStore`
- в БД - `DbDataStore`
