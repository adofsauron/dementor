# Хорошие и плохие практики

Здесь будем описывать проблемные места в коде, которые хотелось бы поменять или о которых стоит знать, как существующим, так и новым разработчикам

## RxJava

1. Если используете `subject`, то при отдаче сериализуйте его
``` java
PublishSubject.create().toSerialized()
```

2. onError у `observer`'ов:
- договорились, что переопределяем его. Логируем ошибки. Таким образом реьювер увидит, что ты не забыл переопределить, а просто нечего делать в этом случае
- для `Completable`, если нечего делать в случае успеха или ошибки ставим `SimpleCOmpletableObserver`
- неотловленные ошибки попадают в наш кастомный handler `RxJavaGlobalErrorsHandler`

3. При тестировании `timeout` в `RxJava`, нужно сначала дёргать `triggerActions` и потом переводить время. Можно заюзать `extension`-иетод

## AutoValue и data классы

1. Если меняете `AutoValue` класс и добавляете новый метод в билдер, не забудь:
- во все места, где используется `Builder`, добавить вызов нового метода
- если у класса есть метод `testInstance()`, добавить новый метод

2. В методах `testInstance()` у нас принято класть:
- непустые строки
- непустые коллекции (хотя бы один экземпляр должен быть)
- не `null` объекты (юзайте такие же `testInstance()` для подобъектов)

3. Не забывай повысить версию `serialVersionUID`, если меняешь `DTO`

## Android

1. При использовании `StateListAnimator` можно словить `JNI` краш на ряде устройств:
- В файл `ViewExtensions.kt` добавлен метод `View.changeState()`, который позволяет безопасно изменять состояние вью на котором висит `StateListAnimator`