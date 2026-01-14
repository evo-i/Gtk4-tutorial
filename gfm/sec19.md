Up: [README.md](../README.md),  Prev: [Section 18](sec18.md), Next: [Section 20](sec20.md)

# UI-файл для меню и записей действий

## UI-файл для меню

Вы, возможно, подумали, что создание меню было действительно утомительным.
Да, программа была сложной, и на её написание требовалось много времени.
Ситуация похожа на создание виджетов.
Когда мы создавали виджеты, использование ui-файла было хорошим способом избежать таких сложностей.
То же самое относится и к меню.

UI-файл для меню содержит теги interface и menu.
Файл начинается и заканчивается тегами interface.

~~~xml
<interface>
  <menu id="menubar">
  </menu>
</interface>
~~~

Тег `menu` соответствует объекту GMenu.
Атрибут `id` определяет имя объекта.
На него будет ссылаться GtkBuilder.

~~~xml
<submenu>
  <attribute name="label">File</attribute>
    <item>
      <attribute name="label">New</attribute>
      <attribute name="action">win.new</attribute>
    </item>
</submenu>
~~~

Тег `item` соответствует элементу в GMenu, который имеет ту же структуру, что и GMenuItem.
Приведённый выше элемент имеет атрибут label.
Его значение — "New".
Элемент также имеет атрибут action, и его значение — "win.new".
"win" — это префикс, а "new" — имя действия.
Тег `submenu` соответствует как GMenuItem, так и GMenu.
GMenuItem содержит ссылку на GMenu.

UI-файл выше можно описать следующим образом.

~~~xml
<item>
  <attribute name="label">File</attribute>
    <link name="submenu">
      <item>
        <attribute name="label">New</attribute>
        <attribute name="action">win.new</attribute>
      </item>
    </link>
</item>
~~~

Тег `link` выражает ссылку на подменю.
И в то же время он также выражает само подменю.
Этот файл иллюстрирует взаимосвязь между меню и элементами лучше, чем предыдущий ui-файл.
Но тег `submenu` прост и понятен.
Поэтому мы обычно предпочитаем первый стиль ui.

Для получения дополнительной информации см. [GTK 4 API reference -- PopoverMenu](https://docs.gtk.org/gtk4/class.PopoverMenu.html#menu-models).

Ниже приведён скриншот примера программы `menu3`.
Она находится в каталоге [src/menu3](../src/menu3).

![menu3](../image/menu3.png)

Ниже приведён ui-файл для `menu3`.

~~~xml
 1 <?xml version="1.0" encoding="UTF-8"?>
 2 <interface>
 3   <menu id="menubar">
 4     <submenu>
 5       <attribute name="label">File</attribute>
 6       <section>
 7         <item>
 8           <attribute name="label">New</attribute>
 9           <attribute name="action">app.new</attribute>
10         </item>
11         <item>
12           <attribute name="label">Open</attribute>
13           <attribute name="action">app.open</attribute>
14         </item>
15       </section>
16       <section>
17         <item>
18           <attribute name="label">Save</attribute>
19           <attribute name="action">win.save</attribute>
20         </item>
21         <item>
22           <attribute name="label">Save As…</attribute>
23           <attribute name="action">win.saveas</attribute>
24         </item>
25       </section>
26       <section>
27         <item>
28           <attribute name="label">Close</attribute>
29           <attribute name="action">win.close</attribute>
30         </item>
31       </section>
32       <section>
33         <item>
34           <attribute name="label">Quit</attribute>
35           <attribute name="action">app.quit</attribute>
36         </item>
37       </section>
38     </submenu>
39     <submenu>
40       <attribute name="label">Edit</attribute>
41       <section>
42         <item>
43           <attribute name="label">Cut</attribute>
44           <attribute name="action">app.cut</attribute>
45         </item>
46         <item>
47           <attribute name="label">Copy</attribute>
48           <attribute name="action">app.copy</attribute>
49         </item>
50         <item>
51           <attribute name="label">Paste</attribute>
52           <attribute name="action">app.paste</attribute>
53         </item>
54       </section>
55       <section>
56         <item>
57           <attribute name="label">Select All</attribute>
58           <attribute name="action">app.selectall</attribute>
59         </item>
60       </section>
61     </submenu>
62     <submenu>
63       <attribute name="label">View</attribute>
64       <section>
65         <item>
66           <attribute name="label">Full Screen</attribute>
67           <attribute name="action">win.fullscreen</attribute>
68         </item>
69       </section>
70     </submenu>
71   </menu>
72 </interface>
~~~

UI-файл преобразуется в ресурс компилятором ресурсов `glib-compile-resouces` с помощью xml-файла.

~~~xml
1 <?xml version="1.0" encoding="UTF-8"?>
2 <gresources>
3   <gresource prefix="/com/github/ToshioCP/menu3">
4     <file>menu3.ui</file>
5   </gresource>
6 </gresources>
~~~

GtkBuilder создаёт меню из ресурса.

~~~C
GtkBuilder *builder = gtk_builder_new_from_resource ("/com/github/ToshioCP/menu3/menu3.ui");
GMenuModel *menubar = G_MENU_MODEL (gtk_builder_get_object (builder, "menubar"));

gtk_application_set_menubar (GTK_APPLICATION (app), menubar);
g_object_unref (builder);
~~~

Экземпляр builder освобождается после того, как GMenuModel `menubar` добавляется в приложение.
Если вы сделаете это до вставки, произойдёт плохая вещь — ваш компьютер может зависнуть.
Это происходит потому, что вы не владеете экземпляром `menubar`.
Функция `gtk_builder_get_object` просто возвращает указатель на `menubar` и не увеличивает счётчик ссылок `menubar`.
Поэтому, если вы освободите `bulder` до `gtk_application_set_menubar`, `builder` будет уничтожен, а вместе с ним и `menubar`.

## Запись действия

Написание кода для создания действий и обработчиков сигналов также является утомительной работой.
Поэтому это следует автоматизировать.
Вы можете легко реализовать их с помощью структуры GActionEntry и функции `g_action_map_add_action_entries`.

GActionEntry содержит имя действия, обработчики сигналов, параметр и состояние.

~~~C
typedef struct _GActionEntry GActionEntry;

struct _GActionEntry
{
  /* action name */
  const char *name;
  /* activate handler */
  void (* activate) (GSimpleAction *action, GVariant *parameter, gpointer user_data);
  /* the type of the parameter given as a single GVariant type string */
  const char *parameter_type;
  /* initial state given in GVariant text format */
  const char *state;
  /* change-state handler */
  void (* change_state) (GSimpleAction *action, GVariant *value, gpointer user_data);
  /*< private >*/
  gsize padding[3];
};
~~~
Например, действия из предыдущего раздела:

~~~C
{ "fullscreen", NULL, NULL, "false", fullscreen_changed }
{ "color", color_activated, "s", "'red'", NULL }
{ "quit", quit_activated, NULL, NULL, NULL },
~~~

- Действие Fullscreen является статическим, но не имеет параметров.
Таким образом, третий элемент (тип параметра) — NULL.
[Текстовый формат GVariant](https://docs.gtk.org/glib/gvariant-text.html) предоставляет "true" и "false" как булевы значения GVariant.
Начальное состояние действия — false (четвёртый элемент).
Оно не имеет обработчика activate, поэтому второй элемент — NULL.
Вместо этого у него есть обработчик change-state.
Пятый элемент `fullscreen_changed` — это обработчик.
- Действие Color является статическим и имеет параметр.
Тип параметра — строка.
[Строки формата GVariant](https://docs.gtk.org/glib/gvariant-format-strings.html) предоставляют строковые форматы для представления типов GVariant.
Третий элемент "s" означает строковый тип GVariant.
Текстовый формат GVariant определяет, что строки окружаются одинарными или двойными кавычками.
Таким образом, строка red — это 'red' или "red".
Четвёртый элемент — `"'red'"`, что является форматом строки C, а строка — 'red'.
Вы можете написать `"\"red\""` вместо этого.
Второй элемент `color_activated` — обработчик activate.
Действие не имеет обработчика change-state, поэтому пятый элемент — NULL.
- Действие Quit не является статическим и не имеет параметра.
Таким образом, третий и четвёртый элементы — NULL.
Второй элемент `quit_activated` — обработчик activate.
Действие не имеет обработчика change-state, поэтому пятый элемент — NULL.

Функция `g_action_map_add_action_entries` выполняет всё необходимое
для создания экземпляров GSimpleAction и добавления их в GActionMap (приложение или окно).

~~~C
const GActionEntry app_entries[] = {
  { "color", color_activated, "s", "'red'", NULL },
  { "quit", quit_activated, NULL, NULL, NULL }
};
g_action_map_add_action_entries (G_ACTION_MAP (app), app_entries,
                                 G_N_ELEMENTS (app_entries), app);
~~~

Код выше выполняет:

- Создаёт действия "color" и "quit"
- Подключает действия и обработчики сигнала "activate" (`color_activated` и `quit_activated`).
- Добавляет действия в карту действий `app`.

То же самое относится к другому действию.

~~~C
const GActionEntry win_entries[] = {
  { "fullscreen", NULL, NULL, "false", fullscreen_changed }
};
g_action_map_add_action_entries (G_ACTION_MAP (win), win_entries,
                                 G_N_ELEMENTS (win_entries), win);
~~~
Код выше выполняет:

- Создаёт действие "fullscreen".
- Подключает действие и обработчик сигнала `fullscreen_changed`
- Его начальное состояние устанавливается в false.
- Добавляет действие в карту действий `win`.

## Пример

Исходные файлы: `menu3.c`, `menu3.ui`, `menu3.gresource.xml` и `meson.build`.
Они находятся в каталоге [src/menu3](../src/menu3).
Ниже приведены `menu3.c` и `meson.build`.

~~~C
  1 #include <gtk/gtk.h>
  2 
  3 static void
  4 new_activated (GSimpleAction *action, GVariant *parameter, gpointer user_data) {
  5 }
  6 
  7 static void
  8 open_activated (GSimpleAction *action, GVariant *parameter, gpointer user_data) {
  9 }
 10 
 11 static void
 12 save_activated (GSimpleAction *action, GVariant *parameter, gpointer user_data) {
 13 }
 14 
 15 static void
 16 saveas_activated (GSimpleAction *action, GVariant *parameter, gpointer user_data) {
 17 }
 18 
 19 static void
 20 close_activated (GSimpleAction *action, GVariant *parameter, gpointer user_data) {
 21   GtkWindow *win = GTK_WINDOW (user_data);
 22 
 23   gtk_window_destroy (win);
 24 }
 25 
 26 static void
 27 cut_activated (GSimpleAction *action, GVariant *parameter, gpointer user_data) {
 28 }
 29 
 30 static void
 31 copy_activated (GSimpleAction *action, GVariant *parameter, gpointer user_data) {
 32 }
 33 
 34 static void
 35 paste_activated (GSimpleAction *action, GVariant *parameter, gpointer user_data) {
 36 }
 37 
 38 static void
 39 selectall_activated (GSimpleAction *action, GVariant *parameter, gpointer user_data) {
 40 }
 41 
 42 static void
 43 fullscreen_changed (GSimpleAction *action, GVariant *state, gpointer user_data) {
 44   GtkWindow *win = GTK_WINDOW (user_data);
 45 
 46   if (g_variant_get_boolean (state))
 47     gtk_window_maximize (win);
 48   else
 49     gtk_window_unmaximize (win);
 50   g_simple_action_set_state (action, state);
 51 }
 52 
 53 static void
 54 quit_activated (GSimpleAction *action, GVariant *parameter, gpointer user_data) {
 55   GApplication *app = G_APPLICATION (user_data);
 56 
 57   g_application_quit (app);
 58 }
 59 
 60 static void
 61 app_activate (GApplication *app) {
 62   GtkWidget *win = gtk_application_window_new (GTK_APPLICATION (app));
 63 
 64   const GActionEntry win_entries[] = {
 65     { "save", save_activated, NULL, NULL, NULL },
 66     { "saveas", saveas_activated, NULL, NULL, NULL },
 67     { "close", close_activated, NULL, NULL, NULL },
 68     { "fullscreen", NULL, NULL, "false", fullscreen_changed }
 69   };
 70   g_action_map_add_action_entries (G_ACTION_MAP (win), win_entries, G_N_ELEMENTS (win_entries), win);
 71 
 72   gtk_application_window_set_show_menubar (GTK_APPLICATION_WINDOW (win), TRUE);
 73 
 74   gtk_window_set_title (GTK_WINDOW (win), "menu3");
 75   gtk_window_set_default_size (GTK_WINDOW (win), 400, 300);
 76   gtk_window_present (GTK_WINDOW (win));
 77 }
 78 
 79 static void
 80 app_startup (GApplication *app) {
 81   GtkBuilder *builder = gtk_builder_new_from_resource ("/com/github/ToshioCP/menu3/menu3.ui");
 82   GMenuModel *menubar = G_MENU_MODEL (gtk_builder_get_object (builder, "menubar"));
 83 
 84   gtk_application_set_menubar (GTK_APPLICATION (app), menubar);
 85   g_object_unref (builder);
 86 
 87   const GActionEntry app_entries[] = {
 88     { "new", new_activated, NULL, NULL, NULL },
 89     { "open", open_activated, NULL, NULL, NULL },
 90     { "cut", cut_activated, NULL, NULL, NULL },
 91     { "copy", copy_activated, NULL, NULL, NULL },
 92     { "paste", paste_activated, NULL, NULL, NULL },
 93     { "selectall", selectall_activated, NULL, NULL, NULL },
 94     { "quit", quit_activated, NULL, NULL, NULL }
 95   };
 96   g_action_map_add_action_entries (G_ACTION_MAP (app), app_entries, G_N_ELEMENTS (app_entries), app);
 97 }
 98 
 99 #define APPLICATION_ID "com.github.ToshioCP.menu3"
100 
101 int
102 main (int argc, char **argv) {
103   GtkApplication *app;
104   int stat;
105 
106   app = gtk_application_new (APPLICATION_ID, G_APPLICATION_DEFAULT_FLAGS);
107   g_signal_connect (app, "startup", G_CALLBACK (app_startup), NULL);
108   g_signal_connect (app, "activate", G_CALLBACK (app_activate), NULL);
109 
110   stat = g_application_run (G_APPLICATION (app), argc, argv);
111   g_object_unref (app);
112   return stat;
113 }
114 
~~~

meson.build

~~~meson
 1 project('menu3', 'c')
 2 
 3 gtkdep = dependency('gtk4')
 4 
 5 gnome=import('gnome')
 6 resources = gnome.compile_resources('resources','menu3.gresource.xml')
 7 
 8 sourcefiles=files('menu3.c')
 9 
10 executable('menu3', sourcefiles, resources, dependencies: gtkdep)
~~~

Обработчики действий должны следовать следующему формату.

~~~C
static void
handler (GSimpleAction *action_name, GVariant *parameter, gpointer user_data) { ... ... ... }
~~~

Вы не можете написать, например, "GApplication *app" вместо "gpointer user_data".
Потому что `g_action_map_add_action_entries` ожидает, что обработчики следуют формату выше.

В каталоге `menu` находятся `menu2_ui.c` и `menu2.ui`.
Это другие примеры, демонстрирующие ui-файл меню и `g_action_map_add_action_entries`.
Он включает статическое действие с параметрами.

~~~xml
<item>
  <attribute name="label">Red</attribute>
  <attribute name="action">app.color</attribute>
  <attribute name="target">red</attribute>
</item>
~~~

Имя действия и цель разделены таким образом.
Атрибут action включает только префикс и имя.
Вы не можете написать `<attribute name="action">app.color::red</attribute>`.

Up: [README.md](../README.md),  Prev: [Section 18](sec18.md), Next: [Section 20](sec20.md)
