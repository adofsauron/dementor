# Введение

{% note info %}

В этом разделе дается довольно высокоуровневое введение в систему сборки исходного кода. Этого должно быть достаточно для простых сценариев. Подробная документация расположена [здесь](https://docs.yandex-team.ru/ya-make/).

{% endnote %}

## Немного предыстории { #history }

Единый репозиторий [Arcadia](https://a.yandex-team.ru/) существует с 1996 года. Первыми языками программирования, на которых писался код в едином репозитории, были [С](https://en.wikipedia.org/wiki/C_(programming_language)) и [С++](https://en.wikipedia.org/wiki/C%2B%2B). Как правило, большие проекты на этих языках программирования состоят из множества файлов с исходными кодами и требуют выполнения множества различных сложных команд, чтобы получить итоговый исполняемый файл. По историческим причинам одним из распространенных инструментов решения этой проблемы для C \ C++ является утилита [make](https://en.wikipedia.org/wiki/Make_(software)), использующая файлы [Makefile](https://en.wikipedia.org/wiki/Makefile) для хранения необходимой последовательности действий. Во многих случаях Makefile оказываются однотипными и становится возможно их генерировать. Одним из инструментов, позволяющих генерировать Makefile и другие подобные конфигурационные файлы, является [CMake](https://cmake.org/). Для хранения своей конфигурации CMake использует файлы формата **CMakeLists.txt**.

До тех пор, пока в едином репозитории были преимущественно проекты на С\С++, использовался **CMake** и конфигурация описывалась в файлах **CMakeLists.txt**. Со временем появились проекты на других языках программирования (например, Python и Java), и CMake перестало хватать. Пользователям единого репозитория хотелось по-прежнему описывать сборку проектов на всех языках программирования единым образом. Кроме этого CMake на постоянно растущем репозитории работал очень медленно. Поэтому было принято решение сделать собственный инструмент сборки под названием **ya make**, а конфигурацию для этого инструмента описывать в текстовых файлах **ya.make**, синтаксис которых похож на **CMakeLists.txt**.

## Поддерживаемые языки программирования { #languages }

Несмотря на то, что в едином репозитории [разрешено](../rules/intro.md#allowed-languages) использовать довольно много языков программирования, инструмент сборки **ya make** поддерживает только часть из них:

* С \ С++
* Go
* Java \ Kotlin
* Python 2 и 3

## Поддерживаемые платформы { #platforms }

{% note info %}

Список поддерживаемых платформ важен только для языков программирования, исходный код которых собирается в двоичный исполняемый файл. Например, для Java и Kotlin поддерживаются все платформы, на которые можно установить [JVM](https://en.wikipedia.org/wiki/Java_virtual_machine). Полный список поддерживаемых компиляторов, архитектур и платформ располагается [здесь](https://docs.yandex-team.ru/ya-make/usage/ya_make).

{% endnote %}

Платформа | Архитектуры
:--- | :---
`Android` | `armv7a`, `armv7a_neon`, `armv8a`, `i686`
`Cygwin` | `x86-64`
`Darwin` (MacOS) | `x86-64`
`iOS` | `i386`, `x86-64`, `armv7`, `arm64`
`Linux` | `x86-64`
`Windows` | `x86-64`

Основными архитектурами являются: `Darwin x86-64`, `Linux x86-64` и `Windows x86-64`.
