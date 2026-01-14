# GtkExpression

GtkExpression — это фундаментальный тип.
Это не потомок GObject.
GtkExpression предоставляет способ описания ссылок на значения.
GtkExpression необходимо вычислить, чтобы получить значение.

Это похоже на арифметическое вычисление.

~~~
1 + 2 = 3
~~~

`1+2` — это выражение.
Оно показывает способ вычисления.
`3` — это значение, полученное из выражения.
Вычисление — это расчёт выражения и получение значения.

GtkExpression — это способ получить значение.
Вычисление похоже на расчёт.
Значение получается путём вычисления выражения.

## Константное выражение

Константное выражение (GtkConstantExpression) предоставляет константное значение или экземпляр при его вычислении.

~~~C
  GValue value = G_VALUE_INIT;
  expression = gtk_constant_expression_new (G_TYPE_INT,100);
  gtk_expression_evaluate (expression, NULL, &value);
~~~

- GtkExpression использует GValue для хранения значения.
GValue — это структура и контейнер для хранения типа и значения.
Сначала она должна быть инициализирована с помощью `G_VALUE_INIT`.
Будьте осторожны, что `value` — это структура, а не указатель на структуру.
- Константное выражение создаётся с помощью функции `gtk_constant_expression_new`.
Параметр функции — это тип (GType) и значение (или экземпляр).
Это выражение содержит константное значение.
`G_TYPE_INT` — это тип, зарегистрированный в системе типов.
Это целочисленный тип.
Некоторые типы показаны в следующей таблице.
- `gtk_expression_evaluate` вычисляет выражение.
Он имеет три параметра: выражение для вычисления, экземпляр `this` и указатель на GValue для установки значения.
Экземпляр `this` не нужен для константных выражений.
Поэтому второй аргумент — NULL.
`gtk_expression_evaluate` возвращает TRUE, если он успешно вычисляет выражение.
В противном случае возвращает FALSE.
- Если он возвращает TRUE, GValue `value` устанавливается со значением выражения.
Тип значения — int.

|GType            |C тип |имя типа  |примечания                         |
|:----------------|:-----|:---------|:----------------------------------|
|G\_TYPE\_CHAR    |char  |gchar     |                                   |
|G\_TYPE\_BOOLEAN |int   |gboolean  |                                   |
|G\_TYPE\_INT     |int   |gint      |                                   |
|G\_TYPE\_FLOAT   |float |gfloat    |                                   |
|G\_TYPE\_DOUBLE  |double|gdouble   |                                   |
|G\_TYPE\_POINTER |void *|gpointer  |общий указатель                    |
|G\_TYPE\_STRING  |char *|gchararray|null-терминированная строка C      |
|G\_TYPE\_OBJECT  |      |GObject   |                                   |
|GTK\_TYPE\_WINDOW|      |GtkWindow |                                   |


Пример программы `exp_constant_simple.c` находится в каталоге `src/expression`.

@@@include
expression/exp_constant_simple.c
@@@

- 9: Создаётся константное выражение. Оно содержит значение int 100. Переменная `expression` указывает на выражение.
- 11-14: Вычисляет выражение. Если это успешно, показывает значение в stdout. В противном случае показывает сообщение об ошибке.
- 15-16: Освобождает выражение и сбрасывает GValue.

Константное выражение обычно используется для передачи константного значения или экземпляра другому выражению.

## Выражение свойства

Выражение свойства (GtkPropertyExpression) ищет свойство в экземпляре GObject.
Например, выражение свойства, которое ссылается на свойство "label" в объекте GtkLabel, создаётся так.

@@@if gfm
~~~C
expression = gtk_property_expression_new (GTK_TYPE_LABEL, another_expression, "label");
~~~
@@@else
~~~{.C}
expression = gtk_property_expression_new (GTK_TYPE_LABEL, another_expression, "label");
~~~
@@@end

Второй параметр `another_expression` — это одно из:

- Выражение, которое даёт экземпляр GtkLabel при его вычислении.
- NULL. Когда передаётся NULL, экземпляр GtkLabel будет передан при вычислении.
Экземпляр называется объектом `this`.

Например,

@@@if gfm
~~~C
label = gtk_label_new ("Hello");
another_expression = gtk_constant_expression_new (GTK_TYPE_LABEL, label);
expression = gtk_property_expression_new (GTK_TYPE_LABEL, another_expression, "label");
~~~
@@@else
~~~{.C}
label = gtk_label_new ("Hello");
another_expression = gtk_constant_expression_new (GTK_TYPE_LABEL, label);
expression = gtk_property_expression_new (GTK_TYPE_LABEL, another_expression, "label");
~~~
@@@end

Если `expression` вычисляется, второй параметр `another_expression` вычисляется заранее.
Значение `another_expression` — это `label` (экземпляр GtkLabel).
Затем `expression` ищет свойство "label" метки, и результат вычисления — "Hello".

В примере выше второй аргумент `gtk_property_expression_new` — это другое выражение.
Но второй аргумент может быть NULL.
Если это NULL, вместо него используется экземпляр `this`.
`this` передаётся функцией `gtk_expression_evaluate`.

Есть простая программа `exp_property_simple.c` в каталоге `src/expression`.

@@@include
expression/exp_property_simple.c
@@@

- 9-10: `gtk_init` инициализирует инструментарий GTK GUI.
Обычно это не нужно, потому что обработчик запуска по умолчанию GtkApplication выполняет инициализацию.
Создаётся экземпляр GtkLabel с текстом "Hello world.".
- 12: Создаётся выражение свойства.
Оно ищет свойство "label" экземпляра GtkLabel.
Но при создании экземпляр не передаётся, потому что второй аргумент — NULL.
Выражение просто знает, как взять свойство из будущего экземпляра GtkLabel.
- 14-17: Функция `gtk_expression_evaluate` вычисляет выражение с экземпляром 'this' `label`.
Результат сохраняется в GValue `value`.
Функция `g_value_get_string` получает строку из GValue.
Но строка принадлежит GValue, поэтому вы не должны освобождать строку.
- 18-19: Освобождает выражение и сбрасывает GValue.
В то же время строка в GValue освобождается.

Если второй аргумент `gtk_property_expression_new` не NULL, это другое выражение.
Выражение принадлежит вновь созданному выражению свойства.
Поэтому, когда выражения бесполезны, вы просто освобождаете последнее выражение.
Затем оно освобождает другое выражение, которое у него есть.

## Выражение замыкания

Выражение замыкания вызывает замыкание при его вычислении.
Замыкание — это обобщённое представление обратного вызова (указателя на функцию).
Для информации о замыканиях см. [GObject API Reference -- The GObject messaging system](https://docs.gtk.org/gobject/concepts.html#the-gobject-messaging-system).
В каталоге `src/expression` есть простые примеры замыканий `closure.c` и `closure_each.c`.

Существует два типа выражений замыканий: GtkCClosureExpression и GtkClosureExpression.
Они соответствуют GCClosure и GClosure соответственно.
Когда вы программируете на языке C, подходят GtkCClosureExpression и GCClosure.

Выражение замыкания создаётся с помощью функции `gtk_cclosure_expression_new`.

@@@if gfm
~~~C
GtkExpression *
gtk_cclosure_expression_new (GType value_type,
                             GClosureMarshal marshal,
                             guint n_params,
                             GtkExpression **params,
                             GCallback callback_func,
                             gpointer user_data,
                             GClosureNotify user_destroy);
~~~
@else
~~~{.C}
GtkExpression *
gtk_cclosure_expression_new (GType value_type,
                             GClosureMarshal marshal,
                             guint n_params,
                             GtkExpression **params,
                             GCallback callback_func,
                             gpointer user_data,
                             GClosureNotify user_destroy);
~~~
@end

- `value_type` — это тип значения при его вычислении.
- `marshal` — это маршаллер.
Вы можете назначить NULL.
Если это NULL, то в качестве маршаллера используется `g_cclosure_marshal_generic ()`.
Это общая функция маршаллера, реализованная через libffi.
- `n_params` — это количество параметров.
- `params` указывает на выражения для каждого параметра функции обратного вызова.
- `callback_func` — это функция обратного вызова.
Ей передаются аргументы `this` и параметры выше.
Таким образом, если `n_params` равно 3, количество аргументов функции равно 4.
(`this` и `params`. См. ниже.)
- `user_data` — это пользовательские данные.
Вы можете добавить их для замыкания.
Это похоже на `user_data` в `g_signal_connect`.
Если они не нужны, назначьте NULL.
- `user_destroy` — это уведомление об уничтожении для `user_data`.
Оно вызывается для уничтожения `user_data`, когда они больше не нужны.
Если NULL назначено `user_data`, назначьте NULL также `user_destroy`.

Функции обратного вызова имеют следующий формат.

~~~
C-type
callback (this, param1, param2, ...)
~~~

For example,

@@@if gfm
~~~C
int
callback (GObject *object, int x, const char *s)
~~~
@@@else
~~~{.C}
int
callback (GObject *object, int x, const char *s)
~~~
@@@end

Следующее — `exp_closure_simple.c` в `src/expression`.

@@@include
expression/exp_closure_simple.c
@@@

- 3-11: Функция обратного вызова.
Параметр только один, и это объект 'this'.
Это GtkLabel, и предполагается, что его метка "(число)+(число)".
- 8-10: Извлекает два целых числа из метки и возвращает их сумму.
Эта функция не имеет отчёта об ошибках.
Если вы хотите вернуть отчёт об ошибке, измените тип возвращаемого значения на указатель на структуру из gboolean и integer.
Один для ошибки, другой для суммы.
Первый аргумент `gtk_cclosure_expression_new` — это `G_TYPE_POINTER`.
Есть пример программы `exp_closure_with_error_report` в каталоге `src/expression`.
- 19: Функция `gtk_init`` инициализирует GTK. Это необходимо для GtkLabel.
- 20: Создаётся экземпляр GtkLabel с "123+456".
- 21: Экземпляр имеет плавающую ссылку. Она изменяется на обычный счётчик ссылок.
- 22-23: Создаёт выражение замыкания. Его тип возвращаемого значения — `G_TYPE_INT`, и нет параметров или объекта 'this'.
- 24: Вычисляет выражение с меткой в качестве объекта 'this'.
- 25: Если вычисление успешно, показывается сумма "123+456", которая равна 579.
- 27: Если это не удаётся, появляется сообщение об ошибке.
- 28-30: Освобождает выражение и метку. Сбрасывает значение.

Выражение замыкания более гибкое, чем другие типы выражений, потому что вы можете указать свою собственную функцию обратного вызова.

## GtkExpressionWatch

GtkExpressionWatch — это структура, а не объект.
Она представляет наблюдаемое GtkExpression.
Две функции создают структуру GtkExpressionWatch.
Это `gtk_expression_bind` и `gtk_expression_watch`.

### Функция gtk\_expression\_bind

Эта функция привязывает свойство целевого объекта к выражению.
Если значение выражения изменяется, свойство немедленно отражает значение.

@@@if gfm
~~~C
GtkExpressionWatch*
gtk_expression_bind (
  GtkExpression* self,
  GObject* target,
  const char* property,
  GObject* this_
)
~~~
@@@else
~~~{.C}
GtkExpressionWatch*
gtk_expression_bind (
  GtkExpression* self,
  GObject* target,
  const char* property,
  GObject* this_
)
~~~
@@@end

Эта функция берёт владение выражением.
Поэтому, если вы хотите владеть выражением, вызовите `gtk_expression_ref ()` для увеличения счётчика ссылок выражения.
И вы должны освободить его, когда оно бесполезно.
Если вы не владеете выражением, вам не нужно беспокоиться об освобождении выражения.

Пример `exp_bind.c` и `exp_bind.ui` находится в каталоге [`src/expression`](expression).

![exp\_bind](../image/exp_bind.png){width=9.2cm height=1.9cm}

Он включает метку и ползунок.
Если вы перемещаете ползунок вправо, значение ползунка увеличивается, и число на метке также увеличивается.
Таким же образом, если вы перемещаете его влево, число на метке уменьшается.
Метка привязана к значению ползунка через adjustment.

@@@include
expression/exp_bind.ui
@@@

Ui-файл описывает следующее отношение родитель-дочерний.

~~~
GtkApplicationWindow (win) -- GtkBox -+- GtkLabel (label)
                                      +- GtkScale
~~~

Определены четыре свойства GtkScale.

- adjustment. GtkAdjustment предоставляет следующее.
  - upper и lower: диапазон ползунка.
  - value: текущее значение ползунка. Оно отражает значение ползунка.
  - step increment и page increment: Когда пользователь нажимает клавишу со стрелкой или клавишу page up/down,
ползунок перемещается на step increment или page increment соответственно.
  - page-size: Когда adjustment используется с ползунком, page-size равно нулю.
- digits: Количество десятичных знаков, отображаемых в значении.
- draw-value: Отображается ли значение.
- has-origin: Имеет ли ползунок начало. Если это true, оранжевая полоса появляется между началом и текущей точкой.
- round-digits: Количество цифр для округления значения при его изменении.
Например, если это ноль, ползунок перемещается к целочисленной точке.

@@@include
expression/exp_bind.c
@@@

Суть программы:

- 41-42: Определены два выражения.
Одно — это выражение свойства, а другое — выражение замыкания.
Выражение свойства ищет свойство "value" экземпляра adjustment.
Выражение замыкания просто преобразует double в integer.
- 43: `gtk_expression_bind` привязывает свойство label экземпляра GtkLabel к integer, возвращённому выражением замыкания.
Оно создаёт структуру GtkExpressionWatch.
Привязка работает в течение жизни наблюдателя.
Когда окно уничтожается, ползунок и adjustment также уничтожаются.
И наблюдатель распознаёт, что значение выражения изменяется, и пытается изменить свойство метки.
Очевидно, это неправильное поведение.
Наблюдатель должен прекратить наблюдение до того, как окно будет уничтожено.
- 37: Подключает сигнал "close-request" на окне к обработчику `close_request_cb`.
Этот сигнал испускается при нажатии кнопки закрытия.
Обработчик вызывается непосредственно перед закрытием окна.
Это правильный момент для прекращения наблюдения GtkExpressionWatch.
- 10-14: Обработчик сигнала "close-request".
Функция `gtk_expression_watch_unwatch (watch)` заставляет наблюдателя прекратить наблюдение за выражением.
Она также освобождает выражение.

Если вы хотите привязать свойство к выражению, `gtk_expression_bind` — лучший выбор.
Вы можете сделать это с помощью функции `gtk_expression_watch`, но это менее подходит.

### Функция gtk\_expression\_watch

@@@if gfm
~~~C
GtkExpressionWatch*
gtk_expression_watch (
  GtkExpression* self,
  GObject* this_,
  GtkExpressionNotify notify,
  gpointer user_data,
  GDestroyNotify user_destroy
)
~~~
@@@else
~~~{.C}
GtkExpressionWatch*
gtk_expression_watch (
  GtkExpression* self,
  GObject* this_,
  GtkExpressionNotify notify,
  gpointer user_data,
  GDestroyNotify user_destroy
)
~~~
@@@end

Функция не берёт владение выражением.
Это отличается от `gtk_expression_bind`.
Поэтому вам нужно освободить выражение, когда оно бесполезно.
Она создаёт структуру GtkExpressionWatch.
Третий параметр `notify` — это обратный вызов для вызова при изменении выражения.
Вы можете установить `user_data`, чтобы передать его обратному вызову.
Последний параметр — это функция для уничтожения `user_data`, когда наблюдатель прекращает наблюдение.
Поместите NULL, если они вам не нужны.

Обратный вызов notify имеет следующий формат.

@@@if gfm
~~~C
void
notify (
  gpointer user_data
)
~~~
@@@else
~~~{.C}
void
notify (
  gpointer user_data
)
~~~
@@@end

Эта функция используется для выполнения чего-либо при изменении значения выражения.
Но если вы хотите привязать свойство к значению, используйте вместо этого `gtk_expression_bind`.

Есть пример программы `exp_watch.c` в каталоге [`src/expression`](expression).
Она выводит ширину окна в стандартный вывод.

![exp\_watch](../image/exp_watch.png){width=9.6cm height=6.9cm}

Когда вы изменяете размер окна, ширина отображается в терминале.

@@@include
expression/exp_watch.c
@@@

- 37: Выражение свойства ищет свойство "default-width" окна.
- 38: Создать структуру наблюдателя для выражения.
Обратный вызов `notify` вызывается каждый раз, когда значение выражения изменяется.
Объект 'this' — это `win`, поэтому выражение возвращает ширину окна по умолчанию.
- 6-14: Функция обратного вызова `notify`.
Она использует `gtk_expression_watch_evaluate` для получения значения выражения.
Объект 'this' задан заранее (при создании наблюдателя).
Она выводит ширину окна в стандартный вывод.
- 16-21: Обработчик сигнала "close-request" на окне.
Он останавливает наблюдатель.
Кроме того, он освобождает ссылку на выражение.
Поскольку `gtk_expression_watch` не берёт владение выражением, вы владеете им.
Поэтому освобождение необходимо.

## GtkExpression в ui-файлах

GtkBuilder поддерживает GtkExpressions.
Есть четыре тега.

- constant тег для создания константного выражения. Атрибут type указывает имя типа значения.
Если тип не указан, предполагается, что тип — объект.
Содержимое — это значение выражения.
- lookup тег для создания выражения свойства. Атрибут type указывает тип объекта.
Атрибут name указывает имя свойства.
Содержимое — это выражение или объект, который имеет свойство для поиска.
Если содержимого нет, используется объект 'this'.
- closure тег для создания выражения замыкания. Атрибут type указывает тип возвращаемого значения.
Атрибут function указывает функцию обратного вызова.
Содержимое тега — это аргументы, которые являются выражениями.
- binding тег для привязки свойства к выражению.
Он помещается в содержимое тега объекта.
Атрибут name указывает имя свойства объекта.
Содержимое — это выражение.

@@@if gfm
~~~xml
<constant type="gchararray">Hello world</constant>
<lookup name="label" type="GtkLabel">label</lookup>
<closure type="gint" function="callback_function"></closure>
<bind name="label">
  <lookup name="default-width">win</lookup>
</bind>
~~~
@@@else
~~~{.xml}
<constant type="gchararray">Hello world</constant>
<lookup name="label" type="GtkLabel">label</lookup>
<closure type="gint" function="callback_function"></closure>
<bind name="label">
  <lookup name="default-width">win</lookup>
</bind>
~~~
@@@end

Эти теги обычно используются для GtkBuilderListItemFactory.

@@@if gfm
~~~xml
<interface>
  <template class="GtkListItem">
    <property name="child">
      <object class="GtkLabel">
        <binding name="label">
          <lookup name="string" type="GtkStringObject">
            <lookup name="item">GtkListItem</lookup>
          </lookup>
        </binding>
      </object>
    </property>
  </template>
</interface>
~~~
@@@else
~~~{.xml}
<interface>
  <template class="GtkListItem">
    <property name="child">
      <object class="GtkLabel">
        <binding name="label">
          <lookup name="string" type="GtkStringObject">
            <lookup name="item">GtkListItem</lookup>
          </lookup>
        </binding>
      </object>
    </property>
  </template>
</interface>
~~~
@@@end

GtkBuilderListItemFactory использует GtkBuilder для расширения GtkListItem с помощью XML-данных.

В xml-файле выше "GtkListItem" — это экземпляр шаблона GtkListItem.
Это объект 'this', переданный выражениям.
(Информация находится в [GTK Development Blog](https://blog.gtk.org/2020/09/)).

GtkBuilder вызывает функцию `gtk_expression_bind` при обнаружении тега binding.
Функция устанавливает объект 'this' следующим образом:

1. Если тег binding имеет атрибут object, объект будет объектом 'this'.
2. Если текущий объект GtkBuilder существует, он будет объектом 'this'.
Вот почему экземпляр GtkListItem является объектом 'this' XML-данных для GtkBuilderListItemFactory.
3. В противном случае целевой объект тега binding будет объектом 'this'.

Документация GTK 4 не описывает информацию об объекте "this", когда выражения определены в ui-файле.
Информация выше найдена из исходных файлов GTK 4, и возможно, она содержит ошибки.
Если у вас есть точная информация, пожалуйста, сообщите мне.

Пример программы `exp.c` и ui-файла `exp.ui` находится в каталоге [`src/expression`](expression).
Ui-файл включает теги lookup, closure и bind.
Тег constant не включён.
Однако теги constant не используются так часто.

![exp.c](../image/exp.png){width=10.3cm height=7.6cm}

Если вы изменяете размер окна, размер показывается в заголовке окна.
Если вы вводите символы в поле ввода, те же символы появляются на метке.

Ui-файл следующий.

@@@include
expression/exp.ui
@@@

- 4-9: Свойство title главного окна привязано к выражению замыкания.
Его функция обратного вызова `set_title` определена в исходном файле C.
Она возвращает строку, потому что атрибут type тега — "gchararray".
Функции передаются два параметра.
Это ширина и высота окна.
Теги lookup не имеют содержимого, поэтому объект 'this' используется для поиска свойств.
Объект 'this' — это `win`, который является целью привязки (`win` включает тег binding).
- 17-21: Свойство "label" экземпляра GtkLabel привязано к свойству "text" `buffer`,
который является буфером GtkEntry, определённым в строке 25.
Если пользователь вводит символы в поле ввода, те же символы появляются на метке.

Исходный файл C следующий.

@@@include
expression/exp.c
@@@

- 4-6: Функция обратного вызова.
Она возвращает строку (w)x(h), где w и h — ширина и высота окна.
Необходимо дублирование строки.

Исходный файл C очень прост, потому что почти всё сделано в ui-файле.

## Преобразование между GValues

Если вы привязываете свойства разных типов, преобразование типов выполняется автоматически.
Предположим, свойство label (строка) привязано к свойству default-width (int).

@@@if gfm
~~~xml
<object class="GtkLabel">
  <binding name="label">
    <lookup name="default-width">
      win
    </lookup>
  </binding>
</object>
~~~
@@@else
~~~{.xml}
<object class="GtkLabel">
  <binding name="label">
    <lookup name="default-width">
      win
    </lookup>
  </binding>
</object>
~~~
@@@end

Выражение, созданное тегом lookup, возвращает GValue типа int.
С другой стороны, свойство "label" содержит GValue типа строки.
Когда GValue копируется в другой GValue, тип автоматически преобразуется, если это возможно.
Если текущая ширина 100, int `100` преобразуется в строку `"100"`.

Если вы используете `g_object_get` и `g_object_set` для копирования свойств, значение автоматически преобразуется. 

## Meson.build

Исходные файлы находятся в каталоге `src/expression`.
Вы можете собрать все файлы сразу.

~~~
$ cd src/expression
$ meson setup _build
$ ninja -C _build
~~~

Например, если вы хотите запустить "exp", который является исполняемым файлом из "exp.c", введите `_build/exp`.
Вы также можете запустить другие программы.

Файл `meson.build` следующий.

@@@include
expression/meson.build
@@@
