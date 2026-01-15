Up: [README.md](../README.md),  Prev: [Section 21](sec21.md), Next: [Section 23](sec23.md)

# TfeWindow

## Окно Tfe и XML-файлы

Ниже показано окно Tfe.

![tfe6](../image/tfe6.png)

- На панели инструментов размещены кнопки открытия, сохранения и закрытия.
Кроме того, на панель инструментов добавлена GtkMenuButton.
Эта кнопка показывает всплывающее меню при нажатии.
Здесь термин "всплывающее" используется в широком смысле, включая выпадающие меню.
- В меню помещены пункты создания, сохранения как, настроек и выхода.

Это позволяет привязать наиболее часто используемые операции к кнопкам панели инструментов.
А остальные скрыты за меню.
Таким образом, это более практично.

Окно является составным виджетом.
Определение описано в XML-файле `tfewindow.ui`.

~~~xml
 1 <?xml version="1.0" encoding="UTF-8"?>
 2 <interface>
 3   <template class="TfeWindow" parent="GtkApplicationWindow">
 4     <property name="title">Text File Editor</property>
 5     <property name="default-width">600</property>
 6     <property name="default-height">400</property>
 7     <child>
 8       <object class="GtkBox" id="boxv">
 9         <property name="orientation">GTK_ORIENTATION_VERTICAL</property>
10         <child>
11           <object class="GtkBox" id="boxh">
12             <property name="orientation">GTK_ORIENTATION_HORIZONTAL</property>
13             <child>
14               <object class="GtkLabel">
15                 <property name="width-chars">10</property>
16               </object>
17             </child>
18             <child>
19               <object class="GtkButton">
20                 <property name="label">Open</property>
21                 <property name="action-name">win.open</property>
22               </object>
23             </child>
24             <child>
25               <object class="GtkButton">
26                 <property name="label">Save</property>
27                 <property name="action-name">win.save</property>
28               </object>
29             </child>
30             <child>
31               <object class="GtkLabel">
32                 <property name="hexpand">TRUE</property>
33               </object>
34             </child>
35             <child>
36               <object class="GtkButton">
37                 <property name="label">Close</property>
38                 <property name="action-name">win.close</property>
39               </object>
40             </child>
41             <child>
42               <object class="GtkMenuButton" id="btnm">
43                 <property name="direction">down</property>
44                 <property name="icon-name">open-menu-symbolic</property>
45               </object>
46             </child>
47             <child>
48               <object class="GtkLabel">
49                 <property name="width-chars">10</property>
50               </object>
51             </child>
52           </object>
53         </child>
54         <child>
55           <object class="GtkNotebook" id="nb">
56             <property name="scrollable">TRUE</property>
57             <property name="hexpand">TRUE</property>
58             <property name="vexpand">TRUE</property>
59           </object>
60         </child>
61       </object>
62     </child>
63   </template>
64 </interface>
65 
~~~

- Определены три кнопки "Open", "Save" и "Close".
Вы можете использовать два способа для перехвата события нажатия кнопки.
Один - это сигнал "clicked", а другой - регистрация действия для кнопки.
Первый способ прост.
Вы можете напрямую соединить сигнал и ваш обработчик.
Второй способ похож на элементы меню.
Когда кнопка нажата, активируется соответствующее действие.
Это немного сложнее, потому что вам нужно заранее создать действие и его обработчик "activate".
Но одно преимущество в том, что вы можете подключить к действию две или более вещи.
Например, к действию можно подключить акселератор.
Акселераторы - это клавиши, которые связываются с действиями.
Например, Ctrl+O часто связывается с действием открытия файла.
Таким образом, и кнопка открытия, и Ctrl+O активируют действие открытия.
В приведенном выше XML-файле используется второй способ.
- Вы можете указать тематическую иконку для GtkMenuButton с помощью свойства "icon-name".
"open-menu-symbolic" - это изображение, которое называется гамбургер-меню.

XML-файл `menu.ui` определяет меню для GtkMenuButton.

~~~xml
 1 <?xml version="1.0" encoding="UTF-8"?>
 2 <interface>
 3   <menu id="menu">
 4     <section>
 5       <item>
 6         <attribute name="label">New</attribute>
 7         <attribute name="action">win.new</attribute>
 8       </item>
 9       <item>
10         <attribute name="label">Save As…</attribute>
11         <attribute name="action">win.saveas</attribute>
12       </item>
13     </section>
14     <section>
15       <item>
16         <attribute name="label">Preference</attribute>
17         <attribute name="action">win.pref</attribute>
18       </item>
19     </section>
20     <section>
21       <item>
22         <attribute name="label">Quit</attribute>
23         <attribute name="action">win.close-all</attribute>
24       </item>
25     </section>
26   </menu>
27 </interface>
~~~

Здесь четыре пункта меню, и они связаны с действиями.

## Заголовочный файл

Ниже приведен код `tfewindow.h`.

~~~C
 1 #pragma once
 2 
 3 #include <gtk/gtk.h>
 4 
 5 #define TFE_TYPE_WINDOW tfe_window_get_type ()
 6 G_DECLARE_FINAL_TYPE (TfeWindow, tfe_window, TFE, WINDOW, GtkApplicationWindow)
 7 
 8 void
 9 tfe_window_notebook_page_new (TfeWindow *win);
10 
11 void
12 tfe_window_notebook_page_new_with_files (TfeWindow *win, GFile **files, int n_files);
13 
14 GtkWidget *
15 tfe_window_new (GtkApplication *app);
~~~

- 5-6: Определение `TFE_TYPE_WINDOW` и макрос `G_DECLARE_FINAL_TYPE`.
- 8-15: Публичные функции. Первые две функции создают страницу блокнота, а последняя функция создает окно.

## C-файл

### Составной виджет

Следующий код извлечен из `tfewindow.c`.

```C
#include <gtk/gtk.h>
#include "tfewindow.h"

struct _TfeWindow {
  GtkApplicationWindow parent;
  GtkMenuButton *btnm;
  GtkNotebook *nb;
  gboolean is_quit;
};

G_DEFINE_FINAL_TYPE (TfeWindow, tfe_window, GTK_TYPE_APPLICATION_WINDOW);

static void
tfe_window_dispose (GObject *gobject) {
  gtk_widget_dispose_template (GTK_WIDGET (gobject), TFE_TYPE_WINDOW);
  G_OBJECT_CLASS (tfe_window_parent_class)->dispose (gobject);
}

static void
tfe_window_init (TfeWindow *win) {
  GtkBuilder *build;
  GMenuModel *menu;

  gtk_widget_init_template (GTK_WIDGET (win));

  build = gtk_builder_new_from_resource ("/com/github/ToshioCP/tfe/menu.ui");
  menu = G_MENU_MODEL (gtk_builder_get_object (build, "menu"));
  gtk_menu_button_set_menu_model (win->btnm, menu);
  g_object_unref(build);
... ... ...
}

static void
tfe_window_class_init (TfeWindowClass *class) {
  GObjectClass *object_class = G_OBJECT_CLASS (class);

  object_class->dispose = tfe_window_dispose;
  gtk_widget_class_set_template_from_resource (GTK_WIDGET_CLASS (class), "/com/github/ToshioCP/tfe/tfewindow.ui");
  gtk_widget_class_bind_template_child (GTK_WIDGET_CLASS (class), TfeWindow, btnm);
  gtk_widget_class_bind_template_child (GTK_WIDGET_CLASS (class), TfeWindow, nb);
}

GtkWidget *
tfe_window_new (GtkApplication *app) {
  return GTK_WIDGET (g_object_new (TFE_TYPE_WINDOW, "application", app, NULL));
}
```

Программа выше похожа на `tfealert.c` и `tfepref.c`.
Она использует тот же способ для построения составного виджета.
Но есть одна новая вещь.
Это меню.
Меню строится из XML-ресурса `menu.ui` и вставляется в кнопку меню.
Это делается в функции инициализации экземпляра `tfe_window_init`.

### Действия

Действия могут принадлежать приложению или окну.
Tfe имеет только одно главное окно, и все действия регистрируются в окне.
Например, действие "close-all" уничтожает окно верхнего уровня, что приводит к завершению приложения.
Вы можете создать действие "app.quit" вместо "win.close-all".
Это ваш выбор.
Если ваше приложение имеет два или более окон, могут понадобиться оба действия: "app.quit" и "win:close-all", которое закрывает все страницы блокнота в окне.
В любом случае, вам нужно решить, должно ли каждое действие принадлежать приложению или окну.

Действия определяются в функции инициализации экземпляра.

```C
static void
tfe_window_init (TfeWindow *win) {
... ... ...
/* ----- action ----- */
  const GActionEntry win_entries[] = {
    { "open", open_activated, NULL, NULL, NULL },
    { "save", save_activated, NULL, NULL, NULL },
    { "close", close_activated, NULL, NULL, NULL },
    { "new", new_activated, NULL, NULL, NULL },
    { "saveas", saveas_activated, NULL, NULL, NULL },
    { "pref", pref_activated, NULL, NULL, NULL },
    { "close-all", close_all_activated, NULL, NULL, NULL }
  };
  g_action_map_add_action_entries (G_ACTION_MAP (win), win_entries, G_N_ELEMENTS (win_entries), win);
... ... ...
}
```

Необходимы две вещи: массив и функция `g_action_map_add_action_entries`.

- Элемент массива - это структура GActionEntry.
Структура имеет следующие члены:
  - имя действия
  - обработчик сигнала activate
  - тип параметра или NULL при отсутствии параметра
  - начальное состояние для действия
  - обработчик сигнала change-state
- Приведенные выше действия не имеют состояния и параметров.
Поэтому третий параметр и последующие - все NULL.
- Функция `g_action_map_add_action_entries` добавляет действия из массива `win_entries` в карту действий `win`.
Последний аргумент `win` - это user_data, который является последним аргументом обработчиков.
- Все обработчики находятся в программе `tfewindow.c` и показаны в следующих подразделах.

### Обработчики действий

#### open\_activated

Функция обратного вызова `open_activated` - это обработчик сигнала activate для действия "open".

~~~C
1 static void
2 open_activated (GSimpleAction *action, GVariant *parameter, gpointer user_data) {
3   TfeWindow *win = TFE_WINDOW (user_data);
4   GtkWidget *tv = tfe_text_view_new ();
5 
6   g_signal_connect (TFE_TEXT_VIEW (tv), "open-response", G_CALLBACK (open_response_cb), win);
7   tfe_text_view_open (TFE_TEXT_VIEW (tv), GTK_WINDOW (win));
8 }
~~~

Она подключает сигнал "open-response" к вновь созданному экземпляру TfeTextView и просто вызывает `tfe_text_view_open`.
Остальную часть задачи она оставляет обработчику сигнала `open_response_cb`.

~~~C
 1 static void
 2 open_response_cb (TfeTextView *tv, int response, gpointer user_data) {
 3   TfeWindow *win = TFE_WINDOW (user_data);
 4   GFile *file;
 5   char *filename;
 6 
 7   if (response != TFE_OPEN_RESPONSE_SUCCESS) {
 8     g_object_ref_sink (tv);
 9     g_object_unref (tv);
10   }else if (! G_IS_FILE (file = tfe_text_view_get_file (tv))) {
11     g_object_ref_sink (tv);
12     g_object_unref (tv);
13   }else {
14     filename = g_file_get_basename (file);
15     g_object_unref (file);
16     notebook_page_build (win, GTK_WIDGET (tv), filename);
17     g_free (filename);
18   }
19 }
~~~

Если экземпляр TfeTextView не смог прочитать файл, он уничтожает экземпляр с помощью `g_object_ref_sink` и `g_object_unref`.
Поскольку вновь созданные виджеты являются плавающими, вам нужно преобразовать плавающую ссылку в обычную ссылку перед её освобождением.
Преобразование выполняется с помощью `g_object_ref_sink`.

Если экземпляр успешно прочитал файл, вызывается `notebook_page_build` для построения страницы блокнота и добавления её к объекту GtkNotebook.

~~~C
 1 static void
 2 notebook_page_build (TfeWindow *win, GtkWidget *tv, char *filename) {
 3   // The arguments win, tb and filename are owned by the caller.
 4   // If tv has a floating reference, it is consumed by the function.
 5   GtkWidget *scr = gtk_scrolled_window_new ();
 6   GtkTextBuffer *tb = gtk_text_view_get_buffer (GTK_TEXT_VIEW (tv));
 7   GtkNotebookPage *nbp;
 8   GtkWidget *lab;
 9   int i;
10 
11   gtk_text_view_set_wrap_mode (GTK_TEXT_VIEW (tv), GTK_WRAP_WORD_CHAR);
12   gtk_scrolled_window_set_child (GTK_SCROLLED_WINDOW (scr), tv);
13   lab = gtk_label_new (filename);
14   i = gtk_notebook_append_page (win->nb, scr, lab);
15   nbp = gtk_notebook_get_page (win->nb, scr);
16   g_object_set (nbp, "tab-expand", TRUE, NULL);
17   gtk_notebook_set_current_page (win->nb, i);
18   g_signal_connect (GTK_TEXT_VIEW (tv), "change-file", G_CALLBACK (file_changed_cb), win->nb);
19   g_signal_connect (tb, "modified-changed", G_CALLBACK (modified_changed_cb), tv);
20 }
~~~

Эта функция - своего рода библиотечная функция, и она вызывается из трех разных мест.

Эта функция создает новый экземпляр GtkScrolledWindow и устанавливает его дочерний элемент в `tv`.
Затем она добавляет его к экземпляру GtkNotebook `win->nb`.
И устанавливает метку вкладки на имя файла.

После построения она подключает два сигнала и обработчика.

- Сигнал "change-file" и обработчик `file_changed_cb`.
Если экземпляр TfeTextView изменяет файл, вызывается обработчик, и вкладка страницы блокнота обновляется.
- Сигнал "modified-changed" и обработчик `modified_changed_cb`.
Если текст в буфере экземпляра TfeTextView изменен, звездочка добавляется в начало имени файла на вкладке страницы блокнота.
Если текст сохранен в файл, звездочка удаляется.
Звездочка сообщает пользователю, был ли текст изменен или нет.

~~~C
 1 static void
 2 file_changed_cb (TfeTextView *tv, gpointer user_data) {
 3   GtkNotebook *nb =  GTK_NOTEBOOK (user_data);
 4   GtkWidget *scr;
 5   GtkWidget *label;
 6   GFile *file;
 7   char *filename;
 8 
 9   file = tfe_text_view_get_file (tv);
10   scr = gtk_widget_get_parent (GTK_WIDGET (tv));
11   if (G_IS_FILE (file)) {
12     filename = g_file_get_basename (file);
13     g_object_unref (file);
14   } else
15     filename = get_untitled ();
16   label = gtk_label_new (filename);
17   g_free (filename);
18   gtk_notebook_set_tab_label (GTK_NOTEBOOK (nb), scr, label);
19 }
20 
21 static void
22 modified_changed_cb (GtkTextBuffer *tb, gpointer user_data) {
23   TfeTextView *tv = TFE_TEXT_VIEW (user_data);
24   GtkWidget *scr = gtk_widget_get_parent (GTK_WIDGET (tv));
25   GtkWidget *nb =  gtk_widget_get_ancestor (GTK_WIDGET (tv), GTK_TYPE_NOTEBOOK);
26   GtkWidget *label;
27   GFile *file;
28   char *filename;
29   char *text;
30 
31   file = tfe_text_view_get_file (tv);
32   filename = g_file_get_basename (file);
33   if (gtk_text_buffer_get_modified (tb))
34     text = g_strdup_printf ("*%s", filename);
35   else
36     text = g_strdup (filename);
37   g_object_unref (file);
38   g_free (filename);
39   label = gtk_label_new (text);
40   g_free (text);
41   gtk_notebook_set_tab_label (GTK_NOTEBOOK (nb), scr, label);
42 }
~~~

#### save\_activated

Функция обратного вызова `save_activated` - это обработчик сигнала activate для действия "save".

~~~C
1 static void
2 save_activated (GSimpleAction *action, GVariant *parameter, gpointer user_data) {
3   TfeWindow *win = TFE_WINDOW (user_data);
4   TfeTextView *tv = get_current_textview (win->nb);
5 
6   tfe_text_view_save (TFE_TEXT_VIEW (tv));
7 }
~~~

Эта функция получает текущий экземпляр TfeTextView с помощью функции `get_current_textview`.
И просто вызывает `tfe_text_view_save`.

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
~~~

#### close\_activated

Функция обратного вызова `close_activated` - это обработчик сигнала activate для действия "close".
Она закрывает текущую страницу блокнота.

~~~C
 1 static void
 2 close_activated (GSimpleAction *action, GVariant *parameter, gpointer user_data) {
 3   TfeWindow *win = TFE_WINDOW (user_data);
 4   TfeTextView *tv;
 5   GtkTextBuffer *tb;
 6   GtkWidget *alert;
 7 
 8   tv = get_current_textview (win->nb);
 9   tb = gtk_text_view_get_buffer (GTK_TEXT_VIEW (tv));
10   if (! gtk_text_buffer_get_modified (tb)) /* is saved? */
11     notebook_page_close (win);
12   else {
13     win->is_quit = FALSE;
14     alert = tfe_alert_new_with_data ("Are you sure?", "Contents aren't saved yet.\nAre you sure to close?", "Close");
15     gtk_window_set_transient_for (GTK_WINDOW (alert), GTK_WINDOW (win));
16     g_signal_connect (TFE_ALERT (alert), "response", G_CALLBACK (alert_response_cb), win);
17     gtk_window_present (GTK_WINDOW (alert));
18   }
19 }
~~~

Если текст на текущей странице был сохранен, она вызывает `notebook_page_close` для закрытия страницы.
В противном случае она устанавливает `win->is_quit` в FALSE и показывает диалог предупреждения.
Сигнал "response" диалога подключен к обработчику `alert_response_cb`.

~~~C
 1 static void
 2 notebook_page_close (TfeWindow *win){
 3   int i;
 4 
 5   if (gtk_notebook_get_n_pages (win->nb) == 1)
 6     gtk_window_destroy (GTK_WINDOW (win));
 7   else {
 8     i = gtk_notebook_get_current_page (win->nb);
 9     gtk_notebook_remove_page (win->nb, i);
10   }
11 }
~~~

Если блокнот имеет только одну страницу, она уничтожает окно, и приложение завершается.
В противном случае она удаляет текущую страницу.

~~~C
 1 static void
 2 alert_response_cb (TfeAlert *alert, int response_id, gpointer user_data) {
 3   TfeWindow *win = TFE_WINDOW (user_data);
 4 
 5   if (response_id == TFE_ALERT_RESPONSE_ACCEPT) {
 6     if (win->is_quit)
 7       gtk_window_destroy(GTK_WINDOW (win));
 8     else
 9       notebook_page_close (win);
10   }
11 }
~~~

Если пользователь нажал на кнопку отмены, она ничего не делает.
Если пользователь нажал на кнопку принятия, которая эквивалентна кнопке закрытия, она вызывает `notebook_page_close`.
Обратите внимание, что `win->is_quit` был установлен в FALSE в функции `close_activated`.

#### new\_activated

Функция обратного вызова `new_activated` - это обработчик сигнала activate для действия "new".

~~~C
1 static void
2 new_activated (GSimpleAction *action, GVariant *parameter, gpointer user_data) {
3   TfeWindow *win = TFE_WINDOW (user_data);
4 
5   tfe_window_notebook_page_new (win);
6 }
~~~

Она просто вызывает `tfe_window_notebook_page_new`, который является публичным методом TfeWindow.

~~~C
 1 void
 2 tfe_window_notebook_page_new (TfeWindow *win) {
 3   GtkWidget *tv;
 4   char *filename;
 5 
 6   tv = tfe_text_view_new ();
 7   filename = get_untitled ();
 8   notebook_page_build (win, tv, filename);
 9   g_free (filename);
10 }
~~~

Эта функция создает новый экземпляр TfeTextView, строку семейства "Untitled" и вызывает `notebook_page_build`.

#### saveas\_activated

Функция обратного вызова `saveas_activated` - это обработчик сигнала activate для действия "saveas".

~~~C
1 static void
2 saveas_activated (GSimpleAction *action, GVariant *parameter, gpointer user_data) {
3   TfeWindow *win = TFE_WINDOW (user_data);
4   TfeTextView *tv = get_current_textview (win->nb);
5 
6   tfe_text_view_saveas (TFE_TEXT_VIEW (tv));
7 }
~~~

Эта функция получает экземпляр TfeTextView текущей страницы и вызывает `tfe_text_view_saveas`.

#### pref\_activated

Функция обратного вызова `pref_activated` - это обработчик сигнала activate для действия "pref".

~~~C
1 static void
2 pref_activated (GSimpleAction *action, GVariant *parameter, gpointer user_data) {
3   TfeWindow *win = TFE_WINDOW (user_data);
4   GtkWidget *pref;
5 
6   pref = tfe_pref_new ();
7   gtk_window_set_transient_for (GTK_WINDOW (pref), GTK_WINDOW (win));
8   gtk_window_present (GTK_WINDOW (pref));
9 }
~~~

Эта функция создает экземпляр TfePref, который является диалогом, и устанавливает временное родительское окно в `win`.
И показывает диалог.

#### close\_all\_activated

Функция обратного вызова `close_all_activated` - это обработчик сигнала activate для действия "close_all".

~~~C
1 static void
2 close_all_activated (GSimpleAction *action, GVariant *parameter, gpointer user_data) {
3   TfeWindow *win = TFE_WINDOW (user_data);
4 
5   if (close_request_cb (win) == FALSE)
6     gtk_window_destroy (GTK_WINDOW (win));
7 }
~~~

Сначала она вызывает функцию `close_request_cb`.
Это функция обратного вызова для сигнала "close-request" главного окна.
Она возвращает FALSE, если все тексты были сохранены.
В противном случае она возвращает TRUE.

Таким образом, функция `close_all_activated` уничтожает главное окно, если все тексты были сохранены.
В противном случае она ничего не делает.
Но функция `close_request_cb` показывает диалог предупреждения, и если пользователь нажмет на кнопку принятия, окно будет уничтожено.

### Сигнал "close-request" окна

GtkWindow имеет сигнал "close-request", который испускается, когда нажимается кнопка закрытия - кнопка в форме X в правом верхнем углу.
И пользовательский обработчик вызывается перед обработчиком по умолчанию.
Если пользовательский обработчик возвращает TRUE, остальная часть процесса закрытия пропускается.
Если он возвращает FALSE, остальное продолжается, и окно будет уничтожено.

~~~C
 1 static gboolean
 2 close_request_cb (TfeWindow *win) {
 3   TfeAlert *alert;
 4 
 5   if (is_saved_all (win->nb))
 6     return FALSE;
 7   else {
 8     win->is_quit = TRUE;
 9     alert = TFE_ALERT (tfe_alert_new_with_data ("Are you sure?", "Contents aren't saved yet.\nAre you sure to quit?", "Quit"));
10     gtk_window_set_transient_for (GTK_WINDOW (alert), GTK_WINDOW (win));
11     g_signal_connect (TFE_ALERT (alert), "response", G_CALLBACK (alert_response_cb), win);
12     gtk_window_present (GTK_WINDOW (alert));
13     return TRUE;
14   }
15 }
~~~

Сначала она вызывает `is_saved_all` и проверяет, были ли сохранены тексты.
Если да, она возвращает FALSE, и процесс закрытия продолжается.
В противном случае она устанавливает `win->is_quit` в TRUE и показывает диалог предупреждения.
Когда пользователь нажимает на кнопку принятия или отмены, диалог исчезает, и испускается сигнал "response".
Затем вызывается обработчик `alert_response_cb`.
Он уничтожает главное окно, если пользователь нажал на кнопку принятия, поскольку `win->is_quit` равен TRUE.
В противном случае он ничего не делает.

~~~C
 1 static gboolean
 2 is_saved_all (GtkNotebook *nb) {
 3   int i, n;
 4   GtkWidget *scr;
 5   GtkWidget *tv;
 6   GtkTextBuffer *tb;
 7 
 8   n = gtk_notebook_get_n_pages (nb);
 9   for (i = 0; i < n; ++i) {
10     scr = gtk_notebook_get_nth_page (nb, i);
11     tv = gtk_scrolled_window_get_child (GTK_SCROLLED_WINDOW (scr));
12     tb = gtk_text_view_get_buffer (GTK_TEXT_VIEW (tv));
13     if (gtk_text_buffer_get_modified (tb))
14       return FALSE;
15   }
16   return TRUE;
17 }
~~~

### Публичные функции

Есть три публичные функции.

- `void tfe_window_notebook_page_new (TfeWindow *win)`
- `void tfe_window_notebook_page_new_with_files (TfeWindow *win, GFile **files, int n_files)`
- `GtkWidget *tfe_window_new (GtkApplication *app)`

Первая функция вызывается, когда приложение испускает сигнал "activate".
Вторая - для сигнала "open".
Ей передаются три аргумента, и они принадлежат вызывающей стороне.

~~~C
 1 void
 2 tfe_window_notebook_page_new_with_files (TfeWindow *win, GFile **files, int n_files) {
 3   int i;
 4   GtkWidget *tv;
 5   char *filename;
 6 
 7   for (i = 0; i < n_files; i++)
 8     if ((tv = tfe_text_view_new_with_file (*(files+i))) != NULL) {
 9       filename = g_file_get_basename (*(files+i));
10       notebook_page_build (win, tv, filename);
11       g_free (filename);
12     }
13   if (gtk_notebook_get_n_pages (win->nb) == 0)
14     tfe_window_notebook_page_new (win);
15 }
~~~

Эта функция имеет цикл для массива `files`.
Она создает экземпляр TfeTextView с текстом из каждого файла.
И строит с ним страницу.

Если происходит ошибка и ни одна страница не создана, она создает новую пустую страницу.

### Полный код tfewindow.c

Ниже приведен полный исходный код `tfewindow.c`.

~~~C
  1 #include <gtk/gtk.h>
  2 #include "tfewindow.h"
  3 #include "tfepref.h"
  4 #include "tfealert.h"
  5 #include "../tfetextview/tfetextview.h"
  6 
  7 struct _TfeWindow {
  8   GtkApplicationWindow parent;
  9   GtkMenuButton *btnm;
 10   GtkNotebook *nb;
 11   gboolean is_quit;
 12 };
 13 
 14 G_DEFINE_FINAL_TYPE (TfeWindow, tfe_window, GTK_TYPE_APPLICATION_WINDOW);
 15 
 16 /* Low level functions */
 17 
 18 /* Create a new untitled string */
 19 /* The returned string should be freed with g_free() when no longer needed. */
 20 static char*
 21 get_untitled () {
 22   static int c = -1;
 23   if (++c == 0) 
 24     return g_strdup_printf("Untitled");
 25   else
 26     return g_strdup_printf ("Untitled%u", c);
 27 }
 28 
 29 /* The returned object is owned by the scrolled window. */
 30 /* The caller won't get the ownership and mustn't release it. */
 31 static TfeTextView *
 32 get_current_textview (GtkNotebook *nb) {
 33   int i;
 34   GtkWidget *scr;
 35   GtkWidget *tv;
 36 
 37   i = gtk_notebook_get_current_page (nb);
 38   scr = gtk_notebook_get_nth_page (nb, i);
 39   tv = gtk_scrolled_window_get_child (GTK_SCROLLED_WINDOW (scr));
 40   return TFE_TEXT_VIEW (tv);
 41 }
 42 
 43 /* This call back is called when a TfeTextView instance emits a "change-file" signal. */
 44 static void
 45 file_changed_cb (TfeTextView *tv, gpointer user_data) {
 46   GtkNotebook *nb =  GTK_NOTEBOOK (user_data);
 47   GtkWidget *scr;
 48   GtkWidget *label;
 49   GFile *file;
 50   char *filename;
 51 
 52   file = tfe_text_view_get_file (tv);
 53   scr = gtk_widget_get_parent (GTK_WIDGET (tv));
 54   if (G_IS_FILE (file)) {
 55     filename = g_file_get_basename (file);
 56     g_object_unref (file);
 57   } else
 58     filename = get_untitled ();
 59   label = gtk_label_new (filename);
 60   g_free (filename);
 61   gtk_notebook_set_tab_label (GTK_NOTEBOOK (nb), scr, label);
 62 }
 63 
 64 static void
 65 modified_changed_cb (GtkTextBuffer *tb, gpointer user_data) {
 66   TfeTextView *tv = TFE_TEXT_VIEW (user_data);
 67   GtkWidget *scr = gtk_widget_get_parent (GTK_WIDGET (tv));
 68   GtkWidget *nb =  gtk_widget_get_ancestor (GTK_WIDGET (tv), GTK_TYPE_NOTEBOOK);
 69   GtkWidget *label;
 70   GFile *file;
 71   char *filename;
 72   char *text;
 73 
 74   file = tfe_text_view_get_file (tv);
 75   filename = g_file_get_basename (file);
 76   if (gtk_text_buffer_get_modified (tb))
 77     text = g_strdup_printf ("*%s", filename);
 78   else
 79     text = g_strdup (filename);
 80   g_object_unref (file);
 81   g_free (filename);
 82   label = gtk_label_new (text);
 83   g_free (text);
 84   gtk_notebook_set_tab_label (GTK_NOTEBOOK (nb), scr, label);
 85 }
 86 
 87 static gboolean
 88 is_saved_all (GtkNotebook *nb) {
 89   int i, n;
 90   GtkWidget *scr;
 91   GtkWidget *tv;
 92   GtkTextBuffer *tb;
 93 
 94   n = gtk_notebook_get_n_pages (nb);
 95   for (i = 0; i < n; ++i) {
 96     scr = gtk_notebook_get_nth_page (nb, i);
 97     tv = gtk_scrolled_window_get_child (GTK_SCROLLED_WINDOW (scr));
 98     tb = gtk_text_view_get_buffer (GTK_TEXT_VIEW (tv));
 99     if (gtk_text_buffer_get_modified (tb))
100       return FALSE;
101   }
102   return TRUE;
103 }
104 
105 static void
106 notebook_page_close (TfeWindow *win){
107   int i;
108 
109   if (gtk_notebook_get_n_pages (win->nb) == 1)
110     gtk_window_destroy (GTK_WINDOW (win));
111   else {
112     i = gtk_notebook_get_current_page (win->nb);
113     gtk_notebook_remove_page (win->nb, i);
114   }
115 }
116 
117 static void
118 notebook_page_build (TfeWindow *win, GtkWidget *tv, char *filename) {
119   // The arguments win, tb and filename are owned by the caller.
120   // If tv has a floating reference, it is consumed by the function.
121   GtkWidget *scr = gtk_scrolled_window_new ();
122   GtkTextBuffer *tb = gtk_text_view_get_buffer (GTK_TEXT_VIEW (tv));
123   GtkNotebookPage *nbp;
124   GtkWidget *lab;
125   int i;
126 
127   gtk_text_view_set_wrap_mode (GTK_TEXT_VIEW (tv), GTK_WRAP_WORD_CHAR);
128   gtk_scrolled_window_set_child (GTK_SCROLLED_WINDOW (scr), tv);
129   lab = gtk_label_new (filename);
130   i = gtk_notebook_append_page (win->nb, scr, lab);
131   nbp = gtk_notebook_get_page (win->nb, scr);
132   g_object_set (nbp, "tab-expand", TRUE, NULL);
133   gtk_notebook_set_current_page (win->nb, i);
134   g_signal_connect (GTK_TEXT_VIEW (tv), "change-file", G_CALLBACK (file_changed_cb), win->nb);
135   g_signal_connect (tb, "modified-changed", G_CALLBACK (modified_changed_cb), tv);
136 }
137 
138 static void
139 open_response_cb (TfeTextView *tv, int response, gpointer user_data) {
140   TfeWindow *win = TFE_WINDOW (user_data);
141   GFile *file;
142   char *filename;
143 
144   if (response != TFE_OPEN_RESPONSE_SUCCESS) {
145     g_object_ref_sink (tv);
146     g_object_unref (tv);
147   }else if (! G_IS_FILE (file = tfe_text_view_get_file (tv))) {
148     g_object_ref_sink (tv);
149     g_object_unref (tv);
150   }else {
151     filename = g_file_get_basename (file);
152     g_object_unref (file);
153     notebook_page_build (win, GTK_WIDGET (tv), filename);
154     g_free (filename);
155   }
156 }
157 
158 /* alert response signal handler */
159 static void
160 alert_response_cb (TfeAlert *alert, int response_id, gpointer user_data) {
161   TfeWindow *win = TFE_WINDOW (user_data);
162 
163   if (response_id == TFE_ALERT_RESPONSE_ACCEPT) {
164     if (win->is_quit)
165       gtk_window_destroy(GTK_WINDOW (win));
166     else
167       notebook_page_close (win);
168   }
169 }
170 
171 /* ----- Close request on the top window ----- */
172 /* ----- The signal is emitted when the close button is clicked. ----- */
173 static gboolean
174 close_request_cb (TfeWindow *win) {
175   TfeAlert *alert;
176 
177   if (is_saved_all (win->nb))
178     return FALSE;
179   else {
180     win->is_quit = TRUE;
181     alert = TFE_ALERT (tfe_alert_new_with_data ("Are you sure?", "Contents aren't saved yet.\nAre you sure to quit?", "Quit"));
182     gtk_window_set_transient_for (GTK_WINDOW (alert), GTK_WINDOW (win));
183     g_signal_connect (TFE_ALERT (alert), "response", G_CALLBACK (alert_response_cb), win);
184     gtk_window_present (GTK_WINDOW (alert));
185     return TRUE;
186   }
187 }
188 
189 /* ----- action activated handlers ----- */
190 static void
191 open_activated (GSimpleAction *action, GVariant *parameter, gpointer user_data) {
192   TfeWindow *win = TFE_WINDOW (user_data);
193   GtkWidget *tv = tfe_text_view_new ();
194 
195   g_signal_connect (TFE_TEXT_VIEW (tv), "open-response", G_CALLBACK (open_response_cb), win);
196   tfe_text_view_open (TFE_TEXT_VIEW (tv), GTK_WINDOW (win));
197 }
198 
199 static void
200 save_activated (GSimpleAction *action, GVariant *parameter, gpointer user_data) {
201   TfeWindow *win = TFE_WINDOW (user_data);
202   TfeTextView *tv = get_current_textview (win->nb);
203 
204   tfe_text_view_save (TFE_TEXT_VIEW (tv));
205 }
206 
207 static void
208 close_activated (GSimpleAction *action, GVariant *parameter, gpointer user_data) {
209   TfeWindow *win = TFE_WINDOW (user_data);
210   TfeTextView *tv;
211   GtkTextBuffer *tb;
212   GtkWidget *alert;
213 
214   tv = get_current_textview (win->nb);
215   tb = gtk_text_view_get_buffer (GTK_TEXT_VIEW (tv));
216   if (! gtk_text_buffer_get_modified (tb)) /* is saved? */
217     notebook_page_close (win);
218   else {
219     win->is_quit = FALSE;
220     alert = tfe_alert_new_with_data ("Are you sure?", "Contents aren't saved yet.\nAre you sure to close?", "Close");
221     gtk_window_set_transient_for (GTK_WINDOW (alert), GTK_WINDOW (win));
222     g_signal_connect (TFE_ALERT (alert), "response", G_CALLBACK (alert_response_cb), win);
223     gtk_window_present (GTK_WINDOW (alert));
224   }
225 }
226 
227 static void
228 new_activated (GSimpleAction *action, GVariant *parameter, gpointer user_data) {
229   TfeWindow *win = TFE_WINDOW (user_data);
230 
231   tfe_window_notebook_page_new (win);
232 }
233 
234 static void
235 saveas_activated (GSimpleAction *action, GVariant *parameter, gpointer user_data) {
236   TfeWindow *win = TFE_WINDOW (user_data);
237   TfeTextView *tv = get_current_textview (win->nb);
238 
239   tfe_text_view_saveas (TFE_TEXT_VIEW (tv));
240 }
241 
242 static void
243 pref_activated (GSimpleAction *action, GVariant *parameter, gpointer user_data) {
244   TfeWindow *win = TFE_WINDOW (user_data);
245   GtkWidget *pref;
246 
247   pref = tfe_pref_new ();
248   gtk_window_set_transient_for (GTK_WINDOW (pref), GTK_WINDOW (win));
249   gtk_window_present (GTK_WINDOW (pref));
250 }
251 
252 static void
253 close_all_activated (GSimpleAction *action, GVariant *parameter, gpointer user_data) {
254   TfeWindow *win = TFE_WINDOW (user_data);
255 
256   if (close_request_cb (win) == FALSE)
257     gtk_window_destroy (GTK_WINDOW (win));
258 }
259 
260 /* --- public functions --- */
261 
262 void
263 tfe_window_notebook_page_new (TfeWindow *win) {
264   GtkWidget *tv;
265   char *filename;
266 
267   tv = tfe_text_view_new ();
268   filename = get_untitled ();
269   notebook_page_build (win, tv, filename);
270   g_free (filename);
271 }
272 
273 void
274 tfe_window_notebook_page_new_with_files (TfeWindow *win, GFile **files, int n_files) {
275   int i;
276   GtkWidget *tv;
277   char *filename;
278 
279   for (i = 0; i < n_files; i++)
280     if ((tv = tfe_text_view_new_with_file (*(files+i))) != NULL) {
281       filename = g_file_get_basename (*(files+i));
282       notebook_page_build (win, tv, filename);
283       g_free (filename);
284     }
285   if (gtk_notebook_get_n_pages (win->nb) == 0)
286     tfe_window_notebook_page_new (win);
287 }
288 
289 static void
290 tfe_window_dispose (GObject *gobject) {
291   gtk_widget_dispose_template (GTK_WIDGET (gobject), TFE_TYPE_WINDOW);
292   G_OBJECT_CLASS (tfe_window_parent_class)->dispose (gobject);
293 }
294 
295 static void
296 tfe_window_init (TfeWindow *win) {
297   GtkBuilder *build;
298   GMenuModel *menu;
299 
300   gtk_widget_init_template (GTK_WIDGET (win));
301 
302   build = gtk_builder_new_from_resource ("/com/github/ToshioCP/tfe/menu.ui");
303   menu = G_MENU_MODEL (gtk_builder_get_object (build, "menu"));
304   gtk_menu_button_set_menu_model (win->btnm, menu);
305   g_object_unref(build);
306 
307 /* ----- action ----- */
308   const GActionEntry win_entries[] = {
309     { "open", open_activated, NULL, NULL, NULL },
310     { "save", save_activated, NULL, NULL, NULL },
311     { "close", close_activated, NULL, NULL, NULL },
312     { "new", new_activated, NULL, NULL, NULL },
313     { "saveas", saveas_activated, NULL, NULL, NULL },
314     { "pref", pref_activated, NULL, NULL, NULL },
315     { "close-all", close_all_activated, NULL, NULL, NULL }
316   };
317   g_action_map_add_action_entries (G_ACTION_MAP (win), win_entries, G_N_ELEMENTS (win_entries), win);
318 
319   g_signal_connect (GTK_WINDOW (win), "close-request", G_CALLBACK (close_request_cb), NULL);
320 }
321 
322 static void
323 tfe_window_class_init (TfeWindowClass *class) {
324   GObjectClass *object_class = G_OBJECT_CLASS (class);
325 
326   object_class->dispose = tfe_window_dispose;
327   gtk_widget_class_set_template_from_resource (GTK_WIDGET_CLASS (class), "/com/github/ToshioCP/tfe/tfewindow.ui");
328   gtk_widget_class_bind_template_child (GTK_WIDGET_CLASS (class), TfeWindow, btnm);
329   gtk_widget_class_bind_template_child (GTK_WIDGET_CLASS (class), TfeWindow, nb);
330 }
331 
332 GtkWidget *
333 tfe_window_new (GtkApplication *app) {
334   return GTK_WIDGET (g_object_new (TFE_TYPE_WINDOW, "application", app, NULL));
335 }
~~~

Up: [README.md](../README.md),  Prev: [Section 21](sec21.md), Next: [Section 23](sec23.md)
