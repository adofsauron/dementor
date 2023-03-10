# xpath

Извлекает из выдачи данного блока значение типа boolean, number или string, адресованное с помощью [ XML Path Language ](http://www.w3.org/TR/xpath) (XPath), и помещает его в переменную контейнера [State](../concepts/state-ov.md) типа boolean, double или string соответственно.

Если XPath-выражением извлекается несколько строковых значений, перед помещением в State они конкатенируются. Если множество строк пустое, то соответствующий State не создается.

Если в теге блока присутствует атрибут _xmlns_, при обработке XPath-выражения будет использоваться пространство имен, указанное в этом атрибуте.

Блок может содержать несколько тегов `<xpath>`.

Если в результате своего выполнения блок возвращает ошибку [<xscript_invoke_failed>](../concepts/error-diag-ov.md), XPath-выражение не выполняется.

## Содержит {#contains}

Данный тег не может содержать других тегов.

## Содержится в {#contained-in}

[auth-block](auth-block.md), [banner-block](banner-block.md), [block](block.md), [custom-morda](custom-morda.md), [file](file.md), [geo](geo.md), [http](http.md), [local](local.md), [lua](lua.md), [mist](mist.md), [while](while.md).

## Атрибуты {#attrs}

#|
|| Наименование | Описание | Тип и варианты значения | Значение по умолчанию ||
|| delim | Символ, используемый в качестве разделителя при конкатенации строковых значений. | Строка. | " " (пробел) ||
|| expr | Выражение, адресующее на языке XPath строку в XML-документе. Если задан атрибут `type`, и ему присвоено значение "StateArg", значением данного атрибута должно быть имя переменной в State, содержащей XPath-выражение. | Строка | - ||
|| result | Наименование переменной в контейнере State, в которой сохраняется адресованное по XPath строковое значение. | Строка. | - ||
|| type | Может иметь только значение "StateArg", являющееся индикатором того, что должно быть выполнено XPath-выражение, содержащееся в переменной контейнера State, имя которой указано в атрибуте `expr`. | Строка - "StateArg". | - ||
|| xmlns | Пространство имен, которое будет использоваться при обработке XPath-выражения. Имеет приоритет перед пространством имен, указанным в корневом теге блока. |  | ||
|#

## Примеры {#examples}

В приведенном ниже примере текстовые представления XML-нод конкатенируются через точку.

```
<mist>
    <xpath expr="/state/@type" result="result_expr" delim="."/>
    ...
</mist>
```

В приведенном ниже примере выполняется XPath-выражение, указанное в переменной контейнера State `xpath_for_region`, а результат сохраняется в переменной `region`.

```
<http xmlns:dd="http://www.yandex.ru/dd">
     <method>getHttp</method>
     <param type="String">http://devel.yandex.ru/index.xml?region=</param>
     <param type="StateArg" as="String">parent</param>
     <xpath expr="xpath_for_region" type="StateArg" result="region"/> </http>
```

