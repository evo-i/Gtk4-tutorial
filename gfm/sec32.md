Up: [README.md](../README.md),  Prev: [Section 31](sec31.md), Next: [Section 33](sec33.md)

# GtkColumnView

## GtkColumnView

GtkColumnView похож на GtkListView, но имеет несколько столбцов.
Каждый столбец является GtkColumnViewColumn.

![Column View](../image/column_view.png)

- GtkColumnView имеет свойство "model".
Свойство указывает на объект GtkSelectionModel.
- Каждый GtkColumnViewColumn имеет свойство "factory".
Свойство указывает на GtkListItemFactory (GtkSignalListItemFactory или GtkBuilderListItemFactory).
- Фабрика связывает GtkListItem и элементы GtkSelectionModel.
И фабрика создаёт виджеты-потомки GtkColumnView для отображения элемента на экране.
Этот процесс такой же, как и в GtkListView.

Следующая диаграмма показывает, как это работает.

![ColumnView](../image/column.png)

Пример в этом разделе — это окно, которое отображает информацию о файлах в текущем каталоге.
Информация — это имя, размер и время последнего изменения файлов.
Таким образом, есть три столбца.

Кроме того, пример использует GtkSortListModel и GtkSorter для сортировки информации.

## column.ui

Файл Ui задаёт виджеты и шаблоны элементов списка.

~~~xml
  1 <?xml version="1.0" encoding="UTF-8"?>
  2 <interface>
  3   <object class="GtkApplicationWindow" id="win">
  4     <property name="title">file list</property>
  5     <property name="default-width">800</property>
  6     <property name="default-height">600</property>
  7     <child>
  8       <object class="GtkScrolledWindow">
  9         <property name="hexpand">TRUE</property>
 10         <property name="vexpand">TRUE</property>
 11         <child>
 12           <object class="GtkColumnView" id="columnview">
 13             <property name="model">
 14               <object class="GtkNoSelection">
 15                 <property name="model">
 16                   <object class="GtkSortListModel">
 17                     <property name="model">
 18                       <object class="GtkDirectoryList" id="directorylist">
 19                         <property name="attributes">standard::name,standard::icon,standard::size,time::modified</property>
 20                       </object>
 21                     </property>
 22                     <binding name="sorter">
 23                       <lookup name="sorter">columnview</lookup>
 24                     </binding>
 25                   </object>
 26                 </property>
 27               </object>
 28             </property>
 29             <child>
 30               <object class="GtkColumnViewColumn">
 31                 <property name="title">Name</property>
 32                 <property name="expand">TRUE</property>
 33                 <property name="factory">
 34                   <object class="GtkBuilderListItemFactory">
 35                     <property name="bytes"><![CDATA[
 36 <?xml version="1.0" encoding="UTF-8"?>
 37 <interface>
 38   <template class="GtkListItem">
 39     <property name="child">
 40       <object class="GtkBox">
 41         <property name="orientation">GTK_ORIENTATION_HORIZONTAL</property>
 42         <property name="spacing">20</property>
 43         <child>
 44           <object class="GtkImage">
 45             <binding name="gicon">
 46               <closure type="GIcon" function="get_icon_factory">
 47                 <lookup name="item">GtkListItem</lookup>
 48               </closure>
 49             </binding>
 50           </object>
 51         </child>
 52         <child>
 53           <object class="GtkLabel">
 54             <property name="hexpand">TRUE</property>
 55             <property name="xalign">0</property>
 56             <binding name="label">
 57               <closure type="gchararray" function="get_file_name_factory">
 58                 <lookup name="item">GtkListItem</lookup>
 59               </closure>
 60             </binding>
 61           </object>
 62         </child>
 63       </object>
 64     </property>
 65   </template>
 66 </interface>
 67                     ]]></property>
 68                   </object>
 69                 </property>
 70                 <property name="sorter">
 71                   <object class="GtkStringSorter">
 72                     <property name="expression">
 73                       <closure type="gchararray" function="get_file_name">
 74                       </closure>
 75                     </property>
 76                   </object>
 77                 </property>
 78               </object>
 79             </child>
 80             <child>
 81               <object class="GtkColumnViewColumn">
 82                 <property name="title">Size</property>
 83                 <property name="factory">
 84                   <object class="GtkBuilderListItemFactory">
 85                     <property name="bytes"><![CDATA[
 86 <?xml version="1.0" encoding="UTF-8"?>
 87 <interface>
 88   <template class="GtkListItem">
 89     <property name="child">
 90       <object class="GtkLabel">
 91         <property name="hexpand">TRUE</property>
 92         <property name="xalign">0</property>
 93         <binding name="label">
 94           <closure type="gint64" function="get_file_size_factory">
 95             <lookup name="item">GtkListItem</lookup>
 96           </closure>
 97         </binding>
 98       </object>
 99     </property>
100   </template>
101 </interface>
102                     ]]></property>
103                   </object>
104                 </property>
105                 <property name="sorter">
106                   <object class="GtkNumericSorter">
107                     <property name="expression">
108                       <closure type="gint64" function="get_file_size">
109                       </closure>
110                     </property>
111                     <property name="sort-order">GTK_SORT_ASCENDING</property>
112                   </object>
113                 </property>
114               </object>
115             </child>
116             <child>
117               <object class="GtkColumnViewColumn">
118                 <property name="title">Date modified</property>
119                 <property name="factory">
120                   <object class="GtkBuilderListItemFactory">
121                     <property name="bytes"><![CDATA[
122 <?xml version="1.0" encoding="UTF-8"?>
123 <interface>
124   <template class="GtkListItem">
125     <property name="child">
126       <object class="GtkLabel">
127         <property name="hexpand">TRUE</property>
128         <property name="xalign">0</property>
129         <binding name="label">
130           <closure type="gchararray" function="get_file_time_modified_factory">
131             <lookup name="item">GtkListItem</lookup>
132           </closure>
133         </binding>
134       </object>
135     </property>
136   </template>
137 </interface>
138                     ]]></property>
139                   </object>
140                 </property>
141                 <property name="sorter">
142                   <object class="GtkNumericSorter">
143                     <property name="expression">
144                       <closure type="gint64" function="get_file_unixtime_modified">
145                       </closure>
146                     </property>
147                     <property name="sort-order">GTK_SORT_ASCENDING</property>
148                   </object>
149                 </property>
150               </object>
151             </child>
152           </object>
153         </child>
154       </object>
155     </child>
156   </object>
157 </interface>
~~~

- 3-12: GtkApplicationWindow имеет дочерний виджет GtkScrolledWindow.
GtkScrolledWindoww имеет дочерний виджет GtkColumnView.
- 12-18: GtkColumnView имеет свойство "model".
Оно указывает на интерфейс GtkSelectionModel.
Класс GtkNoSelection используется как GtkSelectionModel.
И снова, он имеет свойство "model".
Оно указывает на GtkSortListModel.
Эта модель списка поддерживает сортировку списка.
Это будет объяснено в следующем подразделе.
И он также имеет свойство "model".
Оно указывает на GtkDirectoryList.
Следовательно, цепочка такова: GtkColumnView => GtkNoSelection => GtkSortListModel => GtkDirectoryList.
- 18-20: GtkDirectoryList.
Это список GFileInfo, который хранит информацию о файлах в каталоге.
Он имеет свойство "attributes".
Оно указывает, какие атрибуты хранятся в каждом GFileInfo.
  - "standard::name" — это имя файла.
  - "standard::icon" — это объект GIcon файла
  - "standard::size" — это размер файла.
  - "time::modified" — это дата и время последнего изменения файла.
- 29-79: Первый объект GtkColumnViewColumn.
Есть четыре свойства: "title", "expand", factory" и "sorter".
- 31: устанавливает свойство "title" в "Name".
Это заголовок в заголовке столбца.
- 32: устанавливает свойство "expand" в TRUE, чтобы позволить столбцу расширяться насколько возможно.
(См. изображение выше).
- 33- 69: устанавливает свойство "factory" в GtkBuilderListItemFactory.
Фабрика имеет свойство "bytes", которое содержит строку ui для определения шаблона для расширения класса GtkListItem.
Раздел CDATA (строка 36-66) — это строка ui для помещения в свойство "bytes".
Содержимое то же самое, что и в файле ui `factory_list.ui` в разделе 30.
- 70-77: устанавливает свойство "sorter" в объект GtkStringSorter.
Этот объект предоставляет сортировщик, который сравнивает строки.
Он имеет свойство "expression".
Здесь используется тег closure с функцией типа string `get_file_name`.
Функция будет объяснена позже.
- 80-115: Второй объект GtkColumnViewColumn.
Его свойство sorter установлено в GtkNumericSorter.
- 116-151: Третий объект GtkColumnViewColumn.
Его свойство sorter установлено в GtkNumericSorter.

## GtkSortListModel и GtkSorter

GtkSortListModel — это модель списка, которая сортирует свои элементы согласно экземпляру GtkSorter, назначенному свойству "sorter".
Свойство привязано к свойству "sorter" GtkColumnView в строках 22-24.

~~~xml
<object class="GtkSortListModel" id="sortlist">
... ... ...
  <binding name="sorter">
    <lookup name="sorter">columnview</lookup>
  </binding>
~~~

Поэтому `columnview` определяет способ сортировки модели списка.
Свойство "sorter" GtkColumnView — это свойство только для чтения, и это особый сортировщик.
Он отражает выбор сортировки пользователя.
Если пользователь нажимает заголовок столбца, то сортировщик (свойство "sorter") столбца ссылается на свойство "sorter" GtkColumnView.
Если пользователь нажимает заголовок другого столбца, то свойство "sorter" GtkColumnView ссылается на свойство "sorter" вновь нажатого столбца.

Привязка выше создаёт косвенное соединение между свойством "sorter" GtkSortListModel и свойством "sorter" каждого столбца.

GtkSorter сравнивает два элемента (GObject или его потомок).
GtkSorter имеет несколько дочерних объектов.

- GtkStringSorter сравнивает строки, взятые из элементов.
- GtkNumericSorter сравнивает числа, взятые из элементов.
- GtkCustomSorter использует обратный вызов для сравнения.
- GtkMultiSorter объединяет несколько сортировщиков.

Пример использует GtkStringSorter и GtkNumericSorter.

GtkStringSorter использует GtkExpression для получения строк из элементов (объектов).
GtkExpression хранится в свойстве "expression" GtkStringSorter.
Когда GtkStringSorter сравнивает два элемента, он оценивает выражение, вызывая функцию `gtk_expression_evaluate`.
Он назначает каждый элемент второму аргументу (объект 'this') функции.

В файле ui выше GtkExpression находится в строках 71-76.

~~~xml
<object class="GtkStringSorter">
  <property name="expression">
    <closure type="gchararray" function="get_file_name">
    </closure>
  </property>
</object>
~~~

GtkExpression вызывает функцию `get_file_name` при её оценке.

~~~C
1 char *
2 get_file_name (GFileInfo *info) {
3   return G_IS_FILE_INFO (info) ? g_strdup(g_file_info_get_name (info)) : NULL;
4 }
~~~

Функции передаётся элемент (GFileInfo) GtkSortListModel в качестве аргумента (объект `this`).
Но нужно быть осторожным, что он может быть NULL во время переработки элемента списка.
Поэтому `G_IS_FILE_INFO (info)` всегда необходим в функциях обратного вызова.
Функция извлекает имя файла из `info`.
Строка принадлежит `info`, поэтому необходимо дублировать.
И она возвращает скопированную строку.

GtkNumericSorter сравнивает числа.
Он используется в строках 106-112 и строках 142-148.
Строки от 106 до 112:

~~~xml
<object class="GtkNumericSorter">
  <property name="expression">
    <closure type="gint64" function="get_file_size">
    </closure>
  </property>
  <property name="sort-order">GTK_SORT_ASCENDING</property>
</object>
~~~

Тег closure указывает функцию обратного вызова `get_file_size`.

~~~C
1 goffset
2 get_file_size (GFileInfo *info) {
3   return G_IS_FILE_INFO (info) ? g_file_info_get_size (info): -1;
4 }
~~~

Она просто возвращает размер `info`.
Тип размера — `goffset`.
Тип `goffset` такой же, как `gint64`.

Строки от 142 до 148:

~~~xml
<object class="GtkNumericSorter" id="sorter_datetime_modified">
  <property name="expression">
    <closure type="gint64" function="get_file_unixtime_modified">
    </closure>
  </property>
  <property name="sort-order">GTK_SORT_ASCENDING</property>
</object>
~~~

Тег closure указывает функцию обратного вызова `get_file_unixtime_modified`.

~~~C
1 gint64
2 get_file_unixtime_modified (GFileInfo *info) {
3   GDateTime *dt;
4 
5   dt = G_IS_FILE_INFO (info) ? g_file_info_get_modification_date_time (info) : NULL;
6   return dt ? g_date_time_to_unix (dt) : -1;
7 }
~~~

Она получает дату и время модификации (тип GDateTime) из `info`.
Затем она получает unix time из `dt`.
Unix time, иногда называемое unix epoch, — это количество секунд, прошедших с 00:00:00 UTC 1 января 1970 года.
Она возвращает unix time (тип gint64).

## column.c

`column.c` выглядит следующим образом.
Он простой и короткий благодаря `column.ui`.

~~~C
 1 #include <gtk/gtk.h>
 2 
 3 /* functions (closures) for GtkBuilderListItemFactory */
 4 GIcon *
 5 get_icon_factory (GtkListItem *item, GFileInfo *info) {
 6   GIcon *icon;
 7 
 8    /* g_file_info_get_icon can return NULL */
 9   icon = G_IS_FILE_INFO (info) ? g_file_info_get_icon (info) : NULL;
10   return icon ? g_object_ref (icon) : NULL;
11 }
12 
13 char *
14 get_file_name_factory (GtkListItem *item, GFileInfo *info) {
15   return G_IS_FILE_INFO (info) ? g_strdup (g_file_info_get_name (info)) : NULL;
16 }
17 
18 /* goffset is defined as gint64 */
19 /* It is used for file offsets. */
20 goffset
21 get_file_size_factory (GtkListItem *item, GFileInfo *info) {
22   return G_IS_FILE_INFO (info) ? g_file_info_get_size (info) : -1;
23 }
24 
25 char *
26 get_file_time_modified_factory (GtkListItem *item, GFileInfo *info) {
27   GDateTime *dt;
28 
29    /* g_file_info_get_modification_date_time can return NULL */
30   dt = G_IS_FILE_INFO (info) ? g_file_info_get_modification_date_time (info) : NULL;
31   return dt ? g_date_time_format (dt, "%F") : NULL;
32 }
33 
34 /* Functions (closures) for GtkSorter */
35 char *
36 get_file_name (GFileInfo *info) {
37   return G_IS_FILE_INFO (info) ? g_strdup(g_file_info_get_name (info)) : NULL;
38 }
39 
40 goffset
41 get_file_size (GFileInfo *info) {
42   return G_IS_FILE_INFO (info) ? g_file_info_get_size (info): -1;
43 }
44 
45 gint64
46 get_file_unixtime_modified (GFileInfo *info) {
47   GDateTime *dt;
48 
49   dt = G_IS_FILE_INFO (info) ? g_file_info_get_modification_date_time (info) : NULL;
50   return dt ? g_date_time_to_unix (dt) : -1;
51 }
52 
53 static void
54 app_activate (GApplication *application) {
55   GtkApplication *app = GTK_APPLICATION (application);
56   gtk_window_present (gtk_application_get_active_window(app));
57 }
58 
59 static void
60 app_startup (GApplication *application) {
61   GtkApplication *app = GTK_APPLICATION (application);
62   GFile *file;
63   GtkBuilder *build = gtk_builder_new_from_resource ("/com/github/ToshioCP/column/column.ui");
64   GtkWidget *win = GTK_WIDGET (gtk_builder_get_object (build, "win"));
65   GtkDirectoryList *directorylist = GTK_DIRECTORY_LIST (gtk_builder_get_object (build, "directorylist"));
66   g_object_unref (build);
67 
68   gtk_window_set_application (GTK_WINDOW (win), app);
69 
70   file = g_file_new_for_path (".");
71   gtk_directory_list_set_file (directorylist, file);
72   g_object_unref (file);
73 }
74 
75 #define APPLICATION_ID "com.github.ToshioCP.columnview"
76 
77 int
78 main (int argc, char **argv) {
79   GtkApplication *app;
80   int stat;
81 
82   app = gtk_application_new (APPLICATION_ID, G_APPLICATION_DEFAULT_FLAGS);
83 
84   g_signal_connect (app, "startup", G_CALLBACK (app_startup), NULL);
85   g_signal_connect (app, "activate", G_CALLBACK (app_activate), NULL);
86 
87   stat = g_application_run (G_APPLICATION (app), argc, argv);
88   g_object_unref (app);
89   return stat;
90 }
91 
~~~


## Компиляция и выполнение.

Все исходные файлы находятся в каталоге [`src/column`](../src/column).
Измените текущий каталог на этот каталог и введите следующее.

~~~
$ cd src/colomn
$ meson setup _build
$ ninja -C _build
$ _build/column
~~~

Затем появляется окно.

![Column View](../image/column_view.png)

Если вы нажмёте заголовок столбца, то весь список будет отсортирован по этому столбцу.
Если вы нажмёте заголовок другого столбца, то весь список будет отсортирован по вновь выбранному столбцу.

GtkColumnView очень полезен, и он может управлять очень большим GListModel.
Можно использовать его для списка файлов, списка приложений, интерфейса базы данных и так далее.

Up: [README.md](../README.md),  Prev: [Section 31](sec31.md), Next: [Section 33](sec33.md)
