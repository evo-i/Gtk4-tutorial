Up: [README.md](../README.md),  Prev: [Section 6](sec6.md), Next: [Section 8](sec8.md)

# Виджеты (3)

## Сигнал open

### Флаг G\_APPLICATION\_HANDLES\_OPEN

В предыдущем разделе мы создали очень простой редактор с использованием GtkTextView, GtkTextBuffer и GtkScrolledWindow.
Мы добавим в программу возможность чтения файлов и улучшим её до программы просмотра файлов.

Самый простой способ передать имя файла — использовать аргумент командной строки.

~~~
$ ./a.out filename
~~~

Программа откроет файл и вставит его содержимое в GtkTextBuffer.

Для этого нам нужно знать, как GtkApplication (или GApplication) распознаёт аргументы.
Это описано в [GIO API Reference -- Application](https://docs.gtk.org/gio/class.Application.html).

При создании GtkApplication в качестве аргумента передаётся флаг (GApplicationFlags).

~~~C
GtkApplication *
gtk_application_new (const gchar *application_id, GApplicationFlags flags);
~~~

В этом учебнике объясняются только два флага: `G_APPLICATION_DEFAULT_FLAGS` и `G_APPLICATION_HANDLES_OPEN`.

Флаг `G_APPLICATION_FLAGS_NONE` использовался вместо `G_APPLICATION_DEFAULT_FLAGS` до версии GIO 2.73.3 (GLib 2.73.3 от 5 августа 2022 года).
Теперь он устарел, и рекомендуется использовать `G_APPLICATION_DEFAULT_FLAGS`.

Для получения дополнительной информации см. [GIO API Reference -- ApplicationFlags](https://docs.gtk.org/gio/flags.ApplicationFlags.html) и
[GIO API Reference -- g\_application\_run](https://docs.gtk.org/gio/method.Application.run.html).

Мы уже использовали `G_APPLICATION_DEFAULT_FLAGS`, так как это самый простой вариант, и аргументы командной строки не разрешены.
Если вы передадите аргументы, произойдёт ошибка.

Флаг `G_APPLICATION_HANDLES_OPEN` — это второй по простоте вариант.
Он разрешает аргументы, но только имена файлов.

~~~C
app = gtk_application_new ("com.github.ToshioCP.tfv3", G_APPLICATION_HANDLES_OPEN);
~~~

### Сигнал open

Когда приложению передаётся флаг `G_APPLICATION_HANDLES_OPEN`, становятся доступны два сигнала.

- Сигнал activate: этот сигнал испускается, когда нет аргументов.
- Сигнал open: этот сигнал испускается, когда есть хотя бы один аргумент.

Обработчик сигнала "open" определяется следующим образом.

~~~C
void
open (
  GApplication* self,
  gpointer files,
  gint n_files,
  gchar* hint,
  gpointer user_data
)
~~~

Параметры:

- self: экземпляр приложения (обычно GtkApplication)
- files: массив GFile. [array length=n\_files] [element-type GFile]
- n_files: количество элементов в `files`
- hint: подсказка, предоставленная вызывающим экземпляром (обычно может быть проигнорирована)
- user_data: пользовательские данные, которые устанавливаются при подключении обработчика сигнала.

## Программа просмотра файлов

### Что такое программа просмотра файлов?

Программа просмотра файлов — это программа, которая отображает текстовые файлы.
Наша программа просмотра файлов работает следующим образом.

- Когда передаются аргументы, она распознаёт первый аргумент как имя файла и открывает его.
- Второй и последующие аргументы игнорируются.
- Если нет аргументов, она показывает сообщение об ошибке и завершает работу.
- Если файл успешно открыт, она читает содержимое файла, вставляет его в GtkTextBuffer и показывает окно.
- Если не удаётся открыть файл, она показывает сообщение об ошибке и завершает работу.

Программа показана ниже.

~~~C
 1 #include <gtk/gtk.h>
 2 
 3 static void
 4 app_activate (GApplication *app) {
 5   g_printerr ("You need a filename argument.\n");
 6 }
 7 
 8 static void
 9 app_open (GApplication *app, GFile ** files, int n_files, char *hint) {
10   GtkWidget *win;
11   GtkWidget *scr;
12   GtkWidget *tv;
13   GtkTextBuffer *tb;
14   char *contents;
15   gsize length;
16   char *filename;
17   GError *err = NULL;
18 
19   win = gtk_application_window_new (GTK_APPLICATION (app));
20   gtk_window_set_default_size (GTK_WINDOW (win), 400, 300);
21 
22   scr = gtk_scrolled_window_new ();
23   gtk_window_set_child (GTK_WINDOW (win), scr);
24 
25   tv = gtk_text_view_new ();
26   tb = gtk_text_view_get_buffer (GTK_TEXT_VIEW (tv));
27   gtk_text_view_set_wrap_mode (GTK_TEXT_VIEW (tv), GTK_WRAP_WORD_CHAR);
28   gtk_text_view_set_editable (GTK_TEXT_VIEW (tv), FALSE);
29   gtk_scrolled_window_set_child (GTK_SCROLLED_WINDOW (scr), tv);
30 
31   if (g_file_load_contents (files[0], NULL, &contents, &length, NULL, &err)) {
32     gtk_text_buffer_set_text (tb, contents, length);
33     g_free (contents);
34     if ((filename = g_file_get_basename (files[0])) != NULL) {
35       gtk_window_set_title (GTK_WINDOW (win), filename);
36       g_free (filename);
37     }
38     gtk_window_present (GTK_WINDOW (win));
39   } else {
40     g_printerr ("%s.\n", err->message);
41     g_error_free (err);
42     gtk_window_destroy (GTK_WINDOW (win));
43   }
44 }
45 
46 int
47 main (int argc, char **argv) {
48   GtkApplication *app;
49   int stat;
50 
51   app = gtk_application_new ("com.github.ToshioCP.tfv3", G_APPLICATION_HANDLES_OPEN);
52   g_signal_connect (app, "activate", G_CALLBACK (app_activate), NULL);
53   g_signal_connect (app, "open", G_CALLBACK (app_open), NULL);
54   stat = g_application_run (G_APPLICATION (app), argc, argv);
55   g_object_unref (app);
56   return stat;
57 }
~~~

Сохраните её как `tfv3.c`.
Если вы скачали этот репозиторий, файл находится в `src/tfv/tfv3.c`.
Скомпилируйте и запустите.

~~~
$ comp tfv3
$ ./a.out tfv3.c
~~~

![File viewer](../image/screenshot_tfv3.png)

Функция `main` имеет только два изменения по сравнению с предыдущей версией.

- `G_APPLICATION_DEFAULT_FLAGS` заменён на `G_APPLICATION_HANDLES_OPEN`
- Добавлено `g_signal_connect (app, "open", G_CALLBACK (app_open), NULL)`.

Когда функции `gtk_application_new` передаётся флаг `G_APPLICATION_HANDLES_OPEN`, приложение ведёт себя следующим образом:

- Если приложение запускается без аргументов командной строки, оно испускает сигнал "activate" при активации.
- Если приложение запускается с аргументами командной строки, оно испускает сигнал "open" при активации.

Обработчик `app_activate` становится очень простым.
Он просто выводит сообщение об ошибке и возвращается к вызывающей стороне.
Затем приложение немедленно завершает работу, потому что окно не создаётся.

Основная работа выполняется в обработчике `app_open`.

- Создаёт GtkApplicationWindow, GtkScrolledWindow, GtkTextView и GtkTextBuffer и соединяет их вместе
- Устанавливает режим переноса слов в `GTK_WRAP_WORD_CHAR` в GtktextView
- Устанавливает GtkTextView как нередактируемый, потому что программа не редактор, а только просмотрщик
- Читает файл и вставляет текст в GtkTextBuffer (это будет объяснено позже)
- Если файл не открывается, выводит сообщение об ошибке и уничтожает окно. Это приводит к завершению работы приложения.

Ниже приведена часть программы для чтения файла.

~~~C
if (g_file_load_contents (files[0], NULL, &contents, &length, NULL, &err)) {
  gtk_text_buffer_set_text (tb, contents, length);
  g_free (contents);
  if ((filename = g_file_get_basename (files[0])) != NULL) {
    gtk_window_set_title (GTK_WINDOW (win), filename);
    g_free (filename);
  }
  gtk_window_present (GTK_WINDOW (win));
} else {
  g_printerr ("%s.\n", err->message);
  g_error_free (err);
  gtk_window_destroy (GTK_WINDOW (win));
}
~~~

Функция `g_file_load_contents` загружает содержимое файла во временный буфер,
который автоматически выделяется, и устанавливает `contents` для указания на буфер.
Длина буфера присваивается переменной `length`.
Она возвращает `TRUE`, если содержимое файла успешно загружено.
Если возникает ошибка, она возвращает `FALSE` и устанавливает переменную `err` для указания на вновь созданную структуру GError.
Вызывающая сторона становится владельцем структуры GError и отвечает за её освобождение.
Если вы хотите узнать подробности о g\_file\_load\_contents, см. [g file load contents](https://docs.gtk.org/gio/method.File.load_contents.html).

Если файл успешно прочитан, программа вставляет содержимое в GtkTextBuffer,
освобождает временный буфер, на который указывает `contents`, устанавливает заголовок окна,
освобождает память, на которую указывает `filename`, а затем показывает окно.

Если чтение не удаётся, `g_file_load_contents` устанавливает `err` для указания на вновь созданную структуру GError.
Структура имеет вид:

~~~C
struct GError {
  GQuark domain;
  int code;
  char* message;
}
~~~

Элемент `message` используется чаще всего.
Он указывает на сообщение об ошибке.
Функция `g_error_free` используется для освобождения памяти структуры.
См. [GError](https://docs.gtk.org/glib/struct.Error.html).

Приведённая выше программа выводит сообщение об ошибке, освобождает `err`, уничтожает окно и, наконец, завершает работу программы.

## GtkNotebook

GtkNotebook — это виджет-контейнер, который содержит несколько виджетов с вкладками.
Он показывает только один дочерний виджет за раз.
Другой дочерний виджет будет показан при щелчке по его вкладке.

![GtkNotebook](../image/screenshot_gtk_notebook.png)

Левое изображение — это окно при запуске.
Показан файл `pr1.c`, и его имя находится на левой вкладке.
После щелчка по правой вкладке показывается содержимое файла `tfv1.c` (правое изображение).

Ниже приведён `tfv4.c`.
Он имеет виджет GtkNoteBook.
Он вставляется как дочерний элемент GtkApplicationWindow и содержит несколько GtkScrolledWindow.

~~~C
 1 #include <gtk/gtk.h>
 2 
 3 static void
 4 app_activate (GApplication *app) {
 5   g_printerr ("You need filename arguments.\n");
 6 }
 7 
 8 static void
 9 app_open (GApplication *app, GFile ** files, gint n_files, gchar *hint) {
10   GtkWidget *win;
11   GtkWidget *nb;
12   GtkWidget *lab;
13   GtkNotebookPage *nbp;
14   GtkWidget *scr;
15   GtkWidget *tv;
16   GtkTextBuffer *tb;
17   char *contents;
18   gsize length;
19   char *filename;
20   int i;
21   GError *err = NULL;
22 
23   win = gtk_application_window_new (GTK_APPLICATION (app));
24   gtk_window_set_title (GTK_WINDOW (win), "file viewer");
25   gtk_window_set_default_size (GTK_WINDOW (win), 600, 400);
26   nb = gtk_notebook_new ();
27   gtk_window_set_child (GTK_WINDOW (win), nb);
28 
29   for (i = 0; i < n_files; i++) {
30     if (g_file_load_contents (files[i], NULL, &contents, &length, NULL, &err)) {
31       scr = gtk_scrolled_window_new ();
32       tv = gtk_text_view_new ();
33       tb = gtk_text_view_get_buffer (GTK_TEXT_VIEW (tv));
34       gtk_text_view_set_wrap_mode (GTK_TEXT_VIEW (tv), GTK_WRAP_WORD_CHAR);
35       gtk_text_view_set_editable (GTK_TEXT_VIEW (tv), FALSE);
36       gtk_scrolled_window_set_child (GTK_SCROLLED_WINDOW (scr), tv);
37 
38       gtk_text_buffer_set_text (tb, contents, length);
39       g_free (contents);
40       if ((filename = g_file_get_basename (files[i])) != NULL) {
41         lab = gtk_label_new (filename);
42         g_free (filename);
43       } else
44         lab = gtk_label_new ("");
45       gtk_notebook_append_page (GTK_NOTEBOOK (nb), scr, lab);
46       nbp = gtk_notebook_get_page (GTK_NOTEBOOK (nb), scr);
47       g_object_set (nbp, "tab-expand", TRUE, NULL);
48     } else {
49       g_printerr ("%s.\n", err->message);
50       g_clear_error (&err);
51     }
52   }
53   if (gtk_notebook_get_n_pages (GTK_NOTEBOOK (nb)) > 0)
54     gtk_window_present (GTK_WINDOW (win));
55   else
56     gtk_window_destroy (GTK_WINDOW (win));
57 }
58 
59 int
60 main (int argc, char **argv) {
61   GtkApplication *app;
62   int stat;
63 
64   app = gtk_application_new ("com.github.ToshioCP.tfv4", G_APPLICATION_HANDLES_OPEN);
65   g_signal_connect (app, "activate", G_CALLBACK (app_activate), NULL);
66   g_signal_connect (app, "open", G_CALLBACK (app_open), NULL);
67   stat = g_application_run (G_APPLICATION (app), argc, argv);
68   g_object_unref (app);
69   return stat;
70 }
~~~

Большинство изменений находятся в функции `app_open`.
Числа слева от следующих элементов — это номера строк в исходном коде.

- 11-13: определяются переменные `nb`, `lab` и `nbp`. Они указывают соответственно на GtkNotebook, GtkLabel и GtkNotebookPage.
- 24: заголовок окна устанавливается на "file viewer".
- 25: размер окна по умолчанию — 600x400.
- 26-27: создаётся GtkNotebook и вставляется в GtkApplicationWindow как дочерний элемент.
- 29-57: цикл for. Переменная `files[i]` указывает на i-й GFile, который создаётся GtkApplication из i-го аргумента командной строки.
- 31-36: создаются GtkScrollledWindow, GtkTextView. GtkTextBuffer получается из GtkTextView.
GtkTextView подключается к GtkScrolledWindow как дочерний элемент.
- 38-39: вставляет содержимое файла в GtkTextBuffer и освобождает память, на которую указывает `contents`.
- 40-42: если имя файла взято из GFile, создаётся GtkLabel с именем файла. Строка `filename` освобождается.
- 43-44: если не удаётся получить имя файла, создаётся GtkLabel с пустой строкой.
- 45-46: добавляет GtkScrolledWindow к GtkNotebook в качестве дочернего элемента.
И GtkLabel устанавливается как вкладка дочернего элемента.
При этом автоматически создаётся GtkNoteBookPage.
Функция `gtk_notebook_get_page` возвращает GtkNotebookPage дочернего элемента (GtkScrolledWindow).
- 47: GtkNotebookPage имеет свойство "tab-expand".
Если оно установлено в TRUE, то вкладка расширяется по горизонтали настолько, насколько это возможно.
Если оно установлено в FALSE, ширина вкладки определяется размером метки.
`g_object_set` — это общая функция для установки свойств объектов.
См. [GObject API Reference -- g\_object\_set](https://docs.gtk.org/gobject/method.Object.set.html).
- 48-50: если не удаётся прочитать файл, показывается сообщение об ошибке.
Функция `g_clear_error (&err)` работает как `g_error_free (err); err = NULL`.
- 53-56: если существует хотя бы одна страница, окно показывается.
В противном случае окно уничтожается, и приложение завершает работу.

Up: [README.md](../README.md),  Prev: [Section 6](sec6.md), Next: [Section 8](sec8.md)
