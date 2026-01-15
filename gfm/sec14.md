Up: [README.md](../README.md),  Prev: [Section 13](sec13.md), Next: [Section 15](sec15.md)

# Функции в GtkNotebook

GtkNotebook — это очень важный объект в текстовом редакторе файлов `tfe`.
Он соединяет приложение и объекты TfeTextView.
Набор публичных функций объявлен в `tfenotebook.h`.
Слово "tfenotebook" используется только в именах файлов.
Нет объекта "TfeNotebook".

Исходные файлы находятся в каталоге `src/tfe5`.
Вы можете получить их, загрузив [репозиторий](https://github.com/ToshioCP/Gtk4-tutorial).

~~~C
 1 void
 2 notebook_page_save(GtkNotebook *nb);
 3 
 4 void
 5 notebook_page_close (GtkNotebook *nb);
 6 
 7 void
 8 notebook_page_open (GtkNotebook *nb);
 9 
10 void
11 notebook_page_new_with_file (GtkNotebook *nb, GFile *file);
12 
13 void
14 notebook_page_new (GtkNotebook *nb);
15 
~~~

Этот заголовочный файл описывает публичные функции в `tfenotebook.c`.

- 1-2: `notebook_page_save` сохраняет текущую страницу в файл, имя которого указано на вкладке.
Если имя `untitled` или `untitled` с последующими цифрами, появляется диалог выбора файла, и пользователь может выбрать или указать имя файла.
- 4-5: `notebook_page_close` закрывает текущую страницу.
- 7-8: `notebook_page_open` показывает диалог выбора файла, и пользователь может выбрать файл. Содержимое файла вставляется на новую страницу.
- 10-11: `notebook_page_new_with_file` создаёт новую страницу, и файл, переданный в качестве аргумента, читается и вставляется на страницу.
- 13-14: `notebook_page_new` создаёт новую пустую страницу.

Вы, вероятно, обнаружите, что функции, за исключением `notebook_page_close`, являются функциями более высокого уровня для

- `tfe_text_view_save`
- `tef_text_view_open`
- `tfe_text_view_new_with_file`
- `tfe_text_view_new`

соответственно.

Есть два уровня.
Один из них — `tfe_text_view ...`, который является уровнем нижнего уровня.
Другой — `notebook ...`, который является уровнем более высокого уровня.

Теперь давайте посмотрим на программу каждой функции.

## notebook\_page\_new

~~~C
 1 static char*
 2 get_untitled () {
 3   static int c = -1;
 4   if (++c == 0) 
 5     return g_strdup_printf("Untitled");
 6   else
 7     return g_strdup_printf ("Untitled%u", c);
 8 }
 9 
10 static void
11 notebook_page_build (GtkNotebook *nb, GtkWidget *tv, const char *filename) {
12   GtkWidget *scr = gtk_scrolled_window_new ();
13   GtkNotebookPage *nbp;
14   GtkWidget *lab;
15   int i;
16 
17   gtk_scrolled_window_set_child (GTK_SCROLLED_WINDOW (scr), tv);
18   lab = gtk_label_new (filename);
19   i = gtk_notebook_append_page (nb, scr, lab);
20   nbp = gtk_notebook_get_page (nb, scr);
21   g_object_set (nbp, "tab-expand", TRUE, NULL);
22   gtk_notebook_set_current_page (nb, i);
23   g_signal_connect (GTK_TEXT_VIEW (tv), "change-file", G_CALLBACK (file_changed_cb), nb);
24 }
25 
26 void
27 notebook_page_new (GtkNotebook *nb) {
28   g_return_if_fail(GTK_IS_NOTEBOOK (nb));
29 
30   GtkWidget *tv;
31   char *filename;
32 
33   tv = tfe_text_view_new ();
34   filename = get_untitled ();
35   notebook_page_build (nb, tv, filename);
36   g_free (filename);
37 }
~~~

- 26-37: функция `notebook_page_new`.
- 28: функция `g_return_if_fail` проверяет аргумент. Это необходимо, потому что функция является публичной.
- 33: создаёт объект TfeTextView.
- 34: создаёт имя файла, которое будет "Untitled", "Untitled1", ... .
- 1-8: функция `get_untitled`.
- 3: статическая переменная `c` инициализируется при первом вызове этой функции. После этого `c` сохраняет своё значение, если оно не изменено явно.
- 4-7: увеличивает `c` на единицу, и если оно равно нулю, возвращает "Untitled". Если это положительное целое число, возвращает "Untitled\<целое число\>", например, "Untitled1", "Untitled2" и так далее.
Функция `g_strdup_printf` создаёт строку, и она должна быть освобождена с помощью `g_free`, когда станет ненужной.
Вызывающая сторона `get_untitled` отвечает за освобождение строки.
- 36: вызывает `notebook_page_build` для построения новой страницы.
- 37: освобождает `filename`.
- 10-24: функция `notebook_page_build`.
Параметр с квалификатором `const` не изменяется в функции.
Это означает, что аргумент `filename` принадлежит вызывающей стороне.
Вызывающая сторона должна освободить его, когда он станет ненужным.
- 12: создаёт GtkScrolledWindow.
- 17: вставляет `tv` в GtkScrolledWindow в качестве дочернего элемента.
- 18-19: создаёт GtkLabel, затем добавляет `scr` и `lab` к экземпляру GtkNotebook `nb`.
- 20-21: устанавливает свойство "tab-expand" в TRUE.
Функция `g_object_set` устанавливает свойства объекта.
Объект может быть любым объектом, производным от GObject.
Во многих случаях у объекта есть собственная функция для установки его свойств, но иногда её нет.
В этом случае используйте `g_object_set` для установки свойства.
- 22: устанавливает текущую страницу на вновь созданную страницу.
- 23: подключает сигнал "change-file" и обработчик `file_changed_cb`.

## notebook\_page\_new\_with\_file

~~~C
 1 void
 2 notebook_page_new_with_file (GtkNotebook *nb, GFile *file) {
 3   g_return_if_fail(GTK_IS_NOTEBOOK (nb));
 4   g_return_if_fail(G_IS_FILE (file));
 5 
 6   GtkWidget *tv;
 7   char *filename;
 8 
 9   if ((tv = tfe_text_view_new_with_file (file)) == NULL)
10     return; /* read error */
11   filename = g_file_get_basename (file);
12   notebook_page_build (nb, tv, filename);
13   g_free (filename);
14 }
~~~

- 9-10: вызывает `tfe_text_view_new_with_file`.
Если функция возвращает NULL, произошла ошибка.
Тогда она ничего не делает и возвращает управление.
- 11-13: получает имя файла, строит новую страницу и освобождает `filename`.

## notebook\_page\_open

~~~C
 1 static void
 2 open_response_cb (TfeTextView *tv, int response, GtkNotebook *nb) {
 3   GFile *file;
 4   char *filename;
 5 
 6   if (response != TFE_OPEN_RESPONSE_SUCCESS) {
 7     g_object_ref_sink (tv);
 8     g_object_unref (tv);
 9   }else {
10     file = tfe_text_view_get_file (tv);
11     filename = g_file_get_basename (file);
12     g_object_unref (file);
13     notebook_page_build (nb, GTK_WIDGET (tv), filename);
14     g_free (filename);
15   }
16 }
17 
18 void
19 notebook_page_open (GtkNotebook *nb) {
20   g_return_if_fail(GTK_IS_NOTEBOOK (nb));
21 
22   GtkWidget *tv;
23 
24   tv = tfe_text_view_new ();
25   g_signal_connect (TFE_TEXT_VIEW (tv), "open-response", G_CALLBACK (open_response_cb), nb);
26   tfe_text_view_open (TFE_TEXT_VIEW (tv), GTK_WINDOW (gtk_widget_get_ancestor (GTK_WIDGET (nb), GTK_TYPE_WINDOW)));
27 }
~~~

- 18-27: функция `notebook_page_open`.
- 24: создаёт объект TfeTextView.
- 25: подключает сигнал "open-response" и обработчик `open_response_cb`.
- 26: вызывает `tfe_text_view_open`.
Сигнал "open-response" будет испущен позже в этой функции для информирования о результате.
- 1-16: обработчик `open_response_cb`.
- 6-8: если код ответа не `TFE_OPEN_RESPONSE_SUCCESS`, экземпляр `tv` будет уничтожен.
Он имеет плавающую ссылку, которая будет объяснена позже.
Плавающую ссылку необходимо преобразовать в обычную ссылку перед её освобождением.
Функция `g_object_ref_sink` делает это.
После этого функция `g_object_unref` освобождает `tv` и уменьшает счётчик ссылок на единицу.
В конечном итоге счётчик ссылок становится нулевым, и `tv` уничтожается.
- 9-15: в противном случае строится новая страница с `tv`.

## Плавающая ссылка

Все виджеты являются производными от GInitiallyUnowned.
GObject и GInitiallyUnowned почти одинаковы.
Разница вот в чём.
Когда создаётся экземпляр GInitiallyUnowned, экземпляр имеет "плавающую ссылку".
С другой стороны, когда создаётся экземпляр GObject (не GInitiallyUnowned), он имеет "нормальную ссылку".
Их потомки наследуют их, поэтому каждый виджет имеет плавающую ссылку сразу после создания.
Класс, не являющийся виджетом, например, GtkTextBuffer, является прямым подклассом GObject и имеет нормальную ссылку.

Функция `g_object_ref_sink` преобразует плавающую ссылку в нормальную ссылку.
Если экземпляр не имеет плавающей ссылки, `g_object_ref_sink` просто увеличивает счётчик ссылок на единицу.
Она используется, когда виджет добавляется к другому виджету в качестве дочернего элемента.

~~~
GtkTextView *tv = gtk_text_view_new (); // Плавающая ссылка
GtkScrolledWindow *scr = gtk_scrolled_window_new ();
gtk_scrolled_window_set_child (scr, tv); // Scrolled window поглощает плавающую ссылку tv, и счётчик ссылок tv становится единицей.
~~~

Когда `tv` добавляется к `scr` в качестве дочернего элемента, используется `g_object_ref_sink`.

~~~
g_object_ref_sink (tv);
~~~

Таким образом, плавающая ссылка преобразуется в обычную ссылку.
То есть, плавающая ссылка удаляется, и нормальный счётчик ссылок становится единицей.
Благодаря этому вызывающей стороне не нужно уменьшать счётчик ссылок tv.
Если Object\_A не является потомком GInitiallyUnowned, программа выглядит так:

~~~
Object_A *obj_a = object_a_new (); // счётчик ссылок равен единице
GtkScrolledWindow *scr = gtk_scrolled_window_new ();
gtk_scrolled_window_set_child (scr, obj_a); // счётчик ссылок obj_a равен двум
// на obj_a ссылается вызывающая сторона (эта программа) и scrolled window
g_object_unref (obj_a); // счётчик ссылок obj_a равен единице, потому что вызывающая сторона больше не ссылается на obj_a.
~~~

Этот пример показывает нам, что вызывающей стороне нужно выполнить unref для `obj_a`.

Если вы используете `g_object_unref` для экземпляра, имеющего плавающую ссылку, вам нужно предварительно преобразовать плавающую ссылку в нормальную ссылку.
Для получения дополнительной информации см. [GObject API reference](https://docs.gtk.org/gobject/floating-refs.html).

## notebook\_page\_close

~~~C
 1 void
 2 notebook_page_close (GtkNotebook *nb) {
 3   g_return_if_fail(GTK_IS_NOTEBOOK (nb));
 4 
 5   GtkWidget *win;
 6   int i;
 7 
 8   if (gtk_notebook_get_n_pages (nb) == 1) {
 9     win = gtk_widget_get_ancestor (GTK_WIDGET (nb), GTK_TYPE_WINDOW);
10     gtk_window_destroy(GTK_WINDOW (win));
11   } else {
12     i = gtk_notebook_get_current_page (nb);
13     gtk_notebook_remove_page (GTK_NOTEBOOK (nb), i);
14   }
15 }
~~~

Эта функция закрывает текущую страницу.
Если страница является единственной страницей в блокноте, то функция уничтожает окно верхнего уровня и завершает приложение.

- 8-10: если страница является единственной страницей в блокноте, она вызывает `gtk_window_destroy` для уничтожения окна верхнего уровня.
- 11-13: в противном случае удаляет текущую страницу.
Дочерний виджет (TfeTextView) также уничтожается.

## notebook\_page\_save

~~~C
 1 static TfeTextView *
 2 get_current_textview (GtkNotebook *nb) {
 3   int i;
 4   GtkWidget *scr;
 5   GtkWidget *tv;
 6 
 7   i = gtk_notebook_get_current_page (nb);
 8   scr = gtk_notebook_get_nth_page (nb, i);
 9   tv = gtk_scrolled_window_get_child (GTK_SCROLLED_WINDOW (scr));
10   return TFE_TEXT_VIEW (tv);
11 }
12 
13 void
14 notebook_page_save (GtkNotebook *nb) {
15   g_return_if_fail(GTK_IS_NOTEBOOK (nb));
16 
17   TfeTextView *tv;
18 
19   tv = get_current_textview (nb);
20   tfe_text_view_save (tv);
21 }
~~~

- 13-21: `notebook_page_save`.
- 19: получает экземпляр TfeTextView, принадлежащий текущей странице.
Вызывающая сторона не владеет `tv`, поэтому вам не нужно заботиться об его освобождении.
- 20: вызывает `tfe_text_view_save`.
- 1-11: `get_current_textview`.
Эта функция получает объект TfeTextView, принадлежащий текущей странице.
- 7: получает номер страницы текущей страницы.
- 8: получает дочерний виджет `scr`, который является экземпляром GtkScrolledWindow, текущей страницы. Объект `scr` принадлежит блокноту `nb`. Поэтому вызывающей стороне не нужно освобождать его.
- 9-10: получает дочерний виджет `scr`, который является экземпляром TfeTextView, и возвращает его.
Возвращённый экземпляр принадлежит `scr`, и вызывающей стороне `get_cuurent_textview` не нужно заботиться об его освобождении.

## Обработчик file\_changed\_cb

Функция `file_changed_cb` — это обработчик, подключённый к сигналу "change-file".
Если файл в экземпляре TfeTextView изменяется, экземпляр испускает этот сигнал.
Этот обработчик изменяет метку GtkNotebookPage.

~~~C
 1 static void
 2 file_changed_cb (TfeTextView *tv, GtkNotebook *nb) {
 3   GtkWidget *scr;
 4   GtkWidget *label;
 5   GFile *file;
 6   char *filename;
 7 
 8   file = tfe_text_view_get_file (tv);
 9   scr = gtk_widget_get_parent (GTK_WIDGET (tv));
10   if (G_IS_FILE (file)) {
11     filename = g_file_get_basename (file);
12     g_object_unref (file);
13   } else
14     filename = get_untitled ();
15   label = gtk_label_new (filename);
16   g_free (filename);
17   gtk_notebook_set_tab_label (nb, scr, label);
18 }
~~~

- 8: получает экземпляр GFile из `tv`.
- 9: получает экземпляр GkScrolledWindow, который является родительским виджетом `tv`.
- 10-12: если `file` указывает на экземпляр GFile, имя файла GFile присваивается `filename`.
Затем освобождает объект GFile `file`.
- 13-14: в противном случае (file равен NULL) строка `Untitled(число)` присваивается `filename`.
- 15-17: создаёт экземпляр GtkLabel `label` с именем файла и устанавливает метку GtkNotebookPage с `label`.

Up: [README.md](../README.md),  Prev: [Section 13](sec13.md), Next: [Section 15](sec15.md)
