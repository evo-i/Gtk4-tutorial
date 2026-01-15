Up: [README.md](../README.md),  Prev: [Section 22](sec22.md), Next: [Section 24](sec24.md)

# Pango, CSS и приложение

## PangoFontDescription

PangoFontDescription — это структура C для шрифта.
Вы можете получить семейство шрифта, стиль, насыщенность и размер.
Вы также можете получить строку, которая включает атрибуты шрифта.
Например, предположим, что PangoFontDescription содержит шрифт "Noto Sans Mono", "Bold", "Italic" и размер 12 пунктов.
Тогда строка, преобразованная из PangoFontDescription, будет "Noto Sans Mono Bold Italic 12".

- Семейство шрифта — "Noto Sans Mono".
- Стиль шрифта — "Italic".
- Насыщенность шрифта — "Bold", или 700.
- Размер шрифта — 12 pt.

Шрифт в CSS отличается от строки из PangoFontDescription.

- `font: bold italic 12pt "Noto Sans Mono"`
- `Noto Sans Mono Bold Italic 12`

Поэтому может быть проще использовать каждое свойство отдельно, т.е. font-family, font-style, font-weight и font-size, для преобразования данных PangoFontDescription в CSS.

Обратитесь к
[документации Pango](https://docs.gtk.org/Pango/index.html)
и [W3C CSS Fonts Module Level 3](https://www.w3.org/TR/css-fonts-3/) для получения дополнительной информации.

## Конвертер из PangoFontDescription в CSS

Два файла `pfd2css.h` и `pfd2css.c` содержат конвертер из PangoFontDescription в CSS.

~~~C
 1 #pragma once
 2 
 3 #include <pango/pango.h>
 4 
 5 // Pango font description to CSS style string
 6 // Returned string is owned by the caller. The caller should free it when it becomes useless.
 7 
 8 char*
 9 pfd2css (PangoFontDescription *pango_font_desc);
10 
11 // Each element (family, style, weight and size)
12 
13 const char*
14 pfd2css_family (PangoFontDescription *pango_font_desc);
15 
16 const char*
17 pfd2css_style (PangoFontDescription *pango_font_desc);
18 
19 int
20 pfd2css_weight (PangoFontDescription *pango_font_desc);
21 
22 // Returned string is owned by the caller. The caller should free it when it becomes useless.
23 char *
24 pfd2css_size (PangoFontDescription *pango_font_desc);
~~~

Пять функций являются публичными.
Первая функция — это удобная функция для установки четырех других CSS-свойств одновременно.

~~~C
 1 #include <pango/pango.h>
 2 #include "pfd2css.h"
 3 
 4 // Pango font description to CSS style string
 5 // Returned string is owned by caller. The caller should free it when it is useless.
 6 
 7 char*
 8 pfd2css (PangoFontDescription *pango_font_desc) {
 9   char *fontsize;
10 
11   fontsize = pfd2css_size (pango_font_desc);
12   return g_strdup_printf ("font-family: \"%s\"; font-style: %s; font-weight: %d; font-size: %s;",
13               pfd2css_family (pango_font_desc), pfd2css_style (pango_font_desc),
14               pfd2css_weight (pango_font_desc), fontsize);
15   g_free (fontsize); 
16 }
17 
18 // Each element (family, style, weight and size)
19 
20 const char*
21 pfd2css_family (PangoFontDescription *pango_font_desc) {
22   return pango_font_description_get_family (pango_font_desc);
23 }
24 
25 const char*
26 pfd2css_style (PangoFontDescription *pango_font_desc) {
27   PangoStyle pango_style = pango_font_description_get_style (pango_font_desc);
28   switch (pango_style) {
29   case PANGO_STYLE_NORMAL:
30     return "normal";
31   case PANGO_STYLE_ITALIC:
32     return "italic";
33   case PANGO_STYLE_OBLIQUE:
34     return "oblique";
35   default:
36     return "normal";
37   }
38 }
39 
40 int
41 pfd2css_weight (PangoFontDescription *pango_font_desc) {
42   PangoWeight pango_weight = pango_font_description_get_weight (pango_font_desc);
43   switch (pango_weight) {
44   case PANGO_WEIGHT_THIN:
45     return 100;
46   case PANGO_WEIGHT_ULTRALIGHT:
47     return 200;
48   case PANGO_WEIGHT_LIGHT:
49     return 300;
50   case PANGO_WEIGHT_SEMILIGHT:
51     return 350;
52   case PANGO_WEIGHT_BOOK:
53     return 380;
54   case PANGO_WEIGHT_NORMAL:
55     return 400; /* or "normal" */
56   case PANGO_WEIGHT_MEDIUM:
57     return 500;
58   case PANGO_WEIGHT_SEMIBOLD:
59     return 600;
60   case PANGO_WEIGHT_BOLD:
61     return 700; /* or "bold" */
62   case PANGO_WEIGHT_ULTRABOLD:
63     return 800;
64   case PANGO_WEIGHT_HEAVY:
65     return 900;
66   case PANGO_WEIGHT_ULTRAHEAVY:
67     return 900; /* 1000 is available since CSS Fonts level 4 but GTK currently supports level 3. */
68   default:
69     return 400; /* "normal" */
70   }
71 }
72 
73 char *
74 pfd2css_size (PangoFontDescription *pango_font_desc) {
75   if (pango_font_description_get_size_is_absolute (pango_font_desc))
76     return g_strdup_printf ("%dpx", pango_font_description_get_size (pango_font_desc) / PANGO_SCALE);
77   else
78     return g_strdup_printf ("%dpt", pango_font_description_get_size (pango_font_desc) / PANGO_SCALE);
79 }
~~~

- Функция `pfd2css_family` возвращает семейство шрифта.
- Функция `pfd2css_style` возвращает стиль шрифта, который является одним из "normal", "italic" или "oblique".
- Функция `pfd2css_weight` возвращает насыщенность шрифта в виде целого числа. См. список ниже.
- Функция `pfd2css_size` возвращает размер шрифта.
  - Если размер описания шрифта абсолютный, она возвращает размер в единицах устройства, то есть в пикселях. В противном случае единицей измерения является пункт.
  - Функция `pango_font_description_get_size` возвращает целое число размера, но оно умножено на `PANGO_SCALE`.
Поэтому вам нужно разделить его на `PANGO_SCALE`.
`PANGO_SCALE` в настоящее время равен 1024, но это может быть изменено в будущем.
Если размер шрифта 12pt, размер в pango составляет `12*PANGO_SCALE=12*1024=12288`.
- Функция `pfd2css` возвращает строку шрифта.
Например, если задан шрифт "Noto Sans Mono Bold Italic 12",
она возвращает "font-family: Noto Sans Mono; font-style: italic; font-weight: 700; font-size: 12pt;".

Число насыщенности шрифта может быть одним из:

- 100 - Thin (Тонкий)
- 200 - Extra Light (Ultra Light) (Сверхлегкий)
- 300 - Light (Легкий)
- 400 - Normal (Нормальный)
- 500 - Medium (Средний)
- 600 - Semi Bold (Demi Bold) (Полужирный)
- 700 - Bold (Жирный)
- 800 - Extra Bold (Ultra Bold) (Сверхжирный)
- 900 - Black (Heavy) (Черный)

## Объект приложения

### Класс TfeApplication

Класс TfeApplication является дочерним классом GtkApplication.
Он имеет несколько переменных экземпляра.
Заголовочный файл определяет макрос типа и публичную функцию.

~~~C
1 #pragma once
2 
3 #include <gtk/gtk.h>
4 
5 #define TFE_TYPE_APPLICATION tfe_application_get_type ()
6 G_DECLARE_FINAL_TYPE (TfeApplication, tfe_application, TFE, APPLICATION, GtkApplication)
7 
8 TfeApplication *
9 tfe_application_new (const char* application_id, GApplicationFlags flag);
~~~

Следующий код извлечен из `tfeapplication.c`.
Он создает класс и экземпляр TfeApplication.

```C
#include <gtk/gtk.h>
#include "tfeapplication.h"

struct _TfeApplication {
  GtkApplication parent;
  TfeWindow *win;
  GSettings *settings;
  GtkCssProvider *provider;
};

G_DEFINE_FINAL_TYPE (TfeApplication, tfe_application, GTK_TYPE_APPLICATION)

static void
tfe_application_dispose (GObject *gobject) {
  TfeApplication *app = TFE_APPLICATION (gobject);

  g_clear_object (&app->settings);
  g_clear_object (&app->provider);
  G_OBJECT_CLASS (tfe_application_parent_class)->dispose (gobject);
}

static void
tfe_application_init (TfeApplication *app) {
  app->settings = g_settings_new ("com.github.ToshioCP.tfe");
  g_signal_connect (app->settings, "changed::font-desc", G_CALLBACK (changed_font_cb), app);
  app->provider = gtk_css_provider_new ();
}

static void
tfe_application_class_init (TfeApplicationClass *class) {
  G_OBJECT_CLASS (class)->dispose = tfe_application_dispose;
  G_APPLICATION_CLASS (class)->startup = app_startup;
  G_APPLICATION_CLASS (class)->activate = app_activate;
  G_APPLICATION_CLASS (class)->open = app_open;
}

TfeApplication *
tfe_application_new (const char* application_id, GApplicationFlags flag) {
  return TFE_APPLICATION (g_object_new (TFE_TYPE_APPLICATION, "application-id", application_id, "flags", flag, NULL));
}
```

- Определена структура `_TfeApplication`.
Она имеет четыре члена.
Один из них — родительский объект, остальные являются переменными экземпляра.
Члены обычно инициализируются в функции инициализации экземпляра.
И они освобождаются в функции удаления или очищаются в функции финализации.
Члены:
  - win: экземпляр главного окна
  - settings: экземпляр GSettings, привязанный к элементу "font-desc" в GSettings
  - provider: провайдер для шрифта текстового представления.
- Макрос `G_DEFINE_FINAL_TYPE` определяет функцию `tfe_application_get_type` и некоторые другие полезные вещи.
- Функция `tfe_application_class_init` инициализирует класс TfeApplication.
Она переопределяет четыре метода класса.
Три метода класса `startup`, `activate` и `open` указывают на обработчики сигналов по умолчанию.
Переопределение изменяет обработчики по умолчанию.
Вы можете подключить обработчики с помощью макроса `g_signal_connect`, но результат будет другим.
Макрос подключает пользовательский обработчик к сигналу.
Обработчик по умолчанию все еще существует и не изменяется.
- Функция `tfe_application_init` инициализирует экземпляр.
  - Создает новый экземпляр GSettings и устанавливает `app->settings` на него. Затем подключает обработчик `changed_font_cb` к сигналу "changed::font-desc".
  - Создает новый экземпляр GtkCssProvider и устанавливает `app->provider` на него.
- Функция `tfe_application_dispose` освобождает экземпляры GSettings и GtkCssProvider.
Затем вызывает обработчик dispose родителя. Это называется "цепочкой вызовов вверх" (chaining up).
См. [документацию GObject](https://docs.gtk.org/gobject/tutorial.html#chaining-up).

### Обработчики сигнала startup

~~~C
 1 static void
 2 app_startup (GApplication *application) {
 3   TfeApplication *app = TFE_APPLICATION (application);
 4   int i;
 5   GtkCssProvider *provider = gtk_css_provider_new ();
 6   GdkDisplay *display;
 7 
 8   G_APPLICATION_CLASS (tfe_application_parent_class)->startup (application);
 9 
10   app->win = TFE_WINDOW (tfe_window_new (GTK_APPLICATION (app)));
11 
12   gtk_css_provider_load_from_data (provider, "textview {padding: 10px;}", -1);
13   display = gdk_display_get_default ();
14   gtk_style_context_add_provider_for_display (display, GTK_STYLE_PROVIDER (provider),
15                                               GTK_STYLE_PROVIDER_PRIORITY_APPLICATION);
16   g_object_unref (provider);
17   gtk_style_context_add_provider_for_display (display, GTK_STYLE_PROVIDER (app->provider),
18                                               GTK_STYLE_PROVIDER_PRIORITY_APPLICATION);
19 
20   changed_font_cb (app->settings, "font-desc", app); // Sets the text view font to the font from the gsettings data base.
21 
22 /* ----- accelerator ----- */ 
23   struct {
24     const char *action;
25     const char *accels[2];
26   } action_accels[] = {
27     { "win.open", { "<Control>o", NULL } },
28     { "win.save", { "<Control>s", NULL } },
29     { "win.close", { "<Control>w", NULL } },
30     { "win.new", { "<Control>n", NULL } },
31     { "win.saveas", { "<Shift><Control>s", NULL } },
32     { "win.close-all", { "<Control>q", NULL } },
33   };
34 
35   for (i = 0; i < G_N_ELEMENTS(action_accels); i++)
36     gtk_application_set_accels_for_action(GTK_APPLICATION(app), action_accels[i].action, action_accels[i].accels);
37 }
~~~

Функция `app_startup` заменяет обработчики сигналов по умолчанию.
Она выполняет пять действий.

- Вызывает обработчик startup родителя. Это называется "цепочкой вызовов вверх" (chaining up).
Обработчик "startup" по умолчанию запускается до пользовательских обработчиков.
Поэтому вызов обработчика родителя должен быть выполнен в начале.
- Создает главное окно.
Это приложение имеет только одно окно верхнего уровня.
В этом случае хороший способ — создать окно в обработчике startup, который вызывается только один раз.
Обработчики activate или open могут быть вызваны два или более раз.
Поэтому, если вы создаете окно в обработчике activate или open, может быть создано два или более окон.
- Устанавливает CSS по умолчанию для дисплея в "textview {padding: 10px;}".
Это устанавливает отступ для GtkTextView, или TfeTextView, в 10px и делает текст более читабельным.
Этот CSS фиксирован и никогда не меняется в течение жизни приложения.
- Добавляет еще один CSS-провайдер, на который указывает `app->provider`, к дисплею по умолчанию.
Этот CSS зависит от значения "font-desc" в GSettings и может быть изменен в течение времени работы приложения.
И вызывает `changed_font_cb` для обновления настройки CSS шрифта.
- Устанавливает ускорители приложения с помощью функции `gtk_application_set_accels_for_action`.
Ускорители — это разновидность функций горячих клавиш.
Например, `Ctrl+O` — это ускоритель для активации действия "open".
Ускорители записываются в массиве `action-accels[]`.
Его элемент — это структура `struct {const char *action; const char *accels[2];}`.
Член `action` — это имя действия.
Член `accels` — это массив из двух указателей.
Например, `{"win.open", { "<Control>o", NULL }}` сообщает, что ускоритель `Ctrl+O` подключен к действию "win.open".
Второй элемент `accels` — это NULL, который является конечной меткой.
Вы можете определить более одной клавиши ускорителя, и список должен заканчиваться NULL (нулем).
Если вы хотите это сделать, длина массива должна быть три или более.
Например, `{"win.open", { "<Control>o", "<Alt>o", NULL }}` означает, что два ускорителя `Ctrl+O` и `Alt+O` подключены к действию "win.open".
Парсер распознает "\<control\>o", "\<Shift\>\<Alt\>F2", "\<Ctrl\>minus" и так далее.
Если вы хотите использовать символьную клавишу, например "\<Ctrl\>-", используйте вместо этого "\<Ctrl\>minus".
Такая связь между нижним регистром и символом (код символа) указана в [`gdkkeysyms.h`](https://gitlab.gnome.org/GNOME/gtk/-/blob/master/gdk/gdkkeysyms.h) в исходном коде GTK 4.

### Обработчики сигналов activate и open

Две функции `app_activate` и `app_open` заменяют обработчики сигналов по умолчанию.

~~~C
 1 static void
 2 app_activate (GApplication *application) {
 3   TfeApplication *app = TFE_APPLICATION (application);
 4 
 5   tfe_window_notebook_page_new (app->win);
 6   gtk_window_present (GTK_WINDOW (app->win));
 7 }
 8 
 9 static void
10 app_open (GApplication *application, GFile ** files, gint n_files, const gchar *hint) {
11   TfeApplication *app = TFE_APPLICATION (application);
12 
13   tfe_window_notebook_page_new_with_files (app->win, files, n_files);
14   gtk_window_present (GTK_WINDOW (app->win));
15 }
~~~
 
Исходные обработчики по умолчанию не выполняют полезной работы, и вам не нужно вызывать обработчики родителя по умолчанию.
Они просто создают страницы блокнота и показывают окно верхнего уровня.

### Настройка CSS шрифта

~~~C
 1 static void
 2 changed_font_cb (GSettings *settings, char *key, gpointer user_data) {
 3   TfeApplication *app = TFE_APPLICATION (user_data);
 4   char *font, *s, *css;
 5   PangoFontDescription *pango_font_desc;
 6 
 7   if (g_strcmp0(key, "font-desc") != 0)
 8     return;
 9   font = g_settings_get_string (app->settings, "font-desc");
10   pango_font_desc = pango_font_description_from_string (font);
11   g_free (font);
12   s = pfd2css (pango_font_desc); // converts Pango Font Description into CSS style string
13   pango_font_description_free (pango_font_desc);
14   css = g_strdup_printf ("textview {%s}", s);
15   gtk_css_provider_load_from_data (app->provider, css, -1);
16   g_free (s);
17   g_free (css);
18 }
~~~

Функция `changed_font_cb` является обработчиком сигнала "changed::font-desc" на экземпляре GSettings.
Имя сигнала — "changed", а "font-desc" — это имя ключа.
Этот сигнал испускается, когда изменяется значение ключа "font-desc".
Значение привязано к свойству "font-desc" экземпляра GtkFontDialogButton.
Поэтому обработчик `changed_font_cb` вызывается, когда пользователь выбирает и обновляет шрифт через диалог шрифта.

Строка извлекается из базы данных GSetting и преобразуется в описание шрифта pango.
И строка CSS создается функциями `pfd2css` и `g_strdup_printf`.
Затем CSS-провайдер устанавливается в эту строку.
Провайдер уже был заранее добавлен к текущему дисплею.
Таким образом, шрифт применяется к дисплею.

## Другие файлы

main.c

~~~C
 1 #include <gtk/gtk.h>
 2 #include "tfeapplication.h"
 3 
 4 #define APPLICATION_ID "com.github.ToshioCP.tfe"
 5 
 6 int
 7 main (int argc, char **argv) {
 8   TfeApplication *app;
 9   int stat;
10 
11   app = tfe_application_new (APPLICATION_ID, G_APPLICATION_HANDLES_OPEN);
12   stat = g_application_run (G_APPLICATION (app), argc, argv);
13   g_object_unref (app);
14   return stat;
15 }
16 
~~~

Файл ресурсов XML.

~~~xml
1 <?xml version="1.0" encoding="UTF-8"?>
2 <gresources>
3   <gresource prefix="/com/github/ToshioCP/tfe">
4     <file>tfewindow.ui</file>
5     <file>tfepref.ui</file>
6     <file>tfealert.ui</file>
7     <file>menu.ui</file>
8   </gresource>
9 </gresources>
~~~

Файл GSchema XML

~~~xml
 1 <?xml version="1.0" encoding="UTF-8"?>
 2 <schemalist>
 3   <schema path="/com/github/ToshioCP/tfe/" id="com.github.ToshioCP.tfe">
 4     <key name="font-desc" type="s">
 5       <default>'Monospace 12'</default>
 6       <summary>Font</summary>
 7       <description>A font to be used for textview.</description>
 8     </key>
 9   </schema>
10 </schemalist>
~~~

Meson.build

~~~meson
 1 project('tfe', 'c', license : 'GPL-3.0-or-later', meson_version:'>=1.0.1', version: '0.5')
 2 
 3 gtkdep = dependency('gtk4')
 4 
 5 gnome = import('gnome')
 6 resources = gnome.compile_resources('resources','tfe.gresource.xml')
 7 gnome.compile_schemas(depend_files: 'com.github.ToshioCP.tfe.gschema.xml')
 8 
 9 sourcefiles = files('main.c', 'tfeapplication.c', 'tfewindow.c', 'tfepref.c', 'tfealert.c', 'pfd2css.c', '../tfetextview/tfetextview.c')
10 
11 executable(meson.project_name(), sourcefiles, resources, dependencies: gtkdep, export_dynamic: true, install: true)
12 
13 schema_dir = get_option('prefix') / get_option('datadir') / 'glib-2.0/schemas/'
14 install_data('com.github.ToshioCP.tfe.gschema.xml', install_dir: schema_dir)
15 gnome.post_install (glib_compile_schemas: true)
~~~

- Функция `project` определяет проект и инициализирует meson.
Первый аргумент — это имя проекта, а второй — имя языка.
Другие аргументы — это именованные аргументы.
- Функция `dependency` определяет зависимую библиотеку.
Tfe зависит от GTK4.
Это используется для создания опции `pkg-config` в командной строке компилятора C для включения заголовочных файлов и связывания библиотек.
Возвращаемый объект `gtkdep` используется в качестве аргумента функции `executable` позже.
- Функция `import` импортирует модуль расширения.
Модуль GNOME имеет несколько удобных методов, таких как `gnome.compile_resources` и `gnome.compile_schemas`.
- Метод `gnome.compile_resources` компилирует и создает файлы ресурсов.
Первый аргумент — это имя ресурса без расширения, а второй — имя XML-файла.
Возвращаемое значение — это массив `['resources,c', 'resources.h']`.
- Функция `gnome.compile_schemas` компилирует файлы схем в текущем каталоге.
Это просто создает `gschemas.compiled` в каталоге сборки.
Он используется для тестирования исполняемого двоичного файла в каталоге сборки.
Функция не устанавливает файл схемы.
- Функция `files` создает объект File.
- Функция `executable` определяет элементы компиляции, такие как имя цели, исходные файлы, зависимости и установка.
Имя цели — "tfe".
Исходные файлы — это элементы 'sourcefiles' и `resources'.
Он использует библиотеки GTK4.
Он может быть установлен.
- Последние три строки — это работа после установки.
Переменная `schema_dir` — это каталог, в котором хранится файл схемы.
Если meson запускается с аргументом `--prefix=$HOME/.local`, это `$HOME/.local/share/glib-2.9/schemas`.
Функция `install_data` копирует файл первого аргумента в каталог второго аргумента.
Метод `gnome.post_install` запускает `glib-compile-schemas` и обновляет файл `gschemas_compiled`.

## Компиляция и установка

Если вы хотите установить его в вашу локальную область, используйте опцию `--prefix=$HOME/.local` или `--prefix=$HOME`.
Если вы хотите установить его в системную область, никакой опции не требуется.
Он будет установлен в каталог `/user/local`.

~~~
$ meson setup --prefix=$HOME/.local _build
$ ninja -C _build
$ ninja -C _build install
~~~

Вам нужны права root для установки в системную область.

~~~
$ meson setup _build
$ ninja -C _build
$ sudo ninja -C _build install
~~~

Исходные файлы находятся в каталоге [src/tfe6](../src/tfe6).

Мы создали очень маленький текстовый редактор.
Вы можете добавить функции к этому редактору.
Когда вы добавляете новую функцию, будьте осторожны со структурой программы.
Возможно, вам нужно разделить файл на несколько файлов.
Нехорошо помещать много вещей в один файл.
И важно думать о взаимосвязи между исходными файлами и структурами виджетов.

Исходные файлы находятся в [репозитории GitHub учебника Gtk4](https://github.com/ToshioCP/Gtk4-tutorial).
Загрузите его и посмотрите каталог `src/tfe6`.

Примечание: Когда нажимается кнопка меню, печатаются сообщения об ошибках.

```
(tfe:31153): Gtk-CRITICAL **: 13:05:40.746: _gtk_css_corner_value_get_x: assertion 'corner->class == &GTK_CSS_VALUE_CORNER' failed
```

Я нашел [сообщение](https://discourse.gnome.org/t/menu-button-gives-error-messages-with-latest-gtk4/15689) в GNOME Discourse.
Комментарий говорит, что GTK 4.10 имеет ошибку, и она исправлена в версии 4.10.5.
Я еще не проверил 4.10.5, где UBUNTU GTK4 все еще 4.10.4.

Up: [README.md](../README.md),  Prev: [Section 22](sec22.md), Next: [Section 24](sec24.md)
