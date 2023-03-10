# Описание сборки Go

## Общая философия описания Go сборки.
Интеграция сборки `GO` в Аркадии реализована с помощью собственной обёртки к низкоуровневым инструментам стандартного GO тулинга (таким как `compile`, `link`, `asm`, `cgo`). Сборка пакетов для `GO`, так же как и для других языков, поддержанных в Аркадии, описывается в ya.make файлах. Сборка пакета должна быть описана явно, то есть, все исходные файлы, участвующие в сборке пакета, должны быть перечислены в вызовах макросов [SRCS](macros.md#srcs), [CGO_SRCS](macros.md#cgo_srcs), а исходные файлы для тестов должны быть перечислены в вызовах макросов [GO_TEST_SRCS](macros.md#go_test_srcs) и [GO_XTEST_SRCS](macros.md#go_xtest_srcs).

Особенностью аркадийной сборки для `GO` (в отличии от сборки для других языков, поддерживаемых в Аркадии) является то, что нет необходимости явно указывать зависимости в вызове макроса `PEERDIR`. Все необходимые зависимости для исходных файла, расположенных в репозитории, вычисляются в момент построения сборочного графа при (частичном) парсинге исходных файлов на `GO`.

Специальные комментарии для сборки, которые умеет обрабатывать стандартный `GO` тулинг, игнорируются в аркадийной сборке и соответствующие им действия должны быть явно описаны в ya.make файлах: например, включение исходных файлов в сборку в зависимости от целевой архитектуры или операционной системы, дополнительные флаги сборки или кодогенерация.
```
OWNER(g:contrib)

GO_LIBRARY()

SRCS(
    common.go
    format.go
    reader.go
    strconv.go
    writer.go
)

IF (OS_DARWIN)
    SRCS(
        stat_actime2.go
        stat_unix.go
    )
ENDIF()

IF (OS_LINUX)
    SRCS(
        stat_actime1.go
        stat_unix.go
    )
ENDIF()

END()
```

{% note info %}

Для автоматической генерации `ya.make` (файл описания сборки) можно воспользоваться специальной утилитой `yo` ([здесь](https://a.yandex-team.ru/arc/trunk/arcadia/library/go/yo/README.md#avtomaticheskaya-generaciya-ya.make) можно прочитать более подробно) 

{% endnote %}

Кодогенерация должна быть реализована стандартными средствами - с помощью макросов кодогенерации, таких как `RUN_PROGRAM`, `PYTHON` etc.
```
OWNER(g:yatool)

GO_PROGRAM()

PEERDIR(
    ${GOSTD}/fmt
)

RUN_PROGRAM(devtools/dummy_arcadia/go/gen_main OUT main.go CWD ${BINDIR})

END()

```

{% note warning %}

Зависимости, которые будут использоваться в сгенерированном коде должны быть явно перечислены в вызове макроса `PEERDIR`. Путь от корня Аркадии до стандартной библиотеки `GO`, используемой в сборке, определён в конфигурационной переменной [GOSTD](vars.md#gostd). 

{% endnote %}

Аркадийный код на языке GO может размещаться в внутри любой корневой директории кроме директорий `contrib` и `vendor`. В корневой директории `vendor` размещается Go код внешний по отношению к Аркадии (3d-party code). Импортировать аркадийный код на `GO` нужно с префиксом `"a.yandex-team.ru/"`. Для импорта внешних зависимостей (то есть, для кода из корневой директории `vendor`) и зависимостей из стандартной библиотеки добавлять префикс не нужно.
```
import (
    "io/ioutil"
    "testing"

    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/require"

    "a.yandex-team.ru/library/go/core/resource"
)
```

В Аркадии сборка для `GO` поддержана только для `Linux`, `Darwin` и `Windows`. По умолчанию, код для целевых платформ `Linux` и `Darwin` собирается с поддержкой CGO (определяется значением конфигурационной переменной [CGO_ENABLED](vars.md#cgo_enabled)).

Для прямых внешних зависимостей в `GO` сборке используется общеаркадийный механизм ограничения использования (PEERDIR policy). Механизм работает по принципу whitelist - в [файле](https://a.yandex-team.ru/arc/trunk/arcadia/build/rules/go/vendor.policy) описаны проекты, которым разрешено использовать contrib-ы из корневой директории `vendor`.  

## Совместимость со стандартным тулингом
Стандартный `GO` тулинг работает с аркадийным кодом. Аркадию перевели в режим `GO модулей` (Как настроить Golang в этом режиме можно прочитать [здесь](https://wiki.yandex-team.ru/devrules/Go/getting-started/#ide)).

{% note info %}

Надо понимать, что стандартный `GO` тулинг ничего не знает об аркадийной сборке. Например, для сборки проекта стандартным GO тулингом могут потребоваться исходные файлы, которые генерируются макросам кодгенерации, или исходные файлы, генерируемые при сборке `PROTO_LIBRARY`. Для этих целей можно воспользоваться дополнительными флагами аркадийной сборки `--add-result` или `--add-protobuf-result` (и, возможно, `--replace-result`).

{% endnote %}

## Модули (ya.make) для описания GO сборки
Для описания сборки пакетов для `GO` в ya.make файлах используются следующие модули: [GO_LIBRARY](modules.md#go_library), [GO_PROGRAM](modules.md#go_program), [GO_DLL](modules.md#go_dll); а для описания тестов: [GO_TEST](modules.md#go_test) и [GO_TEST_FOR](modules.md#go_test_for).

## Про контрибы и т.п.

[https://wiki.yandex-team.ru/devrules/go/](https://wiki.yandex-team.ru/devrules/go/)

