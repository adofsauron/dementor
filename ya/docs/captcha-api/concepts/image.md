# Метод image {#image}

Возвращает изображение капчи для показа в браузере.

Браузер пользователя вызывает данный метод, чтобы получить изображение капчи. Ссылку на изображение сервис получает от капча-сервера и без изменений размещает ее на HTML-странице, которую передает пользователю.

## Синтаксис запроса {#request}
```
http://u.captcha.yandex.net/image
  ? [key=<captcha_key>]
```
 **Параметр** | **Значение**
 --- | ---
 key | Ключ капчи, полученный при вызове метода [generate](generate.md).
 
> ## Пример
> `http://u.captcha.yandex.net/image?key=10pIGk1YA_uMYERwCg9Zzltn_cQ3bBOF`

## Формат ответа {#answer}

Ответом является изображение капчи в формате GIF (200х60 px, глубина цвета 8 бит).

## Сообщения об ошибках {#image}

Метод возвращает сообщения о стандартных HTTP-ошибках:

- 404 — истек тайм-аут теста;
- 500 — ошибка в работе капча-сервера, причина ошибки не уточняется;
- 502 — капча-сервер не обработал запрос (не запущен или высокая нагрузка).