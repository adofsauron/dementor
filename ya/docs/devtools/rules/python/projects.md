# Правила разработки на Python

При разработке на Python в едином репозитории различают два типа проектов:

1. **Обычный проект** - самостоятельное приложение или библиотека, от которой зависят другие приложения. Например:

    * backend для веб-сервиса
    * расчёт факторов ранжирования, запускаемый по расписанию
    * клиент к [YT](https://yt.yandex-team.ru)

    К таким проектам применяются следующие правила:
    
    * **Зависимости по исходникам**. Можно использовать внешние библиотеки, которые уже доступны в едином репозитории. Запрещается использование [Pip](https://pypi.org/project/pip/) и подобных инструментов.
    * Внешние библиотеки должны располагаться в **contrib/python**. Для каждой библиотеки на весь репозиторий может одновременно использоваться только одна её версия.
    * Исходные коды приложений компилируются в **единый исполняемый файл без внешних зависимостей**. Все внешние библиотеки линкуются в такой файл статически средствами инструметов сборки.
    * Сборка исходных кодов описывается файлами **ya.make**, которые описаны в соответствующем разделе настоящей документации.
    ** Все новые проекты пишутся на **Python 3** или т.н. **PY23**, т.е. на Python 2 с использованием выражений, совместимых с Python 3 (подробнее смотри статьи: [раз](https://python-future.org/compatible_idioms.html), [два](https://docs.python.org/3/howto/pyporting.html)).
    
    Возможные исключения из правил:
    
    * Необходимость применения закрытых (проприетарных) библиотек, которые невозможно использовать с системой сборки;
    * Необходимость использования Django, SQLAlchemy и других библиотек, новые версии которых не поддерживают Python 2. В этом случае допускается хранение и использование нескольких версий таких библиотек одновременно.
    
2. **Аналитические расчёты** - вспомогательные скрипты, например:

    * код для [Jupyter Notebook](https://jupyter.org/)
    * одноразовый расчёт при помощи кода на Python
    
    Специальных правил к таким проектам не применяется.

