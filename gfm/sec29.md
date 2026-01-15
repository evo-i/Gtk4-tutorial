Up: [README.md](../README.md),  Prev: [Section 28](sec28.md), Next: [Section 30](sec30.md)

# GtkListView

GTK 4 добавил новые объекты списков GtkListView, GtkGridView и GtkColumnView.
Новая функциональность описана в [Gtk API Reference -- List Widget Overview](https://docs.gtk.org/gtk4/section-list-widget.html).

GTK 4 имеет другие средства для реализации списков.
Это GtkListBox и GtkTreeView, которые перенесены из GTK 3.
Есть статья в [Gtk Development blog](https://blog.gtk.org/2020/06/07/scalable-lists-in-gtk-4/) о виджетах списков от Matthias Clasen.
Он описал, почему GtkListView разработан для замены GtkTreeView.
GtkTreeView устарел с версии 4.10.

GtkListView, GtkGridView, GtkColumnView и связанные объекты описаны в разделах 29-33.

## Обзор

Список — это последовательная структура данных.
Например, упорядоченная последовательность строк "one", "two", "three", "four" является списком.
Каждый элемент называется элементом (item).
Список похож на массив, но во многих случаях он реализован с помощью указателей, которые указывают на следующие элементы списка.
И у него есть начальная точка.
Таким образом, к каждому элементу можно обратиться по индексу элемента (первый элемент, второй элемент, ..., n-й элемент, ...).
Существует два случая.
Один — индекс начинается с единицы (one-based), другой — с нуля (zero-based).

Gio предоставляет интерфейс GListModel.
Это список с нулевой индексацией, и его элементы являются одним и тем же типом потомков GObject или объектами, реализующими один и тот же интерфейс.
Объект, реализующий GListModel, не является виджетом.
Поэтому список не отображается на экране напрямую.
Существует другой объект GtkListView, который является виджетом для отображения списка.
Элементы в списке должны быть связаны с элементами в GtkListView.
Экземпляр GtkListItemFactory сопоставляет элементы списка с GtkListView.

![List](../image/list.png)

## GListModel и GtkStringList

Если вы хотите создать список строк с помощью GListModel, например, "one", "two", "three", "four", обратите внимание, что строки не могут быть элементами списка.
Потому что GListModel — это список объектов GObject, а строки не являются объектами GObject.
Слово "GObject" здесь означает "класс GObject или его класс-потомок".
Поэтому вам нужна обёртка, которая является GObject и содержит строку.
GtkStringObject — это объект-обёртка, а GtkStringList, реализующий GListModel, — это список объектов GtkStringObject.

~~~C
char *array[] = {"one", "two", "three", "four", NULL};
GtkStringList *stringlist = gtk_string_list_new ((const char * const *) array);
~~~

Функция `gtk_string_list_new` создаёт объект GtkStringList.
Его элементами являются объекты GtkStringObject, которые содержат строки "one", "two", "three" и "four".
Существуют функции для добавления элементов в список или удаления элементов из списка.

- `gtk_string_list_append` добавляет элемент в список
- `gtk_string_list_remove` удаляет элемент из списка
- `gtk_string_list_get_string` получает строку из списка

Для дополнительной информации см. [GTK 4 API Reference -- GtkStringList](https://docs.gtk.org/gtk4/class.StringList.html).

Другие объекты списков будут объяснены позже.

## GtkSelectionModel

GtkSelectionModel — это интерфейс для поддержки выделения.
Благодаря этой модели пользователь может выбирать элементы, нажимая на них.
Он реализован объектами GtkMultiSelection, GtkNoSelection и GtkSingleSelection.
Этих трёх объектов обычно достаточно для создания приложения.
Они создаются с другим GListModel.
Вы также можете создать их отдельно и добавить GListModel позже.

- GtkMultiSelection поддерживает множественный выбор.
- GtkNoSelection поддерживает отсутствие выбора. Это обёртка для GListModel, когда требуется GtkSelectionModel.
- GtkSingleSelection поддерживает одиночный выбор.

## GtkListView

GtkListView — это виджет для отображения элементов GListModel.
GtkListItem используется GtkListView для представления элементов модели списка.
Но сам GtkListItem не является виджетом, поэтому пользователю нужно установить виджет, например GtkLabel, в качестве дочернего элемента GtkListItem для отображения элемента модели списка.
Свойство "item" GtkListItem указывает на объект, принадлежащий модели списка.

![GtkListItem](../image/gtklistitem.png)

В случае, если количество элементов очень велико, например, более тысячи, GtkListItem переиспользуется и связывается с другим элементом, который только что отображается.
Это переиспользование делает количество объектов GtkListItem довольно малым, менее 200.
Это очень эффективно для сдерживания роста потребления памяти, так что GListModel может содержать множество элементов, например, более миллиона элементов.

## GtkListItemFactory

GtkListItemFactory создаёт или переиспользует GtkListItem и связывает его с элементом модели списка.
Существует два дочерних класса этой фабрики: GtkSignalListItemFactory и GtkBuilderListItemFactory.

### GtkSignalListItemFactory

GtkSignalListItemFactory предоставляет сигналы для пользователей для настройки объекта GtkListItem.
Есть четыре сигнала.

1. "setup" испускается для настройки объекта GtkListItem.
Пользователь устанавливает его дочерний виджет в обработчике.
Например, создаёт виджет GtkLabel и устанавливает свойство child GtkListItem на него.
Эта настройка сохраняется, даже если экземпляр GtkListItem переиспользуется (для связывания с другим элементом GListModel).
2. "bind" испускается для связывания элемента в модели списка с виджетом.
Например, пользователь получает элемент из свойства "item" экземпляра GtkListItem.
Затем получает строку элемента и устанавливает свойство label экземпляра GtkLabel этой строкой.
Этот сигнал испускается, когда GtkListItem только что создан, переиспользуется или произошли какие-то изменения с элементом списка.
3. "unbind" испускается для отвязывания элемента.
Пользователь отменяет всё, что было сделано на шаге 2, в обработчике сигнала.
Если какие-то объекты были созданы на шаге 2, они должны быть уничтожены.
4. "teardown" испускается для отмены всего, что было сделано на шаге 1.
Таким образом, виджет, созданный на шаге 1, должен быть уничтожен.
После этого сигнала элемент списка будет уничтожен.

Следующая программа `list1.c` показывает список строк "one", "two", "three" и "four".
Используется GtkNoSelection, поэтому пользователь не может выбрать ни один элемент.

~~~C
 1 #include <gtk/gtk.h>
 2 
 3 static void
 4 setup_cb (GtkSignalListItemFactory *self, GtkListItem *listitem, gpointer user_data) {
 5   GtkWidget *lb = gtk_label_new (NULL);
 6   gtk_list_item_set_child (listitem, lb);
 7   /* Because gtk_list_item_set_child sunk the floating reference of lb, releasing (unref) isn't necessary for lb. */
 8 }
 9 
10 static void
11 bind_cb (GtkSignalListItemFactory *self, GtkListItem *listitem, gpointer user_data) {
12   GtkWidget *lb = gtk_list_item_get_child (listitem);
13   /* Strobj is owned by the instance. Caller mustn't change or destroy it. */
14   GtkStringObject *strobj = gtk_list_item_get_item (listitem);
15   /* The string returned by gtk_string_object_get_string is owned by the instance. */
16   gtk_label_set_text (GTK_LABEL (lb), gtk_string_object_get_string (strobj));
17 }
18 
19 static void
20 unbind_cb (GtkSignalListItemFactory *self, GtkListItem *listitem, gpointer user_data) {
21   /* There's nothing to do here. */
22 }
23 
24 static void
25 teardown_cb (GtkSignalListItemFactory *self, GtkListItem *listitem, gpointer user_data) {
26   /* There's nothing to do here. */
27   /* GtkListItem instance will be destroyed soon. You don't need to set the child to NULL. */
28 }
29 
30 static void
31 app_activate (GApplication *application) {
32   GtkApplication *app = GTK_APPLICATION (application);
33   GtkWidget *win = gtk_application_window_new (app);
34   gtk_window_set_default_size (GTK_WINDOW (win), 600, 400);
35   GtkWidget *scr = gtk_scrolled_window_new ();
36   gtk_window_set_child (GTK_WINDOW (win), scr);
37 
38   char *array[] = {
39     "one", "two", "three", "four", NULL
40   };
41   /* sl is owned by ns */
42   /* ns and factory are owned by lv. */
43   /* Therefore, you don't need to care about their destruction. */
44   GtkStringList *sl =  gtk_string_list_new ((const char * const *) array);
45   GtkNoSelection *ns =  gtk_no_selection_new (G_LIST_MODEL (sl));
46 
47   GtkListItemFactory *factory = gtk_signal_list_item_factory_new ();
48   g_signal_connect (factory, "setup", G_CALLBACK (setup_cb), NULL);
49   g_signal_connect (factory, "bind", G_CALLBACK (bind_cb), NULL);
50   /* The following two lines can be left out. The handlers do nothing. */
51   g_signal_connect (factory, "unbind", G_CALLBACK (unbind_cb), NULL);
52   g_signal_connect (factory, "teardown", G_CALLBACK (teardown_cb), NULL);
53 
54   GtkWidget *lv = gtk_list_view_new (GTK_SELECTION_MODEL (ns), factory);
55   gtk_scrolled_window_set_child (GTK_SCROLLED_WINDOW (scr), lv);
56   gtk_window_present (GTK_WINDOW (win));
57 }
58 
59 /* ----- main ----- */
60 #define APPLICATION_ID "com.github.ToshioCP.list1"
61 
62 int
63 main (int argc, char **argv) {
64   GtkApplication *app;
65   int stat;
66 
67   app = gtk_application_new (APPLICATION_ID, G_APPLICATION_DEFAULT_FLAGS);
68 
69   g_signal_connect (app, "activate", G_CALLBACK (app_activate), NULL);
70 
71   stat = g_application_run (G_APPLICATION (app), argc, argv);
72   g_object_unref (app);
73   return stat;
74 }
~~~

Файл `list1.c` расположен в каталоге [src/misc](../src/misc).
Создайте shell-скрипт ниже и сохраните его в ваш каталог bin, например `$HOME/bin`.

~~~Shell
gcc `pkg-config --cflags gtk4` $1.c `pkg-config --libs gtk4`
~~~

Перейдите в каталог, содержащий `list1.c`, и введите следующее.

~~~
$ chmod 755 $HOME/bin/comp # or chmod 755 (your bin directory)/comp
$ comp list1
$ ./a.out
~~~

Затем появится следующее окно.

![list1](../image/list1.png)

Программа не так сложна.
Если вы чувствуете некоторую сложность, прочитайте этот раздел снова, особенно подраздел GtkSignalListItemFactory.

### GtkBuilderListItemFactory

GtkBuilderListItemFactory — это ещё одна GtkListItemFactory.
Её поведение определяется с помощью ui-файла.

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

Тег template используется для определения GtkListItem.
И его свойство child — это объект GtkLabel.
Фабрика видит этот шаблон и создаёт GtkLabel и устанавливает свойство child GtkListItem.
Это то же самое, что делал обработчик setup GtkSignalListItemFactory.

Затем свяжите свойство label GtkLabel со свойством string объекта GtkStringObject.
Строковый объект ссылается на свойство item GtkListItem.
Таким образом, тег lookup выглядит так:

~~~
label <- string <- GtkStringObject <- item <- GtkListItem
~~~

The last lookup tag has a content `GtkListItem`.
Usually, C type like `GtkListItem` doesn't appear in the content of tags.
This is a special case.
There is an explanation in the [GTK Development Blog](https://blog.gtk.org/2020/09/05/a-primer-on-gtklistview/) by Matthias Clasen.

> Remember that the classname (GtkListItem) in a ui template is used as the “this” pointer referring to the object that is being instantiated.

Therefore, GtkListItem instance is used as the `this` object of the lookup tag when it is evaluated.
`this` object will be explained in [section 31](../src/sec31).

The C source code is as follows.
Its name is `list2.c` and located under [src/misc](../src/misc) directory.

~~~C
 1 #include <gtk/gtk.h>
 2 
 3 static void
 4 app_activate (GApplication *application) {
 5   GtkApplication *app = GTK_APPLICATION (application);
 6   gtk_window_present (gtk_application_get_active_window(app));
 7 }
 8 
 9 static void
10 app_startup (GApplication *application) {
11   GtkApplication *app = GTK_APPLICATION (application);
12   GtkWidget *win = gtk_application_window_new (app);
13   gtk_window_set_default_size (GTK_WINDOW (win), 600, 400);
14   GtkWidget *scr = gtk_scrolled_window_new ();
15   gtk_window_set_child (GTK_WINDOW (win), scr);
16 
17   char *array[] = {
18     "one", "two", "three", "four", NULL
19   };
20   GtkStringList *sl = gtk_string_list_new ((const char * const *) array);
21   GtkSingleSelection *ss = gtk_single_selection_new (G_LIST_MODEL (sl));
22 
23   const char *ui_string =
24 "<interface>"
25   "<template class=\"GtkListItem\">"
26     "<property name=\"child\">"
27       "<object class=\"GtkLabel\">"
28         "<binding name=\"label\">"
29           "<lookup name=\"string\" type=\"GtkStringObject\">"
30             "<lookup name=\"item\">GtkListItem</lookup>"
31           "</lookup>"
32         "</binding>"
33       "</object>"
34     "</property>"
35   "</template>"
36 "</interface>"
37 ;
38   GBytes *gbytes = g_bytes_new_static (ui_string, strlen (ui_string));
39   GtkListItemFactory *factory = gtk_builder_list_item_factory_new_from_bytes (NULL, gbytes);
40 
41   GtkWidget *lv = gtk_list_view_new (GTK_SELECTION_MODEL (ss), factory);
42   gtk_scrolled_window_set_child (GTK_SCROLLED_WINDOW (scr), lv);
43 }
44 
45 /* ----- main ----- */
46 #define APPLICATION_ID "com.github.ToshioCP.list2"
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
62 
~~~

Для GtkBulderListItemFactory не требуется обработчик сигналов.
Используется GtkSingleSelection, поэтому пользователь может выбрать один элемент за раз.

Поскольку это небольшая программа, ui-данные задаются как строка.

## GtkDirectoryList

GtkDirectoryList — это модель списка, содержащая объекты GFileInfo, которые являются информацией о файлах в определённом каталоге.
Она использует `g_file_enumerate_children_async()` для получения объектов GFileInfo.
Модель списка создаётся функцией `gtk_directory_list_new`.

~~~C
GtkDirectoryList *gtk_directory_list_new (const char *attributes, GFile *file);
~~~

`attributes` — это список атрибутов файла, разделённый запятыми.
Атрибуты файла — это пары ключ-значение.
Ключ состоит из пространства имён и имени.
Например, ключ "standard::name" — это имя файла.
"standard" означает общую информацию о файле.
"name" означает имя файла.
В следующей таблице показаны некоторые примеры.

|ключ            |значение                                                                        |
|:---------------|:-------------------------------------------------------------------------------|
|standard::type  |тип файла. например, обычный файл, каталог, символическая ссылка и т.д.         |
|standard::name  |имя файла                                                                       |
|standard::size  |размер файла в байтах                                                          |
|access::can-read|право на чтение, если пользователь может читать файл                            |
|time::modified  |время последнего изменения файла в секундах с начала эпохи UNIX                 |

Текущий каталог — это ".".
Следующая программа создаёт GtkDirectoryList `dl`, и его содержимое — это объекты GFileInfo в текущем каталоге.

~~~C
GFile *file = g_file_new_for_path (".");
GtkDirectoryList *dl = gtk_directory_list_new ("standard::name", file);
g_object_unref (file);
~~~

Не так сложно создать программу для вывода списка файлов, изменив `list2.c` из предыдущего подраздела.
Одна проблема в том, что GInfoFile не имеет свойств.
Тег lookup ищет свойство, поэтому он бесполезен для поиска имени файла из объекта GFileInfo.
Вместо этого в данном случае подходит тег closure.
Тег closure указывает функцию и тип возвращаемого значения функции.

~~~C
char *
get_file_name (GtkListItem *item, GFileInfo *info) {
  return G_IS_FILE_INFO (info) ? g_strdup (g_file_info_get_name (info)) : NULL;
}
... ...
... ...

"<interface>"
  "<template class=\"GtkListItem\">"
    "<property name=\"child\">"
      "<object class=\"GtkLabel\">"
        "<binding name=\"label\">"
          "<closure type=\"gchararray\" function=\"get_file_name\">"
            "<lookup name=\"item\">GtkListItem</lookup>"
          "</closure>"
        "</binding>"
      "</object>"
    "</property>"
  "</template>"
"</interface>"
~~~

- Строка "gchararray" — это имя типа.
Тип "gchar" — это имя типа, и он совпадает с типом C "char".
Поэтому "gchararray" — это "массив типа char", что совпадает с типом строки.
Он используется для получения типа объекта GValue.
GValue — это обобщённое значение, и оно может содержать различные типы значений.
Например, имя типа может быть gboolean, gchar (char), gint (int), gfloat (float), gdouble (double), gchararray (char *) и так далее.
Эти имена типов — это имена фундаментальных типов, зарегистрированных в системе типов.
См. [GObject tutorial](https://github.com/ToshioCP/Gobject-tutorial/blob/main/gfm/sec5.md#gvalue).
- Тег closure имеет атрибут type и атрибут function.
Атрибут function указывает имя функции, а атрибут type указывает тип возвращаемого значения функции.
Содержимое тега closure (между \<closure...\> и\</closure\>) — это параметры функции.
`<lookup name="item">GtkListItem</lookup>` даёт значение свойства item GtkListItem.
Это будет второй аргумент функции.
Первый параметр всегда является экземпляром GListItem, который является объектом 'this'.
Объект 'this' объясняется в разделе 31.
- Функция `gtk_file_name` является функцией обратного вызова для тега closure.
Сначала она проверяет параметр `info`.
Потому что он может быть NULL, когда GListItem `item` не связан.
Если это GFileInfo, она возвращает скопированное имя файла.
Потому что возвращаемое значение (имя файла) `g_file_info_get_name` принадлежит объекту GFileInfo.
Поэтому строку нужно дублировать, чтобы передать владение вызывающей стороне.
Тег binding связывает свойство "label" GtkLabel с тегом closure.

Вся программа (`list3.c`) следующая.
Программа расположена в каталоге [src/misc](../src/misc).

~~~C
 1 #include <gtk/gtk.h>
 2 
 3 char *
 4 get_file_name (GtkListItem *item, GFileInfo *info) {
 5   return G_IS_FILE_INFO (info) ? g_strdup (g_file_info_get_name (info)) : NULL;
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
17   GtkWidget *win = gtk_application_window_new (app);
18   gtk_window_set_default_size (GTK_WINDOW (win), 600, 400);
19   GtkWidget *scr = gtk_scrolled_window_new ();
20   gtk_window_set_child (GTK_WINDOW (win), scr);
21 
22   GFile *file = g_file_new_for_path (".");
23   GtkDirectoryList *dl = gtk_directory_list_new ("standard::name", file);
24   g_object_unref (file);
25   GtkNoSelection *ns = gtk_no_selection_new (G_LIST_MODEL (dl));
26 
27   const char *ui_string =
28 "<interface>"
29   "<template class=\"GtkListItem\">"
30     "<property name=\"child\">"
31       "<object class=\"GtkLabel\">"
32         "<binding name=\"label\">"
33           "<closure type=\"gchararray\" function=\"get_file_name\">"
34             "<lookup name=\"item\">GtkListItem</lookup>"
35           "</closure>"
36         "</binding>"
37       "</object>"
38     "</property>"
39   "</template>"
40 "</interface>"
41 ;
42   GBytes *gbytes = g_bytes_new_static (ui_string, strlen (ui_string));
43   GtkListItemFactory *factory = gtk_builder_list_item_factory_new_from_bytes (NULL, gbytes);
44 
45   GtkWidget *lv = gtk_list_view_new (GTK_SELECTION_MODEL (ns), factory);
46   gtk_scrolled_window_set_child (GTK_SCROLLED_WINDOW (scr), lv);
47 }
48 
49 /* ----- main ----- */
50 #define APPLICATION_ID "com.github.ToshioCP.list3"
51 
52 int
53 main (int argc, char **argv) {
54   GtkApplication *app;
55   int stat;
56 
57   app = gtk_application_new (APPLICATION_ID, G_APPLICATION_DEFAULT_FLAGS);
58 
59   g_signal_connect (app, "startup", G_CALLBACK (app_startup), NULL);
60   g_signal_connect (app, "activate", G_CALLBACK (app_activate), NULL);
61 
62   stat = g_application_run (G_APPLICATION (app), argc, argv);
63   g_object_unref (app);
64   return stat;
65 }
~~~

Ui-данные (xml-данные выше) используются для построения шаблона GListItem во время выполнения.
GtkBuilder обращается к таблице символов для поиска функции `get_file_name`.

Как правило, таблица символов используется компоновщиком для связывания объектов в исполняемый файл.
Она включает имена функций и их местоположение.
Компоновщик обычно не помещает таблицу символов в созданный исполняемый файл.
Но если задана опция `--export-dynamic`, компоновщик добавляет таблицу символов в исполняемый файл.

Для этого компилятору C передаётся опция `-Wl,--export-dynamic`.

- `-Wl` — это опция компилятора C, которая передаёт следующую опцию компоновщику.
- `--export-dynamic` — это опция компоновщика.
Следующее цитируется из документации компоновщика.
"При создании динамически связанного исполняемого файла добавить все символы в динамическую таблицу символов.
Динамическая таблица символов — это набор символов, которые видны из динамических объектов во время выполнения."

Скомпилируйте и выполните её.

~~~
$ gcc -Wl,--export-dynamic `pkg-config --cflags gtk4` list3.c `pkg-config --libs gtk4`
~~~

Вы также можете создать shell-скрипт для компиляции `list3.c`

~~~bash
gcc -Wl,--export-dynamic `pkg-config --cflags gtk4` $1.c `pkg-config --libs gtk4`
~~~

Сохраните эту одну строку в файл `comp`.
Затем скопируйте его в `$HOME/bin` и дайте ему права на выполнение.

~~~
$ cp comp $HOME/bin/comp
$ chmod +x $HOME/bin/comp
~~~

Вы можете скомпилировать `list3.c` и выполнить его так:

~~~
$ comp list3
$ ./a.out
~~~

![screenshot list3](../image/list3.png)

Up: [README.md](../README.md),  Prev: [Section 28](sec28.md), Next: [Section 30](sec30.md)
