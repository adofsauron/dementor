# Описание архитектуры Маркета с точки зрения мобильщика

![alt text](https://wiki.yandex-team.ru/market/development/architecture/.files/beruarchitectruesystems.svg)

## Встреча с рассказом про архитектуру

<#<video src="https://jing.yandex-team.ru/files/apopsuenko/zoom_0.mp4" width="640" height="480" controls="controls"/>Ваш браузер не поддерживает это видео. <a href="https://jing.yandex-team.ru/files/apopsuenko/zoom_0.mp4" target="_blank">Скачайте видео</a></video>#>

Продолжение
<#<video src="https://jing.yandex-team.ru/files/apopsuenko/zoom_1.mp4" width="640" height="480" controls="controls"/>Ваш браузер не поддерживает это видео. <a href="https://jing.yandex-team.ru/files/apopsuenko/zoom_1.mp4" target="_blank">Скачайте видео</a></video>#>

## Наши прокси сервисы

Для начала, зачем мы ходим через прокси-сервисы.

Есть несколько причин:
1. Самая главная. **Безопасность**.
Как мы знаем, мобилки - дырявые. Можно легко вскрыть, узнать о серваке и начать этот сервак долбить.
С помощью КАПИ и ФАПИ мы можем не заботиться о безопасности на стороне клиента. У нас всегда есть входняа точка в виде сервиса, который может заблокировать пользователей по IP, отозвать ключ и тому подобное.
Исходя из того, что у нас есть прокси-сервисы, остальные сервисы Яндекса и Маркета не должны париться о безопасности, так как они доверяют прокси-сервисам, но не пользователям мобилок.
Также здесь можно привести в пример валидацию токена авторизации.
Прокси-сервисы ходят в blackbox, чтобы валидировать токен. Если бы это было в приложении, то это можно было бы оторвать. А остальные компоненты не парятся по поводу токена. Они принимают на вход только `puid` или другие идентификаторы.

2. **Код, общий для мобилок** можно хранить там.
Если есть какой-то код, общий для 2-х платформ, касающийся данных от компонентов, то почему бы не написать его 1 раз, а не 2. И модифицировать тоже легче.

### КАПИ

Существовал для внешних потребителей информации с белого Маркета.
Потом был доработан для нужд Беру. Потихоньку переходим с него, так как он только для мобилок, а у нас появился ФАПИ.

Стек - `Java` + `REST` + `Spring`
Очередь - `MARKETAPI`
[Шаблон на задачу](https://nda.ya.ru/t/PqDqcqwX3VwXLT)

[Документация](https://doc.yandex-team.ru/market/mobile-dev)
[Архитектура КАПИ](https://wiki.yandex-team.ru/users/dimkarp93/api-arch/)

Как развернуть и работать:
- https://wiki.yandex-team.ru/market/development/contentapi/arcadia/
- https://wiki.yandex-team.ru/users/shiko-mst/podnimaem-content-api-lokalno-/
- https://wiki.yandex-team.ru/users/bejibx/rabota-s-kontentnym-api/
- https://wiki.yandex-team.ru/users/yvgrishin/capi-kak-obnovit-chekauter-klient/

Как смотреть логи:
- https://arcanum.yandex-team.ru/arc_vcs/mobile/market/tools/logs-grep-script
- https://wiki.yandex-team.ru/Market/mobile/marketapps/Dezhurstva/#lokalizacija

Сервисы безопасности, которые использует КАПИ:
- [tvm](https://wiki.yandex-team.ru/passport/tvm2/) - для аутентификации между компонентами
- [Паспорт](https://doc.yandex-team.ru/~passport) - сервис авторизации
- [Крипта](https://wiki.yandex-team.ru/crypta/) - сервис для анализа аудитории
- [Data sync](https://beta.wiki.yandex-team.ru/disk/platform/marketing/dataapi/) (устаревшее. Сейчас используется Pers-Address) - адреса и тому подобное

### ФАПИ

Работает не на 2 платформы, как КАПИ, а на 4. По сути это код фронтов, как они ходят в компоненты, только ещё адаптированнный для мобилок.
Вся новая разработка ведётся на ФАПИ, так как это дешевле по ресурсам и легче поддерживать изменения.

Стек - `JS` + `Node` + `Resolvers API`
Очередь - `MARKETFRONTECH` (для инфраструктурных задач)

[Описание](https://wiki.yandex-team.ru/Market/frontend/front-api/)
[Список резолверов с примерами данных](https://wiki.yandex-team.ru/users/dmpolyakov/bluefrontapi/)

Основные понятия:
- резолвер - модуль, агрегирующий и трансформирующий данные для передачи/использования на сервере/клиенте
- ресурс - реализация протокола общения с компонентами
- handler - функция, реализующая поход за данными и формирование ответа.

Как развернуть и работать:
- https://wiki.yandex-team.ru/Market/frontend/front-api/?from=%252Fusers%252Fantsa4%252Fprojects%252Ffront-api%252F#kakpodnjatfapi
- https://wiki.yandex-team.ru/users/zdoo/white-market-ios-fapi

Как смотреть логи:
- https://wiki.yandex-team.ru/users/osialx/kak-smotret-logi-fapi/

## Утилитарные бекенды

### Эксперименты

ABT/Эксперименты

[Как завести эксперимент](https://wiki.yandex-team.ru/market/marketvertical/experimenthowto/)
[Инстукция для мобилок](https://wiki.yandex-team.ru/market/mobile/marketapps/kak-zavodit-jeksperimenty/)

Ручка в КАПИ, которая возвращает эксперименты - **/startup**
[Описание того, как работают экспы в документации](https://nda.ya.ru/t/lGJfcg4G3VxiC4)

[USaaS]((https://nda.ya.ru/t/33kLw7Lp3VxiCG)) - сервис, который управляет распределением пользователей в экспах

**rearr-factors** - это параметр AB-сплита, идентифицирующий изменений логики выдачи различных бекендов
Мы его получаем из ABT и передаём во все запросы.
В некоторых случаях есть захардкоженные значения, которые определяют выдачу например.
**Репорт** - https://wiki.yandex-team.ru/market/verstka/report-query-params/

Также этот параметр поддерживает **CMS**, выдавая разные страницы по этому параметру.

### Firebase

В Firebase хранятся так называемые **remote-конфиги**.
Основная причина использования - фич-тоглы, чтобы можно было включить/выключить функциональность на мобилках
Также его используют в качестве конфигурирования каких-либо частей приложения. Но это не рекомендуется делать. Лучше делать это через CMS.

[Ссылка](https://console.firebase.google.com/u/0/project/yandex-market-7665c/config)
При заведении нового конфига прописываются ещё локальные установки, на случай, если удалённый конфиг недоступен. Обычно фичу выключают по-умолчанию. Вдруг она с багом, а удалённый конфиг не подгрузился. Тогда она будет выключена точно.
Также при заведении конфига отписываются в чат разрабов + QA и добавляют в описание тикет, для которого конфиг заведён.
Конфиги поддерживают версионирование приложений (по регулярке).

## Основные бекенды
Схема бекендо представлена на [странице](https://wiki.yandex-team.ru/market/development/architecture/)
Также есть некоторое (сложное) описание архитектуры Маркета в [документации](https://doc.yandex-team.ru/market/arch-overview)

### Report

Подсистема поиска и выдачи товарных предложений магазинов на Яндекс.Маркете.
В кратце - отдаёт товары по поиску, категории и другим запросам.

Почти все запросы, которые взаимодействуют с товарами ходят в репорт.
Не только КАПИ/ФАПИ ходят в репорт, чтобы вернуть нам выдачу к примеру, но и другие компоненты ходят в него с целью получить информацию по товарам.
Например чекаутер помимо того, чтобы получить инфу о товаре ещё ходит в `place=actual_delivery`, чтобы получить информацию о способах доставки товара, цене доставки и тому подобного.

Термины:
- у репорта нет ручек. За разнородность ответов отвечает параметр `place`. `place` определяет, что мы ищем
[Список плейсов](https://wiki.yandex-team.ru/testirovanie/functesting/market/marketservice/klaster-brigady-testirovanija-bekendov/report-places/)

Полезности:
- запросы в репорт можно воспроизводить и смотреть выдачу
Зайди через `shh` на `public.market.yandex.net` и через `CURL` дёрни запрос, который вызывает у тебя вопросы

[Дока](https://doc.yandex-team.ru/market/arch-overview/concepts/market-report.html)

### Checkouter

Компонент, который отвечает за заказы:
- проверка возможности оформить заказ с выбранными параметрами (`order/options`)
- выдача информации, с которой можно оформить заказ (доступная доставка, способы оплаты и другое) (`order/options`)
- оформление заказа (`order`)
- генерация ссылки на оплату (`orders/pay`)
- выдача ранее оформленных заказов (`orders`)

Термины:
- бакеты/посылки
Это список объектов shop, которые приходят в ручке `order/options` или `order`.
Бакеты формируются на основании разных складов или типов товаров (`click&collect` и тому подобные).
То есть это посылки, на которые разбилась корзина, так как мы не можем привести все товары разом.
Каждый бакет оформляется отдельным заказов, но оплачиваются вместе. По крайней мере в чекауте.
Не все бакеты могут быть оформлены при вызове ручки `order`. Тогда в ответ приходят `orderId` для тех, которые оформились и ошибки для тех, которые не оформились

Ошибки. Чекаутер может присылать разного рода ошибки, которые могут дать понять, что заказ оформить нельзя.
Также чекаутер может отдавать модификации. К примеру, если мы пытаемся купить товар по одной цене, но его цена измениласть. Тогда чекаутер пришлёт новую ценуу и информацию, что та цена, с который мы к нему пошли изменилась.
Также могут быть ошибки доставки, что в такой город нет доставки, что доставка выбранным типом невозможно, что адрес не включает необходимые поля и тому подобное.

[Дока](https://doc.yandex-team.ru/market/arch-overview/concepts/market-marketplace.html)

### Carter

Картер является частью чекаутера. То есть его пишут и отвечают за него те же люди, что и за чекаутер.
Он отвечает за хранение товаров, которые мы добавили в корзину.

Товары добавляются в корзину по той цене, которую мы послали и если она измениласть, то нам об этом скажет чекаутер при попытке оформить заказ.

Основные ручки:
- добавление в корзину
- удаление из корзины
- получение корзины
- перезапись корзины

[Дока](https://doc.yandex-team.ru/market/arch-overview/concepts/market-marketplace.html)

### Loyalty

Компонент, который отвечает за:
- промокоды
- Маркет Бонусы
- персональные акции
- кешбек

### PERS

Компонент, который отвечает за:
- настройки уведомлений
- оценки и отзывы пользователя
- отзывы и рейтинги магазинов и моделей
- избранные
- списки сравнений

[Дока](https://doc.yandex-team.ru/market/arch-overview/reference/market-pers-pers.html)

### Индексатор

В основном нужен для генерации поисковых индексов для быстрого поиска товаров.
Основной потребитель - репорт.

[Дока](https://doc.yandex-team.ru/market/arch-overview/reference/idx-indexator.html)

### Рекоммендатор

Это подсистема подготовки данных, необходимых для формирования персональных и тематических рекомендаций на Яндекс.Маркете.

[Дока](https://doc.yandex-team.ru/market/arch-overview/concepts/market-recommender.html)

### ABO

ABO (Assessor Back Office) — это подсистема контроля качества, которая выполняет проверку магазинов и товарных предложений, размещающихся на Яндекс.Маркете.

[Дока](https://doc.yandex-team.ru/market/arch-overview/concepts/market-abo.html)

### MBO

Это подсистема управления динамически изменяемым контентом.
По идее, там хранятся многие данные Маркета:
- категории
- параметры для категорий
- карточки моделей
- значения параметров для моделей
- CMS-страницы

[Дока](https://doc.yandex-team.ru/market/arch-overview/concepts/market-mbo.html)

### MBI

MBI (Market Billing and Interface) — это подсистема биллинга и управления размещением товарных предложений магазинов на Яндекс.Маркете.
Нужна для взаимодействия с магазинами, которые продают на Маркете.

[Дока](https://doc.yandex-team.ru/market/arch-overview/concepts/market-mbi.html)

### IR

Подсистема по автоматической подготовке данных.

[Дока](https://doc.yandex-team.ru/market/arch-overview/reference/ir.html)

### Stock Storage (SS)

Компонент, который отвечает за количество товаров на складах.

[Дока](https://wiki.yandex-team.ru/delivery/development/apps/market-stock-storage/)

### LMS

Система по работе со службами доставки.

## Как бекенды обмениваются данными

Так как у нас структура маркета выстроена на основе микросервисов-бекендов, они могут обмениваться между собой данными.
1. Обмен данными посредством HTTP запросов
Самый распространненный способ обмена данными. Бекенды обращаются к друг другу посредством предоставляемых методов API. Есть входные и выходные данные - один бекенд передает в запросе нужные данные, второй бекенд принимает эти данные и на их основе формирует ответ.
2. Получение данных через `Logbroker`
`Logbroker` - сервис передачи потоков данных, подробнее о немпочитать можно [тут](https://logbroker.yandex-team.ru/docs/. Логброкер подразумевает асинхронную модель обмена данными между сервисами. Как работает обмен данными через `logbroker` - сервис 1 создает очередь (топик) в логброкере, в которую отправляет данные, сервис 2 слушает эту очередь и читает из нее сообщения. Модель асинхронная, так как сервис 1 не ждет факта получения сообщений от потребителей. Потребителей у очереди может быть несколько (а может и не быть вообще), иными словами, одну очередь могут слушать несколько читателей.
3. Обмен данными через таблицы `YT` (Один из излюбленных способов подвоза данных от аналитиков).
`YT` - система обработки и хранения больших объемов данных, почитать подробнее можно [тут](https://yt.yandex-team.ru/docs/). Можно использовать, чтобы поделиться частью данных из одного сервиса в другой. Чаще всего подходит для каких-то разовых активностей, например, выгрузить от аналитиков `puid`'ы пользователей с купленной подпиской Я.Плюс, чтобы `loyalty` могли забрать эти данные себе и начислить по этим `puid` бонусы.
