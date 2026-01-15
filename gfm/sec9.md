Up: [README.md](../README.md),  Prev: [Section 8](sec8.md), Next: [Section 10](sec10.md)

# GtkBuilder и UI-файл

## Кнопки New, Open, Save и Close

В предыдущем разделе мы создали очень простой редактор.
Он читает файлы в начале и записывает их в конце программы.
Он работает, но не очень хорошо.
Было бы лучше, если бы у нас были кнопки "New", "Open", "Save" и "Close".
Этот раздел описывает, как поместить эти кнопки в окно.

![Screenshot of the file editor](../image/screenshot_tfe2.png)

Снимок экрана выше показывает макет.
Функция `app_open` в исходном коде `tfe2.c` выглядит следующим образом.

~~~C
 1 static void
 2 app_open (GApplication *app, GFile ** files, gint n_files, gchar *hint) {
 3   GtkWidget *win;
 4   GtkWidget *nb;
 5   GtkWidget *lab;
 6   GtkNotebookPage *nbp;
 7   GtkWidget *scr;
 8   GtkWidget *tv;
 9   GtkTextBuffer *tb;
10   char *contents;
11   gsize length;
12   char *filename;
13   int i;
14   GError *err = NULL;
15 
16   GtkWidget *boxv;
17   GtkWidget *boxh;
18   GtkWidget *dmy1;
19   GtkWidget *dmy2;
20   GtkWidget *dmy3;
21   GtkWidget *btnn; /* button for new */
22   GtkWidget *btno; /* button for open */
23   GtkWidget *btns; /* button for save */
24   GtkWidget *btnc; /* button for close */
25 
26   win = gtk_application_window_new (GTK_APPLICATION (app));
27   gtk_window_set_title (GTK_WINDOW (win), "file editor");
28   gtk_window_set_default_size (GTK_WINDOW (win), 600, 400);
29 
30   boxv = gtk_box_new (GTK_ORIENTATION_VERTICAL, 0);
31   gtk_window_set_child (GTK_WINDOW (win), boxv);
32 
33   boxh = gtk_box_new (GTK_ORIENTATION_HORIZONTAL, 0);
34   gtk_box_append (GTK_BOX (boxv), boxh);
35 
36   dmy1 = gtk_label_new(NULL); /* dummy label for left space */
37   gtk_label_set_width_chars (GTK_LABEL (dmy1), 10);
38   dmy2 = gtk_label_new(NULL); /* dummy label for center space */
39   gtk_widget_set_hexpand (dmy2, TRUE);
40   dmy3 = gtk_label_new(NULL); /* dummy label for right space */
41   gtk_label_set_width_chars (GTK_LABEL (dmy3), 10);
42   btnn = gtk_button_new_with_label ("New");
43   btno = gtk_button_new_with_label ("Open");
44   btns = gtk_button_new_with_label ("Save");
45   btnc = gtk_button_new_with_label ("Close");
46 
47   gtk_box_append (GTK_BOX (boxh), dmy1);
48   gtk_box_append (GTK_BOX (boxh), btnn);
49   gtk_box_append (GTK_BOX (boxh), btno);
50   gtk_box_append (GTK_BOX (boxh), dmy2);
51   gtk_box_append (GTK_BOX (boxh), btns);
52   gtk_box_append (GTK_BOX (boxh), btnc);
53   gtk_box_append (GTK_BOX (boxh), dmy3);
54 
55   nb = gtk_notebook_new ();
56   gtk_widget_set_hexpand (nb, TRUE);
57   gtk_widget_set_vexpand (nb, TRUE);
58   gtk_box_append (GTK_BOX (boxv), nb);
59 
60   for (i = 0; i < n_files; i++) {
61     if (g_file_load_contents (files[i], NULL, &contents, &length, NULL, &err)) {
62       scr = gtk_scrolled_window_new ();
63       tv = tfe_text_view_new ();
64       tb = gtk_text_view_get_buffer (GTK_TEXT_VIEW (tv));
65       gtk_text_view_set_wrap_mode (GTK_TEXT_VIEW (tv), GTK_WRAP_WORD_CHAR);
66       gtk_scrolled_window_set_child (GTK_SCROLLED_WINDOW (scr), tv);
67 
68       tfe_text_view_set_file (TFE_TEXT_VIEW (tv),  g_file_dup (files[i]));
69       gtk_text_buffer_set_text (tb, contents, length);
70       g_free (contents);
71       filename = g_file_get_basename (files[i]);
72       lab = gtk_label_new (filename);
73       gtk_notebook_append_page (GTK_NOTEBOOK (nb), scr, lab);
74       nbp = gtk_notebook_get_page (GTK_NOTEBOOK (nb), scr);
75       g_object_set (nbp, "tab-expand", TRUE, NULL);
76       g_free (filename);
77     } else {
78       g_printerr ("%s.\n", err->message);
79       g_clear_error (&err);
80     }
81   }
82   if (gtk_notebook_get_n_pages (GTK_NOTEBOOK (nb)) > 0) {
83     gtk_window_present (GTK_WINDOW (win));
84   } else
85     gtk_window_destroy (GTK_WINDOW (win));
86 }
~~~

Функция `app_open` строит виджеты в главном окне приложения.

- 26-28: создаёт экземпляр GtkApplicationWindow и устанавливает заголовок и размер по умолчанию.
- 30-31: создаёт экземпляр GtkBox `boxv`.
Это вертикальный контейнер и дочерний элемент GtkApplicationWindow.
У него два дочерних элемента.
Первый дочерний элемент — это горизонтальный контейнер.
Второй дочерний элемент — это GtkNotebook.
- 33-34: создаёт экземпляр GtkBox `boxh` и добавляет его к `boxv` как первый дочерний элемент.
- 36-41: создаёт три фиктивные метки.
Метки `dmy1` и `dmy3` имеют ширину в десять символов.
Другая метка `dmy2` имеет свойство hexpand, установленное в TRUE.
Это заставляет метку расширяться по горизонтали настолько, насколько это возможно.
- 42-45: создаёт четыре кнопки.
- 47-53: добавляет эти GtkLabel и GtkButton к `boxh`.
- 55-58: создаёт экземпляр GtkNotebook и устанавливает свойства hexpand и vexpand в TRUE.
Это заставляет его расширяться по горизонтали и вертикали, чтобы быть настолько большим, насколько это возможно.
Он добавляется к `boxv` как второй дочерний элемент.

Количество строк построения виджетов составляет 33(=58-26+1).
Нам также понадобилось много переменных (`boxv`, `boxh`, `dmy1`, ...), и большинство из них используются только для построения виджетов.
Есть ли какое-нибудь хорошее решение для сокращения этой работы?

Gtk предоставляет GtkBuilder.
Он читает данные пользовательского интерфейса (UI) и строит окно.
Это сокращает эту обременительную работу.

## UI-файл

Посмотрите на UI-файл `tfe3.ui`, который определяет структуру виджетов.

~~~xml
 1 <?xml version="1.0" encoding="UTF-8"?>
 2 <interface>
 3   <object class="GtkApplicationWindow" id="win">
 4     <property name="title">file editor</property>
 5     <property name="default-width">600</property>
 6     <property name="default-height">400</property>
 7     <child>
 8       <object class="GtkBox">
 9         <property name="orientation">GTK_ORIENTATION_VERTICAL</property>
10         <child>
11           <object class="GtkBox">
12             <property name="orientation">GTK_ORIENTATION_HORIZONTAL</property>
13             <child>
14               <object class="GtkLabel">
15                 <property name="width-chars">10</property>
16               </object>
17             </child>
18             <child>
19               <object class="GtkButton">
20                 <property name="label">New</property>
21               </object>
22             </child>
23             <child>
24               <object class="GtkButton">
25                 <property name="label">Open</property>
26               </object>
27             </child>
28             <child>
29               <object class="GtkLabel">
30                 <property name="hexpand">TRUE</property>
31               </object>
32             </child>
33             <child>
34               <object class="GtkButton">
35                 <property name="label">Save</property>
36               </object>
37             </child>
38             <child>
39               <object class="GtkButton">
40                 <property name="label">Close</property>
41               </object>
42             </child>
43             <child>
44               <object class="GtkLabel">
45                 <property name="width-chars">10</property>
46               </object>
47             </child>
48           </object>
49         </child>
50         <child>
51           <object class="GtkNotebook" id="nb">
52             <property name="hexpand">TRUE</property>
53             <property name="vexpand">TRUE</property>
54           </object>
55         </child>
56       </object>
57     </child>
58   </object>
59 </interface>
~~~

Это XML-файл.
Теги начинаются с `<` и заканчиваются на `>`.
Существует два типа тегов: открывающий тег и закрывающий тег.
Например, `<interface>` — это открывающий тег, а `</interface>` — это закрывающий тег.
UI-файл начинается и заканчивается тегами interface.
Некоторые теги, например, теги object, могут иметь атрибуты class и id в своём открывающем теге.

- 1: объявление XML.
Оно указывает, что версия XML — 1.0, а кодировка — UTF-8.
- 3-6: тег object с классом `GtkApplicationWindow` и идентификатором `win`.
Это окно верхнего уровня.
Он определяет три свойства:
свойство `title` — "file editor", свойство `default-width` — 600, а свойство `default-height` — 400.
- 7: тег child означает дочерний виджет.
Например, строка 7 говорит нам, что объект GtkBox является дочерним виджетом `win`.

Сравните этот ui-файл и строки 26-58 в функции `app_open` файла `tfe2.c`.
Оба строят одно и то же окно с его дочерними виджетами.

Вы можете проверить ui-файл с помощью `gtk4-builder-tool`.

- `gtk4-builder-tool validate <имя ui-файла>` проверяет ui-файл.
Если ui-файл содержит какую-либо синтаксическую ошибку, `gtk4-builder-tool` выводит ошибку.
- `gtk4-builder-tool simplify <имя ui-файла>` упрощает ui-файл и выводит результат.
Если задана опция `--replace`, он заменяет ui-файл упрощённым.
Если ui-файл указывает значение свойства по умолчанию, это свойство будет удалено.
Например, ориентация по умолчанию — горизонтальная, поэтому упрощение удаляет строку 12.
Некоторые значения также упрощаются.
Например, "TRUE" и "FALSE" становятся "1" и "0" соответственно.
Однако "TRUE" и "FALSE" лучше для поддержки.

Хорошая идея — проверить ваш ui-файл перед компиляцией.

## GtkBuilder

GtkBuilder строит виджеты на основе ui-файла.

~~~C
GtkBuilder *build;

build = gtk_builder_new_from_file ("tfe3.ui");
win = GTK_WIDGET (gtk_builder_get_object (build, "win"));
gtk_window_set_application (GTK_WINDOW (win), GTK_APPLICATION (app));
nb = GTK_WIDGET (gtk_builder_get_object (build, "nb"));
g_object_unref(build);
~~~

Функция `gtk_builder_new_from_file` читает файл `tfe3.ui`.
Затем она строит виджеты и создаёт объект GtkBuilder.
Все виджеты соединяются на основе отношения родитель-дети, описанного в ui-файле.
Мы можем извлечь объекты из объекта builder с помощью функции `gtk_builder_get_object`.
Окно верхнего уровня, которое имеет идентификатор "win" в ui-файле, извлекается и присваивается переменной `win`.
Свойство application окна устанавливается в `app` с помощью функции `gtk_window_set_application`.
GtkNotebook, который имеет идентификатор "nb" в ui-файле, также извлекается и присваивается переменной `nb`.
После того, как окно и приложение соединены, нам больше не нужен экземпляр GtkBuilder.
Он освобождается с помощью функции `g_object_unref`.

UI-файл сокращает строки в исходном файле C.

~~~
$ cd tfe; diff tfe2.c tfe3.c
59a60
>   GtkBuilder *build;
61,104c62,66
<   GtkWidget *boxv;
<   GtkWidget *boxh;
<   GtkWidget *dmy1;
<   GtkWidget *dmy2;
<   GtkWidget *dmy3;
<   GtkWidget *btnn; /* button for new */
<   GtkWidget *btno; /* button for open */
<   GtkWidget *btns; /* button for save */
<   GtkWidget *btnc; /* button for close */
< 
<   win = gtk_application_window_new (GTK_APPLICATION (app));
<   gtk_window_set_title (GTK_WINDOW (win), "file editor");
<   gtk_window_set_default_size (GTK_WINDOW (win), 600, 400);
< 
<   boxv = gtk_box_new (GTK_ORIENTATION_VERTICAL, 0);
<   gtk_window_set_child (GTK_WINDOW (win), boxv);
< 
<   boxh = gtk_box_new (GTK_ORIENTATION_HORIZONTAL, 0);
<   gtk_box_append (GTK_BOX (boxv), boxh);
< 
<   dmy1 = gtk_label_new(NULL); /* dummy label for left space */
<   gtk_label_set_width_chars (GTK_LABEL (dmy1), 10);
<   dmy2 = gtk_label_new(NULL); /* dummy label for center space */
<   gtk_widget_set_hexpand (dmy2, TRUE);
<   dmy3 = gtk_label_new(NULL); /* dummy label for right space */
<   gtk_label_set_width_chars (GTK_LABEL (dmy3), 10);
<   btnn = gtk_button_new_with_label ("New");
<   btno = gtk_button_new_with_label ("Open");
<   btns = gtk_button_new_with_label ("Save");
<   btnc = gtk_button_new_with_label ("Close");
< 
<   gtk_box_append (GTK_BOX (boxh), dmy1);
<   gtk_box_append (GTK_BOX (boxh), btnn);
<   gtk_box_append (GTK_BOX (boxh), btno);
<   gtk_box_append (GTK_BOX (boxh), dmy2);
<   gtk_box_append (GTK_BOX (boxh), btns);
<   gtk_box_append (GTK_BOX (boxh), btnc);
<   gtk_box_append (GTK_BOX (boxh), dmy3);
< 
<   nb = gtk_notebook_new ();
<   gtk_widget_set_hexpand (nb, TRUE);
<   gtk_widget_set_vexpand (nb, TRUE);
<   gtk_box_append (GTK_BOX (boxv), nb);
< 
---
>   build = gtk_builder_new_from_file ("tfe3.ui");
>   win = GTK_WIDGET (gtk_builder_get_object (build, "win"));
>   gtk_window_set_application (GTK_WINDOW (win), GTK_APPLICATION (app));
>   nb = GTK_WIDGET (gtk_builder_get_object (build, "nb"));
>   g_object_unref(build);
138c100
<   app = gtk_application_new ("com.github.ToshioCP.tfe2", G_APPLICATION_HANDLES_OPEN);
---
>   app = gtk_application_new ("com.github.ToshioCP.tfe3", G_APPLICATION_HANDLES_OPEN);
144a107
> 
~~~

`61,104c62,66` означает, что 44 (=104-61+1) строки изменены на 5 (=66-62+1) строк.
Следовательно, 39 строк сокращены.
Использование ui-файла не только сокращает исходные файлы C, но и делает структуру виджетов более ясной.

Теперь я покажу вам функцию `app_open` в файле C `tfe3.c`.

~~~C
 1 static void
 2 app_open (GApplication *app, GFile ** files, gint n_files, gchar *hint) {
 3   GtkWidget *win;
 4   GtkWidget *nb;
 5   GtkWidget *lab;
 6   GtkNotebookPage *nbp;
 7   GtkWidget *scr;
 8   GtkWidget *tv;
 9   GtkTextBuffer *tb;
10   char *contents;
11   gsize length;
12   char *filename;
13   int i;
14   GError *err = NULL;
15   GtkBuilder *build;
16 
17   build = gtk_builder_new_from_file ("tfe3.ui");
18   win = GTK_WIDGET (gtk_builder_get_object (build, "win"));
19   gtk_window_set_application (GTK_WINDOW (win), GTK_APPLICATION (app));
20   nb = GTK_WIDGET (gtk_builder_get_object (build, "nb"));
21   g_object_unref(build);
22   for (i = 0; i < n_files; i++) {
23     if (g_file_load_contents (files[i], NULL, &contents, &length, NULL, &err)) {
24       scr = gtk_scrolled_window_new ();
25       tv = tfe_text_view_new ();
26       tb = gtk_text_view_get_buffer (GTK_TEXT_VIEW (tv));
27       gtk_text_view_set_wrap_mode (GTK_TEXT_VIEW (tv), GTK_WRAP_WORD_CHAR);
28       gtk_scrolled_window_set_child (GTK_SCROLLED_WINDOW (scr), tv);
29 
30       tfe_text_view_set_file (TFE_TEXT_VIEW (tv),  g_file_dup (files[i]));
31       gtk_text_buffer_set_text (tb, contents, length);
32       g_free (contents);
33       filename = g_file_get_basename (files[i]);
34       lab = gtk_label_new (filename);
35       gtk_notebook_append_page (GTK_NOTEBOOK (nb), scr, lab);
36       nbp = gtk_notebook_get_page (GTK_NOTEBOOK (nb), scr);
37       g_object_set (nbp, "tab-expand", TRUE, NULL);
38       g_free (filename);
39     } else {
40       g_printerr ("%s.\n", err->message);
41       g_clear_error (&err);
42     }
43   }
44   if (gtk_notebook_get_n_pages (GTK_NOTEBOOK (nb)) > 0) {
45     gtk_window_present (GTK_WINDOW (win));
46   } else
47     gtk_window_destroy (GTK_WINDOW (win));
48 }
~~~

Полный исходный код `tfe3.c` хранится в директории [src/tfe](../src/tfe).

### Использование ui-строки

GtkBuilder может строить виджеты с помощью строки.
Используйте `gtk_builder_new_from_string` вместо `gtk_builder_new_from_file`.

~~~C
char *uistring;

uistring =
"<interface>"
  "<object class=\"GtkApplicationWindow\" id=\"win\">"
    "<property name=\"title\">file editor</property>"
    "<property name=\"default-width\">600</property>"
    "<property name=\"default-height\">400</property>"
    "<child>"
      "<object class=\"GtkBox\">"
        "<property name=\"orientation\">GTK_ORIENTATION_VERTICAL</property>"
... ... ...
... ... ...
"</interface>";

build = gtk_builder_new_from_string (uistring, -1);
~~~

Этот метод имеет преимущество и недостаток.
Преимущество в том, что ui-строка записана в исходном коде.
Таким образом, ui-файл не нужен во время выполнения.
Недостаток в том, что написание строки C немного обременительно,
поскольку xml требует кавычек, а специальные символы требуют экранирования.
Если вы хотите использовать этот метод, вам следует написать скрипт, который преобразует ui-файлы в C-строки.

- Замените обратные слэши двумя обратными слэшами.
- Добавьте обратный слэш перед каждой двойной кавычкой.
- Добавьте двойные кавычки слева и справа от строки в каждой строке.

Или, если у вас установлен `jq`, вы можете использовать `jq -R < tfe3.ui` для выполнения кавычек и экранирования.

### Gresource

Gresource похож на строку, за исключением того, что Gresource — это сжатые двоичные данные, а не текстовые данные.
Программа `glib-compile-resources` компилирует ui-файлы в Gresources.
Она может компилировать не только текстовые файлы, но и двоичные файлы, такие как изображения, звуки и так далее.
После компиляции она объединяет их в один объект Gresource.

XML-файл необходим для компилятора ресурсов `glib-compile-resources`.
Он описывает файлы ресурсов.

~~~xml
1 <?xml version="1.0" encoding="UTF-8"?>
2 <gresources>
3   <gresource prefix="/com/github/ToshioCP/tfe3">
4     <file>tfe3.ui</file>
5   </gresource>
6 </gresources>
~~~

- 2: тег `gresources` может включать несколько gresources (теги gresource).
Однако этот xml имеет только один gresource.
- 3: gresource имеет префикс `/com/github/ToshioCP/tfe3`.
- 4: имя gresource — `tfe3.ui`.
На ресурс будет указывать GtkBuilder с помощью `/com/github/ToshioCP/tfe3/tfe3.ui`.
Шаблон — "префикс" + "имя".
Если вы хотите добавить больше файлов, вставьте их между строками 4 и 5.

Сохраните этот xml-текст в `tfe3.gresource.xml`.
Компилятор gresource `glib-compile-resources` показывает своё использование с аргументом `--help`.

```
$ glib-compile-resources --help
Usage:
  glib-compile-resources [OPTION..] FILE

Compile a resource specification into a resource file.
Resource specification files have the extension .gresource.xml,
and the resource file have the extension called .gresource.

Help Options:
  -h, --help                   Show help options

Application Options:
  --version                    Show program version and exit
  --target=FILE                Name of the output file
  --sourcedir=DIRECTORY        The directories to load files referenced in FILE from (default: current directory)
  --generate                   Generate output in the format selected for by the target filename extension
  --generate-header            Generate source header
  --generate-source            Generate source code used to link in the resource file into your code
  --generate-dependencies      Generate dependency list
  --dependency-file=FILE       Name of the dependency file to generate
  --generate-phony-targets     Include phony targets in the generated dependency file
  --manual-register            Don't automatically create and register resource
  --internal                   Don't export functions; declare them G_GNUC_INTERNAL
  --external-data              Don't embed resource data in the C file; assume it's linked externally instead
  --c-name                     C identifier name used for the generated source code
  -C, --compiler               The target C compiler (default: the CC environment variable)
  ```

Теперь запустите компилятор.

~~~
$ glib-compile-resources tfe3.gresource.xml --target=resources.c --generate-source
~~~

Затем генерируется исходный файл C `resources.c`.
Измените `tfe3.c` и сохраните его как `tfe3_r.c`.

~~~C
#include "resources.c"
... ... ...
... ... ...
build = gtk_builder_new_from_resource ("/com/github/ToshioCP/tfe3/tfe3.ui");
... ... ...
... ... ...
~~~

Функция `gtk_builder_new_from_resource` строит виджеты из ресурса.

Затем скомпилируйте и запустите.

~~~
$ comp tfe3_r
$ ./a.out tfe2.c
~~~

Появляется окно, и оно такое же, как снимок экрана в начале этой страницы.

Вообще говоря, ресурсы лучше всего подходят для программ на C.
Если вы используете другие языки, такие как Ruby, строки лучше, чем ресурсы.

Up: [README.md](../README.md),  Prev: [Section 8](sec8.md), Next: [Section 10](sec10.md)
