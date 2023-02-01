# Пакетирование и объединение : макросы

Большинство макросов, используемых в модулях [`PACKAGE`](./modules#package) и [`UNION`](./modules#union) — это общие макросы, позволяющие так или иначе сделать файлы частью модуля.

К таким макросам относятся:

- [`RUN_PROGRAM`](../common/macros#run_program) и подобные - генерация файлов программой или скриптом 
- [`FROM_SANDBOX`](../common/data#from_sandbox) - загрузка файлов из Sandbox
- [`BUNDLE`](../common/macros#bundle) - получение файла-результата сборки другого модуля
- [`COPY_FILE`](../common/macros#copy_file) - получение файлов из репозитория

Есть только пара макросов свойственных именно этим модулям.

## PACK

```
PACK(Ext)
```

Позволяет получить результат сборки [`PACKAGE`](./modules#package) в виде архива. Используется только для этого модуля. 

- Параметр `Ext` задаёт расширение, по которому определяется как паковать. Поддерживаются варианты `tar` и `tar.gz`. 

## FILES

```
FILES(Files...)

```

Приносит исходные файлы в сборочную директорию модуля и в модулях [`UNION`](./modules#union) и [`PACKAGE`](./modules#package) делает их результатами модуля.
По сути, это упрощённая и сокращённая форма макроса [`COPY_FILE`](../common/macros#copy_file) для приноса в сборку списка файлов без переименования.

{% note tip %}

При обычной сборке, когда результаты оказываются символьными ссылками в репозитории макрос `FILES` как будто бы не имеет эффекта: его результаты оказались бы на месте существующих файлов,
и потому при создании символьных ссылок они игнорируются. Однако, макрос всегда обрабатывается и имеет два видимых эффекта:

1. При сборке в отдельную директорию с параметром [`-o`/`--output`](../../usage/ya_make/#results) файлы будут скопированы в выходную директорию, как и все остальные результаты модуля.
2. При [*резолвинге*](../../general/how_it_works.md#resolving) входных файлов такой файл может быть выбран, если он будет указан явно (как `${ARCADIA_BUILD_ROOT}/path/to/the.file`) или
   окажется наиболее приоритетным из доступных (например, при [*резолвинге*](../../general/how_it_works.md#resolving) инклудов, где сборочная директория имеет приоритет над репозиторием).

{% endnote %}

**Пример:**

Построим `devtools/examples/package/data`.

**devtools/examples/package/data/ya.make**

```(yamake)
OWNER(g:ymake g:yatool)

UNION()

FILES(
    data.txt
)

PEERDIR(
    devtools/examples/package/union2
)

END()
```

Строим в отдельную директорию, чтобы увидеть эффект макроса [`FILES`](./macros#files) 

```
[=] ya make devtools/examples/package/data -o ~/res
Ok
```

В результате видим файл `~/res/devtools/examples/package/data/data.txt`.

