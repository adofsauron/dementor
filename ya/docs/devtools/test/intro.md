# Введение

{% note info %}

Более подробная информация по системе тестирования располагается в [документации системы сборки](https://docs.yandex-team.ru/ya-make/usage/ya_make/tests).

{% endnote %}

Тестирование в едином репозитории значительно отличается от того, как это происходит во внешнем мире. Основными понятиями являются **test**, **chunk** и **suite**.

* **Test** - одна именованная проверка, описанная в виде кода на поддерживаемом языке программирования. В понятие теста включается не только проверка правильности работы кода, но также, например, проверки того, что код оформлен с использованием рекомендуемого [стандарта оформления кода](https://en.wikipedia.org/wiki/Programming_style). Любые падения/ошибки во время исполнения теста относятся непосредственно к исполняемому тесту.

* **Chunk** -  сущность представляющая собой запуск программы с тестами (представлен в виде узла в графе команд), в рамках которой исполняются тесты. К chunk относятся все ошибки runtime исполнения за рамками тестов. Например ошибки запуска тестирования - тестовая программа упала на инициализации и тесты даже не запускались, ошибки финализации тестирования, утечки памяти после тестирования, падения [тестовых рецептов](https://wiki.yandex-team.ru/yatool/test/recipes).

* **Suite** - набор из нескольких тестов, написанных с помощью одного [фреймворка тестирования](https://en.wikipedia.org/wiki/List_of_unit_testing_frameworks) и имеющих один и тот же набор зависимостей. Suite содежит по крайней мере один chunk (так называемый sole chunk). Все тесты в наборе имеют общий [размер](https://docs.yandex-team.ru/ya-make/usage/ya_make/tests#size), логику начала (set up) и завершения (tear down) работы, теги и так далее. Некоторые ошибки не позволяют вообще запустить тестирование - сломанные зависимости теста, когда невозможно собрать требуемую для тестирования программу, ошибки подбора хоста для `LARGE` тестов из-за комбинации sandbox-тегов, которым не соотвествует ни один хост. Все такие ошибки относятся к suite.

Ранее мы уже выяснили, что описание процедуры сборки исходного кода в едином репозитории на любом языке программирования происходит в файлах **ya.make**. Аналогичным образом устроено и тестирование:
1. В файле **ya.make** описано какой фреймворк следует использовать, списки файлов с кодом тестов, требования к тестовому окружению и так далее.
2. Тесты могут запускаться разработчиком **вручную** перед [отправкой пулл-реквеста](../src/arc/workflow.md) или системой автоматической сборки **во время проверки пулл-реквеста**. При этом зачитывается содержимое **ya.make** и выполняются все описанные в нем проверки. Каким образом запустить тесты, описано в следующих разделах.

## Поддерживаемые фреймворки { #framework }

Для каждого поддерживаемого языка программирования есть один или несколько поддерживаемых фреймворков, на которых пишутся тесты:

Язык | Поддерживаемые фреймворки
:--- | :---
[С++](https://isocpp.org/) | Самописный фреймворк [unittest](https://a.yandex-team.ru/arc/trunk/arcadia/library/cpp/testing/unittest), [Google Test](https://github.com/google/googletest/), [Google Benchmark](https://github.com/google/benchmark)
[Go](https://golang.org/) | Встроенный в язык фреймворк [go test](https://golang.org/pkg/testing/)
[Java](https://www.java.com) / [Kotlin](https://kotlinlang.org/) | [JUnit](https://junit.org/) версий 4.х и 5.х
[Python 3](https://www.python.org/) | [pytest](https://pytest.org/), [doctest](https://docs.python.org/3/library/doctest.html)

## Типы тестов { #type }

В едином репозитории исходные коды на различных языках программирования могут произвольным образом перемешиваться в проектах. **Типом теста** называется выполнение проверок с использованием одного конкретного инструмента, например, фреймворка `pytest` для Python или утилиты проверки форматирования кода `go fmt` для Golang. Полный список типов тестов приведен в таблице:

Тип | Описание
:--- | :---
`classpath.clash` | Проверка наличия дублирующихся классов в classpath при компиляции Java проекта.
`exectest` | Выполнение произвольной команды и проверка её кода возврата
`flake8.py2` | Проверка стиля кода на Python 2 c использованием утилиты [Flake8](https://gitlab.com/pycqa/flake8)
`flake8.py3` | Проверка стиля кода на Python 3 c использованием утилиты [Flake8](https://gitlab.com/pycqa/flake8)
`fuzz` | [Fuzzing](https://en.wikipedia.org/wiki/Fuzzing) тест
`g_benchmark` | Выполнение бенчмарков на C++ библиотекой [Google Benchmark](https://github.com/google/benchmark)
`go_bench` | Выполнение бенчмарков на Go утилитой `go bench`
`gofmt` | Проверка форматирования кода на Go утилитой `go fmt`
`go_test` | Выполнение тестов на Go утилитой `go test`
`govet` | Выполнение статического анализатора кода на Go утилитой `go vet`
`gtest` | Выполнение тестов на С++ с использованием фреймфорка [Google Test](https://github.com/google/googletest/)
`java` | Выполнение тестов на Java с использованием фреймворка [JUnit](https://junit.org/)
`java.style` | Проверка форматирования кода на Java утилитой [checkstyle](https://checkstyle.org/)
`py2test` | Тесты на Python 2 с использованием фреймворка [pytest](https://pytest.org/)
`py3test` | Тесты на Python 3 с использованием фреймворка [pytest](https://pytest.org/)
`pytest` | Тесты на Python любой версии с использованием фреймворка [pytest](https://pytest.org/)
`unittest`| Тесты на C++ с использованием фреймворка [unittest](https://a.yandex-team.ru/arc/trunk/arcadia/library/cpp/testing/unittest)
`validate_resource`| Проверка времени жизни Sandbox [ресурса](https://docs.yandex-team.ru/sandbox/resources)
