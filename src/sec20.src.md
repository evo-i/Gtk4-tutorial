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

![Alert dialog](../image/alert.png){width=4.2cm height=1.7cm}

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

@@@include
tfe6/tfealert.ui
@@@

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

![dialog-warning icon is like ...](../image/dialog_warning.png){width=4.19cm height=1.62cm}

Они сделаны вручную.
Реальное изображение в диалоге предупреждения выглядит лучше.

Можно определить виджет предупреждения как дочерний от GtkDialog.
Но GtkDialog устарел, начиная с версии GTK 4.10.
И пользователям следует использовать GtkWindow вместо GtkDialog.

## Заголовочный файл

Заголовочный файл похож на файл TfeTextView.

@@@include
tfe6/tfealert.h
@@@

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

@@@if gfm
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
@@@else
```{.C}
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
@@@end

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

@@@include
tfe6/tfealert.c
@@@

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

@@@include
tfe6/example/ex_alert.c
@@@

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
