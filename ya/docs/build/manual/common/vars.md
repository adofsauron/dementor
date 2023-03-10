# Общее для всех языков : переменные


### Переменные для ссылок на файлы и директории { #dir_vars }

Для того, чтобы сослаться на файлы более точно, а также чтобы ссылки на файлы можно было переиспользовать через `INCLUDE` есть следующие переменные:

- `${ARCADIA_ROOT}` -- явное указание корня дерева исходных файлов
- `${ARCADIA_BUILD_ROOT}` -- явное указание корня дерева результатов сборки
- `${MODDIR}` -- путь до директории модуля от корня Аркадии без указания корня.
- `${CURDIR}` -- путь до директории модуля в дереве исходных файлов (директория, где лежит ya.make модуля), соответствует `${ARCADIA_ROOT}/${MODDIR}`.
- `${BINDIR}` -- путь до директории модуля в дереве результатов сборки (директория, куда попадут артефакты сборки модуля), соответствует `${ARCADIA_BUILD_ROOT}/${MODDIR}`.

## Переопределяемые переменные { #D }

Общие переменные, которые можно переопределить из командной строки [`ya make -DVAR`](../../usage/ya_make/index.md#D)

- `AUTOCHECK[=yes|no]` -- Эту переменную выставляет автосборка, иногда нужно выставить её локально, чтобы получить части проектов, стоящие под  `IF AUTOCHECK)`, *как в автосборке*.

{% note warning %}

Перемененная не влияет на тип сборки (в автосборке для Linux используется `relwithdebinfo`) и дополнительные настройки (такие как `DEBUG_INFO_LINES_ONLY` и `USE_EAT_MY_DATA`), используемые в автосборке.
Она влияет только на условные конструкции, завязанные на саму переменную.

{% endnote %} 

- `DEBUG_INFO_LINES_ONLY[=yes|no]` -- уменьшает объём отладочной информации в программах на C++ и Python.

- `STRIP[=yes]`-- удалить отладочную информацию из программ. В C++ и go отладочная информация по умолчанию всегда включена.

- `NO_STRIP[=yes|no]` -- не удалять отладочную информацию из программ. В Python по умолчанию отладочная информация всегда выключена.


## Переменные платформ и типов сборки { #platform_vars }
```
OS_WINDOWS/OS_LINUX/OS_DARWIN ...
DEBUG/RELEASE/SANITIZER_TYPE...
```

## Переменные управляющие проверками лицензий

### `DEFAULT_MODULE_LICENSE`

Имя лицензии используемой по умолчанию для модулей, не указавших свою лицензию явно через макрос `LICENSE`. Переменная не может быть переопределена в модуле, лицензия по умолчанию задаётся на всю аркадию в [`ymake.core.conf`](../../extension/core_conf.md).

### `LICENSE_PROPERTIES`

Список всех свойств, которые могут быть использованы в макросе `RESTRICT_LICENSES` для описания ограничений на лицензии прямых и транзитивных зависимостей модуля. Для каждого свойства из этого списка определяются три переменных:

 * `LICENSES_<PROP_NAME>_STATIC`: список лицензий для которых это свойство налагает ограничения на потребителя, линкующегося с лицензированным кодом статически.
 * `LICENSES_<PROP_NAME>_DYNAMIC`: список лицензий для которых это свойство налагает ограничения на потребителя, линкующегося с лицензированным кодом динамически.
 * `LICENSES_<PROP_NAME>`: список лицензий для которых это свойство налагает ограничения на всех потребителей. Указание лицензии в этом списке полностью эквивалентно указанию её в двух списках из предыдущих пунктов. Данная переменная создана для удобства и устранения дублирования в конфиге.

...

