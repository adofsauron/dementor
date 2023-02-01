# Тесты : общие макросы

## Зависимости

### DEPENDS() { #depends }

```
DEPENDS(path1 [path2...])
```

Указывает зависимости на другие проекты, которые нужно собрать и результаты которых должны быть доступны тесту. В параметрах перечисляются относительные пути от корня `arcadia`.


## Параметры suite { #params }

### SIZE() { #size }

```
SIZE(SMALL | MEDIUM | LARGE)
```

Задаёт размер теста. Сейчас существует три размера:

* **SMALL** - максимальный таймаут *60s*. Размер по умолчанию.
* **MEDIUM** - максимальный таймаут *600s*.
* **LARGE** - максимальный таймаут *3600s*.


### TIMEOUT() { #timeout }

```
TIMEOUT(time)
```

Устанавливает таймаут на запуск всей сьюиты. По умолчанию равен:
* *60s (1min)* для **SMALL** тестов;
* *600s (10min)* для **MEDIUM** тестов;
* *3600s (1hour)* для **LARGE** тестов.

{% note info %}

Таймаут не может быть больше, чем максимальная длительность теста данного размера.

Если запуск сьюиты разбит на части ([`FORK_TEST_FILES()`](#fork_test_files)/[`FORK_TESTS()`](#fork_tests)/[`FORK_SUBTESTS()`](#fork_subtests)), то указанный таймаут ограничивает время работы каждого запуска в отдельности.

{% endnote %}


### TAG() { #tag }

```
TAG(tag1 [tag2...])
```

Выставляет пользовательский тег.

**Запуск тестов:**
* `ya:manual`: тест не будет запускаться, если явно не указано запускать тесты с таким тегом;
* `ya:norestart`: тест не будет перезапускаться при определенных ошибках;
* `ya:not_autocheck`: не запускать тест при проверке pull requests;

**Логирование:**
* `ya:trace_output`: включает логирование создаваемых файлов и их размеров при помощи системного вызова [ptrace](https://en.wikipedia.org/wiki/Ptrace). Работает только под Linux. Может замедлять тестирование;
* `ya:sys_info`: добавляет вывод системной информации в лог до и после выполнения всех рецептов и тестов;
* `ya:full_logs`: приводит к падению сьюиты, если размер логов привысил **100Мб**;
* `ya:huge_logs`: увеличивает ограничение на размер логов теста с **100Мб** до **1Гб**;

**large-тесты**:
* `ya:fat`: помечает тест как `LARGE`;
* `ya:force_distbuild`: запускает тест в distbuild вне зависимости от его размера;
* `ya:force_sandbox`: запускает тест в Sandbox. Используется только вместе с тегом `ya:fat`;
* `ya:sandbox_coverage`: включает `LARGE` тесты в подсчет [покрытия](https://docs.yandex-team.ru/devtools/test/coverage). Нужно использовать вместе с `ya:force_sandbox`. 
* `ya:privileged`: запускает тесты в [Sandbox](https://docs.yandex-team.ru/sandbox/dev/requirements#privileged) в контейнере от имени root.
* `ya:noretries`: тесты с таким тегом не будут запускаться в автосборке повторно;

**Остальные:**
* `ya:notags`: используется для фильтрации тестов, не имеющих тегов;
* `ya:no_graceful_shutdown`: завершает выполнение процесса с тестами при помощи сигнала `SIGQUIT` вместо `SIGTERM`. Это позволяет, например, поймать стектрейс состояния, в котором находится тест;
* `ya:external`: уведомляет систему, что тест использует внешние системы (сеть, внешние базы данных). Это значит, что такой тест потенциально нестабильный. Уведомления о поломках таких тестов будут приходить только владельцам теста и не будут приходить авторам комита, на котором тест сломался;

**Sandbox-теги**:
* `sb:XXXX`: позволяет задать набор [тегов](https://docs.yandex-team.ru/sandbox/agents#tags) для выбора агента [Sandbox](https://sandbox.yandex-team.ru/home). При указании нескольких тегов `sb:`, они соединяются через логическое `ИЛИ`;
* `sb:ttl=inf`: позволяет задать `ttl` для создаваемого в таске `YA_MAKE` ресурса `BUILD_OUTPUT`. Тег следует использовать, если автозапуск `LARGE` тестов от лица вашего робота потребляет много дисковой квоты из-за больших выходных данных в этом ресурсе. `ttl` по умолчанию равен **14 дней**.

### REQUIREMENTS() { #requirements }

```
REQUIREMENTS(
    [cpu:<count>]
    [disk_usage:<size>]
    [ram:<size>]
    [ram_disk:<size>]
    [container:<id>]
    [network:<restricted|full>]
    [dns:<default|local|dns64>])
```

Позволяет настроить требования к окружению, в котором будет выполняться тест.

* **container**:  ID контейнера, в котором будет выполняться тест;
* **cpu**: количество процессорных ядер, необходимых тестам для корректной работы. По умолчанию равен 1. Конечное значение рассчитывается как `min(требуемое, количество_ядер_на_хосте)`;
* **disk_usage**: необходимый объём свободного места на файловой системе в *Gb*;
* **dns**: настраивает использование [NAT64](https://en.wikipedia.org/wiki/NAT64) в [Sandbox](https://docs.yandex-team.ru/sandbox/dev/requirements#dns). 
  * `default`: доступ в интернет по IPv4 невозможен. Используется по умолчанию;
  * `local`: используются настройки из файла `/etc/resolv.conf.local`. Имеет смысл только для кастомных контейнеров, в которых этот файл существует;
  * `dns64`: доступ в Интернет по IPv4 возможен. Весь резолвинг имён происходит через ферму `ns64-cache.yandex.net`;
* **network**: тип доступа к сети:
  * `full`: есть доступ во внешнюю сеть;
  * `restricted`: можно использовать только `localhost`;
* **ram**: необходимый объём оперативной памяти в *Gb*;
* **ram_disk**: необходимый объём RAM диска в *Gb*;
* **sb_vault**: sandbox-секрет. Должен соответствовать паттерну: `<ENV_NAME>=<value|file>:<owner>:<vault key>`

{% note info %}

1. Параметры `disk_usage` и `container` работают только для `LARGE` тестов.
2. Параметр `container` позволяет запускать тесты в заранее подготовленном окружении в виде образа LXC или Porto контейнера.
3. Поскольку RAM-диск использует оперативную память, суммарное значение полей `ram` и `ram_disk` не может превышать максимально возможное значение размера оперативной памяти для тестов данного размера.

{% endnote %}

Актуальные требования можно посмотреть [вот тут](https://a.yandex-team.ru/arc/trunk/arcadia/devtools/ya/test/const/__init__.py#L230), словарь `TestSize.DefaultRequirements`.

| Requirement | SMALL     | MEDIUM    | LARGE     |
|:-----------:|:---------:|:---------:|:---------:|
| CPU         | `1 .. 4`  | `1 .. 4`  | `1 .. 4`  |
| RAM         | `8 .. 32` | `8 .. 32` | `8 .. 32` |
| RAM disk    | `0 .. 4`  | `0 .. 4`  | `0 .. 4`  |

Требования для LARGE учитываются только при запуске на Distbuild - для тестов размеченных с помощью тега `TAG(ya:force_distbuild)`.
По умолчанию LARGE тесты сейчас запускаются в Sandbox, где нет этих ограничений.


### ENV() { #envs }

```
ENV(key=[value])
```

Задает значение переменной окружения `key` значение `value` при запуске теста.


## Параллельный запуск тестов { #parallel }

### FORK_TESTS() { #fork_tests }

```
FORK_TESTS(mode)
```

Разбивает запуск suite на несколько chunk'ов. По умолчанию количество chunk'ов равно 10. Это значение можно изменить с помощью макроса [`SPLIT_FACTOR(x)`](#split_factor). В отличие от [`FORK_SUBTESTS`](#fork_subtests) считает тесты, объединенные в один класс, неделимой сущностью.

Каждый chunk наследует общие параметры теста из `ya.make`: размер, таймаут, требования на ресурсы. Поэтому этот макрос удобно использовать, чтобы распараллелить тесты, которые не укладываютсмя в таймаут.

Параметр `mode` определяет, каким образом будет происходить распределение тестов по chunk'ам. Может быть равен `SEQUETIAL` или `MODULO`. Если аргумент не указан, то по умолчанию равен `SEQUENTIAL`. `SEQUENTIAL` распределяет тесты равными диапозонами, предварительно отсортировав их по имени. `MODULO` разделяет тесты по модулю, предварительно их отсортировав.


### FORK_SUBTESTS() { #fork_subtests }

```
FORK_SUBTESTS(mode)
```

Разбивает запуск suite на несколько chunk'ов. По умолчанию количество chunk'ов равно 10. Это значение можно изменить с помощью макроса [`SPLIT_FACTOR(x)`](#split_factor). В отличие от [`FORK_TESTS`](#fork_tests) может разбивать тесты, объединенные в один класс, в разные chunk'и. Нежелательно использовать, если у классов есть тяжелые подготовительные стадии.

Каждый chunk наследует общие параметры теста из `ya.make`: размер, таймаут, требования на ресурсы. Поэтому этот макрос удобно использовать, чтобы распараллелить тесты, которые не укладываютсмя в таймаут.

Параметр `mode` определяет, каким образом будет происходить распределение тестов по chunk'ам. Может быть равен `SEQUETIAL` или `MODULO`. Если аргумент не указан, то по умолчанию равен `SEQUENTIAL`. `SEQUENTIAL` распределяет тесты равными диапозонами, предварительно отсортировав их по имени. `MODULO` разделяет тесты по модулю, предварительно их отсортировав.


### FORK_TEST_FILES() { #fork_test_files }

```
FORK_TEST_FILES()
```

Разбивает прогон тестов из suite на количество chunk'ов, равное количеству файлов с тестами, перечисленных в `ya.make`. Каждый запуск работает только с одним конкретным файлом.

Можно комбинировать вместе с [`FORK_TESTS()`](#fork_tests) и [`FORK_SUBTESTS()`](#fork_subtests). В этом режиме будет происходить дополнительное разбиение тестов для каждого файла.

{% note warning %}

Поддержан только для pytest.

{% endnote %}


### SPLIT_FACTOR() { #split_factor }

```
SPLIT_FACTOR(x)
```

Изменяет количество chunk'ов при параллельном запуске тестов. Можно использовать только с [`FORK_TESTS()`](#fork_tests) и [`FORK_SUBTESTS()`](#fork_subtests).


## Специфичные макросы { #specific }

### USE_RECIPE() { #recipe }

```
USE_RECIPE(path [arg1 arg2...])
```

Подключает рецепт для настройки тестового окружения к тесту. Если тест описан с использованием макросов `FORK_TEST() / FORK_SUBTESTS()`, то рецепт будет подгатавливать отдельное окружение для каждого запуска.

[Здесь](recipe) можно прочитать про то, как настраивать тестовое окружение с помощью рецептов.


### SKIP_TEST() { #skip_test }

```
SKIP_TEST(Reason)
```

Отключает тесты для всей модульной сьюиты. Параметром указывается причина отключения.


### NO_LINT() { #no_lint }

```
NO_LINT()
```

Отключает codestyle проверки для java и python.

{% note warning %}

Использование `NO_LINT()` разрешено только в `contrib`.

{% endnote %}