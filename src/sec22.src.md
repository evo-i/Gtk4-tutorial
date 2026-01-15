# TfeWindow

## Окно Tfe и XML-файлы

Ниже показано окно Tfe.

![tfe6](../image/tfe6.png){width=9.06cm height=6.615cm}

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

@@@include
tfe6/tfewindow.ui
@@@

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

@@@include
tfe6/menu.ui
@@@

Здесь четыре пункта меню, и они связаны с действиями.

## Заголовочный файл

Ниже приведен код `tfewindow.h`.

@@@include
tfe6/tfewindow.h
@@@

- 5-6: Определение `TFE_TYPE_WINDOW` и макрос `G_DECLARE_FINAL_TYPE`.
- 8-15: Публичные функции. Первые две функции создают страницу блокнота, а последняя функция создает окно.

## C-файл

### Составной виджет

Следующий код извлечен из `tfewindow.c`.

@@@if gfm
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
@@@else
```{.C}
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
@@@end

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

@@@if gfm
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
@@@else
```{.C}
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
@@@end

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

@@@include
tfe6/tfewindow.c open_activated
@@@

Она подключает сигнал "open-response" к вновь созданному экземпляру TfeTextView и просто вызывает `tfe_text_view_open`.
Остальную часть задачи она оставляет обработчику сигнала `open_response_cb`.

@@@include
tfe6/tfewindow.c open_response_cb
@@@

Если экземпляр TfeTextView не смог прочитать файл, он уничтожает экземпляр с помощью `g_object_ref_sink` и `g_object_unref`.
Поскольку вновь созданные виджеты являются плавающими, вам нужно преобразовать плавающую ссылку в обычную ссылку перед её освобождением.
Преобразование выполняется с помощью `g_object_ref_sink`.

Если экземпляр успешно прочитал файл, вызывается `notebook_page_build` для построения страницы блокнота и добавления её к объекту GtkNotebook.

@@@include
tfe6/tfewindow.c notebook_page_build
@@@

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

@@@include
tfe6/tfewindow.c file_changed_cb modified_changed_cb
@@@

#### save\_activated

Функция обратного вызова `save_activated` - это обработчик сигнала activate для действия "save".

@@@include
tfe6/tfewindow.c save_activated
@@@

Эта функция получает текущий экземпляр TfeTextView с помощью функции `get_current_textview`.
И просто вызывает `tfe_text_view_save`.

@@@include
tfe6/tfewindow.c get_current_textview
@@@

#### close\_activated

Функция обратного вызова `close_activated` - это обработчик сигнала activate для действия "close".
Она закрывает текущую страницу блокнота.

@@@include
tfe6/tfewindow.c close_activated
@@@

Если текст на текущей странице был сохранен, она вызывает `notebook_page_close` для закрытия страницы.
В противном случае она устанавливает `win->is_quit` в FALSE и показывает диалог предупреждения.
Сигнал "response" диалога подключен к обработчику `alert_response_cb`.

@@@include
tfe6/tfewindow.c notebook_page_close
@@@

Если блокнот имеет только одну страницу, она уничтожает окно, и приложение завершается.
В противном случае она удаляет текущую страницу.

@@@include
tfe6/tfewindow.c alert_response_cb
@@@

Если пользователь нажал на кнопку отмены, она ничего не делает.
Если пользователь нажал на кнопку принятия, которая эквивалентна кнопке закрытия, она вызывает `notebook_page_close`.
Обратите внимание, что `win->is_quit` был установлен в FALSE в функции `close_activated`.

#### new\_activated

Функция обратного вызова `new_activated` - это обработчик сигнала activate для действия "new".

@@@include
tfe6/tfewindow.c new_activated
@@@

Она просто вызывает `tfe_window_notebook_page_new`, который является публичным методом TfeWindow.

@@@include
tfe6/tfewindow.c tfe_window_notebook_page_new
@@@

Эта функция создает новый экземпляр TfeTextView, строку семейства "Untitled" и вызывает `notebook_page_build`.

#### saveas\_activated

Функция обратного вызова `saveas_activated` - это обработчик сигнала activate для действия "saveas".

@@@include
tfe6/tfewindow.c saveas_activated
@@@

Эта функция получает экземпляр TfeTextView текущей страницы и вызывает `tfe_text_view_saveas`.

#### pref\_activated

Функция обратного вызова `pref_activated` - это обработчик сигнала activate для действия "pref".

@@@include
tfe6/tfewindow.c pref_activated
@@@

Эта функция создает экземпляр TfePref, который является диалогом, и устанавливает временное родительское окно в `win`.
И показывает диалог.

#### close\_all\_activated

Функция обратного вызова `close_all_activated` - это обработчик сигнала activate для действия "close_all".

@@@include
tfe6/tfewindow.c close_all_activated
@@@

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

@@@include
tfe6/tfewindow.c close_request_cb
@@@

Сначала она вызывает `is_saved_all` и проверяет, были ли сохранены тексты.
Если да, она возвращает FALSE, и процесс закрытия продолжается.
В противном случае она устанавливает `win->is_quit` в TRUE и показывает диалог предупреждения.
Когда пользователь нажимает на кнопку принятия или отмены, диалог исчезает, и испускается сигнал "response".
Затем вызывается обработчик `alert_response_cb`.
Он уничтожает главное окно, если пользователь нажал на кнопку принятия, поскольку `win->is_quit` равен TRUE.
В противном случае он ничего не делает.

@@@include
tfe6/tfewindow.c is_saved_all
@@@

### Публичные функции

Есть три публичные функции.

- `void tfe_window_notebook_page_new (TfeWindow *win)`
- `void tfe_window_notebook_page_new_with_files (TfeWindow *win, GFile **files, int n_files)`
- `GtkWidget *tfe_window_new (GtkApplication *app)`

Первая функция вызывается, когда приложение испускает сигнал "activate".
Вторая - для сигнала "open".
Ей передаются три аргумента, и они принадлежат вызывающей стороне.

@@@include
tfe6/tfewindow.c tfe_window_notebook_page_new_with_files
@@@

Эта функция имеет цикл для массива `files`.
Она создает экземпляр TfeTextView с текстом из каждого файла.
И строит с ним страницу.

Если происходит ошибка и ни одна страница не создана, она создает новую пустую страницу.

### Полный код tfewindow.c

Ниже приведен полный исходный код `tfewindow.c`.

@@@include
tfe6/tfewindow.c
@@@
