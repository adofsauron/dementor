# Внесение правок во внешние библиотеки

## Rationale {#rationale}
Поскольку разработчики в Яндексе экономят своё время и не страдают тяжёлой формой [NIH](https://en.wikipedia.org/wiki/Not_invented_here), то в аркадии широко используются внешние библиотеки и программы с открытым исходным кодом: порядка пары тысяч C/C++-библиотек, полутора тысяч python-библиотек, трёх тысяч java-библиотек и пяти тысяч go-библиотек (400 в `contrib/` и остальное в `vendor/`).
В библиотеках бывают баги, и маленький фикс может провисеть в состоянии pull request'а не один месяц, а рефакторинг, ускоряющий работу, и вовсе не попасть в апстрим по идеологическим соображениям.
Получается, что объективная небходимость в патченных контрибах присутствует, но нет регламента по их завозу в аркадию.
Данный документ представляет собой инструкцию:
  * Как поступать, если потребовалось пропатчить opensource библиотеку или программу.
  * Как сделать, чтобы не пострадали посторонние.
  * Каких практик придерживаться, чтобы обновление на новую версию происходило без лишней боли.

Дабы сэкономить буквы, далее по тексту все opensource библиотеки и программы завозимые в аркадию будем называть контрибами.

### Виды контрибов {#types}
Зафиксируем терминологию, чтобы далее экономить буквы и избегать путаницы:
1. **Ванильные контрибы** — исходный код данных контрибов не менялся, за исключением правок необходимых для подключения в систему сборки.
2. **Контрибы с фиксами** — в исходный код данных контрибов внесены фиксы багов и уязвимостей (т.е. поведение контриба совпадает с ожиданиями пользователя) и прочие изменения не меняющие поведение контриба для внешнего наблюдателя (например, оптимизации скорости). Решение о том, является изменение меняющим поведение или нет, принимает соответствующий языковой комитет.
3. **Патченные контрибы** — в исходный код данных контрибов внесены изменения меняющие ABI (сборка у нас статическая только с допущениями, `dlopen` всё же много где есть), либо же меняющие поведение контриба (например, изменение формата вывода/хранения данных, нарушение совместимости с rfc и прочие изменения, которые не совпадают с ожиданиями пользователя, знакомого с ванильным контрибом)

Поскольку контрибы с фиксами неотличимы для пользователя от ванильного контриба, то далее по тексту под ванильными контрибами будем понимать и те и другие, если явно не указаны отличия для контрибов с фиксами.

### Основные боли и опасения по поводу патченных контрибов {#issues}
1. Патченный контриб или контриб с фиксами сложно обновить на новую версию, не потеряв какой-то из патчей, наложенных на оригинальный контриб. Какое-то мелкое изменение может затеряться, особенно если в новой версии контриба место патча поменялось, а мелкое изменение было наложено поверх другого патча.
2. Если патч меняет поведение или ABI, и контриб случайно приедет по зависимостям в те проекты, которые к этому не готовы, то в продакшене может случиться конфуз. А если по зависимостям приедут и патченная библиотека и непатченная, то может случиться факап.
3. Комитетам и с обычными контрибами возни хватает, а уж заниматься поддержкой кода, пропатченного другой командой, им точно будет некогда.
4. Иногда в новых версиях библиотек ломается старый функционал, и люди осознанно живут на старой версии. При этом двум разным командам могут оказаться нужны разные версии одной библиотеки, потому что используются разные модули этой библиотеки.
5. Разным командам в рамках своих продуктовых задач могут потребоваться патчи, противоречащие друг другу.
6. В некоторых исключительных случаях, требуется использование в одной программе и патченного и ванильного контриба.

### Основные постулаты {#foundation}
1. Целью данного документа является уменьшение боли и неопределённости при завозе новых контрибов, а не увеличение неудобств и размера техдолга за счёт старых контрибов. Поэтому публикация данного документа не означает, что в обозримом будущем будут заводиться субботники по наведению порядка в старых контрибах, которые данному документу противоречат.
2. По умолчанию должен быть запрещён пирдир любого патченного контриба, кроме тех проектов, которые явно захотели запирдирить этот контриб. Также должны быть предприняты меры по запрету транзитивного пирдира.
3. Ответственность за патченный контриб лежит на команде, которая его принесла.
4. Патченный контриб, как правило, никому не нужен, кроме той команды, которая его принесла.
5. В контрибах могут лежать несколько версий патченного контриба. Для Java и Go это не является большой проблемой, но для остальных языков добавление нескольких версий одной библиотеки должно быть согласовано с языковым комитетом.
6. Патченный контриб всё ещё является контрибом, и добавление патченного контриба должно проходить [стандартную процедуру добавление библиотеки в contrib](./add), с тем лишь исключением, что всю работу по завозу контриба должна выполнять команда, которой понадобился патченный контриб.
7. Необходимость наложить на контриб патчи, меняющие его поведения, происходит из продуктовых задач команды, которая эти патчи накладывает. Поэтому библиотеки общего назначения патченные контрибы использовать не должны.

## Правила работы с патченными контрибами {#rules}

### Изменения в правилах работы с контрибами {#ruleschanges}
1. В `ya.make` всех контрибов нужно указывать директиву [VERSION](https://a.yandex-team.ru/arc/trunk/arcadia/build/docs/readme.md#macro_VERSION), чтобы пользователи контриба знали, какую версию контриба они используют без заглядывания в историю правок. `VERSION` представляет собой либо номер версии (например `0.1.2`), либо (если код взят из мастера/транка) номер релиза/релизной ветки + номер ревизии (например `0.1.2-r123456` или `0.1.2-d38f1f2c`). Для патченных контрибов также добавляется суффикс `+patched`.
2. В `ya.make` всех контрибов (кроме Go) нужно указывать директиву [PROVIDES](https://a.yandex-team.ru/arc/trunk/arcadia/build/docs/readme.md#macro_PROVIDES), чтобы сразу находить ситуации, когда в один бинарь приехал и ванильный и патченный контриб. Как поступать в ситуациях, когда нужны и ванильный и патченный контрибы в одном бинаре, будет рассказано в разделе по [разрешению конфликтов](#conflicts).
3. Для контрибов с фиксами и для патченных контрибов все изменения нужно складывать в подпапку `patches` и патчи должны иметь числовой префикс для гарантированного порядка наложения, например `20-Faster-base64-decoding.patch` или `30-AVX2-base64-encoding.patch`. Данный подход позволяет использовать [yamaker](https://a.yandex-team.ru/arc/trunk/arcadia/devtools/yamaker) для (полу)автоматического добавления и обновления контрибов.

### Расположение кода патченных контрибов {#location}
1. Патченные контрибы располагаются по пути `contrib/<language>/patched/<contrib-name>` (для C/C++ в `contrib/libs/patched/<contrib-name>` и `contrib/tools/patched/<contrib-name>`).
2. Патчи следует делать относительно корневой папки библиотеки так, чтобы патчи можно было наложить при помощи `patch -p1`.
3. Если к контрибу нужны противоречащие патчи от двух разных команд или разные версии одного контриба, то следует к имени контриба дописать соответствующие суффиксы (например `rapidjson_maps` и `rapidjson_passport`) и к `VERSION` дописать те же суффиксы. К `PROVIDES` дописывать суффикс не следует, покуда не возникла необходимость в [разрешении конфликтов](#conflicts).
4. Для любого изменения в патченном контрибе или в контрибе с фиксами следует добавлять новый файл в `patches`. <details><summary>Зачем это нужно:</summary>
tl;dr: Для удобного обновления контриба и отслеживания истории.
При обновлении контриба на новую версию требуется наложить на новую версию все старые патчи. Собрать все старые патчи можно было бы по `arc log`, но это требует ручных действий, и к тому же не очень удобно, если в один PR попало несколько коммитов (удобно скачать отдельные коммиты арканум в этом случае не позволяет, а `interdiff` установлен в Яндексе у единиц).
Куда проще положить папку с новой версией и наложить на неё все патчи из папки, а после этого удалить старую папку.
При данном подходе также сохраняется возможность отследить, зачем нужен тот или иной патч, без лишнего копания в истории: по `arc log` на патче можно адресно найти PR, в котором он был добавлен, открыть описание соответствующего review и даже почитать связанные тикеты. Также, при протаскивания патча в апстрим, достаточно будет удалить один конкретный файл с этим патчем.</details>
5. Патченные контрибы могут иметь зависимости от кода Аркадии за пределами `contrib/`.

### Порядок добавления патченного контриба, меняющего поведение или API {#add}
1. Добавление нового патченного контриба происходит согласно [процедуре добавления библиотеки в contrib/](./add).
2. После того как OK от языкового комитета получен, контриб помещается в папку с патченными контрибами (см. выше), в папку `patches` помещаются все патчи для контриба, патчи накладываются на контриб и контриб подключается к автосборке. Наложение патчей и подключение к автосборке для контрибов на C и C++ рекомендуется делать при помощи утилиты [yamaker](https://a.yandex-team.ru/arc/trunk/arcadia/devtools/yamaker), для Go — при помощи утилиты [yo](https://a.yandex-team.ru/arc/trunk/arcadia/library/go/yo) (но патчи для Go придётся накладывать вручную), для Python — при помощи [pip2arc](https://a.yandex-team.ru/arc/trunk/arcadia/devtools/pip2arc/README.md) (патчи тоже придётся накладывать вручную).
3. Затем в `build/rules/<language>/contrib.policy` перед строкой `DENY .* -> contrib/<language>/patched/` дописывается явное разрешение подключать новый контриб из проекта, который собирается его использовать, и следующей же строкой — запрет подключения этого проекта кем-либо. Это делается для того, чтобы патченный контриб случайно не приехал по транзитивным зависимостям, и чтобы избежать ситуации, как в примере ниже, когда аркадийная библиотека является просто обёрткой над парсером, и при переходе на патченный парсер также меняется и поведение библиотеки-обёртки. Перед этим блоком следует в комментарии указать CONTRIB-тикет, в рамках которого контриб был одобрен, и ответственная за контриб команда. Например, вот так:
```
## CONTRIB-12356. Responsible: g:global-search, g:factextract
ALLOW mail/cpp/json/json_dom -> contrib/cpp/patched/json-c
DENY .* -> mail/cpp/json/json_dom
```
Если транзитивные зависимости на этот проект нужны, то они также явным образом перечисляются в `contrib.policy`. Дополнительно прописываются разрешение на использование проекта и запрет подключения транзитивной зависимости. Например, вот так:
```
## CONTRIB-12356. Responsible: g:global-search, g:factextract
ALLOW mail/cpp/json/json_dom -> contrib/cpp/patched/json-c
ALLOW yweb/mail/json_normalizing_server -> mail/cpp/json/json_dom
DENY .* -> mail/cpp/json/json_dom
DENY .* -> yweb/mail/json_normalizing_server
```

### Порядок обновления патченного контриба {#update}
1. Обновляется исходный код контриба. Также обновляется `ya.make` (в том числе `VERSION`)
2. На новый код накладываются все патчи. Если при наложении какого-либо из патчей появляются смещения или конфликты, то файл патча следует обновить, дабы в будущем не появлялось новых `*.orig`-файлов.
3. После этого старый код удаляется, и создаётся PR. Подтверждение комитетов на обновление контриба не требуется.

### Обязанности сторон {#responsibility}
Владельцы патченного контриба берут на себя обязательства по:
1. Закрытию security issues, найденных в контрибе.
2. В случае, если контриб имеет зависимости на аркадию и в аркадии поменялось API, на которое он завязан, то владельцы патченного контриба должны в скорейшие сроки перейти на новую версию API.
3. В случае, если контриб имеет зависимость на другой контриб, который был обновлён и в котором поменялось API, то владельцы патченного контриба должны в скорейшие сроки перейти на новую версию API. В случае, если новая версия контриба не подходит (например в ней замедлили время работы или оторвали необходимый функционал), то по согласованию с языковым комитетом можно положить фиксированную старую версию контриба в `contrib/<language>/patched`. Процесс завоза фиксированной версии контриба в этом случае тот же, что и для завоза контрибов с патчами, меняющими поведение, с запретом случайных пирдиров и пр. Отличием будет отсутствие папки `patches` для этой версии. <details><summary>Почему так:</summary>
Представим себе, что патченный контриб зависит от библиотеки glibc-2.13 и использует функцию `memcpy` из неё во множестве мест, причём использует неправильно и буферы имеют пересечение.
Допустим, что в аркадии решили обновить glibc до версии 2.14, где работа `memcpy` стала сильно быстрее `memmove`, но имеет undefined behaviour для тех вызовов, которые нарушают требование по отсутствию пересечения буферов копирования.
Когда пользователи решат почитать документацию или результаты бенчмарков `memcpy`, то они вероятнее всего откроют информацию по последней версии библиотеки и будут рассчитывать на высокую производительность. Поэтому нельзя допустить, чтобы они случайно запирдирили старую версию библиотеки.</details>
4. В случае наличия патченного контриба и появления необходимости в ванильном контрибе в том же бинаре, что использует патченный, на владельцев патченного контриба ложится задача по скорейшему [разрешению возникших конфликтов](#conflicts).
5. На авторов патчей, не менящих поведение контриба, возлагается обязанность по донесению патча до апстрима. Попадание в апстрим с одной стороны является индикатором полезности патча, а с другой — снимет необходимость по патчингу контриба в будущем.

### Разрешение конфликтов `PROVIDES` {#conflicts}
В случае, когда в одном бинаре необходимо использование как ванильного так и патченного контриба, необходимо прибегнуть к одному из следующих языкоспецифичных способов разрешения конфликтов. Данные способы не являются ни удобными, ни безболезненными, но пока они видятся как единственные для достижения цели.
Вне зависимости от языка программирования (кроме Go), **после** выполнения указанных действий, к `PROVIDES` следует дописать суффикс (как указано [здесь в п.3](#location) ) для того чтобы конфликта больше не возникало.

#### С {#cconflicts}
1. Дописать префикс `yandex_patched_` ко всем глобальным переменным и функциям в контрибе.
2. Если патченный контриб в `ya.make` использует `ADDINCL(GLOBAL ...)`, то необходимо все инклуды в коде заменить на инклуды от корня аркадии и `ADDINCL(GLOBAL ...)` удалить.

#### С++ {#cppconflicts}
1. Все исходники патченного контриба обернуть в `namespace NPatched {...}`.
2. Если патченный контриб в `ya.make` использует `ADDINCL(GLOBAL ...)`, то необходимо все инклуды в коде заменить на инклуды от корня аркадии и `ADDINCL(GLOBAL ...)` удалить.

#### Java {#javaconflicts}
Все `package`'s патченного контриба префиксовать `ru.yandex.patched.`.

### Языковые особенности {#languagespecific}

#### Java {#javaspecific}
  1. С целью поддержания единообразия структура папок в `contrib/java/patched` должна быть той же, что и в `contrib/java`. Например, патченная библиотека `tika-core` должна находиться в `contrib/java/patched/org/apache/tika/tika-core/<version>`, а патчи должны располагаться в `contrib/java/patched/org/apache/tika/tika-core/patches`.
  2. Поскольку в большинстве случаев Java-библиотеки помещаются в `contrib/` в виде `ya.make` с `EXTERNAL_JAR`, допускается в `contrib/java/patched` также помещать не код, а `EXTERNAL_JAR`. При этом все патчи должны присутствовать в папке `patches`, и, после наложения данных патчей на тарбол, должна работать стандартная процедура сборки этого тарбола. Описание jar-файла, загружаемого в sandbox, должно содержать `<version>`. Например, `tika-1.22-d38f1f2c_patched`.
  3. В случае, когда из одного тарбола собираются сразу несколько библиотек, папку `patches` следует перенести в соответствующую папку уровнем выше. Например, если внесены изменения в API `tika-core`, а в проекте необходимо использование `tika-parsers`, собираемого из того же тарбола, то необходимо:
    1. Поместить патчи в папку `contrib/java/patched/org/apache/tika/patches`.
    2. Поместить в папку `contrib/java/patched/org/apache/tika/tika-core/<version>` собранный `EXTERNAL_JAR` для `tika-core`.
    3. Поместить в папку `contrib/java/patched/org/apache/tika/tika-parsers/<version>` собранный `EXTERNAL_JAR` для `tika-parsers` и прописать зависимость на `tika-core`.
  4. Если контриб приезжает как `EXTERNAL_JAR`, рядом с папкой `patches` необходимо разместить `README.md` с инструкцией по сборке и информацией о том, где брать тарбол, чтобы облегчить обновление контриба и добавление новых патчей.
  5. Если контриб приезжает как `EXTERNAL_JAR`, в `README.md` должна присутствовать ссылка на sandbox-ресурс по которой можно скачать тарбол из которого собирался данный `EXTERNAL_JAR`. Делается это для того, чтобы у нас всегда был исходный код проекта, даже если проект умрёт и умрут все его зеркала с ресурсами.

#### Go {#gospecific}
В Go нет проблем с добавлением нескольких "одинаковых" контрибов в один бинарник. Это обусловлено тем, что в Go "путь" импорта является уникальным идентификатором пакета. С точки зрения Go абсолютно идентичные библиотеки находящиеся по разным путям являются абсолютно разными и несовместимыми библиотеками, их "неймспейсы" никак не пересекаются.

В связи с вышесказанным Go не нуждается в специальных "защитных" мерах.
