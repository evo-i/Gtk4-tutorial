Up: [README.md](../README.md),  Prev: [Section 14](sec14.md), Next: [Section 16](sec16.md)

# Главная программа Tfe

Файл `tfeapplication.c` является главной программой Tfe.
Он включает весь код, кроме `tfetextview.c` и `tfenotebook.c`.
Он выполняет:

- Поддержку приложения, в основном обработку аргументов командной строки.
- Создание виджетов с использованием ui-файла.
- Соединение сигналов кнопок и их обработчиков.
- Управление CSS.

## Функция main

Функция `main` — это первая вызываемая функция в языке C.
Она соединяет командную строку, предоставленную пользователем, и приложение Gtk.

~~~C
 1 #define APPLICATION_ID "com.github.ToshioCP.tfe"
 2
 3 int
 4 main (int argc, char **argv) {
 5   GtkApplication *app;
 6   int stat;
 7
 8   app = gtk_application_new (APPLICATION_ID, G_APPLICATION_HANDLES_OPEN);
 9
10   g_signal_connect (app, "startup", G_CALLBACK (app_startup), NULL);
11   g_signal_connect (app, "activate", G_CALLBACK (app_activate), NULL);
12   g_signal_connect (app, "open", G_CALLBACK (app_open), NULL);
13
14   stat = g_application_run (G_APPLICATION (app), argc, argv);
15   g_object_unref (app);
16   return stat;
17 }
~~~

- 1: Определяет идентификатор приложения.
Благодаря директиве `#define` легко найти идентификатор приложения.
- 8: Создаёт объект GtkApplication.
- 10-12: Соединяет сигналы "startup", "activate" и "open" с их обработчиками.
- 14: Запускает приложение.
- 15-16: Освобождает ссылку на приложение и возвращает статус.

## Обработчик сигнала startup

Сигнал startup испускается сразу после инициализации экземпляра GtkApplication.
Обработчик инициализирует всё приложение, которое включает не только экземпляр GtkApplication, но также виджеты и некоторые другие объекты.

- Создаёт виджеты с использованием ui-файла.
- Соединяет сигналы кнопок и их обработчики.
- Устанавливает CSS.

Обработчик выглядит следующим образом.

~~~C
 1 static void
 2 app_startup (GApplication *application) {
 3   GtkApplication *app = GTK_APPLICATION (application);
 4   GtkBuilder *build;
 5   GtkApplicationWindow *win;
 6   GtkNotebook *nb;
 7   GtkButton *btno;
 8   GtkButton *btnn;
 9   GtkButton *btns;
10   GtkButton *btnc;
11 
12   build = gtk_builder_new_from_resource ("/com/github/ToshioCP/tfe/tfe.ui");
13   win = GTK_APPLICATION_WINDOW (gtk_builder_get_object (build, "win"));
14   nb = GTK_NOTEBOOK (gtk_builder_get_object (build, "nb"));
15   gtk_window_set_application (GTK_WINDOW (win), app);
16   btno = GTK_BUTTON (gtk_builder_get_object (build, "btno"));
17   btnn = GTK_BUTTON (gtk_builder_get_object (build, "btnn"));
18   btns = GTK_BUTTON (gtk_builder_get_object (build, "btns"));
19   btnc = GTK_BUTTON (gtk_builder_get_object (build, "btnc"));
20   g_signal_connect_swapped (btno, "clicked", G_CALLBACK (open_cb), nb);
21   g_signal_connect_swapped (btnn, "clicked", G_CALLBACK (new_cb), nb);
22   g_signal_connect_swapped (btns, "clicked", G_CALLBACK (save_cb), nb);
23   g_signal_connect_swapped (btnc, "clicked", G_CALLBACK (close_cb), nb);
24   g_object_unref(build);
25 
26 GdkDisplay *display;
27 
28   display = gdk_display_get_default ();
29   GtkCssProvider *provider = gtk_css_provider_new ();
30   gtk_css_provider_load_from_data (provider, "textview {padding: 10px; font-family: monospace; font-size: 12pt;}", -1);
31   gtk_style_context_add_provider_for_display (display, GTK_STYLE_PROVIDER (provider), GTK_STYLE_PROVIDER_PRIORITY_APPLICATION);
32 
33   g_signal_connect (win, "destroy", G_CALLBACK (before_destroy), provider);
34   g_object_unref (provider);
35 }
~~~

- 12-15: Создаёт виджеты с использованием ui-ресурса.
Соединяет окно верхнего уровня и приложение с помощью `gtk_window_set_application`.
- 16-23: Получает кнопки и соединяет их сигналы и обработчики.
Макрос `g_signal_connect_swapped` соединяет сигнал и обработчик так же, как `g_signal_connect`.
Разница в том, что `g_signal_connect_swapped` меняет местами пользовательские данные для объекта.
Например, макрос в строке 20 меняет местами `nb` и `btno`.
Таким образом, обработчик ожидает, что первым аргументом будет `nb` вместо `btno`.
- 24: Освобождает ссылку на GtkBuilder.
- 26-31: Устанавливает CSS.
CSS в Gtk похож на CSS в HTML.
С помощью CSS можно задать margin, border, padding, color, font и так далее.
В этой программе CSS находится в строке 30.
Он устанавливает padding, font-family и размер шрифта для GtkTextView.
CSS будет объяснён в следующем подразделе.
- 26-28: GdkDisplay используется для установки CSS.
Объект GdkDisplay по умолчанию можно получить с помощью функции `gfk_display_get_default`.
Эту функцию нужно вызывать после создания окна.
- 33: Соединяет сигнал "destroy" главного окна и обработчик before\_destroy.
Этот обработчик объясняется в следующем подразделе.
- 34: Провайдер бесполезен для обработчика startup, поэтому он освобождается.
Примечание: Это не означает уничтожение провайдера.
На него ссылается дисплей, поэтому счётчик ссылок не равен нулю.

## CSS в Gtk

CSS — это аббревиатура от Cascading Style Sheet (каскадная таблица стилей).
Изначально он используется с HTML для описания семантики представления документа.
Вы могли заметить, что виджеты в Gtk похожи на элементы в HTML.
Это означает, что CSS также может применяться к оконной системе Gtk.

### CSS-узлы, селекторы

Синтаксис CSS выглядит следующим образом.

~~~css
selector { color: yellow; padding-top: 10px; ...}
~~~

Каждый виджет имеет CSS-узел.
Например, GtkTextView имеет узел `textview`.
Если вы хотите установить стиль для GtkTextView, замените `selector` выше на "textview".

~~~css
textview {color: yellow; ...}
~~~

К селектору можно применять класс, идентификатор и некоторые другие вещи, как в веб-CSS.
Для получения дополнительной информации обратитесь к [GTK 4 API Reference -- CSS in Gtk](https://docs.gtk.org/gtk4/css-overview.html).

Код обработчика startup содержит строку CSS в строке 30.

~~~css
textview {padding: 10px; font-family: monospace; font-size: 12pt;}
~~~

- Padding — это пространство между границей и содержимым.
Это пространство делает textview более читаемым.
- font-family — это название шрифта.
Название шрифта "monospace" — одно из ключевых слов семейства общих шрифтов.
- Font-size устанавливается в 12pt.

### GtkStyleContext, GtkCssProvider и GdkDisplay

GtkStyleContext устарел с версии 4.10.
Но две функции `gtk_style_context_add_provider_for_display` и `gtk_style_context_remove_provider_for_display` не устарели.
Они добавляют или удаляют объект css-провайдера из объекта GdkDisplay.

GtkCssProvider — это объект, который разбирает CSS для стилизации виджетов.

Чтобы применить ваш CSS к виджетам, вам нужно добавить GtkStyleProvider (интерфейс GtkCssProvider) в объект GdkDisplay.
Вы можете получить объект дисплея по умолчанию с помощью функции `gdk_display_get_default`.
Возвращённый объект принадлежит функции, и у вас нет его владения.
Поэтому вам не нужно беспокоиться об его освобождении.

Взгляните на исходный файл обработчика `startup` ещё раз.

- 28: Дисплей получается с помощью `gdk_display_get_default`.
- 29: Создаёт экземпляр GtkCssProvider.
- 30: Помещает CSS в провайдер.
Функция `gtk_css_provider_load_from_data` устареет с версии 4.12 (не 4.10).
Новая функция `gtk_css_provider_load_from_string` будет использоваться в будущей версии Tfe.
- 31: Добавляет провайдер к дисплею.
Последний аргумент `gtk_style_context_add_provider_for_display` — это приоритет провайдера стилей.
`GTK_STYLE_PROVIDER_PRIORITY_APPLICATION` — это приоритет для специфичной для приложения информации о стиле.
Для получения дополнительной информации обратитесь к [GTK 4 Reference --- Constants](https://docs.gtk.org/gtk4/index.html#constants).
Вы можете найти другие константы, которые имеют имена с шаблоном "STYLE\_PROVIDER\_PRIORITY\_XXXX".

~~~C
1 static void
2 before_destroy (GtkWidget *win, GtkCssProvider *provider) {
3   GdkDisplay *display = gdk_display_get_default ();
4   gtk_style_context_remove_provider_for_display (display, GTK_STYLE_PROVIDER (provider));
5 }
~~~

Когда виджет уничтожается, или, точнее, во время процесса его удаления, испускается сигнал "destroy".
Обработчик "before\_destroy" подключается к сигналу главного окна.
(См. листинг программы app\_startup.)
Таким образом, он вызывается при уничтожении окна.

Обработчик удаляет CSS-провайдер из GdkDisplay.

Примечание: CSS-провайдеры удаляются автоматически при выходе из приложения.
Поэтому, даже если обработчик `before_destroy` удалён, приложение работает.

## Обработчики сигналов activate и open

Обработчиками сигналов "activate" и "open" являются `app_activate` и `app_open` соответственно.
Они просто создают новую страницу GtkNotebookPage.

~~~C
 1 static void
 2 app_activate (GApplication *application) {
 3   GtkApplication *app = GTK_APPLICATION (application);
 4   GtkWidget *win = GTK_WIDGET (gtk_application_get_active_window (app));
 5   GtkWidget *boxv = gtk_window_get_child (GTK_WINDOW (win));
 6   GtkNotebook *nb = GTK_NOTEBOOK (gtk_widget_get_last_child (boxv));
 7 
 8   notebook_page_new (nb);
 9   gtk_window_present (GTK_WINDOW (win));
10 }
11 
12 static void
13 app_open (GApplication *application, GFile ** files, gint n_files, const gchar *hint) {
14   GtkApplication *app = GTK_APPLICATION (application);
15   GtkWidget *win = GTK_WIDGET (gtk_application_get_active_window (app));
16   GtkWidget *boxv = gtk_window_get_child (GTK_WINDOW (win));
17   GtkNotebook *nb = GTK_NOTEBOOK (gtk_widget_get_last_child (boxv));
18   int i;
19 
20   for (i = 0; i < n_files; i++)
21     notebook_page_new_with_file (nb, files[i]);
22   if (gtk_notebook_get_n_pages (nb) == 0)
23     notebook_page_new (nb);
24   gtk_window_present (GTK_WINDOW (win));
25 }
~~~

- 1-10: `app_activate`.
- 8-10: Создаёт новую страницу и показывает окно.
- 12-25: `app_open`.
- 20-21: Создаёт страницы блокнота с файлами.
- 22-23: Если ни одна страница не создана, возможно из-за ошибки чтения, то создаётся пустая страница.
- 24: Показывает окно.

Эти коды стали действительно простыми благодаря tfenotebook.c и tfetextview.c.

## Первичный экземпляр

Только один экземпляр GApplication может быть запущен одновременно в сеансе.
Сеанс — это немного сложная концепция, зависящая от платформы, но грубо говоря, он соответствует входу в графический рабочий стол.
Когда вы используете свой ПК, вы, вероятно, сначала входите в систему, затем ваш рабочий стол появляется до тех пор, пока вы не выйдете.
Это и есть сеанс.

Однако Linux — многопроцессная ОС, и вы можете запустить два или более экземпляров одного и того же приложения.
Разве это не противоречие?

Когда запускается первый экземпляр, он регистрирует себя со своим идентификатором приложения (например, `com.github.ToshioCP.tfe`).
Сразу после регистрации испускается сигнал startup, затем испускается сигнал activate или open и запускается главный цикл экземпляра.
Я писал "сигнал startup испускается сразу после инициализации экземпляра приложения" в предыдущем подразделе.
Точнее, он испускается после регистрации.

Если запускается другой экземпляр с тем же идентификатором приложения, он также пытается зарегистрировать себя.
Поскольку это второй экземпляр, регистрация идентификатора уже выполнена, поэтому она завершается неудачно.
Из-за неудачи сигнал startup не испускается.
После этого сигнал activate или open испускается в первичном экземпляре, а не во втором экземпляре.
Первичный экземпляр получает сигнал, и вызывается его обработчик.
С другой стороны, второй экземпляр не получает сигнал и немедленно завершает работу.

Попробуйте запустить два экземпляра подряд.

    $ ./_build/tfe &
    [1] 84453
    $ ./build/tfe tfeapplication.c
    $

Сначала первичный экземпляр открывает окно.
Затем, после запуска второго экземпляра, в окне первичного экземпляра появляется новая страница блокнота с содержимым `tfeapplication.c`.
Это происходит потому, что сигнал open испускается в первичном экземпляре.
Второй экземпляр немедленно завершает работу, поэтому приглашение оболочки вскоре появляется.

## Серия обработчиков, соответствующих сигналам кнопок

~~~C
 1 static void
 2 open_cb (GtkNotebook *nb) {
 3   notebook_page_open (nb);
 4 }
 5 
 6 static void
 7 new_cb (GtkNotebook *nb) {
 8   notebook_page_new (nb);
 9 }
10 
11 static void
12 save_cb (GtkNotebook *nb) {
13   notebook_page_save (nb);
14 }
15 
16 static void
17 close_cb (GtkNotebook *nb) {
18   notebook_page_close (GTK_NOTEBOOK (nb));
19 }
~~~

`open_cb`, `new_cb`, `save_cb` и `close_cb` просто вызывают соответствующие функции страницы блокнота.

## meson.build

~~~meson
 1 project('tfe', 'c')
 2 
 3 gtkdep = dependency('gtk4')
 4 
 5 gnome=import('gnome')
 6 resources = gnome.compile_resources('resources','tfe.gresource.xml')
 7 
 8 sourcefiles=files('tfeapplication.c', 'tfenotebook.c', '../tfetextview/tfetextview.c')
 9 
10 executable('tfe', sourcefiles, resources, dependencies: gtkdep)
~~~

В этом файле просто изменены имена исходных файлов по сравнению с предыдущей версией.

## Исходные файлы

Вы можете скачать файлы из [репозитория](https://github.com/ToshioCP/Gtk4-tutorial).
Есть два варианта.

- Использовать git и клонировать.
- Запустить браузер и открыть [главную страницу](https://github.com/ToshioCP/Gtk4-tutorial). Затем нажать на кнопку "Code" и нажать "Download ZIP" во всплывающем меню.
После этого распаковать архивный файл.

Если вы используете git, запустите терминал и введите следующее.

    $ git clone https://github.com/ToshioCP/Gtk4-tutorial.git

Исходные файлы находятся в каталоге [/src/tfe5](../src/tfe5).

Up: [README.md](../README.md),  Prev: [Section 14](sec14.md), Next: [Section 16](sec16.md)
