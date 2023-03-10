TL;DR:
1) Модификатор final рекомендуется всегда ставить в полях класса и объекта, если можете себе позволить неизменяемое поле.
2) Модификатор final не рекомендуется ставить по умолчанию в параметрах метода, локальных переменных, для классов и методов.


Модификатор `final` в Java может использоваться в разных случаях, каждый из которых рассмотрен отдельно:
1. Для полей объекта/класса
2. Для параметров метода
3. Для локальных переменных
4. Для классов и методов

## Статические поля класса и поля объекта
```java
public final class X {
    private static final Logger = …;

    private final int a;
    private final List<Long> list;

    public X(int a, List<Long> list) {
        this.a = a;
        this.list = Collections.unmodifiableList(list);
    }
    
    public int getA() {
        return a;
    }
    
    public List<Long> getList() {
        return list;
    }
}
```
Здесь всё довольно просто: статические поля класса помечаются `final` для использования в качестве констант; `final`, применённый к полям объекта, позволяет создать неизменяемый объект. **И то и другое полезно и не снижает удобство чтения и поддержки кода**. Единственный нюанс связан с геттерами на изменяемые объекты (как `List<Long>` в примере выше): здесь рекомендуется либо сразу в конструкторе оборачивать / копировать, либо возвращать неизменяемую обёртку / копию в геттере.
Также модификатор `final` — один из самых простых и эффективных способов для [safe publication](https://shipilev.net/blog/2014/safe-public-construction/).

## Параметры метода
```java
public void x(final int y, final List<Long> list) {
    ...
    y = 1; // Compilation Error
    ...
}
```
В Java объекты всегда передаются в параметры метода по ссылке, примитивы — копируются. Таким образом, даже если не помечать параметр словом final, в точке вызова метода нельзя увидеть изменения переданных примитивов или ссылок на объекты. В итоге `final` для параметра метода позволяет только защититься от случайного присваивания нового значения этому параметру внутри метода. В будущем в аркадийный чекстайл будет добавлена проверка [ParameterAssignmentCheck](https://checkstyle.sourceforge.io/config_coding.html#ParameterAssignment).

Причины, из-за которых, **не рекомендуется к использованию**:
1. Довольно сильно засоряет код визуально, из-за чего его становится труднее воспринимать. Плюс, из-за постоянного использования `final` строка кода с сигнатурой метода может перестать помещаться в 120 символов.
2. Не даёт каких-либо новых гарантий при вызове и семантически неверно: если параметр является, например, объектом класса `ArrayList`, то его можно спокойно изменить, и этот результат будет заметен из точки вызова метода.  
3. Понятие effectively final появилось ещё в Java 8 и, начиная с этой версии, можно передавать локальные переменные (в том числе параметры метода) в лямбды, не помечая их как `final` явным образом.

## Локальные переменные
```java
final long x = 1;
final var map = new HashMap<Long, Long>();
map.put(x, 1);
```
Данный случай использования сильно похож на предыдущий, учитывая что параметры метода в Java являются по сути локальными переменными. Основных отличия здесь два:
1. Нельзя сделать автоматическую проверку повторного присваивания, потому что достаточно большая часть локальных переменных  всё-таки изменяются и будет много предупреждений на них.
2. Семантические ошибки выражены сильнее: локальные переменные изменяются сильно чаще, как map в примере. Модификатор применяется ради защиты от случайной маловероятной ошибки и сама по себе неизменяемость внутри метода конкретной переменной не важна.
Локальные переменные **не рекомендуется помечать как final**.

## Классы и методы
В этих случаях модификатор `final` позволяет запретить наследование от класса и переопределение методов. Из минусов: становится невозможно использовать моки для этого класса / метода. Рекомендуется помечать `final` не все классы, а только те, для которых есть веские причины, почему от них нельзя наследоваться. То же самое относится и к запрету переопределения методов.