Up: [README.md](../README.md),  Prev: [Section 17](sec17.md), Next: [Section 19](sec19.md)

# Действие с состоянием

Некоторые действия имеют состояния.
Типичные значения состояний — это булевы или строковые значения.
Однако, при желании возможны и другие типы состояний.

Действия, которые имеют состояния, называются действиями с состоянием (stateful).

## Действие с состоянием без параметра

Некоторые меню называются переключаемыми (toggle menu).
Например, меню полноэкранного режима имеет состояние с двумя значениями -- полноэкранный и неполноэкранный режим.
Значение состояния изменяется каждый раз при нажатии на меню.
Действие, соответствующее меню полноэкранного режима, также имеет состояние.
Его значение — TRUE или FALSE, и это называется булевым значением.
TRUE соответствует полноэкранному режиму, а FALSE — неполноэкранному.

Ниже приведён пример кода для реализации меню полноэкранного режима, за исключением обработчика сигнала.
Обработчик сигнала будет показан позже.

~~~C
GSimpleAction *act_fullscreen = g_simple_action_new_stateful ("fullscreen",
                                NULL, g_variant_new_boolean (FALSE));
g_signal_connect (act_fullscreen, "change-state", G_CALLBACK (fullscreen_changed), win);
g_action_map_add_action (G_ACTION_MAP (win), G_ACTION (act_fullscreen));
... ... ...
GMenuItem *menu_item_fullscreen = g_menu_item_new ("Full Screen", "win.fullscreen");
~~~

- `act_fullscreen` — это экземпляр GSimpleAction.
Он создаётся с помощью `g_simple_action_new_stateful`.
Функция имеет три аргумента.
Первый аргумент "fullscreen" — это имя действия.
Второй аргумент — это тип параметра.
`NULL` означает, что действие не имеет параметра.
Третий аргумент — это начальное состояние действия.
Это значение GVariant.
GVariant будет объяснён в следующем подразделе.
Функция `g_variant_new_boolean (FALSE)` возвращает значение GVariant булевого типа, которое равно `FALSE`.
Если есть два или более окон верхнего уровня, каждое окно имеет собственное действие `act_fullscreen`.
Таким образом, количество действий равно количеству окон.
- Действие `act_fullscreen` имеет сигнал "change-state". Сигнал подключён к обработчику `fullscreen_changed`.
Если кликнуть на меню полноэкранного режима, то соответствующее действие `act_fullscreen` активируется.
Но ни один обработчик не подключён к сигналу "activate".
Тогда поведение по умолчанию для действий с булевым состоянием и типом параметра NULL, таких как `act_fullscreen`, заключается в переключении их через сигнал "change-state".
- Действие добавляется к GtkWindow `win`.
Следовательно, область видимости действия — "win".
- `menu_item_fullscreen` — это экземпляр GMenuItem.
Есть два аргумента.
Первый аргумент "Full Screen" — это метка `menu_item_fullscreen`.
Второй аргумент — это действие.
Действие "win.fullscreen" имеет префикс "win" и имя действия "fullscreen".
Префикс указывает, что действие принадлежит окну.

~~~C
1 static void
2 fullscreen_changed(GSimpleAction *action, GVariant *value, GtkWindow *win) {
3   if (g_variant_get_boolean (value))
4     gtk_window_maximize (win);
5   else
6     gtk_window_unmaximize (win);
7   g_simple_action_set_state (action, value);
8 }
~~~

- Обработчик `fullscreen_changed` имеет три параметра.
Первый параметр — это действие, которое испускает сигнал "change-state".
Второй параметр — это значение нового состояния действия.
Третий параметр — это пользовательские данные, которые установлены в `g_signal_connect`.
- Если значение имеет булев тип и равно `TRUE`, то окно разворачивается на весь экран.
В противном случае возвращается к обычному размеру.
- Устанавливает состояние действия с помощью `value`.
Примечание: На этом этапе, то есть на этапе до вызова `g_simple_action_set_state`, состояние действия всё ещё имеет исходное значение.
Поэтому вам нужно установить состояние с новым значением с помощью `g_simple_action_set_state`.

Вы можете использовать сигнал "activate" вместо сигнала "change-state", или оба сигнала.
Но вышеописанный способ является самым простым и лучшим.

### GVariant

GVariant — это фундаментальный тип.
Он не является дочерним по отношению к GObject.
GVariant может содержать булевы значения, строки или значения других типов.
Например, следующая программа присваивает TRUE переменной `value`, тип которой — GVariant.

~~~C
GVariant *value = g_variant_new_boolean (TRUE);
~~~

Другой пример:

~~~C
GVariant *value2 = g_variant_new_string ("Hello");
~~~

`value2` — это GVariant, который имеет строковое значение "Hello".
GVariant может содержать другие типы, такие как int16, int32, int64, double и так далее.

Если вы хотите получить исходное значение, используйте функции серии g\_variant\_get.
Например, вы можете получить булево значение с помощью g\_variant_get_boolean.

~~~C
gboolean bool = g_variant_get_boolean (value);
~~~

Поскольку `value` был создан как GVariant булевого типа со значением `TRUE`, `bool` равно `TRUE`.
Точно так же вы можете получить строку из `value2`

~~~C
const char *str = g_variant_get_string (value2, NULL);
~~~

Второй параметр — это указатель на переменную типа gsize (gsize определён как unsigned long).
Если он не равен NULL, то функция использует указанное значение как длину.
Если он равен NULL, ничего не происходит.
Возвращаемая строка `str` принадлежит экземпляру и не может быть изменена или освобождена вызывающей стороной.

## Действие с состоянием и параметром

Другой пример действий с состоянием — это действие, соответствующее меню выбора цвета.
Например, есть три меню, и каждое меню имеет красный, зелёный или синий цвет соответственно.
Они определяют цвет фона виджета GtkLabel.
Одно действие подключено к трём меню.
Действие имеет состояние, значение которого — "red", "green" или "blue".
Значения являются строками.
Эти цвета передаются обработчику сигнала в качестве параметра.

~~~C
... ... ...
GVariantType *vtype = g_variant_type_new("s");
GSimpleAction *act_color
  = g_simple_action_new_stateful ("color", vtype, g_variant_new_string ("red"));
g_variant_type_free (vtype);
GMenuItem *menu_item_red = g_menu_item_new ("Red", "app.color::red");
GMenuItem *menu_item_green = g_menu_item_new ("Green", "app.color::green");
GMenuItem *menu_item_blue = g_menu_item_new ("Blue", "app.color::blue");
g_signal_connect (act_color, "activate", G_CALLBACK (color_activated), NULL);
... ... ...
~~~

- GVariantType — это структура C, которая хранит тип GVariant.
Она создаётся с помощью функции `g_variant_type_new`.
Аргументом функции является строка типа GVariant.
Таким образом, `g_variant_type_new("s")` возвращает структуру GVariantType, содержащую строковый тип.
Возвращаемое значение, структура GVariantType, принадлежит вызывающей стороне.
Поэтому вам нужно освободить её, когда она станет бесполезной.
- Переменная `act_color` указывает на экземпляр GSimpleAction.
Он создаётся с помощью `g_simple_action_new_stateful`.
Функция имеет три аргумента.
Первый аргумент "color" — это имя действия.
Второй аргумент — это тип параметра, который является GVariantType.
`g_variant_type_new("s")` создаёт GVariantType, который является строковым типом (`G_VARIANT_TYPE_STRING`).
Третий аргумент — это начальное состояние действия.
Это GVariant.
Функция `g_variant_new_string ("red")` возвращает значение GVariant, которое имеет строковое значение "red".
GVariant имеет счётчик ссылок, и функции серии `g_variant_new_...` возвращают значение GVariant с плавающей ссылкой.
Это означает, что вызывающая сторона не владеет значением на этом этапе.
А функция `g_simple_action_new_stateful` поглощает плавающую ссылку, поэтому вам не нужно заботиться об освобождении экземпляра GVariant.
- Структура GVariantType `vtype` становится бесполезной после `g_simple_action_new_stateful`.
Она освобождается с помощью функции `g_variant_type_free`.
- Переменная `menu_item_red` указывает на экземпляр GMenuItem.
Функция `g_menu_item_new` имеет два аргумента.
Первый аргумент "Red" — это метка `menu_item_red`.
Второй аргумент — это детализированное действие.
Его префикс — "app", имя действия — "color", а цель — "red".
Цель отправляется действию в качестве параметра.
То же самое касается `menu_item_green` и `menu_item_blue`.
- Функция `g_signal_connect` подключает сигнал activate действия `act_color` к обработчику `color_activated`.
Если кликнуть на одно из трёх меню, то действие `act_color` активируется с целью (параметром), которая задаётся меню.

Ниже приведён обработчик сигнала "activate".

~~~C
static void
color_activated(GSimpleAction *action, GVariant *parameter) {
  char *color = g_strdup_printf ("label.lb {background-color: %s;}",
                                   g_variant_get_string (parameter, NULL));
  gtk_css_provider_load_from_data (provider, color, -1);
  g_free (color);
  g_action_change_state (G_ACTION (action), parameter);
}
~~~

- Обработчик изначально имеет три параметра.
Третий параметр — это пользовательские данные, установленные в функции `g_signal_connect`.
Но он опущен, потому что четвёртый аргумент `g_signal_connect` был NULL.
Первый параметр — это действие, которое испускает сигнал "activate".
Второй параметр — это параметр, или цель, переданная действию.
Это цвет, указанный меню.
- Переменная `color` — это CSS-строка, созданная с помощью `g_strdup_printf`.
Аргументы `g_strdup_printf` такие же, как у стандартной функции C printf.
Функция `g_variant_get_string` получает строку, содержащуюся в `parameter`.
Строка принадлежит экземпляру, и вы не должны изменять или освобождать её.
Строка `label.lb` — это селектор.
Он состоит из `label`, имени узла GtkLabel, и `lb`, которое является именем класса.
Он выбирает GtkLabel, который имеет класс `lb`.
Например, меню имеют GtkLabel для отображения своих меток, но у них нет класса `lb`.
Поэтому CSS не изменяет цвет их фона.
Строка `{background-color %s}` устанавливает цвет фона `%s`, которому присваивается цвет из `parameter`.
- Освобождает строку `color`.
- Изменяет состояние с помощью `g_action_change_state`.

Примечание: Если вы не установили обработчик сигнала "activate", сигнал перенаправляется на сигнал "change-state".
Таким образом, вы можете использовать сигнал "change-state" вместо сигнала "activate".
См. [`src/menu/menu2_change_state.c`](../src/menu/menu2_change_state.c).

### GVariantType

GVariantType задаёт тип GVariant.
GVariantType создаётся со строкой типа.

- "b" означает булев тип.
- "s" означает строковый тип.

Следующая программа — это простой пример.
В конце она выводит строку "s".

~~~C
1 #include <glib.h>
2 
3 int
4 main (int argc, char **argv) {
5   GVariantType *vtype = g_variant_type_new ("s");
6   const char *type_string = g_variant_type_peek_string (vtype);
7   g_print ("%s\n",type_string);
8   g_variant_type_free (vtype);
9 }
~~~

- Функция `g_variant_type_new` создаёт структуру GVariantType.
Аргумент "s" — это строка типа.
Он означает строку.
Возвращаемая структура принадлежит вызывающей стороне.
Когда она станет бесполезной, вам нужно освободить её с помощью функции `g_variant_type_free`.
- Функция `g_variant_type_peek_string` заглядывает в `vtype`.
Это строка "s", которая была передана `vtype` при его создании.
Строка принадлежит экземпляру, и вызывающая сторона не может изменить или освободить её.
- Выводит строку в терминал.
Вы не можете освободить `vtype` до `g_print`, потому что строка `type_string` принадлежит `vtype`.
- Освобождает `vtype`.

## Пример

Следующий код включает описанные выше действия с состоянием.
Эта программа имеет меню вида:

![menu2](../image/menu2.png)

- Меню Fullscreen переключает размер окна между максимальным и обычным.
Если окно имеет максимальный размер, что называется полным экраном, то перед меткой "fullscreen" ставится галочка.
- Меню Red, Green и Blue определяют цвет фона метки в окне.
Меню имеют переключатели слева от меню.
И переключатель выбранного меню включается.
- Меню Quit завершает работу приложения.

Код выглядит следующим образом.

~~~C
  1 #include <gtk/gtk.h>
  2 
  3 /* The provider below provides application wide CSS data. */
  4 GtkCssProvider *provider;
  5 
  6 static void
  7 fullscreen_changed(GSimpleAction *action, GVariant *value, GtkWindow *win) {
  8   if (g_variant_get_boolean (value))
  9     gtk_window_maximize (win);
 10   else
 11     gtk_window_unmaximize (win);
 12   g_simple_action_set_state (action, value);
 13 }
 14 
 15 static void
 16 color_activated(GSimpleAction *action, GVariant *parameter) {
 17   char *color = g_strdup_printf ("label.lb {background-color: %s;}", g_variant_get_string (parameter, NULL));
 18   /* Change the CSS data in the provider. */
 19   /* Previous data is thrown away. */
 20   gtk_css_provider_load_from_data (provider, color, -1);
 21   g_free (color);
 22   g_action_change_state (G_ACTION (action), parameter);
 23 }
 24 
 25 static void
 26 app_shutdown (GApplication *app, GtkCssProvider *provider) {
 27   gtk_style_context_remove_provider_for_display (gdk_display_get_default(), GTK_STYLE_PROVIDER (provider));
 28 }
 29 
 30 static void
 31 app_activate (GApplication *app) {
 32   GtkWindow *win = GTK_WINDOW (gtk_application_window_new (GTK_APPLICATION (app)));
 33   gtk_window_set_title (win, "menu2");
 34   gtk_window_set_default_size (win, 400, 300);
 35 
 36   GtkWidget *lb = gtk_label_new (NULL);
 37   gtk_widget_add_css_class (lb, "lb"); /* the class is used by CSS Selector */
 38   gtk_window_set_child (win, lb);
 39 
 40   GSimpleAction *act_fullscreen
 41     = g_simple_action_new_stateful ("fullscreen", NULL, g_variant_new_boolean (FALSE));
 42   g_signal_connect (act_fullscreen, "change-state", G_CALLBACK (fullscreen_changed), win);
 43   g_action_map_add_action (G_ACTION_MAP (win), G_ACTION (act_fullscreen));
 44 
 45   gtk_application_window_set_show_menubar (GTK_APPLICATION_WINDOW (win), TRUE);
 46 
 47   gtk_window_present (win);
 48 }
 49 
 50 static void
 51 app_startup (GApplication *app) {
 52   GVariantType *vtype = g_variant_type_new("s");
 53   GSimpleAction *act_color
 54     = g_simple_action_new_stateful ("color", vtype, g_variant_new_string ("red"));
 55   g_variant_type_free (vtype);
 56   GSimpleAction *act_quit
 57     = g_simple_action_new ("quit", NULL);
 58   g_signal_connect (act_color, "activate", G_CALLBACK (color_activated), NULL);
 59   g_signal_connect_swapped (act_quit, "activate", G_CALLBACK (g_application_quit), app);
 60   g_action_map_add_action (G_ACTION_MAP (app), G_ACTION (act_color));
 61   g_action_map_add_action (G_ACTION_MAP (app), G_ACTION (act_quit));
 62 
 63   GMenu *menubar = g_menu_new ();
 64   GMenu *menu = g_menu_new ();
 65   GMenu *section1 = g_menu_new ();
 66   GMenu *section2 = g_menu_new ();
 67   GMenu *section3 = g_menu_new ();
 68   GMenuItem *menu_item_fullscreen = g_menu_item_new ("Full Screen", "win.fullscreen");
 69   GMenuItem *menu_item_red = g_menu_item_new ("Red", "app.color::red");
 70   GMenuItem *menu_item_green = g_menu_item_new ("Green", "app.color::green");
 71   GMenuItem *menu_item_blue = g_menu_item_new ("Blue", "app.color::blue");
 72   GMenuItem *menu_item_quit = g_menu_item_new ("Quit", "app.quit");
 73 
 74   g_menu_append_item (section1, menu_item_fullscreen);
 75   g_menu_append_item (section2, menu_item_red);
 76   g_menu_append_item (section2, menu_item_green);
 77   g_menu_append_item (section2, menu_item_blue);
 78   g_menu_append_item (section3, menu_item_quit);
 79   g_object_unref (menu_item_red);
 80   g_object_unref (menu_item_green);
 81   g_object_unref (menu_item_blue);
 82   g_object_unref (menu_item_fullscreen);
 83   g_object_unref (menu_item_quit);
 84 
 85   g_menu_append_section (menu, NULL, G_MENU_MODEL (section1));
 86   g_menu_append_section (menu, "Color", G_MENU_MODEL (section2));
 87   g_menu_append_section (menu, NULL, G_MENU_MODEL (section3));
 88   g_menu_append_submenu (menubar, "Menu", G_MENU_MODEL (menu));
 89   g_object_unref (section1);
 90   g_object_unref (section2);
 91   g_object_unref (section3);
 92   g_object_unref (menu);
 93 
 94   gtk_application_set_menubar (GTK_APPLICATION (app), G_MENU_MODEL (menubar));
 95 
 96   provider = gtk_css_provider_new ();
 97   /* Initialize the css data */
 98   gtk_css_provider_load_from_data (provider, "label.lb {background-color: red;}", -1);
 99   /* Add CSS to the default GdkDisplay. */
100   gtk_style_context_add_provider_for_display (gdk_display_get_default (),
101         GTK_STYLE_PROVIDER (provider), GTK_STYLE_PROVIDER_PRIORITY_APPLICATION);
102   g_signal_connect (app, "shutdown", G_CALLBACK (app_shutdown), provider);
103   g_object_unref (provider); /* release provider, but it's still alive because the display owns it */
104 }
105 
106 #define APPLICATION_ID "com.github.ToshioCP.menu2"
107 
108 int
109 main (int argc, char **argv) {
110   GtkApplication *app;
111   int stat;
112 
113   app = gtk_application_new (APPLICATION_ID, G_APPLICATION_DEFAULT_FLAGS);
114   g_signal_connect (app, "startup", G_CALLBACK (app_startup), NULL);
115   g_signal_connect (app, "activate", G_CALLBACK (app_activate), NULL);
116 
117   stat = g_application_run (G_APPLICATION (app), argc, argv);
118   g_object_unref (app);
119   return stat;
120 }
~~~

- 6-23: Обработчики сигналов действий.
- 25-28: Обработчик `app_shutdown` вызывается при завершении работы приложения.
Он удаляет провайдер из дисплея.
- 30-48: Обработчик сигнала activate.
- 32-34: Создаётся новое окно и присваивается `win`.
Его заголовок и размер по умолчанию устанавливаются на "menu2" и 400x300 соответственно.
- 36-38: Создаётся новая метка и присваивается `lb`
Метке присваивается CSS-класс "lb".
Она добавляется к `win` в качестве дочернего элемента.
- 40-43: Создаётся переключаемое действие и присваивается `act_fullscreen`.
Оно подключается к обработчику сигнала `fullscreen_changed`.
Оно добавляется к окну, поэтому область видимости действия — "win".
Таким образом, если есть два или более окон, создаётся два или более действий.
- 45: Функция `gtk_application_window_set_show_menubar` добавляет панель меню к окну.
- 47: Показывает окно.
- 50-104: Обработчик сигнала startup.
- 52-61: Создаются два действия `act_color` и `act_quit`.
Эти действия существуют только в одном экземпляре, потому что обработчик startup вызывается один раз.
Они подключаются к своим обработчикам и добавляются к приложению.
Их области видимости — "app".
- 63-92: Создаются меню.
- 94: Панель меню добавляется к приложению.
- 96-103: Создаётся CSS-провайдер с CSS-данными и добавляется к дисплею по умолчанию.
Сигнал "shutdown" приложения подключается к обработчику "app_shutdown".
Таким образом, провайдер удаляется из дисплея и освобождается при завершении работы приложения.

## Компиляция

Измените текущий каталог на `src/menu`.

~~~
$ comp menu2
$./a.out
~~~

Затем вы увидите окно, и цвет фона содержимого будет красным.
Вы можете изменить размер на максимальный и вернуться к исходному размеру.
Вы можете изменить цвет фона на зелёный или синий.

Если вы запустите второе приложение во время работы первого приложения, появится ещё одно окно на том же экране.
Оба окна будут иметь одинаковый цвет фона.
Потому что действие `act_color` имеет область видимости "app", и CSS применяется к дисплею по умолчанию, общему для окон.

```
$ ./a.out & # Запуск первого приложения
[1] 82113
$ ./a.out # Запуск второго приложения
$ 
```
Up: [README.md](../README.md),  Prev: [Section 17](sec17.md), Next: [Section 19](sec19.md)
