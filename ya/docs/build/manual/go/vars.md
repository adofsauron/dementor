# Go : переменные


## Переопределяемые переменные { #D }

Переменные, которые можно переопределить из командной строки [`ya make -DVAR`](../../usage/ya_make/index.md#D)

Переменные управляющие сборкой GO:
- [GOSTD](#gostd)
- [GOSTD_VERSION](#gostd_version)
- [CGO_ENABLED](#cgo_enabled)
- [GO_DEBUG_PATH_RELATIVE](#go_debug_path_relative)
- [GO_VET](#go_vat)
- [GO_VET_FLAGS](#go_vet_flags)
- [GO_VET_TOOL](#go_ve_tool)
- [GO_COMPILE_FLAGS](#go_compile_flags)
- [GO_LINK_FLAGS](#go_link_flags)

### GOSTD
Конфигурационная переменная `GOSTD` определяет путь от корня Аркадии до стандартной библиотеки `GO`.

### GOSTD_VERSION
Конфигурационная  переменная `GOSTD_VERSION` определяет версию стандартной библиотеки `GO` и тулчейна, которая будет использоваться для `GO` сборки. В Аркадии поддерживается правило "двух версий" для стандартной библиотеки и тулчейна. Это правило заключается в том, что кроме текущей версии стандартной библиотеки и тулчейна можно локально или в задачах `sandbox` построить проект с предыдущей версией (в редких случаях, вместо предыдущей версии доступна 'новая' версия стандартной библиотеки и тулчейна - версия на которую в ближайшее время будет переключена умолчательная сборка на `GO`).
Для того чтобы построить `GO` проект с версией отличной от умолчательной нужно на командной строке добавить следующие флаги: `-DGOSTD_VERSION=<version> --host-platform-flag=GOSTD_VERSION=<version>`, где `<version>` - это одна из поддерживаемых в Аркадии версий стандартной библиотеки и тулчейна.

### CGO_ENABLED
Модульная переменная `CGO_ENABLED` определяет включение/выключение поддержки `CGO` для сборки `GO` модуля. В аркадийной сборке для `GO` по умолчанию включена поддержка `CGO` для Linux и Darwin. В том случае, если необходимо проверять собираемость проекта без поддержки `CGO` в автосборке (только для Linux), проект должен быть внесен в соответствующие автосборочный [конфигурационный файл](https://a.yandex-team.ru/arc/trunk/arcadia/autocheck/linux/cgo_targets.inc). Для того чтобы построить `GO` проект без поддержки `CGO` нужно на командной строке добавить следующие флаги: `-DCGO_ENABLED=no`.

### GO_DEBUG_PATH_RELATIVE
Конфигурационная переменная `GO_DEBUG_PATH_RELATIVE` позволяет изменить умолчательную генерацию путей для отладочной информации. Сейчас умолчательное поведение генерации путей для локальной сборки определяется следующим образом: абсолютные пути для файлов исходного кода из репозитория и префикс билдовой директории для путей сгенерированных файлов заменяется на префикс `/-B`, для сборки на дистбилде префикс пути до репозитория для файлов исходного кода из репозитория заменяется на префикс `/-S` и префикс билдовой директории для путей сгенерированных файлов заменяется на префикс `/-B`. Если в локальной сборке по каким-либо причинам хочется чтобы пути в отладочной информации были относительными: для файлов из репозитория относительны корня репозитория, а для сгенерированных файлов - относительно билдовой директории, то на командной строке нужно добавить флаг `-DGO_DEBUG_PATH_RELATIVE=yes`.

### GO_VET
Конфигурационная переменная `GO_VET` позволяет управлять использованием `vet` тула для сборки GO проектов. Значение переменной `GO_VET` может принимать следующие значения: `yolint`, `yolint_next`, `yes`, `no`, `local`. По умолчанию в аркадийной сборке `GO_VET` имеет значение `yolint` (т.е. в качестве `vet` тула используется аркадийный линтер `yolint`, поддерживаемый GO комитетом. `yolint_next` - определяет следующую версию yolint). Значение `yes` переключит GO сборку на использование 'стандартного' `vet` тула. Значение `no` отключит использование `vet` тула в сборке. Значение `local` позволит использовать кастомный `vet` тул (см. [GO_VET_TOOL](#go_vet_tool)).

### GO_VET_FLAGS
Конфигурационная переменная `GO_VET_FLAGS` позволяет передать дополнительные флаги для используемого `vet` тула. Для того чтобы передать дополнительные флаги для `vet` тула нужно на командной строке добавить флаг `-DGO_VET_FLAGS=<additional flags for vet tool>`.

### GO_VET_TOOL
Конфигурационная переменная `GO_VET_TOOL` позволяет переопределить `vet` тул для локальной сборки. Значение переменной `GO_VET_TOOL` должно содержать абсолютный путь до желаемого `vet` тула (`-DGO_VET_TOOL=<absolute path to custom vet tool>`).

### GO_COMPILE_FLAGS
Флаги компиляции .go-файлов

### GO_LINK_FLAGS
Флаги линковки go-модулей
