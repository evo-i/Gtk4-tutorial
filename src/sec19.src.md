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
Она находится в каталоге [src/menu3](menu3).

![menu3](../image/menu3.png){width=6.0cm height=5.055cm}

Ниже приведён ui-файл для `menu3`.

@@@include
menu3/menu3.ui
@@@

UI-файл преобразуется в ресурс компилятором ресурсов `glib-compile-resouces` с помощью xml-файла.

@@@include
menu3/menu3.gresource.xml
@@@

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
Они находятся в каталоге [src/menu3](menu3).
Ниже приведены `menu3.c` и `meson.build`.

@@@include
menu3/menu3.c
@@@

meson.build

@@@include
menu3/meson.build
@@@

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
