# Обработка запроса

Обработка запроса XScript-ом начинается в момент, когда полученный от пользователя запрос передается в XScript из веб-сервера по протоколу FastCGI.

XScript разбирает запрос (анализирует HTTP-заголовки, куки, URL и т.д.). На основании полученной информации создаются [базовые объекты и структуры XScript](auth-ov.md) (Request, State, CustomMorda и другие). Объекты и структуры XScript содержат различные данные, использующиеся при обработке страницы, и могут передаваться CORBA-компонентам.

Если запрошенная страница не присутствует в [кэше разобранных XML-документов](caching-ov.md), выполняется её разбор и компиляция. При разборе XML-файлов проверка на их соответствие определенным для них схемам (XML Schema и DTD) не производится. Парсер XScript проверяет корректность описания блоков (допустимость указанных в них атрибутов и т.д.), исходное описание блоков преобразуется во внутренние структуры. Страница сохраняется в кэше.

Если в XML-файле указано, что должна производиться авторизация, она производится перед выполнением методов блоков, создаются объекты [Auth и SecureAuth](auth-ov.md) (для паспортной и платежной авторизации соответственно).

Блоки могут выполнять запросы по протоколу CORBA или HTTP или набор действий, не подразумевающий удаленных вызовов. Перед выполнением удаленного запроса проверяется, есть ли в кэше запрашиваемые данные. Если данных нет, выполняется запрос и полученный ответ [кэшируется](block-results-caching.md). Вызовы локальных методов также могут кэшироваться.

Получаемые в ходе обработки запроса данные могут помещаться в контейнер [State](state-ov.md) и извлекаться оттуда для дальнейшей обработки методами блоков или CORBA-сервантами.

Результатом работы блока является XML-фрагмент, который помещается в XML-странице на место исходного кода блока. На результаты работы любых блоков могут накладываться [перблочные XSL-преобразования](per-block-transformation-ov.md) и [XPointer](../appendices/xpointer.md).

К сформированному из результатов работы блоков XML-документу применяется [основное XSL-преобразование](general-transformation-ov.md).

Основное и перблочное XSL-преобразования также могут содержать XScript-блоки.

Готовая страница передается веб-серверу, который выдает её пользователю.

### Узнайте больше {#learn-more}

* [Общий процесс обработки запроса](./request-handling-ov.md)