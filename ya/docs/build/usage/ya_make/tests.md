# Запуск тестов

ya make предоставляет развитые возможности запуска тестов. Основными понятиями для тестирование силами ya make являются **test**, **chunk** и **suite** (набор тестов).

* **Test** - одна именованная проверка, описанная в виде кода на поддерживаемом языке программирования. В понятие теста включается не только проверка правильности работы кода, но также, например, проверки того, что код оформлен с использованием рекомендуемого [стандарта оформления кода](https://en.wikipedia.org/wiki/Programming_style). Любые падения/ошибки во время исполнения теста относятся непосредственно к исполняемому тесту.

* **Chunk** -  сущность представляющая собой запуск программы с тестами (представлен в виде узла в графе команд), в рамках которой исполняются тесты. К chunk относятся все ошибки runtime исполнения за рамками тестов. Например ошибки запуска тестирования - тестовая программа упала на инициализации и тесты даже не запускались, ошибки финализации тестирования, утечки памяти после тестирования, падения [тестовых рецептов](../../manual/tests/recipe).

* **Suite** - набор из нескольких тестов одного [типа](#test_type) и имеющих один и тот же набор зависимостей. Suite содежит по крайней мере один chunk (так называемый sole chunk). Все тесты в наборе имеют общий [размер](#size), логику начала (set up) и завершения (tear down) работы, теги и так далее. Некоторые ошибки не позволяют вообще запустить тестирование - сломанные зависимости теста, когда невозможно собрать требуемую для тестирования программу, ошибки подбора хоста для `LARGE` тестов из-за комбинации sandbox-тегов, которым не соотвествует ни один хост. Все такие ошибки относятся к suite.

Каждая suite имеет имя, [размер](#size) (время, отведённое на работу), [тип (используемый фреймворк)](#test_type), а также опциональный набор тегов. Все эти параметры могут быть использованы для фильтрации.
Кроме того, каждый отдельный тест имеет имя и потому фильтрация может делаться с точностью до отдельного теста.

{% note info %}

Имена suite известны системе сборки заранее, а имена тестов выясняются только во время запуска chunk. Это, в частности, означает, что система сборки должна запустить chunk даже если ей надо только узнать имена тестов. Если в коде инициализации chunk есть ошибки, то такой запуск может завершиться ошибкой и система сборки не сможет узнать какие тесты есть в suite и даже сколько их там.

{% endnote %}

Как описывать тесты можно прочитать [в соответствующем разделе](../../manual/tests/index.md) руководства по системе сборки ya make.

## Запуск тестов { #execution }

Для простого запуска тестов есть следующие ключи командной строки.

- `-t`/`-tt`/`-ttt` — запустить тесты. `-t` - только SMALL, `-tt` - SMALL и MEDIUM, `-ttt` - тесты всех размеров.
- `-A`, `--run-all-tests` — запустить тесты всех размеров (тоже, что что и `-ttt`).
- Тесты могут быть параметризованы значениями, передаваемым из командной строки c помощью опции `--test-param=TEST_PARAM`. `TEST _PATARAM` имеет вид `name=val`, доступ к параметрам зависит от используемого фреймворка.

**Пример**

```bash
[arcadia]$ ya make -t devtools/examples/tutorials/python
```

Запустит все тесты, которые найдёт по `RECURSE`/`RECURSE_FOR_TESTS` от `devtools/examples/tutorials/python`, включая тесты стиля и тесты импорта для Python. При этом будут использованы умолчания для сборки:

- Платформа будет определена по *реальной платформе* на которой запущена команда `ya make`.
- Тесты будут собраны в режиме `debug` — он используется по умолчанию.
- Кроме тестов будут собраны все остальные цели (библиотеки и программы), достижимые по `RECURSE`/`RECURSE_FOR_TESTS` от `devtools/examples/tutorials/python`. Это включает сборку всех необходимых зависимостей.

По умолчанию система сборки запустит все запрошенные тесты. После запуска тестов для всех упавших тестов будет выдана краткая информация о падениях (включая ссылки на более полную информацию). Для прошедших и
проигнорированных (отфильтрованных) тестах будет выдан только общий короткий статус (количество тех и других).

Это поведение можно поменять следующими ключами:

- `--fail-fast` — исполнять тесты до первого падения.
- `-P`, `--show-passed-tests` — показывать каждый прошедший тест
- `--show-skipped-tests` — показывать каждый пропущенный (отфильтрованный) тест
- `--show-metrics` — показывать метрики тестов

## Список тестов { #test_list }

Список всех тестов, которые будут запущены можно получить опцией `-L` (`--list-tests`).
Эта опция поддерживает выбор размера тестов, а также все [параметры фильтрации](#test_filtering).

**Примеры**

```bash
[arcadia]$ ya make -tL devtools/examples/tutorials/python # Список всех SMALL тестов
```


```bash
[arcadia]$ ya make -AL --regular-tests devtools/examples/tutorials/python # Список всех пользовательских тестов (без тестов стиля, импортов и подобных проверок)
```

## Размер тестов { #size }

В едином репозитории в одном пулл-реквесте изменения могут вноситься в код и тесты на разных языках программирования. Важно, чтобы проверка любых пулл-реквестов проходила как можно быстрее, поэтому тесты разделены на категории по **максимальному разрешённому времени исполнения**, которое является значением по умолчанию:

1. **SMALL** тесты: выполняются **не более 1 минуты**. Сюда обычно попадает большинство [unit-тестов](https://en.wikipedia.org/wiki/Unit_testing).
2. **MEDIUM** тесты: выполняются **не более 10 минут**. Сюда попадают некоторые медленные unit-тесты, интеграционные и end-to-end тесты.
3. **LARGE** (ранее **"FAT"**) тесты: выполняются **не более 1 часа**. Содержит особо медленные тесты. В отличие от MEDIUM \ SMALL тестов LARGE тесты могут продолжать выполняться даже после того как прошли остальные проверки пулл-реквеста, потому что запускаются в отдельном процессе в [Sandbox](https://sandbox.yandex-team.ru/).

{% note info %}

Строгого соответствия между размером тестов (SMALL, MEDIUM, LARGE) и типом тестов (unit, интеграционные, end to end) нет. В первую очередь нужно ориентироваться на максимальное время исполнения каждого теста.

{% endnote %}

Размер тестов должен быть явно указан в файле **ya.make** при помощи макроса `SIZE`.
Максимальное время выполнения теста можно уменьшить при помощи макроса `TIMEOUT`.

Пример файла **ya.make** для тестов на Java:

```yamake
OWNER(g:my-group)

JUNIT5() # Используемый фреймворк

SIZE(MEDIUM) # Размер тестов

JAVA_SRCS(SRCDIR java **/*) # Исходные коды тестов
JAVA_SRCS(SRCDIR resources **/*)

PEERDIR( # Зависимости
    my-project/src/
    # ...
)

END()
```

## Параллельный запуск тестов { #fork }

По умолчанию тесты внутри одной suite выполняются последовательно в рамках одного chunk (так называемого sole chunk) в виде отдельного узла графа команд `ya`.
Тесты из разных suite (ya.make) выполняются параллельно.

Во время формирования графа команда `ya` не может знать сколько будет тестов в suite, так как для этого требуется сборка и листинг тестов средствами тестового фреймворка.
Так как `ya` оперирует статическим графом команд (он не меняется по мере исполнения и целиком известен для исполнителя), то на этапе конфигурации мы только можем заранее вставить нужное количество chunk'ов, каждый из которых исполнит непересекающееся множество тестов.

Для того, чтобы разбить выполнение тестов из suite на несколько chunk'ов нужно воспользоваться макросами `FORK_TESTS`, `FORK_SUBTESTS` и `FORK_TEST_FILES`.

Каждый chunk наследует общие параметры теста из `ya.make`: размер, таймаут, требования на ресурсы и другие: таким образом удобно распараллелить тесты, которые из-за своего количества перестали укладываться в таймаут.

* `FORK_TESTS`, `FORK_SUBTESTS` – режимы, при которых вместо sole chunk, запуск suite разбивается на несколько chunk'ов, каждый из которых получает дополнительные параметры в виде общего количества chunk'ов, участвующих в разбиении и своего порядкового номера: по этим параметрам каждый chunk определяет подмножество тестов, которое надо запустить.
Количество chunk'ов умолчанию равно `10`, это значение можно изменить с помощью `SPLIT_FACTOR(X)`.
Отличие `FORK_TESTS` от `FORK_SUBTESTS` заключается в том, что `FORK_TESTS` считает тесты, объединенные в классы неделимой сущностью, в то время как `FORK_SUBTESTS` разбивает в том числе тестовые классы и в разных chunk'ах могут оказаться тесты, принадлежащие одному классу, что может быть нежелательно, если у классов есть тяжелые подготовительные стадии.

* `FORK_TEST_FILES` – разбивает прогон тестов из suite на количество chunk'ов, равное количеству файлов с тестами, перечисленных в `ya.make`, каждый запуск работает только с одним конкретным файлом.

`FORK_TESTS`, `FORK_SUBTESTS` принимают опциональный агрумент - `SEQUENTIAL` или `MODULO`, определяющий как будет происходить разделение тестов по chunk'ам.
`SEQUENTIAL` разделяет тесты равными диапозонами, предварительно их отсортировав по имени.
`MODULO` разделяет тесты по модулю, предварительно их отсортировав.
Если агрумент не указан, значение по умолчанию - `SEQUENTIAL`.

`FORK_TEST_FILES` можно комбинировать с `FORK_TESTS` или `FORK_SUBTESTS`, тогда будет дополнительное разбиение запуска тестов из каждого файла.

`FORK_TEST_FILES` поддержан только для `pytest`.

## Фильтрация тестов { #test_filtering }

Suite можно фильтровать по различным свойствам:

- `--test-size=TEST_SIZE_FILTERS` — запускать только выбранные размеры
- `--test-type=TEST_TYPE_FILTERS` — запускать только выбранные [типы тестов](#test_type). Перед именами типов тестов можно использовать `+` для включения suite в фильтр и `-` для исключения. Таким образом `--test-type -import_test` запустит все тесты, кроме `import_test`. Фильтр `--test-type unittest+gtest` запустит только `unittest` и `gtest`.
- `--style` — запускать только тесты стиля
- `--regular-tests` — запускать только пользовательские тесты
- `--test-tag=TEST_TAGS_FILTER` — запускать только сюиты с определёнными тегами. Перед именами тегов можно использовать `+` для включения тега в фильтр и `-` для исключения. Таким образом `--test-tag tag1+tag2-tag3` запустит все тесты, у которых есть `tag1`, `tag2` и нет `tag3`. Фильтр `--test-tag -tag3` запустит все тесты без тега `tag3`.

Помимо фильтрации suite можно фильтровать отдельные тесты:

-  `-F=TESTS_FILTERS`, `--test-filter=TESTS_FILTERS` — фильтрация тестов по шаблону
-  `-F="[X/Y] chunk"`, `--test-filter="[X/Y] chunk"` — фильтрация тестов по chunk'ам. Внутри `[]` можно использовать `*` для фильтрации.
- `--test-filename=TEST_FILES_FILTER` — запускать тесты только из выбранного исходного файла
- `-X`, `--last-failed-tests` — запустить только тесты упавшие в предыдущем запуске


При указании флага `-F` можно использовать [специальные символы](https://docs.python.org/3/library/fnmatch.html):

```bash
$ ya make -t -F <file>.py::<ClassName>::*
```

{% note tip %}

Для правильного указания параметров фильтрации воспользуйтесь опцией получения [списка тестов](#test_list). Её же можно использовать, чтобы проверить, что ваш фильтр работает правильно.

{% endnote %}


## Канонизация (и переканонизация)

Система сборки ya make для некоторых типов тестов поддерживает сравнение с эталонными (каноническими) данными. Если эти данные нужно обновить воспользуйтесь опцией `-Z` (`--canonize-tests`).
В этом режиме вместо сравнения данных с эталонными, сами эталонные данные будут заменены и при необходимости отправлены в Sandbox. В локальное рабочее пространство будут внесены все необходимые изменения.

{% note alert %}

Не забудьте закоммитить локальные изменения, иначе новые эталонные данные не будут доступны в CI.

{% endnote %}


## Типы тестов { #test_type }

**Типом теста** называется выполнение проверок с использованием одного конкретного инструмента, например, фреймворка `pytest` для Python или утилиты проверки форматирования кода `go fmt` для Golang. Полный список типов тестов приведен в таблице:

Тип | Описание
:--- | :---
`black` | Проверка форматирования кода на python3 утилитой [black](../../manual/tests/style#black).
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
`gtest` | Выполнение тестов на С++ с использованием фреймворка [Google Test](https://github.com/google/googletest/)
`java` | Выполнение тестов на Java с использованием фреймворка [JUnit](https://junit.org/)
`java.style` | Проверка форматирования кода на Java утилитой [checkstyle](https://checkstyle.org/)
`py2test` | Тесты на Python 2 с использованием фреймворка [pytest](https://pytest.org/)
`py3test` | Тесты на Python 3 с использованием фреймворка [pytest](https://pytest.org/)
`pytest` | Тесты на Python любой версии с использованием фреймворка [pytest](https://pytest.org/)
`unittest`| Тесты на C++ с использованием фреймворка [unittest](https://a.yandex-team.ru/arc/trunk/arcadia/library/cpp/testing/unittest)
`validate_resource`| Проверка времени жизни Sandbox [ресурса](https://docs.yandex-team.ru/sandbox/resources)
