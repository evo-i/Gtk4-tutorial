Up: [README.md](../README.md),  Prev: [Section 11](sec11.md), Next: [Section 13](sec13.md)

# Сигналы

## Сигналы

Каждый объект инкапсулирован в программировании Gtk.
И не рекомендуется использовать глобальные переменные, потому что они склонны усложнять программу.
Итак, нам нужно что-то для связи между объектами.
Есть два способа сделать это.

- Методы экземпляра:
Методы экземпляра — это функции на экземплярах.
Например, `tb = gtk_text_view_get_buffer (tv)` — это метод экземпляра на экземпляре `tv`.
Вызывающий запрашивает у `tv` предоставить `tb`, который является экземпляром GtkTextBuffer, подключённым к `tv`.
- Сигналы:
Например, сигнал `activate` на объекте GApplication.
Когда приложение активируется, сигнал испускается.
Затем вызывается обработчик, который был подключён к сигналу.

Вызывающий методов или сигналов обычно находится вне объекта.
Одно из различий между ними заключается в том, что объект является активным или пассивным.
В методах объекты пассивно отвечают вызывающему.
В сигналах объекты активно отправляют сигналы обработчикам.

Сигналы GObject регистрируются, подключаются и испускаются.

1. Сигналы регистрируются в классе.
Регистрация обычно выполняется при инициализации класса.
Сигналы могут иметь обработчик по умолчанию, который иногда называется "обработчик метода объекта".
Это не пользовательский обработчик, подключённый функциями семейства `g_signal_connect`.
Обработчик по умолчанию всегда вызывается для любого экземпляра класса.
1. Сигналы подключаются к обработчикам с помощью макроса `g_signal_connect` или его семейства функций.
Подключение обычно выполняется вне объекта.
Одна важная вещь заключается в том, что сигналы подключаются к определённому экземпляру.
Предположим, существуют два экземпляра GtkButton A, B и функция C.
Даже если вы подключили сигнал "clicked" на A к C, B и C *не* подключены.
1. Когда сигналы испускаются, вызываются подключённые обработчики.
Сигналы испускаются на экземпляре класса.

## Регистрация сигналов

В TfeTextView регистрируются два сигнала.

- сигнал "change-file".
Этот сигнал испускается, когда изменяется `tv->file`.
- сигнал "open-response".
Функция `tfe_text_view_open` не возвращает статус, потому что она не может получить статус из диалога выбора файла.
(Вместо этого функция обратного вызова получает статус.)
Этот сигнал испускается вместо возвращаемого значения функции.

Статическая переменная или массив используется для хранения идентификатора сигнала.

~~~C
enum {
  CHANGE_FILE,
  OPEN_RESPONSE,
  NUMBER_OF_SIGNALS
};

static guint tfe_text_view_signals[NUMBER_OF_SIGNALS];
~~~

Сигналы регистрируются в функции инициализации класса.

~~~C
 1 static void
 2 tfe_text_view_class_init (TfeTextViewClass *class) {
 3   GObjectClass *object_class = G_OBJECT_CLASS (class);
 4 
 5   object_class->dispose = tfe_text_view_dispose;
 6   tfe_text_view_signals[CHANGE_FILE] = g_signal_new ("change-file",
 7                                  G_TYPE_FROM_CLASS (class),
 8                                  G_SIGNAL_RUN_LAST | G_SIGNAL_NO_RECURSE | G_SIGNAL_NO_HOOKS,
 9                                  0 /* class offset */,
10                                  NULL /* accumulator */,
11                                  NULL /* accumulator data */,
12                                  NULL /* C marshaller */,
13                                  G_TYPE_NONE /* return_type */,
14                                  0     /* n_params */
15                                  );
16   tfe_text_view_signals[OPEN_RESPONSE] = g_signal_new ("open-response",
17                                  G_TYPE_FROM_CLASS (class),
18                                  G_SIGNAL_RUN_LAST | G_SIGNAL_NO_RECURSE | G_SIGNAL_NO_HOOKS,
19                                  0 /* class offset */,
20                                  NULL /* accumulator */,
21                                  NULL /* accumulator data */,
22                                  NULL /* C marshaller */,
23                                  G_TYPE_NONE /* return_type */,
24                                  1     /* n_params */,
25                                  G_TYPE_INT
26                                  );
27 }
~~~

- 6-15: регистрирует сигнал "change-file".
Используется функция `g_signal_new`.
Сигнал "change-file" не имеет обработчика по умолчанию (обработчик метода объекта), поэтому смещение (строка 9) установлено в ноль.
Обычно вам не нужен обработчик по умолчанию.
Если он вам нужен, используйте функцию `g_signal_new_class_handler` вместо `g_signal_new`.
Для получения дополнительной информации см. [GObject API Reference](https://docs.gtk.org/gobject/func.signal_new_class_handler.html).
- Возвращаемое значение `g_signal_new` — это идентификатор сигнала.
Тип идентификатора сигнала — guint, что совпадает с unsigned int.
Он используется в функции `g_signal_emit`.
- 16-26: регистрирует сигнал "open-response".
Этот сигнал имеет параметр.
- 24: количество параметров.
Сигнал "open-response" имеет один параметр.
- 25: тип параметра.
`G_TYPE_INT` — это тип целого числа.
Такие фундаментальные типы описаны в [руководстве по GObject](https://developer-old.gnome.org/gobject/stable/gobject-Type-Information.html).

Обработчики объявляются следующим образом.

~~~C
/* обработчик сигнала "change-file" */
void
user_function (TfeTextView *tv,
               gpointer user_data)

/* обработчик сигнала "open-response" */
void
user_function (TfeTextView *tv,
               TfeTextViewOpenResponseType response-id,
               gpointer user_data)
~~~

- Сигнал "change-file" не имеет параметра, поэтому аргументы обработчика — это экземпляр TfeTextView и пользовательские данные.
- Сигнал "open-response" имеет один параметр, и его аргументы — это экземпляр TfeTextView, параметр сигнала и пользовательские данные.
- Переменная `tv` указывает на экземпляр, на котором испускается сигнал.
- Последний аргумент `user_data` берётся из четвёртого аргумента `g_signal_connect`.
- `Параметр` (`response-id`) берётся из четвёртого аргумента `g_signal_emit`.

Значения типа `TfeTextViewOpenResponseType` определены в `tfetextview.h`.

```C
/* ответ сигнала "open-response" */
enum TfeTextViewOpenResponseType
{
  TFE_OPEN_RESPONSE_SUCCESS,
  TFE_OPEN_RESPONSE_CANCEL,
  TFE_OPEN_RESPONSE_ERROR
};
```

- Параметр устанавливается в `TFE_OPEN_RESPONSE_SUCCESS`, когда `tfe_text_view_open` успешно открыла файл и прочитала его.
- Параметр устанавливается в `TFE_OPEN_RESPONSE_CANCEL`, когда пользователь отменил действие.
- Параметр устанавливается в `TFE_OPEN_RESPONSE_ERROR`, когда произошла ошибка.
 
## Подключение сигналов

Сигнал и обработчик подключаются с помощью функционального макроса `g_signal_connect`.
Существуют похожие функциональные макросы, такие как `g_signal_connect_after`, `g_signal_connect_swapped` и так далее.
Однако `g_signal_connect` используется чаще всего.
Сигналы "change-file" и "open-response" подключаются к их функциям обратного вызова вне объекта TfeTextView.
Эти функции обратного вызова определяются пользователями.

Например, функции обратного вызова определены в `src/tfe6/tfewindow.c`, и их имена — `file_changed_cb` и `open_response_cb`.
Они будут объяснены позже.

~~~C
g_signal_connect (GTK_TEXT_VIEW (tv), "change-file", G_CALLBACK (file_changed_cb), nb);

g_signal_connect (TFE_TEXT_VIEW (tv), "open-response", G_CALLBACK (open_response_cb), nb);
~~~

## Испускание сигналов

Сигнал испускается на экземпляре.
Функция `g_signal_emit` используется для испускания сигнала.
Следующие строки извлечены из `src/tfetextview/tfetextview.c`.
Каждая строка берётся из разных строк.

~~~C
g_signal_emit (tv, tfe_text_view_signals[CHANGE_FILE], 0);
g_signal_emit (tv, tfe_text_view_signals[OPEN_RESPONSE], 0, TFE_OPEN_RESPONSE_SUCCESS);
g_signal_emit (tv, tfe_text_view_signals[OPEN_RESPONSE], 0, TFE_OPEN_RESPONSE_CANCEL);
g_signal_emit (tv, tfe_text_view_signals[OPEN_RESPONSE], 0, TFE_OPEN_RESPONSE_ERROR);
~~~

- Первый аргумент — это экземпляр, на котором испускается сигнал.
- Второй аргумент — это идентификатор сигнала.
- Третий аргумент — это детали сигнала.
Сигналы "change-file" и "open-response" не имеют деталей, и аргументы равны нулю.
- Сигнал "change-file" не имеет параметров, поэтому нет четвёртого аргумента.
- Сигнал "open-response" имеет один параметр.
Четвёртый аргумент — это параметр.

Up: [README.md](../README.md),  Prev: [Section 11](sec11.md), Next: [Section 13](sec13.md)
