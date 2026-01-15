Up: [README.md](../README.md),  Prev: [Section 27](sec27.md), Next: [Section 29](sec29.md)

# Перетаскивание

## Что такое перетаскивание?

Перетаскивание также записывается как "Drag-and-Drop", или сокращённо "DND".
DND подобно "копировать и вставить" или "вырезать и вставить".
Если пользователь перетаскивает элемент пользовательского интерфейса, который является виджетом, выбранной частью или чем-то ещё, данные передаются из источника в назначение.

Вы, вероятно, имели опыт перемещения файла с помощью DND.

![DND on the GUI file manager](../image/dnd.png)

Когда DND начинается, файл `sample_file.txt` передаётся системе.
Когда DND заканчивается, система передаёт `sample_file.txt` в каталог `sample_folder` в файловом менеджере.
Поэтому это похоже на "вырезать и вставить".
Фактическое поведение может отличаться от объяснения здесь, но концепция похожа.

## Пример для DND

Этот учебник предоставляет простой пример в каталоге `src/dnd`.
Он имеет три метки для источника и одну метку для назначения.
Исходные метки имеют метки "red", "green" или "blue".
Если пользователь перетаскивает метку на метку назначения, цвет шрифта будет изменён.

![DND example](../image/dnd_canvas.png)

## UI-файл

Виджеты определены в XML-файле `dnd.ui`.

~~~xml
 1 <?xml version="1.0" encoding="UTF-8"?>
 2 <interface>
 3   <object class="GtkApplicationWindow" id="win">
 4     <property name="default-width">800</property>
 5     <property name="default-height">600</property>
 6     <property name="resizable">FALSE</property>
 7     <property name="title">Drag and Drop</property>
 8     <child>
 9       <object class="GtkBox">
10         <property name="hexpand">TRUE</property>
11         <property name="vexpand">TRUE</property>
12         <property name="orientation">GTK_ORIENTATION_VERTICAL</property>
13         <property name="spacing">5</property>
14         <child>
15           <object class="GtkBox">
16             <property name="orientation">GTK_ORIENTATION_HORIZONTAL</property>
17             <property name="homogeneous">TRUE</property>
18             <child>
19               <object class="GtkLabel" id="red">
20                 <property name="label">RED</property>
21                 <property name="justify">GTK_JUSTIFY_CENTER</property>
22                 <property name="name">red</property>
23               </object>
24             </child>
25             <child>
26               <object class="GtkLabel" id="green">
27                 <property name="label">GREEN</property>
28                 <property name="justify">GTK_JUSTIFY_CENTER</property>
29                 <property name="name">green</property>
30               </object>
31             </child>
32             <child>
33               <object class="GtkLabel" id="blue">
34                 <property name="label">BLUE</property>
35                 <property name="justify">GTK_JUSTIFY_CENTER</property>
36                 <property name="name">blue</property>
37               </object>
38             </child>
39           </object>
40         </child>
41         <child>
42           <object class="GtkLabel" id="canvas">
43             <property name="label">CANVAS</property>
44             <property name="justify">GTK_JUSTIFY_CENTER</property>
45             <property name="name">canvas</property>
46             <property name="hexpand">TRUE</property>
47             <property name="vexpand">TRUE</property>
48           </object>
49         </child>
50       </object>
51     </child>
52   </object>
53 </interface>
54 
~~~

Он преобразуется в файл ресурсов с помощью `glib-compile-resources`.
Компилятор использует XML-файл `dnd.gresource.xml`.

~~~xml
1 <?xml version="1.0" encoding="UTF-8"?>
2 <gresources>
3   <gresource prefix="/com/github/ToshioCP/dnd">
4     <file>dnd.ui</file>
5   </gresource>
6 </gresources>
~~~

## C-файл dnd.c

C-файл `dnd.c` не является большим файлом.
Количество строк меньше ста.
Объект GtkApplication создаётся в функции `main`.

~~~C
 1 int
 2 main (int argc, char **argv) {
 3   GtkApplication *app;
 4   int stat;
 5 
 6   app = gtk_application_new (APPLICATION_ID, G_APPLICATION_DEFAULT_FLAGS);
 7   g_signal_connect (app, "startup", G_CALLBACK (app_startup), NULL);
 8   g_signal_connect (app, "activate", G_CALLBACK (app_activate), NULL);
 9   stat = g_application_run (G_APPLICATION (app), argc, argv);
10   g_object_unref (app);
11   return stat;
12 }
~~~

ID приложения определён как:

```C
#define APPLICATION_ID "com.github.ToshioCP.dnd"
```

### Обработчик сигнала startup

Большая часть работы выполняется в обработчике сигнала "startup".

Для реализации DND используются два объекта GtkDragSource и GtkDropTarget.

- Источник перетаскивания: источник перетаскивания (экземпляр GtkDragSource) является контроллером событий.
Он инициирует операцию DND, когда пользователь щёлкает и перетаскивает виджет.
Если данные в форме GdkContentProvider установлены заранее, он передаёт данные системе в начале перетаскивания.
- Цель перетаскивания: цель перетаскивания (GtkDropTarget) также является контроллером событий.
Вы можете получить данные в обработчике сигнала GtkDropTarget::drop.

Приведённый ниже пример использует эти объекты очень простым способом.
Вы можете использовать ряд функций, которые есть у этих двух объектов.
Для получения дополнительной информации см. следующие ссылки.

- [Drag-and-Drop in GTK](https://docs.gtk.org/gtk4/drag-and-drop.html)
- [GtkDragSource](https://docs.gtk.org/gtk4/class.DragSource.html)
- [GtkDropTarget](https://docs.gtk.org/gtk4/class.DropTarget.html)

~~~C
 1 static void
 2 app_startup (GApplication *application) {
 3   GtkApplication *app = GTK_APPLICATION (application);
 4   GtkBuilder *build;
 5   GtkWindow *win;
 6   GtkLabel *src_labels[3];
 7   int i;
 8   GtkLabel *canvas;
 9   GtkDragSource *src;
10   GdkContentProvider* content;
11   GtkDropTarget *tgt;
12   GdkDisplay *display;
13   char *s;
14 
15   build = gtk_builder_new_from_resource ("/com/github/ToshioCP/dnd/dnd.ui");
16   win = GTK_WINDOW (gtk_builder_get_object (build, "win"));
17   src_labels[0] = GTK_LABEL (gtk_builder_get_object (build, "red"));
18   src_labels[1] = GTK_LABEL (gtk_builder_get_object (build, "green"));
19   src_labels[2] = GTK_LABEL (gtk_builder_get_object (build, "blue"));
20   canvas = GTK_LABEL (gtk_builder_get_object (build, "canvas"));
21   gtk_window_set_application (win, app);
22   g_object_unref (build);
23 
24   for (i=0; i<3; ++i) {
25     src = gtk_drag_source_new ();
26     content = gdk_content_provider_new_typed (G_TYPE_STRING, gtk_widget_get_name (GTK_WIDGET (src_labels[i])));
27     gtk_drag_source_set_content (src, content);
28     g_object_unref (content);
29     gtk_widget_add_controller (GTK_WIDGET (src_labels[i]), GTK_EVENT_CONTROLLER (src)); // The ownership of src is taken by the instance.
30   }
31 
32   tgt = gtk_drop_target_new (G_TYPE_STRING, GDK_ACTION_COPY);
33   g_signal_connect (tgt, "drop", G_CALLBACK (drop_cb), NULL);
34   gtk_widget_add_controller (GTK_WIDGET (canvas), GTK_EVENT_CONTROLLER (tgt)); // The ownership of tgt is taken by the instance.
35 
36   provider = gtk_css_provider_new ();
37   s = g_strdup_printf (format, "black");
38   gtk_css_provider_load_from_data (provider, s, -1);
39   g_free (s);
40   display = gdk_display_get_default ();
41   gtk_style_context_add_provider_for_display (display, GTK_STYLE_PROVIDER (provider),
42                                               GTK_STYLE_PROVIDER_PRIORITY_APPLICATION);
43   g_object_unref (provider); // The provider is still alive because the display owns it.
44 }
~~~

- 15-22: строит виджеты.
Массив `source_labels[]` указывает на исходные метки red, green и blue в ui-файле.
Переменная `canvas` указывает на метку назначения.
- 24-30: устанавливает виджеты источника DND.
Цикл for перебирает массив `src_labels[]`, каждый из которых указывает на исходный виджет, метку red, green или blue.
- 25: создаёт новый экземпляр GtkDragSource.
- 26: создаёт новый экземпляр GdkContentProvider со строкой "red", "green" или "blue.
Это имена виджетов.
Эти строки являются данными для передачи через операцию DND.
- 27: устанавливает содержимое источника перетаскивания на экземпляр GdkContentProvider выше.
- 28: содержимое бесполезно, поэтому оно уничтожается.
- 29: добавляет контроллер событий, который на самом деле является источником перетаскивания, к виджету.
Если операция DND начинается на виджете, соответствующий источник перетаскивания работает, и данные передаются системе.
- 32-34: устанавливает цель перетаскивания DND.
- 32: создаёт новый экземпляр GtkDropTarget.
Первый параметр — это GType данных.
Второй параметр — это константа перечисления GdkDragAction.
Аргументами здесь являются тип строки и константа для копирования.
- 33: подключает сигнал "drop" и обработчик `drop_cb`.
- 34: добавляет контроллер событий, который на самом деле является целью перетаскивания, к виджету.
- 36-43: устанавливает CSS.
- 37: переменная `format` является статической и определена в верхней части программы.
Статические переменные показаны ниже.

```C
static GtkCssProvider *provider = NULL;
static const char *format = "label {padding: 20px;} label#red {background: red;} "
  "label#green {background: green;} label#blue {background: blue;} "
  "label#canvas {color: %s; font-weight: bold; font-size: 72pt;}";
```

### Обработчик сигнала activate

~~~C
1 static void
2 app_activate (GApplication *application) {
3   GtkApplication *app = GTK_APPLICATION (application);
4   GtkWindow *win;
5 
6   win = gtk_application_get_active_window (app);
7   gtk_window_present (win);
8 }
~~~

Этот обработчик просто показывает окно.

### Обработчик сигнала drop

~~~C
1 static gboolean
2 drop_cb (GtkDropTarget* self, const GValue* value, gdouble x, gdouble y, gpointer user_data) {
3   char *s;
4 
5   s = g_strdup_printf (format, g_value_get_string (value));
6   gtk_css_provider_load_from_data (provider, s, -1);
7   g_free (s);
8   return TRUE;
9 }
~~~

Обработчик сигнала "drop" имеет пять параметров.

- Экземпляр GtkDropTarget, на котором был испущен сигнал.
- GValue, который содержит данные из источника.
- Аргументы `x` и `y` — это координаты мыши при отпускании.
- Пользовательские данные были установлены при подключении сигнала и обработчика.

Строка из GValue — это "red", "green" или "blue".
Она заменяет "%s" в переменной `format`.
Это означает, что цвет шрифта метки `canvas` изменится на этот цвет.

## Meson.build

Файл `meson.build` управляет процессом сборки.

~~~meson
1 project('dnd', 'c')
2 
3 gtkdep = dependency('gtk4')
4 
5 gnome = import('gnome')
6 resources = gnome.compile_resources('resources','dnd.gresource.xml')
7 
8 executable(meson.project_name(), 'dnd.c', resources, dependencies: gtkdep, export_dynamic: true, install: false)
~~~

Вы можете собрать его из командной строки.

```
$ cd src/dnd
$ meson setup _build
$ ninja -C _build
$ _build/dnd
```

Исходные файлы находятся в каталоге `src/dnd` [репозитория](https://github.com/ToshioCP/Gtk4-tutorial).
Загрузите его и посмотрите каталог.

Up: [README.md](../README.md),  Prev: [Section 27](sec27.md), Next: [Section 29](sec29.md)
