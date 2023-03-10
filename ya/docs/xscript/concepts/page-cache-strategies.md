# Стратегии кэширования XML-страниц

Стратегии кэширования позволяют сформировать ключ кэша при кэшировании XML-страниц. Стратегии описываются в [конфигурационном файле XScript](../appendices/config.md) и подключаются в исходном XML-файле.


## Описание стратегии кэширования {#desc}

Описание стратегии кэширования в конфигурационном файле имеет следующий вид:

```
<xscript>
   ...
   <page-cache-strategies>
      <strategy id="my_cache_strategy">
         <query sort="no">var1,var2,var3</query>
         <cookie>service_id1,service_id2</<cookie>
         <cookie-my>15,17</cookie-my>
         <region default="100">23,578</region>
      </strategy>
      ...
   </page-cache-strategies>
   ...
</xscript>
```

Каждая стратегия кэширования описывается в отдельном теге `<strategy>` внутри тега `<page-cache-strategies>`. Атрибут `id` тега `<strategy>` является именем стратегии - по нему производится обращение к стратегии из XML-файла.

Каждая строка внутри тега `<strategy>` представляет собой описание подстратегии кэширования, которая позволяет сформировать часть ключа кэша. Впоследствии полученные части кэша конкатенируются, образуя полный ключ кэширования. Для формирования кэша можно использовать любые доступные подстретегии в любом порядке.

Доступны следующие типы подстратегий:

#### **\<cookie\>**

   Куки запроса, которые будут участвовать в формировании ключа кэширования.

   Запрещено использовать в ключе куки Session_id, Secure_session_id, L, yandexuid, yabs-frequency, yandex_gid, yandex_login, my.

#### **\<cookie-my\>**

   Блоки куки my, которые будут участвовать в формировании ключа кэширования.

#### **\<query\>**

   Параметры запроса, которые будут участвовать в формировании ключа кэширования.

   Если тело тега `<query>` пустое, в ключ войдут все параметры запроса.

   Если тег `<query>` в стратегии отсутствует, параметры запроса в ключ не войдут.

   В теге `<query>` можно указать атрибуты _sort_ и _except_ со значениями "yes" или "no". Если атрибуту _sort_ присвоено значение "yes", параметры перед добавлением в ключ сортируются по имени, а при совпадении имен - по значению. Если значение атрибута - "no", сортировка не выполняется - параметры включаются в ключ в том порядке, в котором они следуют в запросе. По умолчанию сортировка включена.

   Если атрибуту _except_ присвоено значение "yes", при формировании ключа будут использоваться все параметры запроса, кроме указанных.

#### **\<region\>**

   Содержит список идентификаторов регионов. Если пользователь пришел из одного из указанных регионов, идентификатор этого региона будет добавлен в ключ. Если пользователь пришел из региона, входящего в один из указанных (например, указан идентификатор Московской области, а пользователь пришел из Мытищ), идентификатор родительского региона (то есть Московской области) будет добавлен в ключ.

   Если тело тега пустое, в ключ будут добавляться любые регионы.

   В теге `<region>` можно указать атрибут **default** - идентификатор региона, который будет добавлен в ключ, если пользователь не принадлежит ни одному из указанных в теге регионов. Если атрибут не указан, по умолчанию будет использован регион с идентификатором 1000 (Земля).


## Подключение стратегии кэширования {#call}

Для подключения стратегии кэширования в исходном XML-файле в теге `<xscript>` необходимо добавить атрибут **cache-strategy**. Например:

```
<xscript cache-strategy="my_cache_strategy:300">
```

Значение атрибута определяет название стратегии кэширования (до двоеточия) и время кэширования в секундах (после двоеточия).


## Стандартные стратегии кэширования {#standard-strategies}

Существует возможность использовать стандартные стратегии кэширования, распространяемые в пакетах [xscript-cache-strategies](packages.md#cache-strategies) и [xscript-multiple-cache-strategies](packages.md#cache-strategies-multiple).

В стандартный набор входят следующие стратегии:

#|
|| **Имя стратегии** | **Назначение** ||
|| common | Кэширование страницы по URL-у без учета параметров запроса (query). ||
|| common-query | Кэширование страницы по полному URL-у (с query), исключая параметры "\_", "ncrnd", "rnd", "wauth". ||
|| common-region | Кэширование страницы по регионам и URL-у без query. ||
|| common-region-query | Кэширование страницы по регионам и полному URL-у, исключая параметры "\_", "ncrnd", "rnd", "wauth". ||
|| common-yandex_countries | Кэширование по странам присутствия Яндекса (Россия, Украина, Беларусь, Казахстан, Земля) и URL-у без query. ||
|| common-yandex_countries-query | Кэширование по странам присутствия Яндекса (Россия, Украина, Беларусь, Казахстан, Земля) и полному URL-у, исключая параметры "\_", "ncrnd", "rnd", "wauth". ||
|#

Исходный код стандартных стратегий в SVN: [https://svn.yandex.ru/corba-configs/trunk/xscript-cache-strategies/cache-strategies.xml](https://svn.yandex.ru/corba-configs/trunk/xscript-cache-strategies/cache-strategies.xml).

### Узнайте больше {#learn-more}
* [Кэширование XML-страницы](../concepts/caching-ov.md)
* [Кэширование результата работы XScript-блока](../concepts/block-results-caching.md)