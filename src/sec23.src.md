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

@@@include
tfe6/pfd2css.h
@@@

Пять функций являются публичными.
Первая функция — это удобная функция для установки четырех других CSS-свойств одновременно.

@@@include
tfe6/pfd2css.c
@@@

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

@@@include
tfe6/tfeapplication.h
@@@

Следующий код извлечен из `tfeapplication.c`.
Он создает класс и экземпляр TfeApplication.

@@@if gfm
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
@@@else
```{.C}
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
@@@end

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

@@@include
tfe6/tfeapplication.c app_startup
@@@

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

@@@include
tfe6/tfeapplication.c app_activate app_open
@@@
 
Исходные обработчики по умолчанию не выполняют полезной работы, и вам не нужно вызывать обработчики родителя по умолчанию.
Они просто создают страницы блокнота и показывают окно верхнего уровня.

### Настройка CSS шрифта

@@@include
tfe6/tfeapplication.c changed_font_cb
@@@

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

@@@include
tfe6/main.c
@@@

Файл ресурсов XML.

@@@include
tfe6/tfe.gresource.xml
@@@

Файл GSchema XML

@@@include
tfe6/com.github.ToshioCP.tfe.gschema.xml
@@@

Meson.build

@@@include
tfe6/meson.build
@@@

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

Исходные файлы находятся в каталоге [src/tfe6](tfe6).

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
