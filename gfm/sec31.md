Up: [README.md](../README.md),  Prev: [Section 30](sec30.md), Next: [Section 32](sec32.md)

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

~~~C
 1 #include <gtk/gtk.h>
 2 
 3 int
 4 main (int argc, char **argv) {
 5   GtkExpression *expression;
 6   GValue value = G_VALUE_INIT;
 7 
 8   /* Create an expression */
 9   expression = gtk_constant_expression_new (G_TYPE_INT,100);
10   /* Evaluate the expression */
11   if (gtk_expression_evaluate (expression, NULL, &value))
12     g_print ("The value is %d.\n", g_value_get_int (&value));
13   else
14     g_print ("The constant expression wasn't evaluated correctly.\n");
15   gtk_expression_unref (expression);
16   g_value_unset (&value);
17 
18   return 0;
19 }
~~~

- 9: Создаётся константное выражение. Оно содержит значение int 100. Переменная `expression` указывает на выражение.
- 11-14: Вычисляет выражение. Если это успешно, показывает значение в stdout. В противном случае показывает сообщение об ошибке.
- 15-16: Освобождает выражение и сбрасывает GValue.

Константное выражение обычно используется для передачи константного значения или экземпляра другому выражению.

## Выражение свойства

Выражение свойства (GtkPropertyExpression) ищет свойство в экземпляре GObject.
Например, выражение свойства, которое ссылается на свойство "label" в объекте GtkLabel, создаётся так.

~~~C
expression = gtk_property_expression_new (GTK_TYPE_LABEL, another_expression, "label");
~~~

Второй параметр `another_expression` — это одно из:

- Выражение, которое даёт экземпляр GtkLabel при его вычислении.
- NULL. Когда передаётся NULL, экземпляр GtkLabel будет передан при вычислении.
Экземпляр называется объектом `this`.

Например,

~~~C
label = gtk_label_new ("Hello");
another_expression = gtk_constant_expression_new (GTK_TYPE_LABEL, label);
expression = gtk_property_expression_new (GTK_TYPE_LABEL, another_expression, "label");
~~~

Если `expression` вычисляется, второй параметр `another_expression` вычисляется заранее.
Значение `another_expression` — это `label` (экземпляр GtkLabel).
Затем `expression` ищет свойство "label" метки, и результат вычисления — "Hello".

В примере выше второй аргумент `gtk_property_expression_new` — это другое выражение.
Но второй аргумент может быть NULL.
Если это NULL, вместо него используется экземпляр `this`.
`this` передаётся функцией `gtk_expression_evaluate`.

Есть простая программа `exp_property_simple.c` в каталоге `src/expression`.

~~~C
 1 #include <gtk/gtk.h>
 2 
 3 int
 4 main (int argc, char **argv) {
 5   GtkWidget *label;
 6   GtkExpression *expression;
 7   GValue value = G_VALUE_INIT;
 8 
 9   gtk_init ();
10   label = gtk_label_new ("Hello world.");
11   /* Create an expression */
12   expression = gtk_property_expression_new (GTK_TYPE_LABEL, NULL, "label");
13   /* Evaluate the expression */
14   if (gtk_expression_evaluate (expression, label, &value))
15     g_print ("The value is %s.\n", g_value_get_string (&value));
16   else
17     g_print ("The property expression wasn't evaluated correctly.\n");
18   gtk_expression_unref (expression);
19   g_value_unset (&value);
20 
21   return 0;
22 }
~~~

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

Следующее — `exp_closure_simple.c` в `src/expression`.

~~~C
 1 #include <gtk/gtk.h>
 2 
 3 static int
 4 calc (GtkLabel *label) { /* this object */
 5   const char * s;
 6   int i, j;
 7 
 8   s = gtk_label_get_text (label); /* s is owned by the label. */
 9   sscanf (s, "%d+%d", &i, &j);
10   return i+j;
11 }
12 
13 int
14 main (int argc, char **argv) {
15   GtkExpression *expression;
16   GValue value = G_VALUE_INIT;
17   GtkLabel *label;
18 
19   gtk_init ();
20   label = GTK_LABEL (gtk_label_new ("123+456"));
21   g_object_ref_sink (label);
22   expression = gtk_cclosure_expression_new (G_TYPE_INT, NULL, 0, NULL,
23                  G_CALLBACK (calc), NULL, NULL);
24   if (gtk_expression_evaluate (expression, label, &value)) /* 'this' object is the label. */
25     g_print ("%d\n", g_value_get_int (&value));
26   else
27     g_print ("The closure expression wasn't evaluated correctly.\n");
28   gtk_expression_unref (expression);
29   g_value_unset (&value);
30   g_object_unref (label);
31   
32   return 0;
33 }
~~~

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

~~~C
GtkExpressionWatch*
gtk_expression_bind (
  GtkExpression* self,
  GObject* target,
  const char* property,
  GObject* this_
)
~~~

Эта функция берёт владение выражением.
Поэтому, если вы хотите владеть выражением, вызовите `gtk_expression_ref ()` для увеличения счётчика ссылок выражения.
И вы должны освободить его, когда оно бесполезно.
Если вы не владеете выражением, вам не нужно беспокоиться об освобождении выражения.

Пример `exp_bind.c` и `exp_bind.ui` находится в каталоге [`src/expression`](../src/expression).

![exp\_bind](../image/exp_bind.png)

Он включает метку и ползунок.
Если вы перемещаете ползунок вправо, значение ползунка увеличивается, и число на метке также увеличивается.
Таким же образом, если вы перемещаете его влево, число на метке уменьшается.
Метка привязана к значению ползунка через adjustment.

~~~xml
 1 <?xml version='1.0' encoding='UTF-8'?>
 2 <interface>
 3   <object class='GtkApplicationWindow' id='win'>
 4     <property name='default-width'>600</property>
 5     <child>
 6       <object class='GtkBox'>
 7         <property name='orientation'>GTK_ORIENTATION_VERTICAL</property>
 8         <child>
 9           <object class='GtkLabel' id='label'>
10             <property name="label">10</property>
11           </object>
12         </child>
13         <child>
14           <object class='GtkScale'>
15             <property name='adjustment'>
16               <object class='GtkAdjustment' id='adjustment'>
17                 <property name='upper'>20.0</property>
18                 <property name='lower'>0.0</property>
19                 <property name='value'>10.0</property>
20                 <property name='step-increment'>1.0</property>
21                 <property name='page-increment'>5.0</property>
22                 <property name='page-size'>0.0</property>
23               </object>
24             </property>
25             <property name='digits'>0</property>
26             <property name='draw-value'>true</property>
27             <property name='has-origin'>true</property>
28             <property name='round-digits'>0</property>
29           </object>
30         </child>
31       </object>
32     </child>
33   </object>
34 </interface>
~~~

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

~~~C
 1 #include <gtk/gtk.h>
 2 
 3 GtkExpressionWatch *watch;
 4 
 5 static int
 6 f2i (GObject *object, double d) {
 7   return (int) d;
 8 }
 9 
10 static int
11 close_request_cb (GtkWindow *win) {
12   gtk_expression_watch_unwatch (watch);
13   return false;
14 }
15 
16 static void
17 app_activate (GApplication *application) {
18   GtkApplication *app = GTK_APPLICATION (application);
19   gtk_window_present (gtk_application_get_active_window(app));
20 }
21 
22 static void
23 app_startup (GApplication *application) {
24   GtkApplication *app = GTK_APPLICATION (application);
25   GtkBuilder *build;
26   GtkWidget *win, *label;
27   GtkAdjustment *adjustment;
28   GtkExpression *expression, *params[1];
29 
30   /* Builds a window with exp.ui resource */
31   build = gtk_builder_new_from_resource ("/com/github/ToshioCP/exp/exp_bind.ui");
32   win = GTK_WIDGET (gtk_builder_get_object (build, "win"));
33   label = GTK_WIDGET (gtk_builder_get_object (build, "label"));
34   // scale = GTK_WIDGET (gtk_builder_get_object (build, "scale"));
35   adjustment = GTK_ADJUSTMENT (gtk_builder_get_object (build, "adjustment"));
36   gtk_window_set_application (GTK_WINDOW (win), app);
37   g_signal_connect (win, "close-request", G_CALLBACK (close_request_cb), NULL);
38   g_object_unref (build);
39 
40   /* GtkExpressionWatch */
41   params[0] = gtk_property_expression_new (GTK_TYPE_ADJUSTMENT, NULL, "value");
42   expression = gtk_cclosure_expression_new (G_TYPE_INT, NULL, 1, params, G_CALLBACK (f2i), NULL, NULL);
43   watch = gtk_expression_bind (expression, label, "label", adjustment); /* watch takes the ownership of the expression. */
44 }
45 
46 #define APPLICATION_ID "com.github.ToshioCP.exp_watch"
47 
48 int
49 main (int argc, char **argv) {
50   GtkApplication *app;
51   int stat;
52 
53   app = gtk_application_new (APPLICATION_ID, G_APPLICATION_DEFAULT_FLAGS);
54 
55   g_signal_connect (app, "startup", G_CALLBACK (app_startup), NULL);
56   g_signal_connect (app, "activate", G_CALLBACK (app_activate), NULL);
57 
58   stat = g_application_run (G_APPLICATION (app), argc, argv);
59   g_object_unref (app);
60   return stat;
61 }
~~~

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

Функция не берёт владение выражением.
Это отличается от `gtk_expression_bind`.
Поэтому вам нужно освободить выражение, когда оно бесполезно.
Она создаёт структуру GtkExpressionWatch.
Третий параметр `notify` — это обратный вызов для вызова при изменении выражения.
Вы можете установить `user_data`, чтобы передать его обратному вызову.
Последний параметр — это функция для уничтожения `user_data`, когда наблюдатель прекращает наблюдение.
Поместите NULL, если они вам не нужны.

Обратный вызов notify имеет следующий формат.

~~~C
void
notify (
  gpointer user_data
)
~~~

Эта функция используется для выполнения чего-либо при изменении значения выражения.
Но если вы хотите привязать свойство к значению, используйте вместо этого `gtk_expression_bind`.

Есть пример программы `exp_watch.c` в каталоге [`src/expression`](../src/expression).
Она выводит ширину окна в стандартный вывод.

![exp\_watch](../image/exp_watch.png)

Когда вы изменяете размер окна, ширина отображается в терминале.

~~~C
 1 #include <gtk/gtk.h>
 2 
 3 GtkExpression *expression;
 4 GtkExpressionWatch *watch;
 5 
 6 static void
 7 notify (gpointer user_data) {
 8   GValue value = G_VALUE_INIT;
 9 
10   if (gtk_expression_watch_evaluate (watch, &value))
11     g_print ("width = %d\n", g_value_get_int (&value));
12   else
13     g_print ("evaluation failed.\n");
14 }
15 
16 static int
17 close_request_cb (GtkWindow *win) {
18   gtk_expression_watch_unwatch (watch);
19   gtk_expression_unref (expression);
20   return false;
21 }
22 
23 static void
24 app_activate (GApplication *application) {
25   GtkApplication *app = GTK_APPLICATION (application);
26   gtk_window_present (gtk_application_get_active_window(app));
27 }
28 
29 static void
30 app_startup (GApplication *application) {
31   GtkApplication *app = GTK_APPLICATION (application);
32   GtkWidget *win;
33 
34   win = GTK_WIDGET (gtk_application_window_new (app));
35   g_signal_connect (win, "close-request", G_CALLBACK (close_request_cb), NULL);
36 
37   expression = gtk_property_expression_new (GTK_TYPE_WINDOW, NULL, "default-width");
38   watch = gtk_expression_watch (expression, win, notify, NULL, NULL);
39 }
40 
41 #define APPLICATION_ID "com.github.ToshioCP.exp_watch"
42 
43 int
44 main (int argc, char **argv) {
45   GtkApplication *app;
46   int stat;
47 
48   app = gtk_application_new (APPLICATION_ID, G_APPLICATION_DEFAULT_FLAGS);
49 
50   g_signal_connect (app, "startup", G_CALLBACK (app_startup), NULL);
51   g_signal_connect (app, "activate", G_CALLBACK (app_activate), NULL);
52 
53   stat = g_application_run (G_APPLICATION (app), argc, argv);
54   g_object_unref (app);
55   return stat;
56 }
~~~

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

~~~xml
<constant type="gchararray">Hello world</constant>
<lookup name="label" type="GtkLabel">label</lookup>
<closure type="gint" function="callback_function"></closure>
<bind name="label">
  <lookup name="default-width">win</lookup>
</bind>
~~~

Эти теги обычно используются для GtkBuilderListItemFactory.

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

Пример программы `exp.c` и ui-файла `exp.ui` находится в каталоге [`src/expression`](../src/expression).
Ui-файл включает теги lookup, closure и bind.
Тег constant не включён.
Однако теги constant не используются так часто.

![exp.c](../image/exp.png)

Если вы изменяете размер окна, размер показывается в заголовке окна.
Если вы вводите символы в поле ввода, те же символы появляются на метке.

Ui-файл следующий.

~~~xml
 1 <?xml version="1.0" encoding="UTF-8"?>
 2 <interface>
 3   <object class="GtkApplicationWindow" id="win">
 4     <binding name="title">
 5       <closure type="gchararray" function="set_title">
 6         <lookup name="default-width" type="GtkWindow"></lookup>
 7         <lookup name="default-height" type="GtkWindow"></lookup>
 8       </closure>
 9     </binding>
10     <property name="default-width">600</property>
11     <property name="default-height">400</property>
12     <child>
13       <object class="GtkBox">
14         <property name="orientation">GTK_ORIENTATION_VERTICAL</property>
15         <child>
16           <object class="GtkLabel">
17             <binding name="label">
18               <lookup name="text">
19                 buffer
20               </lookup>
21             </binding>
22           </object>
23         </child>
24         <child>
25           <object class="GtkEntry">
26             <property name="buffer">
27               <object class="GtkEntryBuffer" id="buffer"></object>
28             </property>
29           </object>
30         </child>
31       </object>
32     </child>
33   </object>
34 </interface>
~~~

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

~~~C
 1 #include <gtk/gtk.h>
 2 
 3 char *
 4 set_title (GtkWidget *win, int width, int height) {
 5   return g_strdup_printf ("%d x %d", width, height);
 6 }
 7 
 8 static void
 9 app_activate (GApplication *application) {
10   GtkApplication *app = GTK_APPLICATION (application);
11   gtk_window_present (gtk_application_get_active_window(app));
12 }
13 
14 static void
15 app_startup (GApplication *application) {
16   GtkApplication *app = GTK_APPLICATION (application);
17   GtkBuilder *build;
18   GtkWidget *win;
19 
20   build = gtk_builder_new_from_resource ("/com/github/ToshioCP/exp/exp.ui");
21   win = GTK_WIDGET (gtk_builder_get_object (build, "win"));
22   gtk_window_set_application (GTK_WINDOW (win), app);
23   g_object_unref (build);
24 }
25 
26 #define APPLICATION_ID "com.github.ToshioCP.exp"
27 
28 int
29 main (int argc, char **argv) {
30   GtkApplication *app;
31   int stat;
32 
33   app = gtk_application_new (APPLICATION_ID, G_APPLICATION_DEFAULT_FLAGS);
34 
35   g_signal_connect (app, "startup", G_CALLBACK (app_startup), NULL);
36   g_signal_connect (app, "activate", G_CALLBACK (app_activate), NULL);
37 
38   stat = g_application_run (G_APPLICATION (app), argc, argv);
39   g_object_unref (app);
40   return stat;
41 }
~~~

- 4-6: Функция обратного вызова.
Она возвращает строку (w)x(h), где w и h — ширина и высота окна.
Необходимо дублирование строки.

Исходный файл C очень прост, потому что почти всё сделано в ui-файле.

## Преобразование между GValues

Если вы привязываете свойства разных типов, преобразование типов выполняется автоматически.
Предположим, свойство label (строка) привязано к свойству default-width (int).

~~~xml
<object class="GtkLabel">
  <binding name="label">
    <lookup name="default-width">
      win
    </lookup>
  </binding>
</object>
~~~

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

~~~meson
 1 project('exp', 'c')
 2 
 3 gtkdep = dependency('gtk4')
 4 
 5 gnome=import('gnome')
 6 resources = gnome.compile_resources('resources','exp.gresource.xml')
 7 
 8 sourcefiles=files('exp.c')
 9 
10 executable('exp', sourcefiles, resources, dependencies: gtkdep, export_dynamic: true, install: false)
11 executable('exp_constant', 'exp_constant.c', dependencies: gtkdep, export_dynamic: true, install: false)
12 executable('exp_constant_simple', 'exp_constant_simple.c', dependencies: gtkdep, export_dynamic: true, install: false)
13 executable('exp_property_simple', 'exp_property_simple.c', dependencies: gtkdep, export_dynamic: true, install: false)
14 executable('closure', 'closure.c', dependencies: gtkdep, export_dynamic: true, install: false)
15 executable('closure_each', 'closure_each.c', dependencies: gtkdep, export_dynamic: true, install: false)
16 executable('exp_closure_simple', 'exp_closure_simple.c', dependencies: gtkdep, export_dynamic: true, install: false)
17 executable('exp_closure_with_error_report', 'exp_closure_with_error_report.c', dependencies: gtkdep, export_dynamic: true, install: false)
18 executable('exp_bind', 'exp_bind.c', resources, dependencies: gtkdep, export_dynamic: true, install: false)
19 executable('exp_watch', 'exp_watch.c', dependencies: gtkdep, export_dynamic: true, install: false)
20 executable('exp_test', 'exp_test.c', resources, dependencies: gtkdep, export_dynamic: true, install: false)
~~~

Up: [README.md](../README.md),  Prev: [Section 30](sec30.md), Next: [Section 32](sec32.md)
