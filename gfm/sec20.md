Up: [README.md](../README.md),  Prev: [Section 19](sec19.md), Next: [Section 21](sec21.md)

# Составные виджеты и диалог предупреждения

Исходные файлы находятся в [репозитории GitHub Gtk4 tutorial](https://github.com/ToshioCP/Gtk4-tutorial).
Загрузите его и посмотрите директорию `src/tfe6`.

## Структура нового текстового редактора Tfe

Текстовый редактор Tfe будет реструктурирован.
Программа разделена на шесть частей.

- Главная программа: функция main на C.
- Объект TfeApplication: Похож на GtkApplication, но хранит GSettings и CSS Provider.
- Объект TfeWindow: Это окно с кнопками и блокнотом.
- Объект TfePref: Диалог настроек.
- Объект TfeAlert: Диалог предупреждения.
- pdf2css.h и pdf2css.c: Утилиты для работы с шрифтами и CSS.

Этот раздел описывает TfeAlert.
Остальные будут объяснены в следующих разделах.

## Составные виджеты

Диалог предупреждения выглядит так:

![Alert dialog](../image/alert.png)

Tfe использует его, когда пользователь закрывает приложение или закрывает блокнот без сохранения данных в файлы.

Диалог имеет заголовок, кнопки, иконку и сообщение.
Следовательно, он состоит из нескольких виджетов.
Такой диалог называется составным виджетом.

Составные виджеты определяются с помощью шаблонов XML.
Класс создается в функции инициализации класса, а экземпляры создаются и освобождаются следующими функциями.

- gtk\_widget\_init\_template
- gtk\_widget\_dispose\_template

TfeAlert — хороший пример для изучения составных виджетов.
Он определен в трех файлах.

- tfealert.ui: XML-файл
- tfealert.h: Заголовочный файл
- tfealert.c: Файл программы на C

## XML-файл

В XML составного виджета используется тег template.

~~~xml
 1 <?xml version="1.0" encoding="UTF-8"?>
 2 <interface>
 3   <template class="TfeAlert" parent="GtkWindow">
 4     <property name="resizable">FALSE</property>
 5     <property name="modal">TRUE</property>
 6     <property name="titlebar">
 7       <object class="GtkHeaderBar">
 8         <property name="show-title-buttons">FALSE</property>
 9         <property name="title-widget">
10           <object class="GtkLabel" id="lb_title">
11             <property name="label">Are you sure?</property>
12             <property name="single-line-mode">True</property>
13           </object>
14         </property>
15         <child type="start">
16           <object class="GtkButton" id="btn_cancel">
17             <property name="label">Cancel</property>
18             <style>
19               <class name="suggested-action"/>
20             </style>
21             <signal name="clicked" handler="cancel_cb" swapped="TRUE" object="TfeAlert"></signal>
22           </object>
23         </child>
24         <child type="end">
25           <object class="GtkButton" id="btn_accept">
26             <property name="label">Close</property>
27             <style>
28               <class name="destructive-action"/>
29             </style>
30             <signal name="clicked" handler="accept_cb" swapped="TRUE" object="TfeAlert"></signal>
31           </object>
32         </child>
33       </object>
34     </property>
35     <child>
36       <object class="GtkBox">
37         <property name="orientation">GTK_ORIENTATION_HORIZONTAL</property>
38         <property name="spacing">12</property>
39         <property name="margin-top">12</property>
40         <property name="margin-bottom">12</property>
41         <property name="margin-start">12</property>
42         <property name="margin-end">12</property>
43         <child>
44           <object class="GtkImage">
45             <property name="icon-name">dialog-warning</property>
46             <property name="icon-size">GTK_ICON_SIZE_LARGE</property>
47           </object>
48         </child>
49         <child>
50           <object class="GtkLabel" id="lb_message">
51           </object>
52         </child>
53       </object>
54     </child>
55   </template>
56 </interface>
57 
~~~

- 3: Тег template определяет составной виджет.
Атрибут class указывает имя класса составного виджета.
Атрибут parent указывает родительский класс составного виджета.
Таким образом, TfeAlert является дочерним классом GtkWindow.
Атрибут parent необязателен, и его можно опустить.
Но рекомендуется указывать его в теге template.
- 4-6: Определены три его свойства.
Эти свойства унаследованы от GtkWindow.
Свойство titlebar содержит виджет для пользовательской строки заголовка.
Типичный виджет — GtkHeaderBar.
- 8: Если свойство "show-title-buttons" имеет значение TRUE, отображаются кнопки заголовка, такие как закрыть, свернуть и развернуть.
В противном случае они не отображаются.
Объект TfeAlert не изменяется в размерах.
Он закрывается при нажатии любой из двух кнопок: отмена или принять.
Поэтому кнопки заголовка не нужны, и это свойство установлено в FALSE.
- 9-14: Панель имеет заголовок, который является виджетом GtkLabel.
Заголовок по умолчанию — "Are you sure?", но его можно заменить методом экземпляра.
- 15-32: На панели есть две кнопки: отмена и принять.
Кнопка отмены находится слева, поэтому тег child имеет атрибут `type="start"`.
Кнопка принять находится справа, поэтому тег child имеет атрибут `type="end"`.
Диалог отображается, когда пользователь нажал кнопку закрытия или меню выхода без сохранения данных.
Поэтому для пользователя безопаснее нажать кнопку отмены в диалоге предупреждения.
Таким образом, кнопка отмены имеет CSS-класс "suggested-action".
Ubuntu окрашивает кнопку в зеленый цвет, но цвет может быть синим или другим подходящим, определенным системой.
Точно так же кнопка принять имеет CSS-класс "destructive-action" и окрашена в красный цвет.
Обе кнопки имеют сигналы, которые определены тегами signal.
- 35-54: Горизонтальный блок содержит иконку изображения и метку.
- 44-47: Виджет GtkImage отображает изображение.
Свойство "icon-name" — это имя иконки в теме иконок.
Тема зависит от вашей системы.
Вы можете проверить это с помощью браузера иконок.

~~~
$ gtk4-icon-browser
~~~

Иконка "dialog-warning" выглядит примерно так.

![dialog-warning icon is like ...](../image/dialog_warning.png)

Они сделаны вручную.
Реальное изображение в диалоге предупреждения выглядит лучше.

Можно определить виджет предупреждения как дочерний от GtkDialog.
Но GtkDialog устарел, начиная с версии GTK 4.10.
И пользователям следует использовать GtkWindow вместо GtkDialog.

## Заголовочный файл

Заголовочный файл похож на файл TfeTextView.

~~~C
 1 #pragma once
 2 
 3 #include <gtk/gtk.h>
 4 
 5 #define TFE_TYPE_ALERT tfe_alert_get_type ()
 6 G_DECLARE_FINAL_TYPE (TfeAlert, tfe_alert, TFE, ALERT, GtkWindow)
 7 
 8 /* "response" signal id */
 9 enum TfeAlertResponseType
10 {
11   TFE_ALERT_RESPONSE_ACCEPT,
12   TFE_ALERT_RESPONSE_CANCEL
13 };
14 
15 const char *
16 tfe_alert_get_title (TfeAlert *alert);
17 
18 const char *
19 tfe_alert_get_message (TfeAlert *alert);
20 
21 const char *
22 tfe_alert_get_button_label (TfeAlert *alert);
23 
24 void
25 tfe_alert_set_title (TfeAlert *alert, const char *title);
26 
27 void
28 tfe_alert_set_message (TfeAlert *alert, const char *message);
29 
30 void
31 tfe_alert_set_button_label (TfeAlert *alert, const char *btn_label);
32 
33 GtkWidget *
34 tfe_alert_new (void);
35 
36 GtkWidget *
37 tfe_alert_new_with_data (const char *title, const char *message, const char* btn_label);
~~~

- 5-6: Эти две строки всегда необходимы для определения нового объекта.
`TFE_TYPE_ALERT` — это тип объекта TfeAlert, и это макрос, разворачиваемый в `tfe_alert_get_type ()`.
Макрос G\_DECLARE\_FINAL\_TYPE разворачивается в:
  - Объявление функции `tfe_alert_get_type`
  - `TfeAlert` определяется как typedef для `struct _TfeAlert`, которая определена в C-файле.
  - Макросы `TFE_ALERT` и `TFE_IS_ALERT` определяются как функции приведения типа и проверки типа.
  - Структура `TfeAlertClass` определяется как финальный класс.
- 8-13: Класс TfeAlert имеет сигнал "response".
Он имеет параметр, и тип параметра определен как перечисляемая константа `TfeAlertResponseType`.
- 15-31: Методы получения и установки.
- 33-37: Функции для создания экземпляра.
Функция `tfe_alert_new_with_data` — это вспомогательная функция, которая создает экземпляр и устанавливает данные одновременно.

## C-файл

### Функции для составных виджетов

Следующий код извлечен из `tfealert.c`.

```C
#include <gtk/gtk.h>
#include "tfealert.h"

struct _TfeAlert {
  GtkWindow parent;
  GtkLabel *lb_title;
  GtkLabel *lb_message;
  GtkButton *btn_accept;
  GtkButton *btn_cancel;
};

G_DEFINE_FINAL_TYPE (TfeAlert, tfe_alert, GTK_TYPE_WINDOW);

static void
cancel_cb (TfeAlert *alert) {
  ... ... ...
}

static void
accept_cb (TfeAlert *alert) {
  ... ... ...
}

static void
tfe_alert_dispose (GObject *gobject) { // gobject is actually a TfeAlert instance.
  gtk_widget_dispose_template (GTK_WIDGET (gobject), TFE_TYPE_ALERT);
  G_OBJECT_CLASS (tfe_alert_parent_class)->dispose (gobject);
}

static void
tfe_alert_init (TfeAlert *alert) {
  gtk_widget_init_template (GTK_WIDGET (alert));
}

static void
tfe_alert_class_init (TfeAlertClass *class) {
  G_OBJECT_CLASS (class)->dispose = tfe_alert_dispose;
  gtk_widget_class_set_template_from_resource (GTK_WIDGET_CLASS (class), "/com/github/ToshioCP/tfe/tfealert.ui");
  gtk_widget_class_bind_template_child (GTK_WIDGET_CLASS (class), TfeAlert, lb_title);
  gtk_widget_class_bind_template_child (GTK_WIDGET_CLASS (class), TfeAlert, lb_message);
  gtk_widget_class_bind_template_child (GTK_WIDGET_CLASS (class), TfeAlert, btn_accept);
  gtk_widget_class_bind_template_child (GTK_WIDGET_CLASS (class), TfeAlert, btn_cancel);
  gtk_widget_class_bind_template_callback (GTK_WIDGET_CLASS (class), cancel_cb);
  gtk_widget_class_bind_template_callback (GTK_WIDGET_CLASS (class), accept_cb);
  ... ... ...
}

GtkWidget *
tfe_alert_new (void) {
  return GTK_WIDGET (g_object_new (TFE_TYPE_ALERT, NULL));
}
```

- Макрос `G_DEFINE_FINAL_TYPE` доступен с версии GLib 2.70.
Он используется только для финальных классов.
Вместо него можно использовать макрос `G_DEFINE_TYPE`.
Они разворачиваются в:
  - Объявление функций `tfe_alert_init` и `tfe_alert_class_init`.
Они определены в следующей части программы на C.
  - Определение переменной `tfe_alert_parent_class`.
  - Определение функции `tfe_alert_get_type`.
- Имена членов `_TfeAlert`, которые являются `lb_title`, `lb_message`, `btn_accept` и `btn_cancel`,
должны совпадать с атрибутом id в XML-файле `tfealert.ui`.
- Функция `tfe_alert_class_init` инициализирует класс составного виджета.
  - Функция `gtk_widget_class_set_template_from_resource` устанавливает шаблон класса.
Шаблон строится из XML-ресурса "tfealert.ui".
В этот момент экземпляр не создается.
Это просто заставляет класс распознать структуру объекта.
Вот почему тег верхнего уровня в XML-файле — template, а не object.
  - Макрос-функция `gtk_widget_class_bind_template_child` связывает член TfeAlert и класс объекта в шаблоне.
Таким образом, например, вы можете получить доступ к экземпляру GtkLabel `lb_title` через `alert->lb_title`, где `alert` — это экземпляр класса TfeAlert.
  - Функция `gtk_widget_class_bind_template_callback` связывает функцию обратного вызова и атрибут `handler` тега signal в XML.
Например, сигнал "clicked" на кнопке отмены имеет обработчик с именем "cancel\_cb" в теге signal.
И функция `cancel_cb` существует в C-файле выше.
Они связаны, поэтому когда сигнал испускается, вызывается функция `cancel_cb`.
Благодаря этой связи вы можете добавить класс хранения `static` к функции обратного вызова.
- Функция `tfe_alert_init` инициализирует вновь созданный экземпляр.
Необходимо вызвать `gtk_widget_init_template`, чтобы создать и инициализировать дочерние виджеты в шаблоне.
- Функция `tfe_alert_despose` освобождает объекты.
Функция `gtk_widget_despose_template` очищает дочерние элементы шаблона.
- Функция `tfe_alert_new` создает экземпляр составного виджета TfeAlert.
Она создает не только сам TfeAlert, но и все дочерние виджеты, которые имеет составной виджет.

### Другие функции

Ниже приведен полный код `tfealert.c`.

~~~C
  1 #include <gtk/gtk.h>
  2 #include "tfealert.h"
  3 
  4 struct _TfeAlert {
  5   GtkWindow parent;
  6   GtkLabel *lb_title;
  7   GtkLabel *lb_message;
  8   GtkButton *btn_accept;
  9   GtkButton *btn_cancel;
 10 };
 11 
 12 G_DEFINE_FINAL_TYPE (TfeAlert, tfe_alert, GTK_TYPE_WINDOW);
 13 
 14 enum {
 15   RESPONSE,
 16   NUMBER_OF_SIGNALS
 17 };
 18 
 19 static guint tfe_alert_signals[NUMBER_OF_SIGNALS];
 20 
 21 static void
 22 cancel_cb (TfeAlert *alert) {
 23   g_signal_emit (alert, tfe_alert_signals[RESPONSE], 0, TFE_ALERT_RESPONSE_CANCEL);
 24   gtk_window_destroy (GTK_WINDOW (alert));
 25 }
 26 
 27 static void
 28 accept_cb (TfeAlert *alert) {
 29   g_signal_emit (alert, tfe_alert_signals[RESPONSE], 0, TFE_ALERT_RESPONSE_ACCEPT);
 30   gtk_window_destroy (GTK_WINDOW (alert));
 31 }
 32 
 33 const char *
 34 tfe_alert_get_title (TfeAlert *alert) {
 35   return gtk_label_get_text (alert->lb_title);
 36 }
 37 
 38 const char *
 39 tfe_alert_get_message (TfeAlert *alert) {
 40     return gtk_label_get_text (alert->lb_message);
 41 }
 42 
 43 const char *
 44 tfe_alert_get_button_label (TfeAlert *alert) {
 45   return gtk_button_get_label (alert->btn_accept);
 46 }
 47 
 48 void
 49 tfe_alert_set_title (TfeAlert *alert, const char *title) {
 50   gtk_label_set_text (alert->lb_title, title);
 51 }
 52 
 53 void
 54 tfe_alert_set_message (TfeAlert *alert, const char *message) {
 55   gtk_label_set_text (alert->lb_message, message);
 56 }
 57 
 58 void
 59 tfe_alert_set_button_label (TfeAlert *alert, const char *btn_label) {
 60   gtk_button_set_label (alert->btn_accept, btn_label);
 61 }
 62 
 63 static void
 64 tfe_alert_dispose (GObject *gobject) { // gobject is actually a TfeAlert instance.
 65   gtk_widget_dispose_template (GTK_WIDGET (gobject), TFE_TYPE_ALERT);
 66   G_OBJECT_CLASS (tfe_alert_parent_class)->dispose (gobject);
 67 }
 68 
 69 static void
 70 tfe_alert_init (TfeAlert *alert) {
 71   gtk_widget_init_template (GTK_WIDGET (alert));
 72 }
 73 
 74 static void
 75 tfe_alert_class_init (TfeAlertClass *class) {
 76   G_OBJECT_CLASS (class)->dispose = tfe_alert_dispose;
 77   gtk_widget_class_set_template_from_resource (GTK_WIDGET_CLASS (class), "/com/github/ToshioCP/tfe/tfealert.ui");
 78   gtk_widget_class_bind_template_child (GTK_WIDGET_CLASS (class), TfeAlert, lb_title);
 79   gtk_widget_class_bind_template_child (GTK_WIDGET_CLASS (class), TfeAlert, lb_message);
 80   gtk_widget_class_bind_template_child (GTK_WIDGET_CLASS (class), TfeAlert, btn_accept);
 81   gtk_widget_class_bind_template_child (GTK_WIDGET_CLASS (class), TfeAlert, btn_cancel);
 82   gtk_widget_class_bind_template_callback (GTK_WIDGET_CLASS (class), cancel_cb);
 83   gtk_widget_class_bind_template_callback (GTK_WIDGET_CLASS (class), accept_cb);
 84 
 85   tfe_alert_signals[RESPONSE] = g_signal_new ("response",
 86                                 G_TYPE_FROM_CLASS (class),
 87                                 G_SIGNAL_RUN_LAST | G_SIGNAL_NO_RECURSE | G_SIGNAL_NO_HOOKS,
 88                                 0 /* class offset */,
 89                                 NULL /* accumulator */,
 90                                 NULL /* accumulator data */,
 91                                 NULL /* C marshaller */,
 92                                 G_TYPE_NONE /* return_type */,
 93                                 1     /* n_params */,
 94                                 G_TYPE_INT
 95                                 );
 96 }
 97 
 98 GtkWidget *
 99 tfe_alert_new (void) {
100   return GTK_WIDGET (g_object_new (TFE_TYPE_ALERT, NULL));
101 }
102 
103 GtkWidget *
104 tfe_alert_new_with_data (const char *title, const char *message, const char* btn_label) {
105   GtkWidget *alert = tfe_alert_new ();
106   tfe_alert_set_title (TFE_ALERT (alert), title);
107   tfe_alert_set_message (TFE_ALERT (alert), message);
108   tfe_alert_set_button_label (TFE_ALERT (alert), btn_label);
109   return alert;
110 }
~~~

Функция `tfe_alert_new_with_data` используется чаще, чем `tfe_alert_new`, для создания нового экземпляра.
Она создает экземпляр и устанавливает три данных одновременно.
Ниже приведен общий процесс при использовании класса TfeAlert.

- Вызовите `tfe_alert_new_with_data` и создайте экземпляр.
- Вызовите `gtk_window_set_transient_for`, чтобы установить временное родительское окно.
- Вызовите `gtk_window_present`, чтобы показать диалог TfeAlert.
- Подключите сигнал "response" и обработчик.
- Пользователь нажимает кнопку отмены или принятия.
Затем диалог испускает сигнал "response" и уничтожает себя.
- Пользователь перехватывает сигнал и что-то делает.

Остальная часть программы:

- 14-19: Массив для идентификатора сигнала. Можно использовать переменную вместо массива, потому что у класса только один сигнал.
Но использование массива — это обычный способ.
- 21-31: Обработчики сигналов. Они испускают сигнал "response" и уничтожают сам экземпляр.
- 33-61: Методы получения и установки.
- 85-95: Создает сигнал "response".
- 103-110: Вспомогательная функция `tfe_alert_new_with_data` создает экземпляр и устанавливает метки.

## Пример

В директории `src/tfe6/example` есть пример.
Он показывает, как использовать TfeAlert.
Программа находится в `src/example/ex_alert.c`.

~~~C
 1 #include <gtk/gtk.h>
 2 #include "../tfealert.h"
 3 
 4 static void
 5 alert_response_cb (TfeAlert *alert, int response, gpointer user_data) {
 6   if (response == TFE_ALERT_RESPONSE_ACCEPT)
 7     g_print ("%s\n", tfe_alert_get_button_label (alert));
 8   else if (response == TFE_ALERT_RESPONSE_CANCEL)
 9     g_print ("Cancel\n");
10   else
11     g_print ("Unexpected error\n");
12 }
13 
14 static void
15 app_activate (GApplication *application) {
16   GtkWidget *alert;
17   char *title, *message, *btn_label;
18 
19   title = "Example for TfeAlert"; message = "Click on Cancel or Accept button"; btn_label = "Accept";
20   alert = tfe_alert_new_with_data (title, message, btn_label);
21   g_signal_connect (TFE_ALERT (alert), "response", G_CALLBACK (alert_response_cb), NULL);
22   gtk_window_set_application (GTK_WINDOW (alert), GTK_APPLICATION (application));
23   gtk_window_present (GTK_WINDOW (alert));
24 }
25 
26 static void
27 app_startup (GApplication *application) {
28 }
29 
30 #define APPLICATION_ID "com.github.ToshioCP.example_tfe_alert"
31 
32 int
33 main (int argc, char **argv) {
34   GtkApplication *app;
35   int stat;
36 
37   app = gtk_application_new (APPLICATION_ID, G_APPLICATION_DEFAULT_FLAGS);
38   g_signal_connect (app, "startup", G_CALLBACK (app_startup), NULL);
39   g_signal_connect (app, "activate", G_CALLBACK (app_activate), NULL);
40   stat = g_application_run (G_APPLICATION (app), argc, argv);
41   g_object_unref (app);
42   return stat;
43 }
~~~

Обработчик сигнала "activate" `app_activate` инициализирует диалог предупреждения.

- Создается экземпляр TfeAlert.
- Его сигнал "response" подключается к обработчику `alert_response_cb`.
- Класс TfeAlert является подклассом GtkWindow, поэтому он может быть окном верхнего уровня, подключенным к экземпляру приложения.
Функция `gtk_window_set_application` делает это.
- Диалог отображается.

Пользователь нажимает либо на кнопку отмены, либо на кнопку принятия.
Затем испускается сигнал "response", и диалог уничтожается.
Обработчик сигнала `alert_response_cb` проверяет ответ и печатает "Accept" или "Cancel".
Если происходит ошибка, он печатает "Unexpected error".

Вы можете скомпилировать его с помощью meson и ninja.

```
$ cd src/tfe6/example
$ meson setup _build
$ ninja -C _build
$ _build/ex_alert
Accept #<= если вы нажали на кнопку принятия
```

Up: [README.md](../README.md),  Prev: [Section 19](sec19.md), Next: [Section 21](sec21.md)
