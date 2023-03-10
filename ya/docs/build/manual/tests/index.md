# Описание тестов

В этом разделе подробно рассказано как описывать тесты в `ya.make` файлах. Про запуск тестов можно прочитать вот [здесь](../../usage/ya_make/tests)

В нашей системе сборки `ya make` можно описать тесты для основных языков в аркадии. Поддержка тестов реализована поверх фреймворков для тестирования в этих языках. Подробную информацию про устройство тестов в языке можете найти на соответствующих страницах ([C++](cpp), [Python](python), [Java](java), [Go](go)).

## Общие понятия

Два основных понятия для тестов в `ya make` это `test` и `suite`.
* `test` - это конкретная именованная проверка.
* `suite` - сущность, включающая в себя тесты в рамках описываемого модуля.

## Как описывать ya.make

По умолчанию один тестовый модуль является одной `suite`. `suite` аккумулирует в себе ошибки тестирования, которые выходят за пределы определения теста, например:
- ошибки получения списка тестов
- ошибки инициализации тестирования (до фактического выполнения тестов)
- ошибки финализации тестирования

Сьюита имеет несколько параметров:
- Указание фрэймворка тестирования
- Список файлов с тестами
- [Список зависимостей](#dependecy)
- [Размер](#size)
- [Тэги](#tag)
- [Требования к запуску тестов](#requirements)
- [Переменные окружения](#env)
- Таймаут на запуск


### Список зависимостей {#dependecy}

Мы придерживаемся идеи герметичности тестов. Это значит, что тест должен быть зависимым только от входных данных, которые были явно задекларированы в `ya.make`. Чтобы обеспечить герметичность тестов, каждый запуск проходит в чистом окружении, которое содержит только указанные явно зависимости из `ya.make`.

Помимо сборочных зависимостей (описываются макросом [`PEERDIR()`](../common/macros#peerdir)), для тестов нужно описывать зависимости на входные данные для запуска тестов. Они бывают двух типов:

1. **Другие проекты из единого репозитория**. Например, вам может потребоваться исполняемый файл, исходные коды которого расположены в другом проекте. Такие зависимости описываются при помощи макроса `DEPENDS()`. Все пути строятся относительно корня `arcadia` и указываются через пробел до `ya.make`.

2. **Тестовые данные**. Например, сюда относятся различные эталонные файлы: логи, списки, тестовые дампы баз данных. Такие файлы могут храниться как и в `arcadia`, так и в `Sandbox`. Для описания тестовых данных можно использовать макросы:
   * [**`DATA()`**](../common/data#data)
   * [**`FROM_SANDBOX()`**](../common/data#from_sandbox)

Более полную информацию про использование данных в тестах можно прочитать [здесь](../common/data).


### Размер тестов {#size}

Размер тестов задается макросом `SIZE()`. Сейчас существуют три размера:

* `SIZE(SMALL)` - максимальный таймаут *60s (1min)*. Размер по умолчанию.
* `SIZE(MEDIUM)` - максимальный таймаут *600s (10min)*.
* `SIZE(LARGE)` - максимальный таймаут *3600s (1hour)*.


### Тэги теста {#tag}

Тесты можно фильтровать по тэгам. Тэги проставляются с помощью макроса `TAG()`.


### Требования к запуску теста {#requirements}

Макрос `REQUIREMENTS()` позволяет указать требования, необходимые для запуска тестов. С помощью этого макроса можно задать:

* количество ядер
* необходимый объем свободного места на диске
* необходимый объем оперативной памяти
* необходимый объем RAM-диска
* ID контейнера
* ограничения сети

Все возможные описания требований указаны в разделе [Общие макросы](common)


### Переменные окружения {#env}

С помощью макроса `ENV(key=value)` можно задать значение переменной окружения при запуске теста. Каждая переменная окружения задается отдельным макросомм.


## Иерархия проекта с тестами

Предлагается в папке проекта иметь `lib`, `bin`, `tests` или `lib/tests`. Мы не рекомендуем указывать `RECURSE` в `ya.make`, содержащем описание модуля, но можно после директивы `END()` добавлять `RECURSE_FOR_TESTS` на тесты, проверяющие этот модуль. Тогда `project/ya.make` может состоять из

```
RECURSE(
    project/bin
    project/lib
    project/lib/tests
)
```

В `project/lib` не должно быть `RECURSE` на `tests` или определение модуля тестов, однако можно поставить `RECURSE_FOR_TESTS`.

В этом случае при запуске `ya make -t` из каталога `project` инициируется сборка проекта с прогоном тестов. А другие проекты, желая подключать вашу библиотеку, будут писать `PEERDIR(project/lib)` без установления зависимости от ваших тестов.


## Возможности тестов

### Канонизация

У тестов есть возможность сообщить о своих результатах в виде данных, которые необходимо верифицировать с каноническими. Такие тесты необходимо первый раз канонизировать, после чего системе будут доступны референсные данные для сравнения. Канонический результат будет сохранен рядом с тестом в директорию `canondata/<test name>/result.json` или вынесен в ресурс Sandbox, в зависимости от переданных параметров в тесте.

Каждый фреймворк для написания тестов предоставляет свою поддержку механизма канонизации. Канонизация поддержана для всех языков, кроме C++.


### Метрики

Наш CI поддерживает выгрузку пользовательских метрик из тестов. При подключении проекта к автосборке история теста с метриками будет сохраняться и отображаться в CI. При локальной сборке будет доступен отчет.


### Параллельный запуск тестов

По умолчанию тесты внутри одного `ya.make` файла выполняются последовательно в рамках отдельной задачи сборочного графа, а тесты из разных `ya.make` выполняются параллельно. Для того, чтобы разбить выполнение тестов из одного `ya.make` на несколько параллельных запусков, можно воспользоваться макросами `FORK_TESTS`, `FORK_SUBTESTS` и `FORK_TEST_FILES`. Каждый запуск будет выполнен в отдельной задаче сборочного графа.

Важно отметить, что каждый подзапуск наследует общие параметры теста из `ya.make`, такие как размер, таймаут, требования на ресурсы и т.д. Таким образом удобно распараллелить тесты, которые из-за своего количества перестали укладываться в таймаут, но если в `ya.make` стоит требование `cpu(4)`, то при большом количестве параллельных задач, локальный запуск ya make может замедлиться.

Не все `FORK`-макросы поддержаны для конкретного тестового фреймворка и языка: перед их использованием, уточните текущий статус в документации.


## Exec-тесты

Помимо обычных тестов, которые будут описаны в специфичных для языков разделах, есть возможность писать exec-тесты. Exec-тесты позволяют выполнить произвольную команду и убедиться, что она успешно завершается. Успешным считается завершение команды с кодом возврата 0.

Подробнее про описание Exec-тестов написано [здесь](exectest).


## Как описывать тесты для языков

### C++

Сейчас поддержаны два тестовых фреймворка: 
* [unittest](https://a.yandex-team.ru/arc/trunk/arcadia/library/cpp/testing/unittest) - собственная разработка
* [gtest](https://github.com/google/googletest) - популярное решение от Google

Также есть отдельная библиотека [library/cpp/testing/common](https://a.yandex-team.ru/arc/trunk/arcadia/library/cpp/testing/common), в которой находятся полезные утилиты, независящие от фреймворка.

**Бенчмарки:** Используется библиотека [google benchmark](https://a.yandex-team.ru/arc/trunk/arcadia/contrib/libs/benchmark/README.md) и модуль `G_BENCHMARK`. Все подключенные к автосборке бенчмарки запускаются в CI по релевантным коммитам и накапливают историю метрик.

**Fuzzing:** Фаззинг - это техника тестирования, заключающаяся в передаче приложению на вход неправильных, неожиданных или случайных данных. Все подробности [здесь](fuzzing).

**Linting:** Поддержан статический анализ файлов с помощью `clang-tidy`. Подробнее про подключение своего проекта к линтингу в автосборке можно прочитать [здесь](style#clang-tidy).

**Метрики:** Помимо метрик от бенчмарков также можно сообщать числовые метрики из `UNITTEST()` и `GTEST()`. Для добавления метрик используйте функцию `testing::Test::RecordProperty` если работаете с `gtest`, или макрос `UNIT_ADD_METRIC` если работаете с `unittest`.

{% list tabs %}

- Gtest

  ```cpp
  TEST(Solver, TrivialCase) {
      // ...
      RecordProperty("num_iterations", 10);
      RecordProperty("score", "0.93");
  }
  ```
- Unittest

  ```cpp
  Y_UNIT_TEST_SUITE(Solver) {
      Y_UNIT_TEST(TrivialCase) {
          // ...
          UNIT_ADD_METRIC("num_iterations", 10);
          UNIT_ADD_METRIC("score", 0.93);
      }
  }
  ```

{% endlist %}

---

Минимальный `ya.make` для тестов выглядит так:

```
UNITTEST() | GTEST() | G_BENCHMARK()

OWNER(...)

SRCS(tests.cpp)

END()
```

Подробная документация о тестах на C++ расположена [здесь](cpp).

### Python

Основным фреймворком для написания тестов на Python является [pytest](https://pytest.org/). 

Поддерживаются Python 2 (модуль `PY2TEST`), Python 3 (модуль `PY3TEST`) и модуль `PY23_TEST`. Все тестовые файлы перечисляются в макросе `TEST_SRCS()`.

Для работы с файлами, внешними программами, сетью в тестах следует использовать специальную библиотеку [yatest](https://a.yandex-team.ru/arc/trunk/arcadia/library/python/testing/yatest_common). 

**Метрики**: Чтобы сообщить метрики из теста, необходимо использовать funcarg metrics.
```python
def test(metrics):
    metrics.set("name1", 12)
    metrics.set("name2", 12.5)
```

**Бенчмарки**: Для бенчмарков следует использовать функцию `yatest.common.execute_benchmark(path, budget=None, threads=None)`. Чтобы результаты отображались в CI, результаты нужно записывать в метрики.

**Канонизация**: Можно канонизировать простые типы данных, списки, словари, файлы и директории. Тест сообщает о данных, которые нужно сравнить с каноническими, через возврат их из тестовой функции командой return.

**Linting**: Все python файлы, используемые в сборке и тестах, подключаемые через `ya.make` в секциях `PY_SRCS()` и `TEST_SRCS()`, автоматически проверяются `flake8` линтером.

**Python imports**: Для программ `PY2_PROGRAM`, `PY3_PROGRAM`, `PY2TEST`, `PY3TEST`, `PY23_TEST`, собранных из модулей на питоне, добавлена проверка внутренних модулей на их импортируемость - `import_test`. Это позволит обнаруживать на ранних стадиях конфликты между библиотеками, которые подключаются через `PEERDIR`, а также укажет на неперечисленные в `PY_SRCS` файлы (но не `TEST_SRCS`).

Подробная документация о тестах на Python расположена [здесь](python).

### Java

Для тестов используется фреймворк [JUnit](https://junit.org/junit5/) версий 4.x и 5.x.

Тестовый модуль для JUnit4 описывается модулем `JTEST()` или `JTEST_FOR(path/to/testing/module)`. 
* `JTEST()`: система сборки будет искать тесты в `JAVA_SRCS()` данного модуля.
* `JTEST_FOR(path/to/testing/module)`: система сборки будет искать тесты в тестируемом модуле.

Для включения JUnit5 вместо `JTEST()` необходимо использовать `JUNIT5()`. 

Содержание `ya.make` файла для `JUNIT5()` и `JTEST()` отличается только набором зависимостей.

Минимальный `ya.make` файл выглядит так:

{% list tabs %}

- JUnit4

  ```
  JTEST()

  JAVA_SRCS(FileTest.java)

  PEERDIR(
    # Сюда же необходимо добавить зависимости от исходных кодов вашего проекта
    contrib/java/junit/junit/4.12 # Сам фреймворк JUnit 4
    contrib/java/org/hamcrest/hamcrest-all # Можно подключить набор Hamcrest матчеров
  )

  END()
  ```
- JUnit5
  ```
  JUNIT5()
  JAVE_SRCS(FileTest.java)
  PEERDIR(
    # Сюда же необходимо добавить зависимости от исходных кодов вашего проекта
    contrib/java/org/junit/jupiter/junit-jupiter # Сам фреймворк JUnit 5
    contrib/java/org/hamcrest/hamcrest-all # Набор Hamcrest матчеров
  )
  END()
  ```

{% endlist %}

**Java classpath clashes**: Есть возможность включить автоматическую проверку на наличие нескольких одинаковых классов в [Java Classpath](https://en.wikipedia.org/wiki/Classpath). В проверке участвует не только имя класса, но и хэш-сумма файла с его исходным кодом, так как идентичные классы из разных библиотек проблем вызывать не должны. Для включения этого типа тестов в ya.make файл соответствующего проекта нужно добавить макрос `CHECK_JAVA_DEPS(yes)`.

**Linting**: На все исходные тексты на Java, которые подключены в секции `JAVA_SRCS`, включён статический анализ. Для проверки используется утилита [checkstyle](https://checkstyle.org/). 

**Канонизация**: Для работы с канонизированными данными используйте функции из [devtools/jtest](https://a.yandex-team.ru/arc/trunk/arcadia/devtools/jtest).

Подробная документация о тестах на Java расположена [здесь](java).

### Go

Тесты работают поверх [стандартного тулинга для Go](https://pkg.go.dev/testing). Для работы с зависимостями теста следует использовать библиотеку [`library/go/test/yatest`](https://a.yandex-team.ru/arc/trunk/arcadia/library/go/test/yatest/env.go).

Все тестовые файлы должны иметь суффикс `_test.go`. Они перечисляются в макросе `GO_TEST_SRCS`.

Тестовый модуль описывается модулем `GO_TEST()` или `GO_TEST_FOR(path/to/testing/module)`. 
* `GO_TEST()`: система сборки будет искать тесты в `GO_TEST_SRCS()` данного модуля.
* `GO_TEST_FOR(path/to/testing/module)`: система сборки будет искать тесты в `GO_TEST_SRCS` в тестируемом модуле.

Минимальные `ya.make` файлы выглядят так:

{% list tabs %}

- GO_TEST()

  ```
  GO_TEST()

  GO_TEST_SRCS(file_test.go)

  END()
  ```
- GO_TEST_FOR()
  
  В `project/ya.make` в макросе `GO_TEST_SRCS` перечисляются тестовые файлы:

  ```
  GO_LIBRARY() | GO_PROGRAM()

  SRCS(file.go)

  GO_TEST_SRCS(file_test.go)

  END()

  RECURSE(tests)
  ```

  В `project/tests/ya.make` указывается относительный путь от корня аркадии на  тестируемый модуль через `GO_TEST_FOR`:

  ```
  GO_TEST_FOR(relative/path/to/project)

  END()
  ```

{% endlist %}

**Канонизация**: Для работы с такими тестами используйте [library/go/test/canon](https://a.yandex-team.ru/arc/trunk/arcadia/library/go/test/canon/canon.go). [Пример](https://a.yandex-team.ru/arc/trunk/arcadia/devtools/ya/test/tests/canonize_new_format/canonize_file_with_diff_tool).

**Бенчмарки**: Чтобы включить бенчмарки в проекте, нужно добавить тэг `ya:run_go_benchmark` в `ya.make` проекта. [Пример](https://a.yandex-team.ru/arc/trunk/arcadia/devtools/ya/test/tests/go/data/benchmark_fast).

Подробная документация о тестах на Go расположена [здесь](go).
