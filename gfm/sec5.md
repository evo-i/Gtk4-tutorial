Up: [README.md](../README.md),  Prev: [Section 4](sec4.md), Next: [Section 6](sec6.md)

# Виджеты (2)

## GtkTextView, GtkTextBuffer и GtkScrolledWindow

### GtkTextView и GtkTextBuffer

GtkTextView — это виджет для многострочного редактирования текста.
GtkTextBuffer — это текстовый буфер, который подключен к GtkTextView.
Посмотрите пример программы `tfv1.c` ниже.

~~~C
 1 #include <gtk/gtk.h>
 2 
 3 static void
 4 app_activate (GApplication *app) {
 5   GtkWidget *win;
 6   GtkWidget *tv;
 7   GtkTextBuffer *tb;
 8   gchar *text;
 9 
10   text =
11       "Once upon a time, there was an old man who was called Taketori-no-Okina. "
12       "It is a japanese word that means a man whose work is making bamboo baskets.\n"
13       "One day, he went into a hill and found a shining bamboo. "
14       "\"What a mysterious bamboo it is!,\" he said. "
15       "He cut it, then there was a small cute baby girl in it. "
16       "The girl was shining faintly. "
17       "He thought this baby girl is a gift from Heaven and took her home.\n"
18       "His wife was surprized at his story. "
19       "They were very happy because they had no children. "
20       ;
21   win = gtk_application_window_new (GTK_APPLICATION (app));
22   gtk_window_set_title (GTK_WINDOW (win), "Taketori");
23   gtk_window_set_default_size (GTK_WINDOW (win), 400, 300);
24 
25   tv = gtk_text_view_new ();
26   tb = gtk_text_view_get_buffer (GTK_TEXT_VIEW (tv));
27   gtk_text_buffer_set_text (tb, text, -1);
28   gtk_text_view_set_wrap_mode (GTK_TEXT_VIEW (tv), GTK_WRAP_WORD_CHAR);
29 
30   gtk_window_set_child (GTK_WINDOW (win), tv);
31 
32   gtk_window_present (GTK_WINDOW (win));
33 }
34 
35 int
36 main (int argc, char **argv) {
37   GtkApplication *app;
38   int stat;
39 
40   app = gtk_application_new ("com.github.ToshioCP.tfv1", G_APPLICATION_DEFAULT_FLAGS);
41   g_signal_connect (app, "activate", G_CALLBACK (app_activate), NULL);
42   stat = g_application_run (G_APPLICATION (app), argc, argv);
43   g_object_unref (app);
44   return stat;
45 }
~~~

Посмотрите на строку 25.
Создается экземпляр GtkTextView, и его указатель присваивается `tv`.
Когда создается экземпляр GtkTextView, экземпляр GtkTextBuffer также создается и подключается к GtkTextView автоматически.
"Экземпляр GtkTextBuffer" будет упоминаться просто как "GtkTextBuffer" или "буфер".
В следующей строке указатель на буфер присваивается `tb`.
Затем текст со строки 10 по 20 присваивается буферу.
Если третий аргумент `gtk_text_buffer_set_text` является положительным целым числом, это длина текста.
Если это -1, строка заканчивается NULL.

GtkTextView имеет режим переноса.
Когда он установлен в `GTK_WRAP_WORD_CHAR`, текст переносится между словами, или, если этого недостаточно, также между графемами.

Режим переноса описан в [Gtk\_WrapMode](https://docs.gtk.org/gtk4/enum.WrapMode.html) в документации GTK 4 API.

В строке 30 `tv` добавляется в `win` как дочерний элемент.

Теперь скомпилируйте и запустите программу.
Если вы загрузили этот репозиторий, его путь — `src/tfv/tfv1.c`.

```
$ cd src/tfv
$ comp tfv1
$ ./a.out
```

![GtkTextView](../image/screenshot_tfv1.png)

В окне есть I-образный указатель.
Вы можете добавлять или удалять любые символы в GtkTextView, и ваши изменения сохраняются в GtkTextBuffer.
Если вы добавите больше символов за пределы окна, высота увеличится, и окно расширится.
Если высота станет больше высоты экрана,
вы не сможете контролировать размер окна или вернуть его к исходному размеру.
Это проблема, то есть ошибка.
Это можно решить, добавив GtkScrolledWindow между GtkApplicationWindow и GtkTextView.

### GtkScrolledWindow

Что нам нужно сделать:

- Создать GtkScrolledWindow и вставить его как дочерний элемент GtkApplicationWindow
- Вставить виджет GtkTextView в GtkScrolledWindow как дочерний элемент.

Модифицируйте `tfv1.c` и сохраните его как `tfv2.c`.
Между этими двумя файлами есть лишь несколько различий.

~~~
$ cd tfv; diff tfv1.c tfv2.c
5a6
>   GtkWidget *scr;
24a26,28
>   scr = gtk_scrolled_window_new ();
>   gtk_window_set_child (GTK_WINDOW (win), scr);
> 
30c34
<   gtk_window_set_child (GTK_WINDOW (win), tv);
---
>   gtk_scrolled_window_set_child (GTK_SCROLLED_WINDOW (scr), tv);
40c44
<   app = gtk_application_new ("com.github.ToshioCP.tfv1", G_APPLICATION_DEFAULT_FLAGS);
---
>   app = gtk_application_new ("com.github.ToshioCP.tfv2", G_APPLICATION_DEFAULT_FLAGS);
~~~

Полный код `tfv2.c` следующий.

~~~C
 1 #include <gtk/gtk.h>
 2 
 3 static void
 4 app_activate (GApplication *app) {
 5   GtkWidget *win;
 6   GtkWidget *scr;
 7   GtkWidget *tv;
 8   GtkTextBuffer *tb;
 9   gchar *text;
10 
11   text =
12       "Once upon a time, there was an old man who was called Taketori-no-Okina. "
13       "It is a japanese word that means a man whose work is making bamboo baskets.\n"
14       "One day, he went into a hill and found a shining bamboo. "
15       "\"What a mysterious bamboo it is!,\" he said. "
16       "He cut it, then there was a small cute baby girl in it. "
17       "The girl was shining faintly. "
18       "He thought this baby girl is a gift from Heaven and took her home.\n"
19       "His wife was surprized at his story. "
20       "They were very happy because they had no children. "
21       ;
22   win = gtk_application_window_new (GTK_APPLICATION (app));
23   gtk_window_set_title (GTK_WINDOW (win), "Taketori");
24   gtk_window_set_default_size (GTK_WINDOW (win), 400, 300);
25 
26   scr = gtk_scrolled_window_new ();
27   gtk_window_set_child (GTK_WINDOW (win), scr);
28 
29   tv = gtk_text_view_new ();
30   tb = gtk_text_view_get_buffer (GTK_TEXT_VIEW (tv));
31   gtk_text_buffer_set_text (tb, text, -1);
32   gtk_text_view_set_wrap_mode (GTK_TEXT_VIEW (tv), GTK_WRAP_WORD_CHAR);
33 
34   gtk_scrolled_window_set_child (GTK_SCROLLED_WINDOW (scr), tv);
35 
36   gtk_window_present (GTK_WINDOW (win));
37 }
38 
39 int
40 main (int argc, char **argv) {
41   GtkApplication *app;
42   int stat;
43 
44   app = gtk_application_new ("com.github.ToshioCP.tfv2", G_APPLICATION_DEFAULT_FLAGS);
45   g_signal_connect (app, "activate", G_CALLBACK (app_activate), NULL);
46   stat = g_application_run (G_APPLICATION (app), argc, argv);
47   g_object_unref (app);
48   return stat;
49 }
~~~

Скомпилируйте и запустите программу.

Теперь окно не расширяется, даже если вы введете много символов,
оно просто прокручивается.

Up: [README.md](../README.md),  Prev: [Section 4](sec4.md), Next: [Section 6](sec6.md)
