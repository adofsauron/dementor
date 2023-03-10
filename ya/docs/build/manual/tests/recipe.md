# Тестовые окружения aka Рецепты

Часто различным тестам нужно сначала подготовить для себя окружение: запустить "внешние сервисы" или их заглушки, подготовить данные и т.д. Обычно, код, который этим занимается, существует как часть теста и работает в том же процессе (пишется на том же языке с использованием текущего тестового фреймворка). Такой подход сильно усложняет переиспользование этого кода в других тестах, например, процедуру подготовки тестового стенда в pytest практически нельзя переиспользовать в тестах на C++. Тестовые окружения (или рецепты, recipe) призваны исправить эту ситуацию.

Рецепт позволяет перенести запуск тестового окружения на отдельный уровень. Он может:

* Настроить и запустить необходимые программы;
* Сообщить тесту всю необходимую информацию о подготовленном окружениии;
* Корректно остановить стенд после прогона теста.


Технически, рецепт представляет собой программу, которая запускается в двух режимах:

1. Start
   * Запускается перед тестом;
   * Имеет с тестом общую рабочую директорию;
   * Может задать переменные окружения для теста;
   * Имеет доступ ко всем `DEPENDS/DATA`, то есть может запускать другие программы так же, как это делает тест;
   * Завершается перед тестом, оставляя после себя запущенный стенд.
2. Stop
   * Запускается после теста;
   * Имеет с тестом общую рабочую директорию, из которой может извлечь информацию о ранее запущенном стенде и погасить его.

Для того, чтобы подключить рецепт к тесту, нужно воспользоваться макросом `USE_RECIPE()`  в ya.make :

```
USE_RECIPE(path/to/recipe arg1 arg2 ...)
```

Если тест описан с использованием макросов `FORK_TESTS / FORK_SUBTESTS`, то рецепт будет подготавливать отдельное окружение для каждого запуска.


## Как написать рецепт

Рецепты удобнее всего оформлять в `PY_PROGRAM`, так как в них доступна привычная библиотека `yatest.common`, предоставляющая интерфейсы доступа к зависимостям теста.

Рассмотрим демонстрационный пример в `devtools/dummy_arcadia/recipe`. Этот рецепт запускает http-сервер, который на любой запрос отвечает константной строкой, заданной в параметрах макроса `USE_RECIPE()`, и оставляет номер порта этого сервера в файле `recipe.port`.
С точки зрения системы сборки это обычная `PY_PROGRAM`, примечательная только своими PEERDIR'ами:

```
PEERDIR(
    library/python/testing/recipe           # библиотека для инициализации recipe
    library/python/testing/yatest_common    # yatest.common от PYTEST
)
```

Первое, что нужно сделать рецепту – правильно распарсить свои аргументы:

```
from library.python.testing.recipe import declare_recipe, set_env

# <...>

if __name__ == "__main__":
    declare_recipe(start, stop)
```

Основная работа выполняется в `start`:

```
def start(argv):

    # находим свободный порт, на котором поднимем сервер, и записываем в переменную окружения, которая будет доступна в тесте
    port = find_free_port()
    set_env("RECIPE_PORT", str(port))

    # рецепт имеет общую рабочую директорию с тестом, поэтому создаваемые файлы будут тоже доступны в тесте
    with open("recipe.port", "w") as f:
        f.write(str(port))

    # инициализируемся некоторым образом
    # в argv окажутся те значения, которые были написаны в USE_RECIPE
    global RESPONSE_STRING
    RESPONSE_STRING = " ".join(argv)

    # планируем запустить сервер в этом же процессе, но рецепт должен завершиться перед запуском теста, поэтому форкаемся
    pid = os.fork()
    if pid == 0:
        run(port=port)
    else:
        # сохраняем pid запущенного процесса с сервером в файл, чтобы потом его остановить
        with open("recipe.pid", "w") as f:
            f.write(str(pid))
```

Остановка рецепта выглядит проще:

```
def stop(argv):
    # файл recipe.pid был записан ранее функцией start
    with open("recipe.pid") as f:
        pid = int(f.read())
        os.kill(pid, 9)
```

Для демонстрации этот рецепт использует бинарник `devtools/dummy_arcadia/cat`, получить и запустить его в рецепте можно через `yatest.common`:
```
yatest.common.execute([
    yatest.common.build_path("devtools/dummy_arcadia/cat/cat"), 
    arg
])
```

Чтобы использовать этот рецепт, в файле `ya.make` теста нужно написать следующие макросы (при желании их можно положить в файл `recipe.inc` и подключать в тесте через `INCLUDE`):

```
USE_RECIPE(devtools/dummy_arcadia/recipe/recipe/recipe test1 test2 test3)  
DEPENDS(
  devtools/dummy_arcadia/recipe/recipe
  devtools/dummy_arcadia/cat
)
```

Теперь рассмотрим тест, который этот рецепт использует:

```
# ya.make
UNITTEST()

SRCS(main.cpp)

# заказываем рецепт
INCLUDE(${ARCADIA_ROOT}/devtools/dummy_arcadia/recipe/recipe/recipe.inc)

PEERDIR(yweb/news/fetcher_lib)

END()
```
```
// main.cpp
#include <util/stream/file.h>
#include <library/unittest/registar.h>
#include <yweb/news/fetcher_lib/fetcher.h>


SIMPLE_UNIT_TEST_SUITE(Suite)
{
    SIMPLE_UNIT_TEST(Test)
    {
        // узнаем порт сервера из рецепта
        TString port = getenv("RECIPE_PORT");
        
        // делаем запрос, проверяем результат
        NHttpFetcher::TRequestRef req = new NHttpFetcher::TRequest();
        req->Url = "http://localhost:" + port;
        NHttpFetcher::TResultRef result = NHttpFetcher::FetchNow(req);

        UNIT_ASSERT(result->DecodeData().Contains("test1 test2 test3"));
    }
};
```

## Рецепт для Local YT

В `mapreduce/yt/python/recipe` есть готовый рецепт для тестов, которым нужен локальный `YT`. Рецепт пишет порт прокси в `yt_proxy_port.txt`, а также устанавливает переменную среды `YT_PROXY`.

Рядом лежат примеры таких тестов на `cpp-ut`, `java-junit`, `pytest`, `exectest`.


## Популярные рецепты

Некоторые наиболее часто используемые рецепты можно найти в каталоге [`library/recipes`](https://a.yandex-team.ru/arc/trunk/arcadia/library/recipes). При этом, поскольку рецепт может иметь свои зависимости, распространенной практикой является использование `*.inc` файлов, в которых объявлены все макросы, необходимые для подключения конкретного рецепта. Такой `*.inc` файл можно добавить в свой `ya.make` при помощи инструкции `INCLUDE`.


## Дальнейшие планы

Мы планируем сделать отдельный тип сборочного таргета `RECIPE`, в котором можно будет явно указывать необходимые рецепту `DEPENDS` и `DATA` (сейчас рецепт не имеет своих зависимостей, все зависимости указываются только в конечном тесте, поэтому приходится пользоваться хаком с `recipe.inc` и `INCLUDE`).

Мы ожидаем, что команды, разрабатывающие общие компоненты, напишут свои рецепты, которые будут поддерживать и развивать; а также найдутся энтузиасты, которые сделают удобные рецепты для опенсорсных компонент, таких как Postgres, Mongo, etc.

Рецепты для конкретных проектов следует хранить в директориях этих проектов, рецепты для общих технологий можно складывать в `library/python/testing`.