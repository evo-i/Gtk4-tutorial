Up: [README.md](../README.md),  Prev: [Section 12](sec12.md), Next: [Section 14](sec14.md)

# Класс TfeTextView

Класс TfeTextView будет окончательно завершён в этом разделе.
Оставшаяся тема - функции.
Функции TfeTextView, которые являются конструкторами и методами экземпляра, описаны в этом разделе.

Исходные файлы находятся в каталоге `src/tfetextview`.
Вы можете получить их, скачав [репозиторий](https://github.com/ToshioCP/Gtk4-tutorial).

## tfetextview.h

Заголовочный файл `tfetextview.h` предоставляет:

- Тип TfeTextView, который является `TFE_TYPE_TEXT_VIEW`.
- Макрос `G_DECLARE_FINAL_TYPE`, раскрытие которого включает некоторые полезные функции и определения.
- Константы для сигнала `open-response`.
- Публичные функции из `tfetextview.c`. Это конструкторы и методы экземпляра.

Поэтому любые программы, использующие TfeTextView, должны включать `tfetextview.h`.

~~~C
 1 #pragma once
 2 
 3 #include <gtk/gtk.h>
 4 
 5 #define TFE_TYPE_TEXT_VIEW tfe_text_view_get_type ()
 6 G_DECLARE_FINAL_TYPE (TfeTextView, tfe_text_view, TFE, TEXT_VIEW, GtkTextView)
 7 
 8 /* "open-response" signal response */
 9 enum TfeTextViewOpenResponseType
10 {
11   TFE_OPEN_RESPONSE_SUCCESS,
12   TFE_OPEN_RESPONSE_CANCEL,
13   TFE_OPEN_RESPONSE_ERROR
14 };
15 
16 GFile *
17 tfe_text_view_get_file (TfeTextView *tv);
18 
19 void
20 tfe_text_view_open (TfeTextView *tv, GtkWindow *win);
21 
22 void
23 tfe_text_view_save (TfeTextView *tv);
24 
25 void
26 tfe_text_view_saveas (TfeTextView *tv);
27 
28 GtkWidget *
29 tfe_text_view_new_with_file (GFile *file);
30 
31 GtkWidget *
32 tfe_text_view_new (void);
~~~

- 1: Директива препроцессора `#pragma once` делает так, что заголовочный файл включается только один раз.
Она нестандартная, но широко используется.
- 3: Включает заголовочные файлы gtk4.
Заголовочный файл `gtk4` также имеет такой же механизм, чтобы избежать множественного включения.
- 5-6: Эти две строки определяют тип TfeTextView, его структуру класса и некоторые полезные определения.
  - `TfeTextView` и `TfeTextViewClass` объявлены как typedef структур C.
  - Вам нужно будет определить структуру `_TfeTextView` позже.
  - Структура класса `_TfeTextViewClass` определена здесь. Вам не нужно определять её самостоятельно.
  - Определены вспомогательные функции `TFE_TEXT_VIEW ()` для приведения типа и `TFE_IS_TEXT_VIEW` для проверки типа.
- 8-14: Определение значений параметров сигнала "open-response".
- 16-32: Объявления публичных функций для TfeTextView.

## Конструкторы

Экземпляр TfeTextView создаётся с помощью `tfe_text_view_new` или `tfe_text_view_new_with_file`.
Эти функции называются конструкторами.

~~~C
GtkWidget *tfe_text_view_new (void);
~~~

Она просто создаёт новый экземпляр TfeTextView и возвращает указатель на новый экземпляр.

~~~C
GtkWidget *tfe_text_view_new_with_file (GFile *file);
~~~

Ей передаётся объект GFile в качестве аргумента, и она загружает файл в экземпляр GtkTextBuffer, затем возвращает указатель на новый экземпляр.
Аргумент `file` принадлежит вызывающей стороне, и функция его не изменяет.
Если во время процесса создания произойдёт ошибка, будет возвращено NULL.

Каждая функция определена следующим образом.

~~~C
 1 GtkWidget *
 2 tfe_text_view_new_with_file (GFile *file) {
 3   g_return_val_if_fail (G_IS_FILE (file), NULL);
 4 
 5   GtkWidget *tv;
 6   GtkTextBuffer *tb;
 7   char *contents;
 8   gsize length;
 9 
10   if (! g_file_load_contents (file, NULL, &contents, &length, NULL, NULL)) /* read error */
11     return NULL;
12 
13   tv = tfe_text_view_new();
14   tb = gtk_text_view_get_buffer (GTK_TEXT_VIEW (tv));
15   gtk_text_buffer_set_text (tb, contents, length);
16   TFE_TEXT_VIEW (tv)->file = g_file_dup (file);
17   gtk_text_buffer_set_modified (tb, FALSE);
18   g_free (contents);
19   return tv;
20 }
21 
22 GtkWidget *
23 tfe_text_view_new (void) {
24   return GTK_WIDGET (g_object_new (TFE_TYPE_TEXT_VIEW, "wrap-mode", GTK_WRAP_WORD_CHAR, NULL));
25 }
~~~

- 22-25: Функция `tfe_text_view_new`.
Просто возвращает значение функции `g_object_new`, но приводит его к указателю на GtkWidget.
Функция `g_object_new` создаёт любые экземпляры классов-потомков.
Аргументы - это тип класса, список свойств и NULL, который является конечным маркером списка свойств.
Свойство "wrap-mode" TfeTextView имеет GTK\_WRAP\_WORD\_CHAR в качестве значения по умолчанию.
- 1-20: Функция `tfe_text_view_new_with_file`.
- 3: `g_return_val_if_fail` описана в [GLib API Reference -- g\_return\_val\_if\_fail](https://docs.gtk.org/glib/func.return_val_if_fail.html).
А также [GLib API Reference -- Message Logging](https://docs.gtk.org/glib/logging.html).
Она проверяет, является ли аргумент `file` указателем на GFile.
Если это истина, программа переходит к следующей строке.
Если это ложь, она немедленно возвращает NULL (второй аргумент).
И в то же время выводит сообщение об ошибке в лог (обычно лог выводится в stderr или stdout).
Эта функция используется для проверки ошибок программиста.
Если произойдёт ошибка, решение обычно заключается в изменении программы (вызывающей стороны) и исправлении бага.
Вам нужно различать ошибки программиста и ошибки времени выполнения.
Не следует использовать эту функцию для обнаружения ошибок времени выполнения.
- 10-11: Читает файл. Если произойдёт ошибка, возвращается NULL.
- 13: Вызывает функцию `tfe_text_view_new`.
Функция создаёт экземпляр TfeTextView и возвращает указатель на экземпляр.
- 14: Получает указатель на экземпляр GtkTextBuffer, соответствующий `tv`.
Указатель присваивается `tb`
- 15: Присваивает содержимое, прочитанное из файла, буферу `tb`.
- 16: Дублирует `file` и устанавливает `tv->file` указывающим на него.
GFile *не является* потокобезопасным.
Дублирование гарантирует, что экземпляр GFile в `tv` сохраняет информацию о файле, даже если оригинал будет изменён другим потоком.
- 17: Функция `gtk_text_buffer_set_modified (tb, FALSE)` устанавливает флаг модификации `tb` в FALSE.
Флаг модификации указывает, что содержимое было изменено.
Он используется при сохранении содержимого.
Если флаг модификации равен FALSE, сохранять содержимое не нужно.
- 18: Освобождает память, на которую указывает `contents`.
- 19: Возвращает `tv`, который является указателем на только что созданный экземпляр TfeTextView.
Если произойдёт ошибка, возвращается NULL.

## Функции save и saveas

Функции save и saveas записывают содержимое из GtkTextBuffer в файл.

~~~C
void tfe_text_view_save (TfeTextView *tv)
~~~

Функция `tfe_text_view_save` записывает содержимое из GtkTextBuffer в файл, указанный в `tv->file`.
Если `tv->file` равен NULL, то она показывает диалог выбора файла и запрашивает у пользователя выбор файла для сохранения.
Затем она сохраняет содержимое в файл и устанавливает `tv->file` указывающим на экземпляр GFile для файла.

~~~C
void tfe_text_view_saveas (TfeTextView *tv)
~~~

Функция `saveas` показывает диалог выбора файла и запрашивает у пользователя выбор существующего файла или указание нового файла для сохранения.
Затем функция изменяет `tv->file` и сохраняет содержимое в указанный файл.
Если произойдёт ошибка, она отображается пользователю через диалог оповещения.
Ошибка обрабатывается только внутри TfeTextView, и вызывающей стороне не передаётся никакой информации.

### Функция save\_file

~~~C
 1 static gboolean
 2 save_file (GFile *file, GtkTextBuffer *tb, GtkWindow *win) {
 3   GtkTextIter start_iter;
 4   GtkTextIter end_iter;
 5   char *contents;
 6   gboolean stat;
 7   GtkAlertDialog *alert_dialog;
 8   GError *err = NULL;
 9 
10   gtk_text_buffer_get_bounds (tb, &start_iter, &end_iter);
11   contents = gtk_text_buffer_get_text (tb, &start_iter, &end_iter, FALSE);
12   stat = g_file_replace_contents (file, contents, strlen (contents), NULL, TRUE, G_FILE_CREATE_NONE, NULL, NULL, &err);
13   if (stat)
14     gtk_text_buffer_set_modified (tb, FALSE);
15   else {
16     alert_dialog = gtk_alert_dialog_new ("%s", err->message);
17     gtk_alert_dialog_show (alert_dialog, win);
18     g_object_unref (alert_dialog);
19     g_error_free (err);
20   }
21   g_free (contents);
22   return stat;
23 }
~~~

- Функция `save_file` вызывается из `saveas_dialog_response` и `tfe_text_view_save`.
Эта функция сохраняет содержимое буфера в файл, переданный в качестве аргумента.
Если произойдёт ошибка, она отображает сообщение об ошибке.
Таким образом, вызывающей стороне этой функции не нужно заботиться об ошибках.
Класс этой функции - `static`.
Поэтому только функции в этом файле (`tfetextview.c`) вызывают эту функцию.
Такие статические функции обычно не имеют функций `g_return_val_if_fail`.
- 10-11: Получает текстовое содержимое из буфера.
- 12: Функция `g_file_replace_contents` записывает содержимое в файл и возвращает статус (true = успех / false = неудача).
У неё много параметров, но некоторым из них почти всегда передаются одинаковые значения.
  - GFile* file: GFile, в который сохраняется содержимое.
  - const char* contents: содержимое для сохранения. Строка принадлежит вызывающей стороне.
  - gsize length: длина содержимого
  - const char* etag: тег сущности. Обычно NULL.
  - gboolean make_backup: true для создания резервной копии, если файл существует. false, чтобы не создавать её. файл будет перезаписан.
  - GFileCreateFlags flags: обычно `G_FILE_CREATE_NONE` подходит.
  - char** new_etag: новый тег сущности. Обычно NULL.
  - GCancellable* cancellable: Если установлен экземпляр cancellable, другой поток может отменить эту операцию. обычно NULL.
  - GError** error: Если произойдёт ошибка, будет установлен GError.
- 13,14: Если ошибка не произошла, устанавливается флаг modified в FALSE.
Это означает, что буфер не был изменён с момента его сохранения.
- 16-19: Если не удаётся сохранить содержимое, будет отображено сообщение об ошибке.
- 16: Создаёт диалог оповещения. Параметры - это строка формата в стиле printf, за которой следуют значения для вставки в строку.
GtkAlertDialog доступен с версии 4.10.
Если ваша версия старше 4.10, используйте GtkMessageDialog вместо этого.
GtkMessageDialog устарел с версии 4.10.
- 17: Показывает диалог оповещения. Параметры - это диалог и родительское окно.
Это позволяет менеджерам окон держать диалог поверх родительского окна или центрировать диалог над родительским окном.
Можно не указывать родительское окно для диалога, передав NULL в качестве аргумента.
Однако рекомендуется указывать родителей для диалогов.
- 18: Освобождает диалог.
- 19: Освобождает структуру GError, на которую указывает `err`, функцией `g_error_free`.
- 21: Освобождает `contents`.
- 22: Возвращает статус вызывающей стороне.

### Функция save\_dialog\_cb

~~~C
 1 static void
 2 save_dialog_cb(GObject *source_object, GAsyncResult *res, gpointer data) {
 3   GtkFileDialog *dialog = GTK_FILE_DIALOG (source_object);
 4   TfeTextView *tv = TFE_TEXT_VIEW (data);
 5   GtkTextBuffer *tb = gtk_text_view_get_buffer (GTK_TEXT_VIEW (tv));
 6   GFile *file;
 7   GtkWidget *win = gtk_widget_get_ancestor (GTK_WIDGET (tv), GTK_TYPE_WINDOW);
 8   GError *err = NULL;
 9   GtkAlertDialog *alert_dialog;
10 
11   if (((file = gtk_file_dialog_save_finish (dialog, res, &err)) != NULL) && save_file(file, tb, GTK_WINDOW (win))) {
12     // The following is complicated. The comments here will help your understanding
13     // G_IS_FILE(tv->file) && tv->file == file  => nothing to do
14     // G_IS_FILE(tv->file) && tv->file != file  => unref(tv->file), tv->file=file, emit change_file signal
15     // tv->file==NULL                           =>                  tv->file=file, emit change_file signal
16     if (! (G_IS_FILE (tv->file) && g_file_equal (tv->file, file))) {
17       if (G_IS_FILE (tv->file))
18         g_object_unref (tv->file);
19       tv->file = file; // The ownership of 'file' moves to TfeTextView.
20       g_signal_emit (tv, tfe_text_view_signals[CHANGE_FILE], 0);
21     }
22   }
23   if (err) {
24     alert_dialog = gtk_alert_dialog_new ("%s", err->message);
25     gtk_alert_dialog_show (alert_dialog, GTK_WINDOW (win));
26     g_object_unref (alert_dialog);
27     g_clear_error (&err);
28   }
29 }
~~~

- Функция `save_dialog_cb` является функцией обратного вызова, которая передаётся функции `gtk_file_dialog_save` в качестве аргумента.
Функция `gtk_file_dialog_save` показывает диалог выбора файла пользователю.
Пользователь выбирает или вводит имя файла и нажимает кнопку `Save` или просто нажимает кнопку `Cancel`.
Затем вызывается функция обратного вызова с результатом.
Это общий способ в GIO для управления асинхронными операциями.
Пара функций `g_data_input_stream_read_line_async` и `g_data_input_stream_read_line_finish` - один из примеров.
Эти функции потокобезопасны.
Аргументы `save_dialog_cb`:
  - GObject *source_object: Экземпляр GObject, с которым была начата операция.
На самом деле это экземпляр GtkFileDialog, который показывается пользователю.
Однако функция обратного вызова определена как `AsyncReadyCallback`, что является общей функцией обратного вызова для асинхронной операции.
Поэтому тип - GObject, и вам нужно будет привести его к GtkFileDialog позже.
  - GAsyncResult *res: Результат асинхронной операции.
Он будет передан функции `gtk_dialog_save_finish`.
  - gpointer data: Пользовательские данные, установленные в функции `gtk_dialog_save`.
- 11: Вызывает `gtk_dialog_save_finish`.
Ей передаётся результат `res` в качестве аргумента, и она возвращает указатель на объект GFile, выбранный пользователем.
Если пользователь отменил операцию или произошла ошибка, она возвращает NULL, создаёт объект GError и устанавливает `err` указывающим на него.
Если `gtk_dialog_save_finish` возвращает GFile, вызывается функция `save_file`.
- 12-21: Если файл успешно сохранён, выполняются эти строки.
См. комментарии, строки 12-15, для деталей.
- 23-28: Если произойдёт ошибка, показывается сообщение об ошибке через диалог оповещения.

### Функция tfe\_text\_view\_save

~~~C
 1 void
 2 tfe_text_view_save (TfeTextView *tv) {
 3   g_return_if_fail (TFE_IS_TEXT_VIEW (tv));
 4 
 5   GtkTextBuffer *tb = gtk_text_view_get_buffer (GTK_TEXT_VIEW (tv));
 6   GtkWidget *win = gtk_widget_get_ancestor (GTK_WIDGET (tv), GTK_TYPE_WINDOW);
 7 
 8   if (! gtk_text_buffer_get_modified (tb))
 9     return; /* no need to save it */
10   else if (tv->file == NULL)
11     tfe_text_view_saveas (tv);
12   else
13     save_file (tv->file, tb, GTK_WINDOW (win));
14 }
~~~

- Функция `tfe_text_view_save` записывает содержимое в файл `tv->file`.
Она вызывает `tfe_text_view_saveas` или `save_file`.
- 1-3: Функция публичная, т.е. она открыта для других объектов.
Поэтому у неё нет класса `static`.
Публичные функции должны проверять тип параметра с помощью функции `g_return_if_fail`.
Если `tv` не является указателем на экземпляр TfeTextView, то она записывает сообщение об ошибке в лог и немедленно возвращается.
Эта функция похожа на `g_return_val_if_fail`, но не возвращает значение, потому что `tfe_text_view_save` не возвращает значение (void).
- 5-6: Устанавливаются GtkTextBuffer `tb` и GtkWidget (GtkWindow) `win`.
Функция `gtk_widget_get_ancestor (widget, type)` возвращает первого предка виджета с типом, который является GType.
Отношение родитель-потомок здесь - это отношение для виджетов, а не классов.
Точнее, тип возвращаемого виджета - это `type` или тип объекта-потомка типа `type`.
Будьте осторожны, "объект-потомок" в предыдущем предложении - это *не* "виджет-потомок".
Например, тип GtkWindow - это `GTK_TYPE_WINDOW`, а тип TfeTextView - `TFE_TYPE_TEXT_VIEW`.
Окно верхнего уровня может быть GtkApplicationWindow, но оно является потомком GtkWindow.
Поэтому `gtk_widget_get_ancestor (GTK_WIDGET (tv), GTK_TYPE_WINDOW)` может вернуть GtkWindow или GtkApplicationWindow.
- 8-9: Если буфер не был изменён, его не нужно сохранять.
- 10-11: Если `tv->file` равен NULL, что означает, что файл ещё не был задан, вызывается `tfe_text_view_saveas`, чтобы запросить у пользователя выбор файла и сохранить содержимое.
- 12-13: В противном случае вызывается `save_file` для сохранения содержимого в файл `tv->file`.

### Функция tfe\_text\_view\_saveas

~~~C
 1 void
 2 tfe_text_view_saveas (TfeTextView *tv) {
 3   g_return_if_fail (TFE_IS_TEXT_VIEW (tv));
 4 
 5   GtkWidget *win = gtk_widget_get_ancestor (GTK_WIDGET (tv), GTK_TYPE_WINDOW);
 6   GtkFileDialog *dialog;
 7 
 8   dialog = gtk_file_dialog_new ();
 9   gtk_file_dialog_save (dialog, GTK_WINDOW (win), NULL, save_dialog_cb, tv);
10   g_object_unref (dialog);
11 }
~~~

Функция `tfe_text_view_saveas` показывает диалог выбора файла и запрашивает у пользователя выбор файла и сохранение содержимого.

- 1-3: Проверяет тип `tv`, потому что функция публичная.
- 6: GtkWidget `win` устанавливается в окно, которое является предком `tv`.
- 8: Создаёт экземпляр GtkFileDialog.
GtkFileDialog доступен с версии 4.10.
Если ваша версия Gtk старше 4.10, используйте GtkFileChooserDialog вместо этого.
GtkFileChooserDialog устарел с версии 4.10.
- 9: Вызывает функцию `gtk_file_dialog_save`. Аргументы:
  - dialog: GtkFileDialog.
  - GTK_WINDOW (win): родительское окно.
  - NULL: NULL означает отсутствие объекта cancellable.
Если вы поместите сюда объект cancellable, вы сможете отменить операцию из другого потока.
Во многих случаях это NULL.
См. [GCancellable](https://docs.gtk.org/gio/class.Cancellable.html) для дополнительной информации.
  - `save_dialog_cb`: Обратный вызов, который вызывается, когда операция завершена.
Тип указателя на функцию обратного вызова - [GAsyncReadyCallback](https://docs.gtk.org/gio/callback.AsyncReadyCallback.html).
Если задан объект cancellable и операция отменена, обратный вызов не будет вызван.
  - `tv`: Это необязательные пользовательские данные типа gpointer.
Они используются в функции обратного вызова.
- 10: Освобождает экземпляр GtkFileDialog, потому что он больше не нужен.

Эта функция просто показывает диалог выбора файла.
Остальная часть операции выполняется функцией обратного вызова.

![Процесс saveas](../image/saveas.png)

## Функции, связанные с открытием

Функция открытия показывает диалог выбора файла пользователю и запрашивает выбор файла.
Затем она читает файл и помещает текст в GtkTextBuffer.

~~~C
void tfe_text_view_open (TfeTextView *tv, GtkWindow *win);
~~~

Параметр `win` - это родительское окно.
Диалог выбора файла будет показан в центре окна.

Эта функция может быть вызвана сразу после создания `tv`.
В этом случае `tv` ещё не включён в иерархию виджетов.
Поэтому невозможно получить окно верхнего уровня из `tv`.
Вот почему функция нуждается в параметре `win`.

Эта функция обычно вызывается, когда буфер `tv` пустой.
Однако, даже если буфер не пуст, `tfe_text_view_open` не рассматривает это как ошибку.
Если вы хотите вернуть буфер к исходному состоянию, вызов этой функции уместен.

Процесс открытия и чтения разделён на две фазы.
Одна - это создание и показ диалога выбора файла, а другая - функция обратного вызова.
Первая - это `tfe_text_view_open`, а вторая - `open_dialog_cb`.

### Функция open\_dialog\_cb

~~~C
 1 static void
 2 open_dialog_cb (GObject *source_object, GAsyncResult *res, gpointer data) {
 3   GtkFileDialog *dialog = GTK_FILE_DIALOG (source_object);
 4   TfeTextView *tv = TFE_TEXT_VIEW (data);
 5   GtkTextBuffer *tb = gtk_text_view_get_buffer (GTK_TEXT_VIEW (tv));
 6   GtkWidget *win = gtk_widget_get_ancestor (GTK_WIDGET (tv), GTK_TYPE_WINDOW);
 7   GFile *file;
 8   char *contents;
 9   gsize length;
10   gboolean file_changed;
11   GtkAlertDialog *alert_dialog;
12   GError *err = NULL;
13 
14   if ((file = gtk_file_dialog_open_finish (dialog, res, &err)) != NULL
15       && g_file_load_contents (file, NULL, &contents, &length, NULL, &err)) {
16     gtk_text_buffer_set_text (tb, contents, length);
17     g_free (contents);
18     gtk_text_buffer_set_modified (tb, FALSE);
19     // G_IS_FILE(tv->file) && tv->file == file => unref(tv->file), tv->file=file, emit response with SUCCESS
20     // G_IS_FILE(tv->file) && tv->file != file => unref(tv->file), tv->file=file, emit response with SUCCESS, emit change-file
21     // tv->file==NULL =>                                           tv->file=file, emit response with SUCCESS, emit change-file
22     // The order is important. If you unref tv->file first, you can't compare tv->file and file anymore.
23     // And the signals are emitted after new tv->file is set. Or the handler can't catch the new file.
24     file_changed = (G_IS_FILE (tv->file) && g_file_equal (tv->file, file)) ? FALSE : TRUE;
25     if (G_IS_FILE (tv->file))
26       g_object_unref (tv->file);
27     tv->file = file; // The ownership of 'file' moves to TfeTextView
28     if (file_changed)
29       g_signal_emit (tv, tfe_text_view_signals[CHANGE_FILE], 0);
30     g_signal_emit (tv, tfe_text_view_signals[OPEN_RESPONSE], 0, TFE_OPEN_RESPONSE_SUCCESS);
31   } else {
32     if (err->code == GTK_DIALOG_ERROR_DISMISSED) // The user canceled the file chooser dialog
33       g_signal_emit (tv, tfe_text_view_signals[OPEN_RESPONSE], 0, TFE_OPEN_RESPONSE_CANCEL);
34     else {
35       alert_dialog = gtk_alert_dialog_new ("%s", err->message);
36       gtk_alert_dialog_show (alert_dialog, GTK_WINDOW (win));
37       g_object_unref (alert_dialog);
38       g_signal_emit (tv, tfe_text_view_signals[OPEN_RESPONSE], 0, TFE_OPEN_RESPONSE_ERROR);
39     }
40     g_clear_error (&err);
41   }
42 }
~~~

Эта функция похожа на `save_dialog_cb`.
Обе являются функциями обратного вызова для объекта GtkFileDialog.

- 2: У неё три параметра, как у `save_dialog_cb`.
Это:
  - GObject *source_object: Экземпляр GObject, с которым была начата операция.
На самом деле это экземпляр GtkFileDialog, который показывается пользователю.
Позже он будет приведён к GtkFileDialog.
  - GAsyncResult *res: Результат асинхронной операции.
Он будет передан функции `gtk_dialog_open_finish`.
  - gpointer data: Пользовательские данные, установленные в функции `gtk_dialog_open`.
На самом деле это экземпляр TfeTextView, и позже он будет приведён к TfeTextView.
- 14: Функция `gtk_file_dialog_open_finish` возвращает объект GFile, если операция прошла успешно.
В противном случае она возвращает NULL.
- 16-30: Если пользователь выбрал файл и файл был успешно прочитан, будут выполнены строки с 16 по 30.
- 16-18: Устанавливает буфер `tv` с текстом, прочитанным из файла. И освобождает `contents`.
Затем устанавливает статус modified в false.
- 19-30: Код немного сложный.
См. комментарии.
Если файл (`tv->file`) изменился, испускается сигнал "change-file".
Испускается сигнал "open-response" с параметром `TFE_OPEN_RESPONSE_SUCCESS`.
- 31-41: Если операция не удалась, будут выполнены строки с 31 по 41.
- 32-33: Если код ошибки - `GTK_DIALOG_ERROR_DISMISSED`, это означает, что пользователь нажал кнопку "Cancel" или кнопку закрытия на заголовке.
Затем испускается сигнал "open-response" с параметром `TFE_OPEN_RESPONSE_CANCEL`.
Ошибка Dialog описана [здесь](https://docs.gtk.org/gtk4/error.DialogError.html) в справочнике GTK API.
- 35-38: Если произошла другая ошибка, показывается диалог оповещения для сообщения об ошибке и испускается сигнал "open-response" с параметром `TFE_OPEN_RESPONSE_ERROR`.
- 40: Очищает структуру ошибки.

### Функция tfe\_text\_view\_open

~~~C
 1 void
 2 tfe_text_view_open (TfeTextView *tv, GtkWindow *win) {
 3   g_return_if_fail (TFE_IS_TEXT_VIEW (tv));
 4   // 'win' is used for a transient window of the GtkFileDialog.
 5   // It can be NULL.
 6   g_return_if_fail (GTK_IS_WINDOW (win) || win == NULL);
 7 
 8   GtkFileDialog *dialog;
 9 
10   dialog = gtk_file_dialog_new ();
11   gtk_file_dialog_open (dialog, win, NULL, open_dialog_cb, tv);
12   g_object_unref (dialog);
13 }
~~~

- 3-6: Проверяет тип аргументов `tv` и `win`.
Публичные функции всегда должны проверять аргументы.
- 10: Создаёт экземпляр GtkFileDialog.
- 11: Вызывает `gtk_file_dialog_open`. Аргументы:
  - `dialog`: экземпляр GtkFileDialog
  - `win`: родительское окно для диалога выбора файла
  - `NULL`: NULL означает отсутствие объекта cancellable
  - `open_dialog_cb`: функция обратного вызова
  - `tv`: пользовательские данные, которые используются в функции обратного вызова
- 12: Освобождает экземпляр диалога, потому что он больше не нужен.

Весь процесс между вызывающей стороной и TfeTextView показан на следующей диаграмме.
Он действительно сложный.
Потому что `gtk_file_dialog_open` не может вернуть статус операции.

![Вызывающая сторона и TfeTextView](../image/open.png)

1. Вызывающая сторона получает указатель `tv` на экземпляр TfeTextView, вызывая `tfe_text_view_new`.
2. Вызывающая сторона подключает обработчик (слева внизу на диаграмме) и сигнал "open-response".
3. Она вызывает `tfe_text_view_open`, чтобы запросить у пользователя выбор файла из диалога выбора файла.
4. Когда диалог закрывается, вызывается обратный вызов `open_dialog_cb`.
5. Функция обратного вызова читает файл и вставляет текст в GtkTextBuffer и испускает сигнал для информирования о статусе в виде кода ответа.
6. Вызывается обработчик сигнала "open-response", и статус операции передаётся ему в качестве аргумента (параметра сигнала).

## Получение GFile в TfeTextView

Вы можете получить GFile в экземпляре TfeTextView с помощью `tfe_text_view_get_file`.
Это очень просто.

~~~C
1 GFile *
2 tfe_text_view_get_file (TfeTextView *tv) {
3   g_return_val_if_fail (TFE_IS_TEXT_VIEW (tv), NULL);
4 
5   if (G_IS_FILE (tv->file))
6     return g_file_dup (tv->file);
7   else
8     return NULL;
9 }
~~~

Важно дублировать `tv->file`.
В противном случае, если вызывающая сторона освободит объект GFile, `tv->file` больше не гарантированно будет указывать на GFile.
Другая причина использования `g_file_dup` заключается в том, что GFile не является потокобезопасным.
Если вы используете GFile в другом потоке, дублирование необходимо.
См. [Gio API Reference -- g\_file\_dup](https://docs.gtk.org/gio/method.File.dup.html).

## Документация API и исходный файл tfetextview.c

Обратитесь к [документации API TfeTextView](tfetextview_doc.md).
Она находится в каталоге `src/tfetextview`.

Вы можете найти все исходные коды TfeTextView в каталоге [src/tfetextview](../src/tfetextview).


Up: [README.md](../README.md),  Prev: [Section 12](sec12.md), Next: [Section 14](sec14.md)
