# Java : переменные

## Переопределяемые переменные { #D }

Переменные, которые можно переопределить из командной строки [`ya make -DVAR`](../../usage/ya_make/index.md#D)

### JDK_VERSION

`JDK_VERSION=<ver>` -- Собирать с использованием определённой версии JDK. На данный момент поддерживаются 8, 10-16 и 17 (по умолчанию)

### KOTLIN_VERSION

`KOTLIN_VERSION=<ver>` -- Собирать с использованием определённой версии Kotlin. На данный момент поддерживаются 1.3.71, 1.3.72, 1.4.20, 1.4.30, 1.6.21 (по умолчанию).

### USE_SYSTEM_JDK

`USE_SYSTEM_JDK=/path/to/system/jdk/java_home` -- Собирать с использованием указанного стороннего JDK установленного в системе.

### USE_SYSTEM_JDK_FOR_TESTS

`USE_SYSTEM_JDK_FOR_TESTS=/path/to/system/jdk/java_home` -- Использовать установленную в системе JDK для запуска тестов.
