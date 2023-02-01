# Описание сборки Python

В Аркадии реализована сборка Python в замкнутые [*программы*](#arcadia_python). Такая сборка является замкнутой, т.е. требуют описания всего своего исходного кода и зависимостей.
Результатом сборки является исполняемая программа под целевую платформу. Она содержит код интерпретатора Python c модифицированным загрузчиком,
а также прекомпилированный Python код программы и всех её зависимостей.

На основе сборки Python можно создавать [виртуальное окружение](../../usage/ya_ide/venv.md).

Кроме сборки в *программы*  ya make позволяет собирать динамически загружаемые бинарные [*модули*](#py_modules) - внешние расширения для использования с обычным интерпретатором Python.

## Модули, исходный код и зависимости{#arcadia_python}

С точки зрения сборки в программу все пакеты Python, кроме `main` являются библиотеками и описываются как [`PY2_LIBRARY`](modules.md#py_library)/[`PY3_LIBRARY`](modules.md#py3_library)/[`PY23_LIBRARY`](modules.md#py23_library).
В библиотеке-пакете в макросе [`PY_SRCS`](macros.md#py_srcs) перечисляются исходные файлы пакета, составляющие его модули.


**Минимальный пример:**
```
OWNER(g:my_group)
PY23_LIBRARY()
    PY_SRCS(x.py)
END()
```

В качестве исходных файлов поддержаны код на питоне (`.py`), код на Cython (`.pyx`), SWIG-биндинги (`.swg`), a также Protobuf (`.proto`) и Flatbuffers (`.fbs`).
Файл `__init__.py` в корне пакета не требуется, но если он есть и указан в [`PY_SRCS`](macros.md#py_srcs), он будет импортироваться при импорте такого пакета.


По умолчанию имя пакета для импорта — это путь до него от корня Аркадии с '.' вместо '/'.
Однако, имя можно переопределить использовав параметры `TOP_LEVEL` или `NAMESPACE` в макросе [`PY_SRCS`](macros.md#py_srcs), а также макросом [`PY_NAMESPACE`](macros.md#py_namespace).


Если у нас описана сборка пакета `x/y/ya.make`, то в зависимости от настройки импорты будут выглядеть так:

 - | - | - | -
:--- | :--- | :--- | :---
Путь в Аркадии | `x/y/a.py` | `x/y/z/b.py` | `x/y/__init__.py`
Default | `import x.y.a` | `import x.y.z.b` | `import x.y`
`TOP_LEVEL` | `import a` | `import z.b` | **запрещено**
`NAMESPACE n.s` | `import n.s.a` | `import n.s.z.b` | `import n.s`


Чтобы импортировать модули из других пакетов, то системе сборки необходимо явно указать зависимости, используя макрос `PEERDIR(path/to/package)`.
К сожалению, наличие `TOP_LEVEL` и `NAMESPACE` не позволяют системе сборки находить зависимости автоматически. В силу динамической природы исполнения
import-ов в Python система сборки также не в состоянии заранее обнаружить отсутствующие зависимости. Однако, для каждой программы на Python она порождает
import-тест, который проверяет импортируемость всех модулей, включенных в программу. В частности, для безусловных глобальных импортов это обеспечивает
проверку доступности необходимых пакетов в программе.

Для библиотек автоматически создаются тесты стиля. Чтобы убедиться, что у вас нет проблем со стилем кода нужно запустить сборку библиотеки с флагом `-t`.

{% note info %}

Сборка библиотеки порождает либо статическую библиотеку, либо объектные файлы. Это не означает компиляцию Python в исполняемый код: эти файлы содержат данные. Никакого разумного применения у них нет,
они используются только для сборки Python в программы системой сборки `ya make`. Никакой причины собирать библиотеку отдельно нет, вместо этого лучше собирать программу или тесты.

{% endnote %}


В Python-программе должен быть указан главный модуль. Если в [`PY_SRCS`](macros.md#py_srcs) программы указан `__main__.py`, то он и будет главным. Другой модуль может
быть сделан главным указанием `MAIN` в [`PY_SRCS`](macros.md#py_srcs) или отдельным макросом [`PY_MAIN`](macros.md#py_main).


Таким образом проект на питоне должен быть структурирован примерно так:

**Дерево:**
```
a/
|- b/
|--- lib/
|------ x.py
|------ ya.make
|--- bin/
|------ __main__.py
|------ ya.make
```
**a/b/lib/ya.make**
```
OWNER(g:my_group)
PY23_LIBRARY()
    PY_SRCS(x.py)
END()
```
**a/b/bin/ya.make**
```
OWNER(g:my_group)
PY3_PROGRAM()
    PEERDIR(a/b/lib)
    PY_SRCS(__main__.py)
END()
```
**a/b/bin/__main__.py**
```
import a.b.lib.x

print x.some_func()
```

Для программ автоматически создаются тесты стиля и импортов. Чтобы убедиться, что у вас нет проблем со стилем кода или импортами нужно запустить сборку программы с флагом `-t`.

{% note alert %}

Импорты проверяются только для программ целиком, замкнутость библиотек по импортам не контролируется. Для библиотек проверяется только стиль.

{% endnote %}


## Поддержка Python2 и Python3

В Аркадии сейчас одновременно поддерживаются Python2 и Python3. Для их описания используются парные модули [`PY2_LIBRARY`](modules.md#py_library)/[`PY3_LIBRARY`](modules.md#py3_library),
[`PY2_PROGRAM`](modules.md#py_program)/`PY3_PROGRAM`, `PY2TEST`/`PY3TEST`. Однако, чтобы избежать дублирования описаний для Python2/3-совместимого кода
предусмотрены *мультимодули* [`PY23_LIBRARY`](modules.md#py23_library) и `PY23_TEST`.


[`PY23_LIBRARY`](modules.md#py23_library) порождает 2 библиотеки, совместимые соответственно со 2м и 3м Питоном. При их использовании по `PEERDIR` выбирается та версия,
какая программа или тест собираются. Т.е. [`PY2_PROGRAM`](modules.md#py_program) получит из всех [`PY23_LIBRARY`](modules.md#py23_library) транзитивно версии для 2го питона (как будто [`PY23_LIBRARY`](modules.md#py23_library) — это [`PY2_LIBRARY`](modules.md#py_library)).
`PY3_PROGRAM` получит из них версии для 3го питона (как будто [`PY23_LIBRARY`](modules.md#py23_library) — это [`PY3_LIBRARY`](modules.md#py3_library)). При этом можно одновременно (через командную строку или `RECURSE`)
собирать и [`PY2_PROGRAM`](modules.md#py_program) и `PY3_PROGRAM` из одних и тех же [`PY23_LIBRARY`](modules.md#py23_library) и никаких проблем не будет.


`PY23_TEST` порождает 2 версии теста и запускает их как две отдельных сюиты: одну на 2м питоне (как `PY2TEST`) и одну на третьем (как `PY3TEST`). В их сборку библиотеки,
описанные как [`PY23_LIBRARY`](modules.md#py23_library), попадают по правилу, описанному выше.


Если набор исходных файлов или зависимостей различается для одного мультимодуля, то в зависимости от версии Python предусмотрены две переменные `PYTHON2` и `PYTHON3`.
Их можно использовать для условного описания сборки.

**Например:**
```
OWNER(g:my_group)
PY23_LIBRARY()

    IF (PYTHON2)
        PEERDIR(contrib/python/six)
    ENDIF()

    PY_SRCS(my23.py)
END()
```

## Из чего состоит Python-программа{#python_program}

Аркадийная программа на Python является полноценным исполняемым файлом под целевую архитектору (Linux, macOS, Windows и т.п.). Она состоит из:
- Прекомпилированного и исходного Python-кода, как взятого из Аркадии, так и сгенерированного, например, из `.proto` в виде бинарных *ресурсов*.
- Откомпилированного кода на С++ для Cython, SWIG и явно написанного в SRCS.
- Оглавления для поиска модулей внутри этого файла.
- Модифицированного интерпретатора Python, который умеет обрабатывать `import` из ресурсов, а не из файловой системы.

Программа на старте запускает интерпретатор для main-модуля и с этого места исполняется Python-код. Важно понимать, что все пакеты будут браться из
самой программы, а не с файловой системы. Окружения, в которых может запускаться такая программа, могут вообще не содержать установленного локального питона
и там может не быть файлов, которые питоновые модули ожидают видеть на файловой системе.

{% note info %}

В Аркадии лежит *разархиватор* программ на Python. Чтобы им воспользоваться нужно собрать проект `library/python/sfx/bin` и запустить получившуюся программу `sfx`.

{% endnote %}

## Модули расширения{%py_modules}

Кроме сборки Python в программы в Аркадии поддержана сборка расширений для внешнего Python. Такое расширение описывается как модуль [`PY2MODULE`](macros.md#pymodule) в котором пишется код на С++ или Cython в макросе `SRCS`.

[`PY2MODULE`](macros.md#pymodule) компилируется в динамическую библиотеку и импортируется во внешний, по отношению к сборке, Python. [`PY2MODULE`](macros.md#pymodule) может содержать определение только одного python-модуля (не более одного `.pyx` в `SRCS`)
и должен лежать на файловой системе так, чтобы внешний интерпретатор Рython его нашёл. При этом в сборку [`PY2MODULE`](macros.md#pymodule) не должно попадать Аркадийной сборки Python, поскольку она встраивает Python, а [`PY2MODULE`](macros.md#pymodule)
должен использоваться именно с внешним.

{% note warning %}

[`PY2MODULE`](macros.md#pymodule) не должен иметь `PEERDIR` на [`PY2_LIBRARY`](modules.md#py_library)/[`PY3_LIBRARY`](modules.md#py3_library)/[`PY23_LIBRARY`](modules.md#py23_library).

{% endnote %}

Поскольку [`PY2MODULE`](macros.md#pymodule) собирается с системным питоном и его версия определяет то, для какого питона будет модуль: для 2го или для 3го. Обычно для правильной
сборки достаточно использовать параметр `-DUSE_SYSTEM_PYTHON=<ver>`.

Однако, эта разница между Python 2 и 3 может влиять не только на сам модуль, но и на его зависимости. Чтобы весь C++ код работал с одной версией
Python его надо собирать с `Python.h` от правильной версии Python.  Чтобы поддержать переиспользование С/C++ кода между 2м и 3м питонами без
зависимости от аркадийного питона существует *мультимодуль* [`PY23_NATIVE_LIBRARY`](macros.md#py23_native_library). Такая библиотека собирается с использованием `Python.h`
для 2го или 3го питона в зависимости от контекста. Однако в отличие от [`PY23_LIBRARY`](modules.md#py23_library) она не зависит от Аркадийного Python runtime. Такую
библиотеку можно использовать как из [`PY2MODULE`](macros.md#pymodule) (которому библиотека Python не нужна), так и из Аркадийной сборки Python ([`PY2_LIBRARY`](modules.md#py_library) и т.п.),
которая приносит зависимость на нужную библиотеку.

{% note warning %}

[`PY23_NATIVE_LIBRARY`](macros.md#py23_native_library), также как и из [`PY2MODULE`](macros.md#pymodule) запрещено иметь `PEERDIR` на [`PY2_LIBRARY`](modules.md#py_library)/[`PY3_LIBRARY`](modules.md#py3_library)/[`PY23_LIBRARY`](modules.md#py23_library)

{% endnote %}

[`PY2MODULE`](macros.md#pymodule) - это модуль совместимый со 2м питоном: он будет всегда выбирать из [`PY23_NATIVE_LIBRARY`](macros.md#py23_native_library) версию для 2го питона и использовать
`Python.h` для 2го питона. В сборке с внешним питоном это не важно - всё будет определяться выбранным внешним питоном, но для дефолтной
сборки это существенное отличие. Чтобы позволить выбрать версию питона в дефолтной сборке и настроить в соответствии с ней зависимости и
путь поиска `Python.h` на [`PY23_NATIVE_LIBRARY`](macros.md#py23_native_library) есть модуль [`PY_ANY_MODULE`](macros.md#py_any_module). Это не мультимодуль - в каждой сборке у него только один
артефакт, но его можно переключать параметром  командной строки. Это модуль с базовым набором свойств [`PY2MODULE`](macros.md#pymodule) вне зависимости от версии
Python. Он не имеет дефолтного поведения и чтобы сделать его модулем для фиксированного питона в нём надо вызвать макрос [`PYTHON2_MODULE` или `PYTHON3_MODULE()`](macros.md#python2_modulepython3_module).
Это позволяет в частности управлять нужной версией.


## Взаимодействие с PROTO_LIBRARY

`PROTO_LIBRARY` может собираться в различные варианты в зависимости от того, из какого модуля используется. В частности она может
собираться в Python 2 и Python 3 вариант. В таблице приведён выбираемый вариант в зависимости от того, из какого модуля приходит зависимость:

Исходящий модуль | Вариант PROTO_LIBRARY
:--- | :---
`PY2_LIBRARY`, `PY2_PROGRAM`, `PY2TEST` | Python 2
`PY3_LIBRARY`, `PY3_PROGRAM`, `PY3TEST` | Python 3
`PY23_LIBRARY` | По входящей зависимости
`PY23_TEST` | Python 2 и Python3 для соответствующих вариантов теста
`PY2MODULE`, `PY_ANY_MODULE`, `PY23_NATIVE_LIBRARY` | C++
`LIBRARY`, `PROGRAM` c `USE_PYTHON2()`/`USE_PYTHON23()` | C++


## С/C++ в Python-сборке

В Аркадийной сборке предпочтительным способом вызова С/C++ кода из Python является Cython. Модули на Cython (`.pyx`) можно класть в [`PY_SRCS`](macros.md#py_srcs) наравне с обычными Python-модулями.
Поддерживаются зависимости от `pxd`-файлов, в которых можно описать через `cdef extern` прототипы внешних функций, классов и т.д. Эти файлы
можно импортировать (через `cimport`) как из текущей директории, так и по абсолютному пути от корня Аркадии. При импорте по абсолютному пути
можно класть `.pxd` файлы рядом с хедерами/определениями сущностей, описываемых в pxd.

Например, таким образом можно импортировать часто употребляемые типы из `util`:
```
from util.generic.maybe cimport TMaybe
from util.generic.string cimport TString
from util.system.types cimport ui64

cdef extern from "..." nogil:
    TMaybe<ui64> GetSomeStringProperty(const TString&)

def get_some_string_property(TString param):
    return GetSomeStringProperty(param)

# ------------------

assert get_some_string_property(s1) is None  # GetSomeStringProperty returned Nothing()
assert get_some_string_property(s2) == 42  # GetSomeStringProperty returned TMaybe<ui64>(42)
```

Также, этот пример демонстрирует некоторые другие возможности, добавленные в Cython для более удобной интеграции с разработкой в Аркадии:
- Опциональный name mangling для регистрации модулей не на верхнем уровне иерархии по абсолютному пути, что позволяет регистрировать такие модули без `TOP_LEVEL`.
- При поиске модулей для `cimport`, не проверяется наличие файлов `__init__.py` в подкаталогах — валидны все поддеревья Аркадии. Это позволяет импортировать готовые заголовки к популярным типам.
- Добавлены автоматические преобразования для основных контейнеров из `util` (преобразования работают в обе стороны). Этот механизм в общем случае нерасширяем (поддержка добавляется только внутри кодогенератора `cython`), но позволяет существенно упростить работу с самыми популярными типами.
  - `TVector`, `THashMap` преобразуются аналогично `std::vector`, `std::unordered_map`.
  - `TString`, `TStringBuf` преобразуются аналогично `std::string` в строки. Следует соблюдать осторожность при преобразовании в `TStringBuf` — кодогенератор `cython` не имеет возможности проверять время жизни данных, и может сохранить ссылку на освобожденную память. Следует воспринимать это как потенциально опасную оптимизацию.
  - `TMaybe<T>` преобразуется как `T`, если значение есть, либо в `None`, если значения нет.


Кроме исходного кода для Python, описанного в [`PY_SRCS`](macros.md#py_srcs), в библиотеки и программы на Python можно включать код на С++, работающий напрямую с
рантайм-библиотекой Python. Такой код надо писать в макросе `SRCS`. Никаких дополнительных зависимостей на рантайм библиотеку Python
добавлять не нужно — все нужные зависимости, а также пути для поиска `<Python.h>` будут доступны автоматически.


Python знает о том, какие встроенные модули можно импортировать, благодаря их регистрации при сборке или при старте интерпретатора.
Все модули из исходников, перечисленных в [`PY_SRCS`](macros.md#py_srcs), регистрируются автоматически с учётом `TOP_LEVEL` и `NAMESPACE`.
Чтобы зарегистрировать модули на С++, нужно воспользоваться макросом [`PY_REGISTER`](macros.md#py_register).


## Python в С++ сборке

C++ код может использовать код на Python. Чтобы добавить зависимость на библиотеку для соответствующего Python можно использовать макросы `USE_PYTHON2()`/`USE_PYTHON23()`.
Кроме зависимостей это правильно настроит пути для поиска `Python.h`, а также определит какой из вариантов выберется в случае `PEERDIR` на [`PY23_LIBRARY`](modules.md#py23_library).

{% note info %}

Модули `PROGRAM` и `LIBRARY` по умолчанию настроены на использование c Python 2, но могут быть перенастроены на использование Python 3 помощью макроса `USE_PYTHON3()`

{% endnote %}

Однако, даже с `USE_PYTHON2()`/`USE_PYTHON23()` при `PEERDIR` на `PROTO_LIBRARY` будет выбираться С++-вариант, а не вариант для соответствующего Python.
Если нужны именно Python версия, используйте [`PY2_LIBRARY`](modules.md#py_library)/[`PY3_LIBRARY`](modules.md#py3_library)/[`PY23_LIBRARY`](modules.md#py23_library).


Можно положить C++ код в `SRCS` модулей `PY_LRBRARY`/[`PY3_LIBRARY`](modules.md#py3_library) и даже [`PY23_LIBRARY`](modules.md#py23_library). При этом совсем не обязательно класть туда [`PY_SRCS`](macros.md#py_srcs) на
выходе получатся библиотеки с зависимостью соответственно, от Python 2, Python 3 и даже мультимодуль с зависимостью выбираемой по контексту. Единственным
отличием будет взаимодействие с `PROTO_LIBRARY`.

## Более подробно

- [Пошаговая инструкция по сборке Python](../../tutorials/python.md)
- [Модули для сборки Python](modules.md)
- [Макросы для сборки Python](macros.md)
- [Переменные управляющие сборкой и исполнением Python](vars.md)
- [Вспомогательные команды для работы с Аркадийным Python](vars.md)
- [Каталог примеров проектов на Python](examples.md)