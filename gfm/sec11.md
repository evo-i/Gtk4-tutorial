Up: [README.md](../README.md),  Prev: [Section 10](sec10.md), Next: [Section 12](sec12.md)

# Инициализация и уничтожение экземпляра

Новая версия текстового редактора файлов (`tfe`) будет создана в этом разделе и следующих четырёх разделах.
Это `tfe5`.
Есть много изменений по сравнению с предыдущей версией.
Они находятся в двух каталогах, [src/tfe5](../src/tfe5) и [src/tfetextview](../src/tfetextview).

## Инкапсуляция

Мы разделили исходный файл C на две части.
Но этого недостаточно с точки зрения инкапсуляции.

- `tfe.c` включает всё, кроме TfeTextView.
Его следует разделить как минимум на две части, `tfeapplication.c` и `tfenotebook.c`.
- Заголовочные файлы также необходимо организовать.

Однако прежде всего я хотел бы сосредоточиться на объекте TfeTextView.
Это дочерний объект GtkTextView и имеет новый член `file` в нём.
Важно управлять объектом GFile, на который указывает `file`.

- Что необходимо для GFile при создании (или инициализации) TfeTextView?
- Что необходимо для GFile при уничтожении TfeTextView?
- Должен ли TfeTextView читать/записывать файл самостоятельно или нет?
- Как он взаимодействует с объектами вне?

Вам нужно знать хотя бы о классах, экземплярах и сигналах, прежде чем думать о них.
Я объясню их в этом разделе и следующем разделе.
После этого я объясню:

- Организацию функций.
- Как использовать GtkFileDialog.
Это новый класс, созданный в версии 4.10, который заменяет GtkFileChooserDialog.

## GObject и его потомки

GObject и его потомки — это объекты, которые имеют как класс, так и объектные структуры C.
Сначала подумайте об экземплярах.
Экземпляр — это память, которая имеет объектную структуру.
Ниже приведена структура TfeTextView.

~~~C
/* Это оператор typedef автоматически генерируется макросом G_DECLARE_FINAL_TYPE */
typedef struct _TfeTextView TfeTextView;

struct _TfeTextView {
  GtkTextView parent;
  GFile *file;
};
~~~

Члены структуры:

- Член `parent` — это структура C GtkTextView.
Она объявлена в `gtktextview.h`.
GtkTextView является родителем TfeTextView.
- Член `file` — это указатель на GFile. Он может быть NULL, если экземпляр TfeTextView не имеет файла.
Наиболее распространённый случай — это когда экземпляр только что создан.

Вы можете найти объявление структур предков в исходных файлах в GTK или GLib.
Ниже приведено извлечение из исходных файлов (не совсем то же самое).

~~~C
typedef struct _GObject GObject;
typedef struct _GObject GInitiallyUnowned;
struct  _GObject
{
  GTypeInstance  g_type_instance;
  volatile guint ref_count;
  GData         *qdata;
};

typedef struct _GtkWidget GtkWidget;
struct _GtkWidget
{
  GInitiallyUnowned parent_instance;
  GtkWidgetPrivate *priv;
};

typedef struct _GtkTextView GtkTextView;
struct _GtkTextView
{
  GtkWidget parent_instance;
  GtkTextViewPrivate *priv;
};
~~~

В каждой структуре её родитель объявлен в верхней части членов.
Таким образом, все предки включены в дочерний объект.
Структура `TfeTextView` похожа на следующую диаграмму.

![The structure of the instance TfeTextView](../image/TfeTextView.png)

Производные классы (классы предков) имеют свою собственную область частных данных, которая не включена в структуру выше.
Например, GtkWidget имеет GtkWidgetPrivate (структура C) для своих частных данных.

Обратите внимание, что объявления — это не определения.
Поэтому память не выделяется при объявлении структур C.
Память выделяется им из области кучи при вызове функции `tfe_text_view_new`.
В то же время для TfeTetView выделяется частная область предков.
Они скрыты от TfeTextView, и он не может напрямую обращаться к ним.
Созданная память называется экземпляром.
Когда создаётся экземпляр TfeTextView, ему предоставляются три области данных.

- Экземпляр (структура C).
- Структура GtkWidgetPrivate.
- Структура GtkTextViewPrivate.

Функции TfeTextView могут обращаться только к своему экземпляру.
GtkWidgetPrivate и GtkTextViewPrivate используются функциями предков.
См. следующий пример.

~~~C
GtkWidget *tv = tfe_text_view_new ();
GtkTextBuffer *buffer = gtk_text_view_get_buffer (GTK_TEXT_VIEW (tv));
~~~

Функция родителя `gtk_text_view_get_buffer` обращается к данным GtkTextViewPrivate (принадлежащим `tv`).
В частной области есть указатель, который указывает на GtkBuffer, и функция возвращает указатель.
(Фактическое поведение немного сложнее.)

Экземпляры TfeTextView наследуют функции предков таким образом.

Экземпляр TfeTextView создаётся каждый раз при вызове функции `tfe_text_view_new`.
Поэтому может существовать несколько экземпляров TfeTextView.

## Инициализация экземпляров TfeTextView

Функция `tfe_text_view_new` создаёт новый экземпляр TfeTextView.

~~~C
1 GtkWidget *
2 tfe_text_view_new (void) {
3   return GTK_WIDGET (g_object_new (TFE_TYPE_TEXT_VIEW, "wrap-mode", GTK_WRAP_WORD_CHAR, NULL));
4 }
~~~

Когда эта функция вызывается, создаётся и инициализируется экземпляр TfeTextView.
Процесс инициализации следующий.

1. При создании экземпляра также создаются структуры GtkWidgetPrivate и GtkTextViewPrivate
2. Инициализирует часть GObject (GInitiallyUnowned) в экземпляре TfeTextView.
3. Инициализирует часть GtkWidget (первый `priv`) в экземпляре TfeTextView и структуру GtkWidgetPrivate.
4. Инициализирует часть GtkTextView (второй `priv`) в экземпляре TfeTextView и структуру GtkTextViewPrivate.
5. Инициализирует часть TfeTextView (`file`) в экземпляре TfeTextView.

Шаги со второго по четвёртый выполняются функциями `g_object_init`, `gtk_widget_init` и `gtk_text_view_init`.
Они вызываются системой автоматически, и вам не нужно о них заботиться.
Шаг пятый выполняется функцией `tfe_text_view_init` в `tfetextview.c`.

~~~C
1 static void
2 tfe_text_view_init (TfeTextView *tv) {
3   tv->file = NULL;
4 }
~~~

Эта функция просто инициализирует `tv->file` как `NULL`.

## Функции и классы

В Gtk все объекты, производные от GObject, имеют классы и экземпляры (но абстрактные объекты имеют только классы).
Экземпляры — это память структуры C, которая описана в предыдущих двух подразделах.
Каждый объект может иметь более одного экземпляра.
Эти экземпляры имеют одинаковую структуру.
Экземпляры просто имеют данные.
Поэтому они не определяют поведение объекта.
Нам нужны как минимум две вещи.
Одна — это функции, а другая — методы класса.

Последняя документация GTK 4 классифицирует функции на конструктор, функции и методы экземпляра.

- конструкторы: их имя всегда `gtk_(имя_объекта)_new`. Они создают объекты.
- функции: их первый параметр (аргумент) *НЕ* является экземпляром. Обычно функции являются утилитарными функциями для класса.
- методы экземпляра: их первый параметр (аргумент) является экземпляром. Они выполняют некоторую задачу для конкретного экземпляра.

Этот учебник использует `функции` в двух смыслах, широком или узком.

Вы уже видели много функций.
Например,

- `TfeTextView *tfe_text_view_new (void);` — это функция (конструктор) для создания экземпляра TfeTextView.
- `GtkTextBuffer *gtk_text_view_get_buffer (GtkTextView *textview)` — это функция (метод экземпляра) для получения GtkTextBuffer из GtkTextView.

Функции являются публичными, что означает, что ожидается их использование другими объектами.
Они похожи на публичные методы в объектно-ориентированных языках.

Класс (структура C) в основном состоит из указателей на функции C.
Они называются *методами класса* и используются самим объектом или его потомками.
Например, класс GObject объявлен в `gobject.h` в исходных файлах GLib.

~~~C
 1 typedef struct _GObjectClass             GObjectClass;
 2 typedef struct _GObjectClass             GInitiallyUnownedClass;
 3 
 4 struct  _GObjectClass
 5 {
 6   GTypeClass   g_type_class;
 7 
 8   /*< private >*/
 9   GSList      *construct_properties;
10 
11   /*< public >*/
12   /* seldom overridden */
13   GObject*   (*constructor)     (GType                  type,
14                                  guint                  n_construct_properties,
15                                  GObjectConstructParam *construct_properties);
16   /* overridable methods */
17   void       (*set_property)		(GObject        *object,
18                                          guint           property_id,
19                                          const GValue   *value,
20                                          GParamSpec     *pspec);
21   void       (*get_property)		(GObject        *object,
22                                          guint           property_id,
23                                          GValue         *value,
24                                          GParamSpec     *pspec);
25   void       (*dispose)			(GObject        *object);
26   void       (*finalize)		(GObject        *object);
27   /* seldom overridden */
28   void       (*dispatch_properties_changed) (GObject      *object,
29 					     guint	   n_pspecs,
30 					     GParamSpec  **pspecs);
31   /* signals */
32   void	     (*notify)			(GObject	*object,
33 					 GParamSpec	*pspec);
34 
35   /* called when done constructing */
36   void	     (*constructed)		(GObject	*object);
37 
38   /*< private >*/
39   gsize		flags;
40 
41   gsize         n_construct_properties;
42 
43   gpointer pspecs;
44   gsize n_pspecs;
45 
46   /* padding */
47   gpointer	pdummy[3];
48 };
49 
~~~

В строке 25 есть указатель на функцию `dispose`.

~~~C
void (*dispose) (GObject *object);
~~~

Объявление немного сложное.
Звёздочка перед идентификатором `dispose` означает указатель.
Таким образом, указатель `dispose` указывает на функцию, которая имеет один параметр, указывающий на структуру GObject, и не возвращает значения.
Аналогично, строка 26 говорит, что `finalize` — это указатель на функцию, которая имеет один параметр, указывающий на структуру GObject, и не возвращает значения.

~~~C
void (*finalize) (GObject *object);
~~~

Посмотрите на объявление `_GObjectClass`, чтобы обнаружить, что большинство членов являются указателями на функции.

- 13: функция, на которую указывает `constructor`, вызывается при создании экземпляра. Она завершает инициализацию экземпляра.
- 25: функция, на которую указывает `dispose`, вызывается, когда экземпляр уничтожает себя.
Процесс уничтожения разделён на две фазы.
Первая называется disposing (освобождение).
На этой фазе экземпляр освобождает все ссылки на другие экземпляры.
Вторая фаза — finalizing (финализация).
- 26: функция, на которую указывает `finalize`, завершает процесс уничтожения.
- Другие указатели указывают на функции, которые вызываются во время жизни экземпляра.

Эти функции называются методами класса.
Методы открыты для его потомков.
Но не открыты для объектов, которые не являются потомками.

## Класс TfeTextView

Класс TfeTextView — это структура, и он включает в себя все классы своих предков.
Поэтому классы имеют иерархию, аналогичную экземплярам.

~~~
GObjectClass (GInitiallyUnownedClass) -- GtkWidgetClass -- GtkTextViewClass -- TfeTextViewClass
~~~

Ниже приведено извлечение из исходных кодов (не совсем то же самое).

~~~C
  1 struct _GtkWidgetClass
  2 {
  3   GInitiallyUnownedClass parent_class;
  4 
  5   /*< public >*/
  6 
  7   /* basics */
  8   void (* show)                (GtkWidget        *widget);
  9   void (* hide)                (GtkWidget        *widget);
 10   void (* map)                 (GtkWidget        *widget);
 11   void (* unmap)               (GtkWidget        *widget);
 12   void (* realize)             (GtkWidget        *widget);
 13   void (* unrealize)           (GtkWidget        *widget);
 14   void (* root)                (GtkWidget        *widget);
 15   void (* unroot)              (GtkWidget        *widget);
 16   void (* size_allocate)       (GtkWidget           *widget,
 17                                 int                  width,
 18                                 int                  height,
 19                                 int                  baseline);
 20   void (* state_flags_changed) (GtkWidget        *widget,
 21                                 GtkStateFlags     previous_state_flags);
 22   void (* direction_changed)   (GtkWidget        *widget,
 23                                 GtkTextDirection  previous_direction);
 24 
 25   /* size requests */
 26   GtkSizeRequestMode (* get_request_mode)               (GtkWidget      *widget);
 27   void              (* measure) (GtkWidget      *widget,
 28                                  GtkOrientation  orientation,
 29                                  int             for_size,
 30                                  int            *minimum,
 31                                  int            *natural,
 32                                  int            *minimum_baseline,
 33                                  int            *natural_baseline);
 34 
 35   /* Mnemonics */
 36   gboolean (* mnemonic_activate)        (GtkWidget           *widget,
 37                                          gboolean             group_cycling);
 38 
 39   /* explicit focus */
 40   gboolean (* grab_focus)               (GtkWidget           *widget);
 41   gboolean (* focus)                    (GtkWidget           *widget,
 42                                          GtkDirectionType     direction);
 43   void     (* set_focus_child)          (GtkWidget           *widget,
 44                                          GtkWidget           *child);
 45 
 46   /* keyboard navigation */
 47   void     (* move_focus)               (GtkWidget           *widget,
 48                                          GtkDirectionType     direction);
 49   gboolean (* keynav_failed)            (GtkWidget           *widget,
 50                                          GtkDirectionType     direction);
 51 
 52   gboolean     (* query_tooltip)      (GtkWidget  *widget,
 53                                        int         x,
 54                                        int         y,
 55                                        gboolean    keyboard_tooltip,
 56                                        GtkTooltip *tooltip);
 57 
 58   void         (* compute_expand)     (GtkWidget  *widget,
 59                                        gboolean   *hexpand_p,
 60                                        gboolean   *vexpand_p);
 61 
 62   void         (* css_changed)                 (GtkWidget            *widget,
 63                                                 GtkCssStyleChange    *change);
 64 
 65   void         (* system_setting_changed)      (GtkWidget            *widget,
 66                                                 GtkSystemSetting      settings);
 67 
 68   void         (* snapshot)                    (GtkWidget            *widget,
 69                                                 GtkSnapshot          *snapshot);
 70 
 71   gboolean     (* contains)                    (GtkWidget *widget,
 72                                                 double     x,
 73                                                 double     y);
 74 
 75   /*< private >*/
 76 
 77   GtkWidgetClassPrivate *priv;
 78 
 79   gpointer padding[8];
 80 };
 81 
 82 struct _GtkTextViewClass
 83 {
 84   GtkWidgetClass parent_class;
 85 
 86   /*< public >*/
 87 
 88   void (* move_cursor)           (GtkTextView      *text_view,
 89                                   GtkMovementStep   step,
 90                                   int               count,
 91                                   gboolean          extend_selection);
 92   void (* set_anchor)            (GtkTextView      *text_view);
 93   void (* insert_at_cursor)      (GtkTextView      *text_view,
 94                                   const char       *str);
 95   void (* delete_from_cursor)    (GtkTextView      *text_view,
 96                                   GtkDeleteType     type,
 97                                   int               count);
 98   void (* backspace)             (GtkTextView      *text_view);
 99   void (* cut_clipboard)         (GtkTextView      *text_view);
100   void (* copy_clipboard)        (GtkTextView      *text_view);
101   void (* paste_clipboard)       (GtkTextView      *text_view);
102   void (* toggle_overwrite)      (GtkTextView      *text_view);
103   GtkTextBuffer * (* create_buffer) (GtkTextView   *text_view);
104   void (* snapshot_layer)        (GtkTextView      *text_view,
105 			          GtkTextViewLayer  layer,
106 			          GtkSnapshot      *snapshot);
107   gboolean (* extend_selection)  (GtkTextView            *text_view,
108                                   GtkTextExtendSelection  granularity,
109                                   const GtkTextIter      *location,
110                                   GtkTextIter            *start,
111                                   GtkTextIter            *end);
112   void (* insert_emoji)          (GtkTextView      *text_view);
113 
114   /*< private >*/
115 
116   gpointer padding[8];
117 };
118 
119 /* The following definition is generated by the macro G_DECLARE_FINAL_TYPE */
120 typedef struct {
121   GtkTextView parent_class;
122 } TfeTextViewClass;
123 
~~~

- 120-122: эти три строки генерируются макросом `G_DECLARE_FINAL_TYPE`.
Поэтому они не написаны ни в `tfe_text_view.h`, ни в `tfe_text_view.c`.
- 3, 84, 121: каждый класс имеет свой родительский класс в качестве первого члена своей структуры.
Это то же самое, что и структуры экземпляров.
- Члены класса в предках открыты для класса-потомка.
Поэтому их можно изменить в функции `tfe_text_view_class_init`.
Например, указатель `dispose` в GObjectClass будет переопределён позже в `tfe_text_view_class_init`.
(Override — это термин объектно-ориентированного программирования.
Override — это переписывание методов класса предков в классе-потомке.)
- Некоторые методы класса часто переопределяются.
`set_property`, `get_property`, `dispose`, `finalize` и `constructed` — такие методы.

TfeTextViewClass включает класс своих предков в себя.
Это проиллюстрировано на следующей диаграмме.

![The structure of TfeTextView Class](../image/TfeTextViewClass.png)

## Уничтожение TfeTextView

Каждый объект, производный от GObject, имеет счётчик ссылок.
Если объект A ссылается на объект B, то A хранит указатель на B в A и в то же время увеличивает счётчик ссылок B на единицу с помощью функции `g_object_ref (B)`.
Если A больше не нуждается в B, A отбрасывает указатель на B (обычно это делается путём присвоения NULL указателю) и уменьшает счётчик ссылок B на единицу с помощью функции `g_object_unref (B)`.

Если два объекта A и B ссылаются на C, то счётчик ссылок C равен двум.
Если A больше не нуждается в C, A отбрасывает указатель на C и уменьшает счётчик ссылок в C на единицу.
Теперь счётчик ссылок C равен единице.
Аналогично, если B больше не нуждается в C, B отбрасывает указатель на C и уменьшает счётчик ссылок в C на единицу.
В этот момент ни один объект не ссылается на C, и счётчик ссылок C равен нулю.
Это означает, что C больше не полезен.
Тогда C уничтожает себя, и, наконец, память, выделенная для C, освобождается.

![Reference count of B](../image/refcount.png)

Идея выше основана на предположении, что объект, на который никто не ссылается, имеет счётчик ссылок равный нулю.
Когда счётчик ссылок падает до нуля, объект начинает свой процесс уничтожения.
Процесс уничтожения разделён на две фазы: disposing и finalizing.
В процессе disposing объект вызывает функцию, на которую указывает `dispose` в его классе, чтобы освободить все ссылки на другие экземпляры.
После этого он вызывает функцию, на которую указывает `finalize` в его классе, чтобы завершить процесс уничтожения.

В процессе уничтожения TfeTextView нужно выполнить unref для GFile, на который указывает `tv->file`.
Вы должны написать обработчик dispose `tfe_text_view_dispose`.

~~~C
1 static void
2 tfe_text_view_dispose (GObject *gobject) {
3   TfeTextView *tv = TFE_TEXT_VIEW (gobject);
4 
5   if (G_IS_FILE (tv->file))
6     g_clear_object (&tv->file);
7 
8   G_OBJECT_CLASS (tfe_text_view_parent_class)->dispose (gobject);
9 }
~~~

- 5,6: если `tv->file` указывает на GFile, уменьшает счётчик ссылок экземпляра GFile.
Функция `g_clear_object` уменьшает счётчик ссылок и присваивает NULL переменной `tv->file`.
В обработчиках dispose мы обычно используем `g_clear_object`, а не `g_object_unref`.
- 8: вызывает обработчик dispose родителя. (Это будет объяснено позже.)

В процессе disposing объект использует указатель в своём классе для вызова обработчика.
Поэтому `tfe_text_view_dispose` должен быть зарегистрирован в классе при инициализации класса TfeTextView.
Функция `tfe_text_view_class_init` — это функция инициализации класса, и она объявлена в раскрытии макроса `G_DEFINE_TYPE`.

~~~C
static void
tfe_text_view_class_init (TfeTextViewClass *class) {
  GObjectClass *object_class = G_OBJECT_CLASS (class);

  object_class->dispose = tfe_text_view_dispose;
}
~~~

Каждый класс предков был создан до создания TfeTextViewClass.
Поэтому есть четыре класса, и каждый класс имеет указатель на каждый обработчик dispose.
Посмотрите на следующую диаграмму.
Есть четыре класса -- GObjectClass (GInitiallyUnownedClass), GtkWidgetClass, GtkTextViewClass и TfeTextViewClass.
Каждый класс имеет свой собственный обработчик dispose -- `dh1`, `dh2`, `dh3` и `tfe_text_view_dispose`.

![dispose handlers](../image/dispose_handler.png)

Теперь посмотрите на программу `tfe_text_view_dispose` выше.
Она сначала освобождает ссылку на объект GFile, на который указывает `tv->file`.
Затем она вызывает обработчик dispose родителя в строке 8.

~~~C
G_OBJECT_CLASS (tfe_text_view_parent_class)->dispose (gobject);
~~~

Переменная `tfe_text_view_parent_class`, которая создана макросом `G_DEFINE_FINAL_TYPE`, является указателем, который указывает на родительский класс объекта.
Переменная `gobject` — это указатель на экземпляр TfeTextView, который приведён к типу экземпляра GObject.
Поэтому `G_OBJECT_CLASS (tfe_text_view_parent_class)->dispose` указывает на обработчик `dh3` на диаграмме выше.
Оператор `G_OBJECT_CLASS (tfe_text_view_parent_class)->dispose (gobject)` то же самое, что `dh3 (gobject)`, что означает освобождение всех ссылок на другие экземпляры в GtkTextViewPrivate в экземпляре TfeTextView.
После этого `dh3` вызывает `dh2`, а `dh2` вызывает `dh1`.
В конечном итоге все ссылки освобождаются.

Up: [README.md](../README.md),  Prev: [Section 10](sec10.md), Next: [Section 12](sec12.md)
