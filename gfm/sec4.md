Up: [README.md](../README.md),  Prev: [Section 3](sec3.md), Next: [Section 5](sec5.md)

# Виджеты (1)

## GtkLabel, GtkButton и GtkBox

### GtkLabel

Мы создали окно и показали его на экране в предыдущем разделе.
Теперь перейдем к следующей теме: виджеты.
Самый простой виджет — это GtkLabel.
Это виджет с текстом внутри.

~~~C
 1 #include <gtk/gtk.h>
 2 
 3 static void
 4 app_activate (GApplication *app) {
 5   GtkWidget *win;
 6   GtkWidget *lab;
 7 
 8   win = gtk_application_window_new (GTK_APPLICATION (app));
 9   gtk_window_set_title (GTK_WINDOW (win), "lb1");
10   gtk_window_set_default_size (GTK_WINDOW (win), 400, 300);
11 
12   lab = gtk_label_new ("Hello.");
13   gtk_window_set_child (GTK_WINDOW (win), lab);
14 
15   gtk_window_present (GTK_WINDOW (win));
16 }
17 
18 int
19 main (int argc, char **argv) {
20   GtkApplication *app;
21   int stat;
22 
23   app = gtk_application_new ("com.github.ToshioCP.lb1", G_APPLICATION_DEFAULT_FLAGS);
24   g_signal_connect (app, "activate", G_CALLBACK (app_activate), NULL);
25   stat = g_application_run (G_APPLICATION (app), argc, argv);
26   g_object_unref (app);
27   return stat;
28 }
29 
~~~

Сохраните эту программу в файл `lb1.c`.
(Вы можете использовать `src/misc/lb1.c`, если загрузили этот репозиторий.)
Затем скомпилируйте и запустите её.

    $ comp lb1
    $ ./a.out

Появится окно с сообщением "Hello.".

![Скриншот метки](../image/screenshot_lb1.png)

Между `pr4.c` и `lb1.c` есть лишь несколько изменений.
Программа `diff` полезна для определения различий.

~~~
$ cd misc; diff pr4.c lb1.c
4c4
< app_activate (GApplication *app, gpointer user_data) {
---
> app_activate (GApplication *app) {
5a6
>   GtkWidget *lab;
8c9
<   gtk_window_set_title (GTK_WINDOW (win), "pr4");
---
>   gtk_window_set_title (GTK_WINDOW (win), "lb1");
9a11,14
> 
>   lab = gtk_label_new ("Hello.");
>   gtk_window_set_child (GTK_WINDOW (win), lab);
> 
18c23
<   app = gtk_application_new ("com.github.ToshioCP.pr4", G_APPLICATION_DEFAULT_FLAGS);
---
>   app = gtk_application_new ("com.github.ToshioCP.lb1", G_APPLICATION_DEFAULT_FLAGS);
~~~

Это говорит нам:

- Обработчик сигнала `app_activate` не имеет параметра `user_data`.
Если четвертый аргумент `g_signal_connect` равен NULL, вы можете опустить `user_data`.
- Добавлено определение новой переменной `lab`.
- Заголовок окна изменен.
- Создана метка и подключена к окну как дочерний элемент.

Функция `gtk_window_set_child (GTK_WINDOW (win), lab)` делает метку `lab` дочерним виджетом окна `win`.
Будьте осторожны.
Дочерний виджет отличается от дочернего объекта.
Объекты имеют отношения родитель-потомок, и виджеты также имеют отношения родитель-потомок.
Но эти два отношения совершенно разные.
Не путайте их.
В программе `lb1.c`, `lab` является дочерним виджетом `win`.
Дочерние виджеты всегда располагаются внутри своего родительского виджета на экране.
Посмотрите, как появилось окно на экране.
Окно приложения включает метку.

Окно `win` не имеет родителей.
Мы называем такое окно окном верхнего уровня.
Приложение может иметь более одного окна верхнего уровня.

### GtkButton

Следующий виджет — GtkButton.
Он отображает кнопку на экране с меткой или значком на ней.
В этом подразделе мы создадим кнопку с меткой.
Когда кнопка нажимается, она испускает сигнал "clicked".
Следующая программа показывает, как перехватить сигнал и что-то сделать.

~~~C
 1 #include <gtk/gtk.h>
 2 
 3 static void
 4 click_cb (GtkButton *btn) {
 5   g_print ("Clicked.\n");
 6 }
 7 
 8 static void
 9 app_activate (GApplication *app) {
10   GtkWidget *win;
11   GtkWidget *btn;
12 
13   win = gtk_application_window_new (GTK_APPLICATION (app));
14   gtk_window_set_title (GTK_WINDOW (win), "lb2");
15   gtk_window_set_default_size (GTK_WINDOW (win), 400, 300);
16 
17   btn = gtk_button_new_with_label ("Click me");
18   gtk_window_set_child (GTK_WINDOW (win), btn);
19   g_signal_connect (btn, "clicked", G_CALLBACK (click_cb), NULL);
20 
21   gtk_window_present (GTK_WINDOW (win));
22 }
23 
24 int
25 main (int argc, char **argv) {
26   GtkApplication *app;
27   int stat;
28 
29   app = gtk_application_new ("com.github.ToshioCP.lb2", G_APPLICATION_DEFAULT_FLAGS);
30   g_signal_connect (app, "activate", G_CALLBACK (app_activate), NULL);
31   stat = g_application_run (G_APPLICATION (app), argc, argv);
32   g_object_unref (app);
33   return stat;
34 }
35 
~~~

Посмотрите на строки с 17 по 19.
Сначала создается экземпляр GtkButton `btn` с меткой "Click me".
Затем кнопка добавляется в окно `win` как дочерний элемент.
Наконец, сигнал "clicked" кнопки подключается к обработчику `click_cb`.
Таким образом, если `btn` нажата, вызывается функция `click_cb`.
Суффикс "cb" означает "call back" (обратный вызов).

Назовите программу `lb2.c` и сохраните её.
Теперь скомпилируйте и запустите её.

![Скриншот метки](../image/screenshot_lb2.png)

Появится окно с кнопкой.
Нажмите на кнопку (это большая кнопка, вы можете нажать в любом месте окна), и строка "Clicked." появится в терминале.
Это показывает, что обработчик был вызван нажатием кнопки.

Хорошо, что мы убедились, что сигнал clicked был перехвачен и обработчик был вызван с помощью `g_print`.
Однако использование `g_print` не гармонирует с GTK, которая является библиотекой GUI.
Поэтому мы изменим обработчик.
Следующий код извлечен из `lb3.c`.

~~~C
 1 static void
 2 click_cb (GtkButton *btn, GtkWindow *win) {
 3   gtk_window_destroy (win);
 4 }
 5 
 6 static void
 7 app_activate (GApplication *app) {
 8   GtkWidget *win;
 9   GtkWidget *btn;
10 
11   win = gtk_application_window_new (GTK_APPLICATION (app));
12   gtk_window_set_title (GTK_WINDOW (win), "lb3");
13   gtk_window_set_default_size (GTK_WINDOW (win), 400, 300);
14 
15   btn = gtk_button_new_with_label ("Close");
16   gtk_window_set_child (GTK_WINDOW (win), btn);
17   g_signal_connect (btn, "clicked", G_CALLBACK (click_cb), win);
18 
19   gtk_window_present (GTK_WINDOW (win));
20 }
~~~

А разница между `lb2.c` и `lb3.c` следующая.

~~~
$ cd misc; diff lb2.c lb3.c
4,5c4,5
< click_cb (GtkButton *btn) {
<   g_print ("Clicked.\n");
---
> click_cb (GtkButton *btn, GtkWindow *win) {
>   gtk_window_destroy (win);
14c14
<   gtk_window_set_title (GTK_WINDOW (win), "lb2");
---
>   gtk_window_set_title (GTK_WINDOW (win), "lb3");
17c17
<   btn = gtk_button_new_with_label ("Click me");
---
>   btn = gtk_button_new_with_label ("Close");
19c19
<   g_signal_connect (btn, "clicked", G_CALLBACK (click_cb), NULL);
---
>   g_signal_connect (btn, "clicked", G_CALLBACK (click_cb), win);
29c29
<   app = gtk_application_new ("com.github.ToshioCP.lb2", G_APPLICATION_DEFAULT_FLAGS);
---
>   app = gtk_application_new ("com.github.ToshioCP.lb3", G_APPLICATION_DEFAULT_FLAGS);
35d34
< 
~~~

Изменения:

- Функция `g_print` в `lb2.c` была удалена и вставлены две строки.
  - `click_cb` имеет второй параметр, который берется из четвертого аргумента `g_signal_connect` в строке 17.
Важно отметить, что типы различны между вторым параметром `click_cb` и четвертым аргументом `g_signal_connect`.
Первый — `GtkWindow *`, а второй — `GtkWidget *`.
Компилятор не жалуется, потому что `g_signal_connect` использует gpointer (общий тип указателя).
В этой программе экземпляр, на который указывает `win`, является объектом GtkApplicationWindow.
Он является потомком класса GtkWindow и GtkWidget, поэтому оба типа `GtkWindow *` и `GtkWidget *` являются правильными типами экземпляра.
  - `gtk_destroy (win)` уничтожает окно верхнего уровня. Затем приложение завершается.
- Метка `btn` изменена с "Click me" на "Close".
- Четвертый аргумент `g_signal_connect` изменен с `NULL` на `win`.

Наиболее важное изменение — это четвертый аргумент `g_signal_connect`.
Этот аргумент описан как "данные для передачи обработчику" в определении [`g_signal_connect`](https://docs.gtk.org/gobject/func.signal_connect.html).

### GtkBox

GtkWindow и GtkApplicationWindow могут иметь только одного потомка.
Если вы хотите добавить два или более виджетов в окно, вам нужен контейнерный виджет.
GtkBox — один из контейнеров.
Он размещает два или более дочерних виджетов в одной строке или столбце.
Следующая процедура показывает способ добавления двух кнопок в окно.

- Создать экземпляр GtkApplicationWindow.
- Создать экземпляр GtkBox и добавить его в GtkApplicationWindow как дочерний элемент.
- Создать экземпляр GtkButton и добавить его в GtkBox.
- Создать еще один экземпляр GtkButton и добавить его в GtkBox.

После этого виджеты соединены, как показано на следующей диаграмме.

![Отношения родитель-потомок](../image/box.png)

Программа `lb4.c` выглядит следующим образом.

~~~C
 1 #include <gtk/gtk.h>
 2 
 3 static void
 4 click1_cb (GtkButton *btn) {
 5   const char *s;
 6 
 7   s = gtk_button_get_label (btn);
 8   if (g_strcmp0 (s, "Hello.") == 0)
 9     gtk_button_set_label (btn, "Good-bye.");
10   else
11     gtk_button_set_label (btn, "Hello.");
12 }
13 
14 static void
15 click2_cb (GtkButton *btn, GtkWindow *win) {
16   gtk_window_destroy (win);
17 }
18 
19 static void
20 app_activate (GApplication *app) {
21   GtkWidget *win;
22   GtkWidget *box;
23   GtkWidget *btn1;
24   GtkWidget *btn2;
25 
26   win = gtk_application_window_new (GTK_APPLICATION (app));
27   gtk_window_set_title (GTK_WINDOW (win), "lb4");
28   gtk_window_set_default_size (GTK_WINDOW (win), 400, 300);
29 
30   box = gtk_box_new (GTK_ORIENTATION_VERTICAL, 5);
31   gtk_box_set_homogeneous (GTK_BOX (box), TRUE);
32   gtk_window_set_child (GTK_WINDOW (win), box);
33 
34   btn1 = gtk_button_new_with_label ("Hello.");
35   g_signal_connect (btn1, "clicked", G_CALLBACK (click1_cb), NULL);
36 
37   btn2 = gtk_button_new_with_label ("Close");
38   g_signal_connect (btn2, "clicked", G_CALLBACK (click2_cb), win);
39 
40   gtk_box_append (GTK_BOX (box), btn1);
41   gtk_box_append (GTK_BOX (box), btn2);
42 
43   gtk_window_present (GTK_WINDOW (win));
44 }
45 
46 int
47 main (int argc, char **argv) {
48   GtkApplication *app;
49   int stat;
50 
51   app = gtk_application_new ("com.github.ToshioCP.lb4", G_APPLICATION_DEFAULT_FLAGS);
52   g_signal_connect (app, "activate", G_CALLBACK (app_activate), NULL);
53   stat = g_application_run (G_APPLICATION (app), argc, argv);
54   g_object_unref (app);
55   return stat;
56 }
~~~

Посмотрите на функцию `app_activate`.

После создания экземпляра GtkApplicationWindow создается экземпляр GtkBox.

~~~C
box = gtk_box_new(GTK_ORIENTATION_VERTICAL, 5);
gtk_box_set_homogeneous (GTK_BOX (box), TRUE);
~~~

Первый аргумент размещает дочерние элементы блока вертикально.
Константы ориентации определены так:

- GTK\_ORIENTATION\_VERTICAL: дочерние виджеты размещаются вертикально
- GTK\_ORIENTATION\_HORIZONTAL: дочерние виджеты размещаются горизонтально

Второй аргумент — размер пространства между дочерними элементами.
Единица длины — пиксель.

Следующая функция заполняет блок дочерними элементами, давая им одинаковое пространство.

После этого создаются две кнопки `btn1` и `btn2`, и устанавливаются обработчики сигналов.
Затем эти две кнопки добавляются в блок.

~~~C
 1 static void
 2 click1_cb (GtkButton *btn) {
 3   const char *s;
 4 
 5   s = gtk_button_get_label (btn);
 6   if (g_strcmp0 (s, "Hello.") == 0)
 7     gtk_button_set_label (btn, "Good-bye.");
 8   else
 9     gtk_button_set_label (btn, "Hello.");
10 }
~~~

Функция `gtk_button_get_label` возвращает текст из метки.
Строка принадлежит кнопке, и вы не можете изменять или освобождать её.
Квалификатор `const` необходим для строки `s`.
Если вы измените строку, ваш компилятор выдаст предупреждение.

Вам всегда нужно быть осторожными с квалификатором const, когда вы смотрите справочник GTK 4 API.

![Скриншот блока](../image/screenshot_lb4.png)

Обработчик, соответствующий `btn1`, переключает свою метку.
Обработчик, соответствующий `btn2`, уничтожает окно верхнего уровня, и приложение завершается.

Up: [README.md](../README.md),  Prev: [Section 3](sec3.md), Next: [Section 5](sec5.md)
