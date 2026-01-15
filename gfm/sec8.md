Up: [README.md](../README.md),  Prev: [Section 7](sec7.md), Next: [Section 9](sec9.md)

# Определение финального класса

## Очень простой редактор

В предыдущем разделе мы создали очень простую программу просмотра файлов.
Теперь мы перепишем её и превратим в очень простой редактор.
Её исходный файл — `tfe1.c` (текстовый редактор файлов 1) в директории `tfe`.

GtkTextView — это многострочный редактор.
Поэтому нам не нужно писать редактор с нуля.
Мы просто добавим две вещи к программе просмотра файлов:

- Указатели на экземпляры GFile.
- Функцию сохранения текста.

Есть несколько способов хранения указателей.

- Использовать глобальные переменные
- Создать дочерний класс GtkTextView, и каждый его экземпляр будет хранить указатель на экземпляр GFile.

Использование глобальных переменных легко реализовать.
Определите массив указателей на GFile достаточного размера.
Например,

~~~C
GFile *f[20];
~~~

Переменная `f[i]` соответствует файлу, связанному с i-й GtkNotebookPage.

Однако есть две проблемы.
Первая — размер массива.
Если пользователь передаст слишком много аргументов (более 20 в приведённом выше примере), невозможно сохранить все указатели на экземпляры GFile.
Вторая — сложность поддержки программы.
Пока у нас небольшая программа.
Но чем больше вы развиваете программу, тем больше становится её размер.
Вообще говоря, очень сложно поддерживать глобальные переменные в большой программе.
Когда вы проверяете глобальную переменную, вам нужно проверить весь код, который использует эту переменную.

Создание дочернего класса — хорошая идея с точки зрения поддержки.
И мы предпочитаем его, а не глобальную переменную.

Будьте внимательны, что мы думаем о "дочернем классе", а не о "дочернем виджете".
Дочерний класс и дочерний виджет — это совершенно разные вещи.
Класс — это термин системы GObject.
Если вы не знакомы с GObject, см.:

- [GObject API reference](https://docs.gtk.org/gobject/)
- [GObject tutorial for beginners](https://toshiocp.github.io/Gobject-tutorial/)

Дочерний класс наследует всё от родительского и, кроме того, расширяет его возможности.
Мы определим TfeTextView как дочерний класс GtkTextView.
Он имеет всё, что есть у GtkTextView, и добавляет указатель на GFile.

![Child object of GtkTextView](../image/child.png)

## Как определить дочерний класс GtkTextView

Вам нужно знать соглашения системы GObject.
Сначала посмотрите на программу ниже.

~~~C
#define TFE_TYPE_TEXT_VIEW tfe_text_view_get_type ()
G_DECLARE_FINAL_TYPE (TfeTextView, tfe_text_view, TFE, TEXT_VIEW, GtkTextView)

struct _TfeTextView
{
  GtkTextView parent;
  GFile *file;
};

G_DEFINE_FINAL_TYPE (TfeTextView, tfe_text_view, GTK_TYPE_TEXT_VIEW);

static void
tfe_text_view_init (TfeTextView *tv) {
}

static void
tfe_text_view_class_init (TfeTextViewClass *class) {
}

void
tfe_text_view_set_file (TfeTextView *tv, GFile *f) {
  tv -> file = f;
}

GFile *
tfe_text_view_get_file (TfeTextView *tv) {
  return tv -> file;
}

GtkWidget *
tfe_text_view_new (void) {
  return GTK_WIDGET (g_object_new (TFE_TYPE_TEXT_VIEW, NULL));
}
~~~

- TfeTextView разделено на две части.
Tfe и TextView.
Tfe называется префиксом или пространством имён.
TextView называется объектом.
- Существует три различных шаблона идентификаторов.
TfeTextView (верблюжий регистр), tfe\_text\_view (используется для функций) и TFE\_TEXT\_VIEW (используется для приведения объекта к типу TfeTextView).
- Сначала определите макрос `TFE_TYPE_TEXT_VIEW` как `tfe_text_view_get_type ()`.
Имя всегда (префикс)\_TYPE\_(объект), и буквы в верхнем регистре.
А заменяющий текст всегда (префикс)\_(объект)\_get\_type (), и буквы в нижнем регистре.
Это определение размещается перед макросом `G_DECLARE_FINAL_TYPE`.
- Аргументы макроса `G_DECLARE_FINAL_TYPE` — это имя дочернего класса в верблюжьем регистре, в нижнем регистре с подчёркиванием, префикс (в верхнем регистре),
объект (в верхнем регистре с подчёркиванием) и имя родительского класса (в верблюжьем регистре).
Следующие две структуры C объявляются при раскрытии макроса.
  - `typedef struct _TfeTextView TfeTextView`
  - `typedef struct {GtkTextViewClass parent_class; } TfeTextViewClass;`
- Эти объявления говорят нам, что TfeTextView и TfeTextViewClass — это структуры C.
"TfeTextView" имеет два значения: имя класса и имя структуры C.
Структура C TfeTextView называется объектом.
Аналогично, TfeTextViewClass называется классом.
- Объявите структуру `_TfeTextView`.
Подчёркивание необходимо.
Первый элемент — это родительский объект (структура C).
Обратите внимание, что это не указатель, а сам объект.
Второй и последующие элементы — это элементы дочернего объекта.
Структура TfeTextView имеет указатель на экземпляр GFile в качестве элемента.
- Макрос `G_DEFINE_FINAL_TYPE`.
Аргументы — это имя дочернего объекта в верблюжьем регистре, в нижнем регистре с подчёркиванием и тип родительского объекта (префикс)\_TYPE\_(модуль).
Этот макрос в основном используется для регистрации нового класса в системе типов.
Система типов — это базовая система GObject.
Каждый класс имеет свой собственный тип.
Типы GObject, GtkWidget и TfeTextView — это `G_TYPE_OBJECT`, `GTK_TYPE_WIDGET` и `TFE_TYPE_TEXT_VIEW` соответственно.
Например, `TFE_TYPE_TEXT_VIEW` — это макрос, который раскрывается в функцию `tfe_text_view_get_type()`.
Он возвращает целое число, которое уникально среди всех классов системы GObject.
- Функция инициализации экземпляра `tfe_text_view_init` вызывается при создании экземпляра.
Это то же самое, что конструктор в других объектно-ориентированных языках.
- Функция инициализации класса `tfe_text_view_class_init` вызывается при создании класса.
- Две функции `tfe_text_view_set_file` и `tfe_text_view_get_file` — это публичные функции.
Публичные функции открыты, и вы можете вызывать их где угодно.
Они такие же, как публичные методы в других объектно-ориентированных языках.
`tv` — это указатель на объект TfeTextView (структура C).
Он имеет элемент `file`, и на него указывает `tv->file`.
- Функция создания экземпляра TfeTextView — это `tfe_text_view_new`.
Её имя — (префикс)\_(объект)\_new.
Она использует функцию `g_object_new` для создания экземпляра.
Аргументы — это (префикс)\_TYPE\_(объект), список для инициализации свойств и NULL.
NULL — это конечный маркер списка свойств.
Здесь не инициализируется ни одно свойство.
И возвращаемое значение приводится к типу GtkWidget.

Эта программа показывает общую схему определения дочернего класса.

## Сигнал close-request

Представьте, что вы используете этот редактор.
Сначала вы запускаете редактор с аргументами.
Аргументы — это имена файлов.
Редактор читает файлы и показывает окно с текстом файлов в нём.
Затем вы редактируете текст.
После завершения редактирования вы нажимаете на кнопку закрытия окна и выходите из редактора.
Редактор обновляет файлы непосредственно перед закрытием окна.

GtkWindow испускает сигнал "close-request", когда нажата кнопка закрытия.
Мы подключим сигнал и обработчик `before_close`.
(Обработчик — это функция C, которая подключена к сигналу.)
Функция `before_close` вызывается, когда испускается сигнал "close-request".

~~~C
g_signal_connect (win, "close-request", G_CALLBACK (before_close), NULL);
~~~

Аргумент `win` — это GtkApplicationWindow, в котором определён сигнал "close-request", а `before_close` — это обработчик.
Приведение типа `G_CALLBACK` необходимо для обработчика.
Программа `before_close` выглядит следующим образом.

~~~C
 1 static gboolean
 2 before_close (GtkWindow *win, GtkWidget *nb) {
 3   GtkWidget *scr;
 4   GtkWidget *tv;
 5   GFile *file;
 6   char *pathname;
 7   GtkTextBuffer *tb;
 8   GtkTextIter start_iter;
 9   GtkTextIter end_iter;
10   char *contents;
11   unsigned int n;
12   unsigned int i;
13   GError *err = NULL;
14 
15   n = gtk_notebook_get_n_pages (GTK_NOTEBOOK (nb));
16   for (i = 0; i < n; ++i) {
17     scr = gtk_notebook_get_nth_page (GTK_NOTEBOOK (nb), i);
18     tv = gtk_scrolled_window_get_child (GTK_SCROLLED_WINDOW (scr));
19     file = tfe_text_view_get_file (TFE_TEXT_VIEW (tv));
20     tb = gtk_text_view_get_buffer (GTK_TEXT_VIEW (tv));
21     gtk_text_buffer_get_bounds (tb, &start_iter, &end_iter);
22     contents = gtk_text_buffer_get_text (tb, &start_iter, &end_iter, FALSE);
23     if (! g_file_replace_contents (file, contents, strlen (contents), NULL, TRUE, G_FILE_CREATE_NONE, NULL, NULL, &err)) {
24       g_printerr ("%s.\n", err->message);
25       g_clear_error (&err);
26     }
27     g_free (contents);
28     g_object_unref (file);
29   }
30   return FALSE;
31 }
~~~

Числа слева — это номера строк.

- 15: количество страниц блокнота присваивается переменной `n`.
- 16-29: цикл for относительно индекса каждой страницы.
- 17-19: `scr`, `tv` и `file` присваиваются указателями на GtkScrolledWindow, TfeTextView и GFile.
GFile объекта TfeTextView был сохранён при вызове обработчика `app_open`. Это будет показано позже.
- 20-22: `tb` присваивается GtkTextBuffer объекта TfeTextView.
К содержимому буфера осуществляется доступ с помощью итераторов.
Итераторы указывают где-то в буфере.
Функция `gtk_text_buffer_get_bounds` присваивает начало и конец буфера переменным `start_iter` и `end_iter` соответственно.
Затем функция `gtk_text_buffer_get_text` возвращает текст между `start_iter` и `end_iter`, который является всем текстом в буфере.
- 23-26: текст сохраняется в файл.
Если это не удаётся, отображаются сообщения об ошибках.
Экземпляр GError должен быть освобождён, и указатель `err` должен быть NULL для следующего запуска в цикле.
- 27: `contents` освобождается.
- 28: GFile больше не нужен. `g_object_unref` уменьшает счётчик ссылок GFile.
Счётчик ссылок будет объяснён в следующем разделе.
Счётчик ссылок станет нулевым, и экземпляр GFile уничтожит себя.

## Исходный код tfe1.c

Ниже приведён полный исходный код `tfe1.c`.

~~~C
  1 #include <gtk/gtk.h>
  2 
  3 /* Define TfeTextView Widget which is the child class of GtkTextView */
  4 
  5 #define TFE_TYPE_TEXT_VIEW tfe_text_view_get_type ()
  6 G_DECLARE_FINAL_TYPE (TfeTextView, tfe_text_view, TFE, TEXT_VIEW, GtkTextView)
  7 
  8 struct _TfeTextView
  9 {
 10   GtkTextView parent;
 11   GFile *file;
 12 };
 13 
 14 G_DEFINE_FINAL_TYPE (TfeTextView, tfe_text_view, GTK_TYPE_TEXT_VIEW);
 15 
 16 static void
 17 tfe_text_view_init (TfeTextView *tv) {
 18   tv->file = NULL;
 19 }
 20 
 21 static void
 22 tfe_text_view_class_init (TfeTextViewClass *class) {
 23 }
 24 
 25 void
 26 tfe_text_view_set_file (TfeTextView *tv, GFile *f) {
 27   tv->file = f;
 28 }
 29 
 30 GFile *
 31 tfe_text_view_get_file (TfeTextView *tv) {
 32   return tv -> file;
 33 }
 34 
 35 GtkWidget *
 36 tfe_text_view_new (void) {
 37   return GTK_WIDGET (g_object_new (TFE_TYPE_TEXT_VIEW, NULL));
 38 }
 39 
 40 /* ---------- end of the definition of TfeTextView ---------- */
 41 
 42 static gboolean
 43 before_close (GtkWindow *win, GtkWidget *nb) {
 44   GtkWidget *scr;
 45   GtkWidget *tv;
 46   GFile *file;
 47   char *pathname;
 48   GtkTextBuffer *tb;
 49   GtkTextIter start_iter;
 50   GtkTextIter end_iter;
 51   char *contents;
 52   unsigned int n;
 53   unsigned int i;
 54   GError *err = NULL;
 55 
 56   n = gtk_notebook_get_n_pages (GTK_NOTEBOOK (nb));
 57   for (i = 0; i < n; ++i) {
 58     scr = gtk_notebook_get_nth_page (GTK_NOTEBOOK (nb), i);
 59     tv = gtk_scrolled_window_get_child (GTK_SCROLLED_WINDOW (scr));
 60     file = tfe_text_view_get_file (TFE_TEXT_VIEW (tv));
 61     tb = gtk_text_view_get_buffer (GTK_TEXT_VIEW (tv));
 62     gtk_text_buffer_get_bounds (tb, &start_iter, &end_iter);
 63     contents = gtk_text_buffer_get_text (tb, &start_iter, &end_iter, FALSE);
 64     if (! g_file_replace_contents (file, contents, strlen (contents), NULL, TRUE, G_FILE_CREATE_NONE, NULL, NULL, &err)) {
 65       g_printerr ("%s.\n", err->message);
 66       g_clear_error (&err);
 67     }
 68     g_free (contents);
 69     g_object_unref (file);
 70   }
 71   return FALSE;
 72 }
 73 
 74 static void
 75 app_activate (GApplication *app) {
 76   g_print ("You need to give filenames as arguments.\n");
 77 }
 78 
 79 static void
 80 app_open (GApplication *app, GFile ** files, gint n_files, gchar *hint) {
 81   GtkWidget *win;
 82   GtkWidget *nb;
 83   GtkWidget *lab;
 84   GtkNotebookPage *nbp;
 85   GtkWidget *scr;
 86   GtkWidget *tv;
 87   GtkTextBuffer *tb;
 88   char *contents;
 89   gsize length;
 90   char *filename;
 91   int i;
 92   GError *err = NULL;
 93 
 94   win = gtk_application_window_new (GTK_APPLICATION (app));
 95   gtk_window_set_title (GTK_WINDOW (win), "file editor");
 96   gtk_window_set_default_size (GTK_WINDOW (win), 600, 400);
 97 
 98   nb = gtk_notebook_new ();
 99   gtk_window_set_child (GTK_WINDOW (win), nb);
100 
101   for (i = 0; i < n_files; i++) {
102     if (g_file_load_contents (files[i], NULL, &contents, &length, NULL, &err)) {
103       scr = gtk_scrolled_window_new ();
104       tv = tfe_text_view_new ();
105       tb = gtk_text_view_get_buffer (GTK_TEXT_VIEW (tv));
106       gtk_text_view_set_wrap_mode (GTK_TEXT_VIEW (tv), GTK_WRAP_WORD_CHAR);
107       gtk_scrolled_window_set_child (GTK_SCROLLED_WINDOW (scr), tv);
108 
109       tfe_text_view_set_file (TFE_TEXT_VIEW (tv),  g_file_dup (files[i]));
110       gtk_text_buffer_set_text (tb, contents, length);
111       g_free (contents);
112       filename = g_file_get_basename (files[i]);
113       lab = gtk_label_new (filename);
114       gtk_notebook_append_page (GTK_NOTEBOOK (nb), scr, lab);
115       nbp = gtk_notebook_get_page (GTK_NOTEBOOK (nb), scr);
116       g_object_set (nbp, "tab-expand", TRUE, NULL);
117       g_free (filename);
118     } else {
119       g_printerr ("%s.\n", err->message);
120       g_clear_error (&err);
121     }
122   }
123   if (gtk_notebook_get_n_pages (GTK_NOTEBOOK (nb)) > 0) {
124     g_signal_connect (win, "close-request", G_CALLBACK (before_close), nb);
125     gtk_window_present (GTK_WINDOW (win));
126   } else
127     gtk_window_destroy (GTK_WINDOW (win));
128 }
129 
130 int
131 main (int argc, char **argv) {
132   GtkApplication *app;
133   int stat;
134 
135   app = gtk_application_new ("com.github.ToshioCP.tfe1", G_APPLICATION_HANDLES_OPEN);
136   g_signal_connect (app, "activate", G_CALLBACK (app_activate), NULL);
137   g_signal_connect (app, "open", G_CALLBACK (app_open), NULL);
138   stat = g_application_run (G_APPLICATION (app), argc, argv);
139   g_object_unref (app);
140   return stat;
141 }
~~~

- 109: указатель GFile объекта TfeTextView устанавливается на копию `files[i]`, который является GFile, созданным из аргумента командной строки.
GFile будет уничтожен системой позже.
Поэтому его нужно скопировать перед назначением.
`g_file_dup` дублирует GFile.
Примечание: GFile *не* является потокобезопасным. Дублирование GFile позволяет избежать проблем, возникающих из-за разных потоков.
- 124: сигнал "close-request" подключается к обработчику `before_close`.
Четвёртый аргумент называется "пользовательские данные" и будет вторым аргументом обработчика сигнала.
Таким образом, `nb` передаётся `before_close` в качестве второго аргумента.

Теперь пришло время скомпилировать и запустить.

~~~
$ cd src/tfe
$ comp tfe1
$ ./a.out taketori.txt`.
~~~

Измените содержимое и закройте окно.
Убедитесь, что файл изменён.

Теперь у нас есть очень простой редактор.
Он не очень умный.
Нам нужно больше функций, таких как открытие, сохранение, сохранение как, изменение шрифта и так далее.
Мы добавим их в следующем разделе и далее.

Up: [README.md](../README.md),  Prev: [Section 7](sec7.md), Next: [Section 9](sec9.md)
