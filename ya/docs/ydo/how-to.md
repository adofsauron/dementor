# Инструкция как добавить новый раздел или поправить существующий

{% note warning %}

Общие соглашения и техно надо добавлять на уровне общих страниц. То, что кастомно для сервиса - на уровень сервиса. Документация для компонентов должна лежать рядом с компонентом

**Перед тем как добавить информацию проверьте нет ли уже её в доке.** Сделать это можно с помощью поиска. Не должно быть дублирования. Если нужно упомянуть добавленную информацию - добавьте ссылку на нужное место в доке

**При внесении крупных изменений призывайте на ревью [@lakate](https://staff.yandex-team.ru/lakate)**

**Нельзя параллельно вносить изменнеия в документацию и в сервис. Автоматика деплоя документации на прод не срабатает в таком случае**

{% endnote %}

## Как добавить новый раздел/ страницу документации

1. При добавлении нового раздела доки надо создать новую папку в корне проекта или внутри родительского раздела
2. Создать внутри нужного раздела md-файл
3. В файле описать документацию той части, которую добавляете. Синтаксис md-файлов можно посмотреть [тут](https://docs.yandex-team.ru/docstools/examples)
4. Теперь необходимо добавить отображение новой части доки в оглавлении. Сделать это нужно в файле `toc.yaml`, [дока](https://docs.yandex-team.ru/docstools/settings/toc)

## Как отредактировать существующую страницу документации

При редактировании существующей страницы документации необходимо только поправить md-файл. Вносить изменения в `toc.yaml` не нужно

## Как вносить изменения в документацию:

Сделать это можно 2-мя способами:
1. Внести изменения локально и затем закоммитить их и запушить
2. Внести изменения в интерфейсе арканума
    * Внести изменения, нажав на карандаш

    {% cut "Изображение" %}

    ![text](_assets/edit-icon.png "Как внести изменения в файл" =800x300)

    {% endcut %}

    * После того, как файл поправлен, надо закоммитить изменения

    {% cut "Изображение" %}

    ![text](_assets/commit-icon.png "Как закоммитить изменения в файл" =800x200)

    {% endcut %}

    * Оставьте выбранными фильтры, только напишите валидное название коммита

    {% cut "Изображение" %}

    ![text](_assets/commit-modal.png "Как закоммитить изменения в файл" =500x300)

    {% endcut %}

    * Дальше все сделает автоматика: создастся ПР, после прохождения проверок изменения автоматически задеплоятся на тестинг, затем ПР вольется и изменения автоматически раскатятся на прод (~5-7 минут). Чтобы проверить что изменения выкатились на прод, можно посмотреть какая версия в проде

    {% cut "Изображения" %}

    ![text](_assets/manage-docs.png "Manage docs" =800x400)

    ![text](_assets/manage-docs-modal.png "Какая версия в проде" =400x300)

    {% endcut %}


{% note warning %}


1. Деплой на тестинг и прод происходит автоматически
2. В каждом PR есть бета, она находится в комментарии

{% cut "Изображение" %}

![text](_assets/beta-comment.png "Бета" =400x100)

{% endcut %}

{% endnote %}

{% note info %}

Со всеми вопросами можно приходить к [@lakate](https://staff.yandex-team.ru/lakate)

{% endnote %}