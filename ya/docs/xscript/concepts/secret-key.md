# Секретный ключ

В целях безопасности XScript предоставляет возможность [сгенерировать секретный ключ](../appendices/xslt-functions.md#get-secret-key) для подписи запроса. Ключ можно добавить в список параметров POST-запроса и [проверить](../appendices/xslt-functions.md#check-secret-key) на вызываемой странице.

Ключ генерируется с использованием шифрования md5 по следующей формуле:

```
key = md5([salt:]uid:yandexuid:days)
```

где:
- `salt` — Фраза свободного формата. Задается в файле настроек ключем [yandex/secret-key-salt](../appendices/config-params.md#yandex-secret-key-sault). Не используется в формуле, если значение не задано;
- `uid` — LiteUID пользователя. Заполняется всегда, при отсутствии авторизации - нулем (в этом случае в расчете секретного ключа используется yandexuid);
- `yandexuid` — yandexuid пользователя. Если uid равен нулю, заполняется значением из куки yandexuid, а при его отсутствии, расчитывается новое значение, которое используется для генерации ключа и сохраняется в куке yandexuid;
- `days` - это количество дней, прошедших с 1 января 1970 года, то есть результат целочисленного деления текущего unix_time на 86400.

Ключ действителен в течение текущих и следующих суток с момента генерации.

При проверке ключа учитывается uid пользователя, а если пользователь не авторизован - значение куки yandexuid. Если uid нулевой и кука отсутствует, считается, ключ проверку не прошел.

Кроме uid или yandexuid при проверке учитывается дата: в превую очередь рассчитывается ключ c текущей датой, а при его несовпадении с проверяемым ключом, рассчитывается ключ с датой предыдущего дня. Если и он не совпадает со входным, считается, что ключ проверку не прошел.

Секретный ключ также может быть сгенерирован по одной лишь куке `yandexuid` без использования авторизационной информации пользователя. Его можно получить xslt-функцией [get-easy-secret-key](../appendices/xslt-functions.md#get-easy-secret-key).


### Узнайте больше {#learn-more}
* [Параметр SecretKey](./parameters-matching-ov.md#secret-key)
* [Параметр EasySecretKey](./parameters-matching-ov.md#easy-secret-key)
* [Параметр CheckSecretKey](./parameters-matching-ov.md#check-secret-key)