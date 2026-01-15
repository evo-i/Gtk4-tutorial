Up: [README.md](../README.md),  Prev: [Section 9](sec9.md), Next: [Section 11](sec11.md)

# Система сборки

## Управление большими исходными файлами

До сих пор мы компилировали небольшой редактор.
Программа пока не сложная, но если она станет больше, её будет трудно поддерживать.
Поэтому нам следует сделать следующее сейчас.

- У нас был только один исходный файл C, и мы поместили всё в него.
Нам нужно разделить и организовать его.
- Есть два компилятора, `gcc` и `glib-compile-resources`.
Мы должны управлять ими с помощью одного инструмента сборки.

## Разделение исходного файла C на две части.

Когда вы разделяете исходный файл C на несколько частей, каждый файл должен содержать одну вещь.
Например, наш исходный код имеет две вещи: определение TfeTextView и функции, связанные с GtkApplication и GtkApplicationWindow.
Хорошая идея — разделить их на два файла, `tfetextview.c` и `tfe.c`.

- `tfetextview.c` включает определение и функции TfeTextView.
- `tfe.c` включает функции типа `main`, `app_activate`, `app_open` и так далее, которые относятся к GtkApplication и GtkApplicationWindow.

Теперь у нас есть три исходных файла: `tfetextview.c`, `tfe.c` и `tfe3.ui`.
Цифра `3` в `tfe3.ui` похожа на номер версии.
Управление версиями с помощью имён файлов — одна из возможных идей, но у неё также есть проблема.
Вам нужно переписывать имя файла в каждой версии, и это влияет на содержимое исходных файлов, которые ссылаются на имена файлов.
Поэтому мы должны убрать `3` из имени файла.

В `tfe.c` функция `tfe_text_view_new` вызывается для создания экземпляра TfeTextView.
Но она определена в `tfetextview.c`, а не в `tfe.c`.
Отсутствие объявления (не определения) `tfe_text_view_new` вызывает ошибку при компиляции `tfe.c`.
Объявление необходимо в `tfe.c`.
Эта публичная информация обычно записывается в заголовочные файлы.
Они имеют суффикс `.h`, например `tfetextview.h`.
Заголовочные файлы включаются в исходные файлы C.
Например, `tfetextview.h` включается в `tfe.c`.

Исходные файлы показаны ниже.

`tfetextview.h`

~~~C
 1 #include <gtk/gtk.h>
 2 
 3 #define TFE_TYPE_TEXT_VIEW tfe_text_view_get_type ()
 4 G_DECLARE_FINAL_TYPE (TfeTextView, tfe_text_view, TFE, TEXT_VIEW, GtkTextView)
 5 
 6 void
 7 tfe_text_view_set_file (TfeTextView *tv, GFile *f);
 8 
 9 GFile *
10 tfe_text_view_get_file (TfeTextView *tv);
11 
12 GtkWidget *
13 tfe_text_view_new (void);
14 
~~~

`tfetextview.c`

~~~C
 1 #include <gtk/gtk.h>
 2 #include "tfetextview.h"
 3 
 4 struct _TfeTextView
 5 {
 6   GtkTextView parent;
 7   GFile *file;
 8 };
 9 
10 G_DEFINE_TYPE (TfeTextView, tfe_text_view, GTK_TYPE_TEXT_VIEW);
11 
12 static void
13 tfe_text_view_init (TfeTextView *tv) {
14 }
15 
16 static void
17 tfe_text_view_class_init (TfeTextViewClass *class) {
18 }
19 
20 void
21 tfe_text_view_set_file (TfeTextView *tv, GFile *f) {
22   tv->file = f;
23 }
24 
25 GFile *
26 tfe_text_view_get_file (TfeTextView *tv) {
27   return tv->file;
28 }
29 
30 GtkWidget *
31 tfe_text_view_new (void) {
32   return GTK_WIDGET (g_object_new (TFE_TYPE_TEXT_VIEW, NULL));
33 }
34 
~~~

`tfe.c`

~~~C
 1 #include <gtk/gtk.h>
 2 #include "tfetextview.h"
 3 
 4 static void
 5 app_activate (GApplication *app) {
 6   g_print ("You need a filename argument.\n");
 7 }
 8 
 9 static void
10 app_open (GApplication *app, GFile ** files, gint n_files, gchar *hint) {
11   GtkWidget *win;
12   GtkWidget *nb;
13   GtkWidget *lab;
14   GtkNotebookPage *nbp;
15   GtkWidget *scr;
16   GtkWidget *tv;
17   GtkTextBuffer *tb;
18   char *contents;
19   gsize length;
20   char *filename;
21   int i;
22   GError *err = NULL;
23   GtkBuilder *build;
24 
25   build = gtk_builder_new_from_resource ("/com/github/ToshioCP/tfe/tfe.ui");
26   win = GTK_WIDGET (gtk_builder_get_object (build, "win"));
27   gtk_window_set_application (GTK_WINDOW (win), GTK_APPLICATION (app));
28   nb = GTK_WIDGET (gtk_builder_get_object (build, "nb"));
29   g_object_unref (build);
30   for (i = 0; i < n_files; i++) {
31     if (g_file_load_contents (files[i], NULL, &contents, &length, NULL, &err)) {
32       scr = gtk_scrolled_window_new ();
33       tv = tfe_text_view_new ();
34       tb = gtk_text_view_get_buffer (GTK_TEXT_VIEW (tv));
35       gtk_text_view_set_wrap_mode (GTK_TEXT_VIEW (tv), GTK_WRAP_WORD_CHAR);
36       gtk_scrolled_window_set_child (GTK_SCROLLED_WINDOW (scr), tv);
37 
38       tfe_text_view_set_file (TFE_TEXT_VIEW (tv),  g_file_dup (files[i]));
39       gtk_text_buffer_set_text (tb, contents, length);
40       g_free (contents);
41       filename = g_file_get_basename (files[i]);
42       lab = gtk_label_new (filename);
43       gtk_notebook_append_page (GTK_NOTEBOOK (nb), scr, lab);
44       nbp = gtk_notebook_get_page (GTK_NOTEBOOK (nb), scr);
45       g_object_set (nbp, "tab-expand", TRUE, NULL);
46       g_free (filename);
47     } else {
48       g_printerr ("%s.\n", err->message);
49       g_clear_error (&err);
50     }
51   }
52   if (gtk_notebook_get_n_pages (GTK_NOTEBOOK (nb)) > 0) {
53     gtk_window_present (GTK_WINDOW (win));
54   } else
55     gtk_window_destroy (GTK_WINDOW (win));
56 }
57 
58 int
59 main (int argc, char **argv) {
60   GtkApplication *app;
61   int stat;
62 
63   app = gtk_application_new ("com.github.ToshioCP.tfe", G_APPLICATION_HANDLES_OPEN);
64   g_signal_connect (app, "activate", G_CALLBACK (app_activate), NULL);
65   g_signal_connect (app, "open", G_CALLBACK (app_open), NULL);
66   stat = g_application_run (G_APPLICATION (app), argc, argv);
67   g_object_unref (app);
68   return stat;
69 }
70 
~~~

UI-файл `tfe.ui` такой же, как `tfe3.ui` в предыдущем разделе.

`tfe.gresource.xml`

~~~xml
1 <?xml version="1.0" encoding="UTF-8"?>
2 <gresources>
3   <gresource prefix="/com/github/ToshioCP/tfe">
4     <file>tfe.ui</file>
5   </gresource>
6 </gresources>
~~~

Разделение файла облегчает поддержку.
Но теперь мы сталкиваемся с новой проблемой.
Количество шагов сборки увеличилось.

- Компиляция ui-файла `tfe.ui` в `resources.c`.
- Компиляция `tfe.c` в `tfe.o` (объектный файл).
- Компиляция `tfetextview.c` в `tfetextview.o`.
- Компиляция `resources.c` в `resources.o`.
- Компоновка всех объектных файлов в приложение `tfe`.

Инструменты сборки управляют этими шагами.

## Meson и Ninja

Я объясню инструменты сборки Meson и Ninja.

Другие возможные инструменты — Make и Autotools.
Это традиционные инструменты, но они медленнее, чем Ninja.
Поэтому многие разработчики в последнее время переключились на Meson и Ninja.
Например, GTK 4 использует их.

Сначала вам нужно создать файл `meson.build`.

~~~meson
 1 project('tfe', 'c')
 2 
 3 gtkdep = dependency('gtk4')
 4 
 5 gnome=import('gnome')
 6 resources = gnome.compile_resources('resources','tfe.gresource.xml')
 7 
 8 sourcefiles=files('tfe.c', 'tfetextview.c')
 9 
10 executable('tfe', sourcefiles, resources, dependencies: gtkdep, install: false)
~~~

- 1: функция `project` определяет вещи о проекте.
Первый аргумент — это имя проекта, а второй — язык программирования.
- 3: функция `dependency` определяет зависимость, которая берётся с помощью `pkg-config`.
Мы помещаем `gtk4` в качестве аргумента.
- 5: функция `import` импортирует модуль.
В строке 5 импортируется модуль gnome и присваивается переменной `gnome`.
Модуль gnome предоставляет вспомогательные инструменты для построения программ GTK.
- 6: метод `.compile_resources` из модуля gnome.
Он компилирует файлы в ресурсы, как указано в xml-файле.
В строке 6 имя файла ресурса — `resources`, что означает `resources.c` и `resources.h`, а xml-файл — `tfe.gresource.xml`.
Этот метод по умолчанию генерирует исходный файл C.
- 8: определяет исходные файлы.
- 10: функция `executable` генерирует целевой файл путём компиляции исходных файлов.
Первый аргумент — это имя файла цели. Следующие аргументы — исходные файлы.
Последние два аргумента имеют ключи и значения.
Например, четвёртый аргумент имеет ключ `dependencies`, разделитель (`:`) и значение `gtkdep`.
Этот тип параметра называется *ключевым параметром* или *kwargs*.
Значение `gtkdep` определено в строке 3.
Последний аргумент сообщает Meson и Ninja, что этот проект не должен устанавливать исполняемый файл.
Поэтому он просто компилируется в каталоге сборки.

Теперь запустите meson и ninja.

    $ meson setup _build
    $ ninja -C _build

meson имеет два аргумента.

- setup: первый аргумент — это команда meson.
Setup является значением по умолчанию, поэтому вы можете его опустить.
Но рекомендуется писать его явно начиная с версии 0.64.0.
- Второй аргумент — это имя каталога сборки.

Затем исполняемый файл `tfe` генерируется в каталоге `_build`.

    $ _build/tfe tfe.c tfetextview.c

Появляется окно.
Оно включает блокнот с двумя страницами.
Одна — `tfe.c`, а другая — `tfetextview.c`.

Для получения дополнительной информации см. [The Meson Build system](https://mesonbuild.com/).

Up: [README.md](../README.md),  Prev: [Section 9](sec9.md), Next: [Section 11](sec11.md)
