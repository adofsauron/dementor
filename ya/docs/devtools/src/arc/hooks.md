# Клиентские хуки

**Хук** (англ. hook - обработчик) — произвольный набор команд, запускаемый в рабочей копии по наступлению некоторых событий. Например, можно запустить какие-то дополнительные проверки при создании коммита, позволяющие найти очевидные проблемы в коде, и сообщить об этом. Хуки настраиваются в локальном конфигурационном файла Arc (например, `~/.arcconfig`). Для настройки хуков скриптами можно пользоваться командой `arc config`. Каждый пользователь может настроить собственный набор хуков и не предполагается хранение хуков в едином репозитории. В данный момент поддерживаются следующие хуки:

## Pre-commit Hook { #pre-commit }

Команды, привязанные к этому хуку выполняются при выполнении команды `arc commit` перед подготовкой сообщения коммита.
Проверка описывается отслеживаемой директорией и выполняемой командой:

```
[pre-commit-hook "project/path"]     # проверки директории "project/path"
    my-hook-name = command/to/check  # какую команду запускать
```

* Если пользователь вносит изменения в отслеживаемую директорию, то будет запущена указанная команда в данной директории. Если запуск команды завершится с ошибкой, то создание коммита будет прервано с ошибкой.
* Если путь к команде абсолютный, то он берётся с локальной ФС пользователя, а если относительный — то из репозитория относительно его корня.
* Путь к отслеживаемой директории всегда рассчитывается относительно корня репозитория. Пустой путь или `/` означает проверку на любые изменения в репозитории.
* Можно пропустить все проверки ключом `--no-verify`. Можно пропустить конкретную проверку ключом `--skip-hook=HOOK-NAME`. Если есть несколько разных проверок с таким именем на разные директории, то будут пропущены все.
* Весь вывод проверки отправляется в стандартный поток ошибок `stderr`.
* Нельзя иметь несколько проверок с одинаковым именем в пределах одной проверяемой директории — будет использована только одна из них.
* Можно иметь несколько разных проверок с одинаковой директорией, и несколько разных проверок с одинаковой командой. Они будут запускаться независимо друг от друга.
* Проверки запускаются последовательно, в лексикографическом порядке директорий и названий проверок. Если вам важен порядок запуска проверок, то рекомендуем начинать названия проверок с числа.

Пример настройки хуков в конфигурационном файле:

```
[pre-commit-hook ""]
    01-check-all = junk/user/some-check
    02-check-other-all = /home/user/some-other-check

[pre-commit-hook "project/lib"]
    03-check-project-lib = project/path/common-check

[pre-commit-hook "project/bin"]
    04-check-project-bin = project/path/common-check
```

## Post-commit Hook { #post-commit }

Команды, привязанные к этому хуку выполняются после выполнения `arc commit`. Их результат не влияет уже случившийся коммит, но влияет на итоговый exit code команды `arc commit`. Правила выбора хуков по путям, поведение ключей `--no-verify` и `--skip-hook=HOOK-NAME`, а также порядок запуска эквивалентны пре-коммитным хукам. При вычислении затронутых хуков используется фактическое содержимое итогового коммита, которое может отличаться от состояния индекса при использовании ключа `--amend`.

```
[post-commit-hook "project/lib"]
    01-check-project-lib = project/path/common-check
```


## Prepare-commit-msg Hook { #prepare-commit-msg }

Команды, привязанные к этому хуку выполняются при выполнении команды `arc commit`
перед созданием коммита сразу после создания файла с заготовкой сообщения коммита,
но перед запуском редактора.
Хуки будут запущены даже если редактирование не требуется.
Проверка описывается отслеживаемой директорией и выполняемой командой:

```
[prepare-commit-msg-hook "project/path"]  # проверки директории "project/path"
    my-hook-name = command/to/check       # какую команду запускать
```

Данный хук предназначен для проверки и редактирования сообщения коммита по указанному ему пути.
Запуск модифицирующих команд `arc` из данного хука не предполагается и не рекомендуется.
Не следуетс использовать данный хук вместо `pre`

Команде передаются от 2 до 3 аргументов командной строки:
1. Путь до файла с заготовкой сообщения коммита.
   Хук может редактировать содержимое данного файла.
2. Источник сообщения коммита в виде строки:
   `"message"`, если была использована опция `-m` или `-F`,
   и `"commit"`, если использована опция `--amend`.
3. Если второй аргумент имеет значение `"commit"`,
   то третьим аргументом передается хеш этого коммита.
   Для `--amend` это будет хеш `HEAD^1`.

Например, в скрипте на `bash` принять аргументы нужно так:

```shell
#!/bin/bash
commitMsgFile = $1
msgSource = $2
commitId = $3

echo >> $commitMsgFile
echo Message is from $msgSource $commitId >> $commitMsgFile
```

За единственным исключением порядок и правила запуска хуков такие же,
как и для `pre-commit` хуков.
Отличие от `pre-commit` хуков в том, что хуки `prepare-commit-msg` запускаются даже с ключом `--no-verify`,
но можно пропустить конкретную проверку ключом `--skip-hook=HOOK-NAME`
(если есть несколько разных проверок с таким именем на разные директории, то будут пропущены все).

## Commit-msg Hook { #commit-msg }

Команды, привязанные к этому хуку выполняются при выполнении команды `arc commit` сразу после закрытия редактора.
Хуки будут запущены даже если редактирование не требуется.
Проверка описывается отслеживаемой директорией и выполняемой командой:

```
[commit-msg-hook "project/path"]     # проверки директории "project/path"
    my-hook-name = command/to/check  # какую команду запускать
```

Команде передается единственный аргумент командной строки --- путь до файла с заготовкой сообщения коммита. Хук может редактировать содержимое данного файла.

Порядок и правила запуска хуков такие же, как и для `pre-commit` хуков.
Этот хук будет также пропущен по опции `--no-verify`

## Post-HEAD-change hook { #post-head-change }

Команды привязанные к этому хуку выполняются после изменения текущего коммита рабочей копии. Это происходит, например, при `arc commit`, `arc checkout <branch>` и `arc rebase`. Эти хуки обычно предназначены для настройки окружения в соответствии с текущим коммитом рабочей копии, например, прописывают тулы из конфига в репозитории в `PATH`.

* Запуск хуков производится при любом изменении головного коммита, без каких-либо проверок изменений файлов.
* Команды запускаются в корне репозитория с двумя аргументами: хэшами предыдущего и нового коммита рабочей копии.

```
[post-head-change-hook]
    hook-1 = hook-command
    hook-2 = other-hook-command
```

## Pre-push Hook { #pre-push }

Команды, привязанные к этому хуку выполняются при выполнении команды `arc push` перед отправкой изменений на сервер:
```
[pre-push-hook]
    hook-1 = hook-command
    hook-2 = other-hook-command
```

* Порядок запуска команд работает аналогично `pre-commit hook`.
* Команды запускаются в корне репозитория с двумя аргументами: `имя репозитория` (обычно "arcadia") и `адрес репозитория` (обычно "arc-vcs.yandex-team.ru").
* На стандартный вход (`stdin`) скрипта ему дадут набор строк вида (по одной строке на каждую отправляемую ветку или тэг):
    `localref localhash remoteref remotehash`
    Здесь `localref` — имя локальной ветки, `localhash` — хэш коммита новой головы, `remoteref` — имя серверной ветки, `remotehash` — хэш коммита старой головы. Будет .
* При удалении ветки или тега всегда `localref = (delete)`, `localhash = 0000000000000000000000000000000000000000`.
* При создании новой ветки или тега всегда `remotehash = 0000000000000000000000000000000000000000`. Например, при создании новой ветки, на вход команде придёт: `mybranch 05912859355de9ea60116ab247ff7e30614001d4 users/ivanselin/mybranch 0000000000000000000000000000000000000000`