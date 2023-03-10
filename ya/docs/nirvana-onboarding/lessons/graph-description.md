# Урок 20. Описание графа

{% include [note-alert](../_includes/onboarding-alert.md) %}

Писать описание и комментарии к продакшн-процессам считается хорошим тоном. Не пренебрегайте этой практикой, описания и комментарии помогут коллегам быстрее разобраться в ваших процессах.Нирвана предоставляет возможности для описания и комментирования ваших процессов. В этом уроке речь пойдет про:

## Описание воркфлоу {##step-1}

Чтобы добавить описание к воркфлоу:
1. Откройте любой инстанс в нужном воркфлоу
1. Перейдите в **Workflow details**

   ![](https://jing.yandex-team.ru/files/kalmykov-ad/2021-07-27_15-06-43.png)

1. В открывшемся окне в правом нижнем углу нажимаем **Edit**
1. Редактируем описание воркфлоу в поле **Description**. Для разметки поддерживается синтаксис [Markdown](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet). Чтобы включить режим предпросмотра, нажмите **Preview**.

   Описание воркфлоу можно увидеть в двух местах:

1. На странице любого инстанса данного воркфлоу, на панели **Instance Details** на вкладке **Details**:

   ![](https://jing.yandex-team.ru/files/kalmykov-ad/2021-07-27_16-10-45.png)

1. На карточке воркфлоу в дереве навигации. На этой странице можно также отредактировать описание, нажав на **Details**

   ![](https://jing.yandex-team.ru/files/kalmykov-ad/2021-07-27_16-21-37.png)

## Комментарий к инстансу {#step-2}

Комментарий к инстансу можно добавить двумя способами:

1. На панели **Instances** слева навести мышку на нужный инстанс, в появившемся окошке нажать **Edit comment**:

   ![](https://jing.yandex-team.ru/files/kalmykov-ad/Untitled.png)

1. Нажать карандашик напортив **Comment** на панели **Instance Details** на вкладке **Details**:

   ![](https://jing.yandex-team.ru/files/kalmykov-ad/2021-07-27_17-28-16.png)

После добавления комментария не забудьте сохранить инстанс, нажав **Save**

Комментарий к инстансу отображается на панели **Instances** и **Instance Details** на вкладке **Details**:

  ![](https://jing.yandex-team.ru/files/kalmykov-ad/2021-07-27_17-35-02.png)

   > Pro tips: Хорошей практикой является создание нового инстанса на каждое изменение в базовом воркфлоу ({red}(есть ли к этому моменту знание, что такое базовый воркфлоу?)). При создании нового инстанса в базовых воркфлоу старайтесь писать краткий комментарий: что было изменено, кто внёс эти изменения и в рамках какого тикета (если такой имеется). Пример: ![](https://jing.yandex-team.ru/files/kalmykov-ad/2021-07-27_17-46-26.png)

## Текстовые блоки {#step-3}

Текстовые блоки позволяют описывать, что происходит в вашем графе, а также акцентировать внимание на его важных местах.![](https://jing.yandex-team.ru/files/kalmykov-ad/2021-07-29_17-31-51.png =800x)Текстовые блоки бывают трех видов "важности" – Info, Warning и Critical. У каждого такого блока есть основной текст и дополнительное описание, которое видно только на большом масштабе.Есть два способа добавления текстового блока:
1. Откройте вкладку **Tools** в палитре и перетяните нужный блок себе на граф:

   ![https://jing.yandex-team.ru/files/madfriend/block-comments.gif](https://jing.yandex-team.ru/files/madfriend/block-comments.gif)

1. Выделите несколько блоков на графе и нажмите кнопку **Wrap in a text block** в правой панели – это создаст обрамляющий текстовый блок:

   ![https://jing.yandex-team.ru/files/madfriend/2020-05-15T10:11:44Z.d2b08f6.png](https://jing.yandex-team.ru/files/madfriend/2020-05-15T10:11:44Z.d2b08f6.png)

   Чтобы отредактировать текст, дважды кликните по блоку – или просто выделите блок кликом и отредактируйте все свойства в правой панели. Чтобы изменить положение на графе, выделите блок, а затем перетащите.
