Up: [README.md](../README.md),  Prev: [Section 32](sec32.md)

# GtkSignalListItemFactory

## GtkSignalListItemFactory и GtkBulderListItemFactory

GtkBuilderlistItemFactory удобен, когда GtkListView просто показывает содержимое списка.
Его направление привязки всегда от элемента списка к дочернему элементу GtkListItem.

Когда дело доходит до динамического соединения, этого недостаточно.
Например, предположим, что вы хотите редактировать содержимое списка.
Вы устанавливаете дочерний элемент GtkListItem как экземпляр GtkText, чтобы пользователь мог редактировать текст с его помощью.
Вам нужно привязать элемент в списке с буфером GtkText.
Направление противоположно тому, что в GtkBuilderListItemFactory.
Оно от экземпляра GtkText к элементу в списке.
Вы можете реализовать это с помощью GtkSignalListItemFactory, который более гибкий, чем GtkBuilderListItemFactory.

Этот раздел показывает лишь некоторые части исходного файла `listeditor.c`.
Если вы хотите увидеть весь код, см. каталог `src/listeditor` [репозитория учебника Gtk4](https://github.com/ToshioCP/Gtk4-tutorial).

## Редактор списка

Пример программы — это редактор списка, и данные списка — это строки.
Это то же самое, что и редактор строк.
Он читает текстовый файл построчно.
Каждая строка является элементом списка.
Список отображается с помощью GtkColumnView.
Есть два столбца.
Один — это кнопка, которая показывает, является ли строка текущей строкой.
Если строка является текущей строкой, кнопка окрашена в красный цвет.
Другой — это строка, которая является содержимым соответствующего элемента списка.

![List editor](../image/listeditor.png)

Исходные файлы расположены в каталоге `src/listeditor`.
Вы можете скомпилировать и выполнить его следующим образом.

- Загрузите программу из [репозитория](https://github.com/ToshioCP/Gtk4-tutorial).
- Измените текущий каталог на `src/listeditor`.
- Введите следующее в командной строке.

~~~
$ meson setup _build
$ ninja -C _build
$ _build/listeditor
~~~

- Кнопка Append: добавляет строку после текущей строки или в конец, если текущей строки не существует.
- Кнопка Insert: вставляет строку перед текущей строкой или в начало, если текущей строки не существует.
- Кнопка Remove: удаляет текущую строку.
- Кнопка Read: читает файл.
- Кнопка Write: записывает содержимое в файл.
- Кнопка close: закрывает содержимое.
- Кнопка quit: завершает приложение.
- Кнопка в столбце select: делает строку текущей.
- Столбец String: GtkText. Вы можете редактировать строку в поле.

Номер текущей строки (начиная с нуля) показан слева от панели инструментов.
Имя файла показано справа от кнопки write.

## Соединение экземпляра GtkText и элемента в списке

Второй столбец (GtkColumnViewColumn) устанавливает свойство factory в GtkSignalListItemFactory.
Он использует три сигнала: setup, bind и unbind.
Следующее показывает обработчики сигналов.

~~~C
 1 static void
 2 setup2_cb (GtkListItemFactory *factory, GtkListItem *listitem) {
 3   GtkWidget *text = gtk_text_new ();
 4   gtk_list_item_set_child (listitem, GTK_WIDGET (text));
 5   gtk_editable_set_alignment (GTK_EDITABLE (text), 0.0);
 6 }
 7 
 8 static void
 9 bind2_cb (GtkListItemFactory *factory, GtkListItem *listitem) {
10   GtkWidget *text = gtk_list_item_get_child (listitem);
11   GtkEntryBuffer *buffer = gtk_text_get_buffer (GTK_TEXT (text));
12   LeData *data = LE_DATA (gtk_list_item_get_item(listitem));
13   GBinding *bind;
14 
15   gtk_editable_set_text (GTK_EDITABLE (text), le_data_look_string (data));
16   gtk_editable_set_position (GTK_EDITABLE (text), 0);
17 
18   bind = g_object_bind_property (buffer, "text", data, "string", G_BINDING_DEFAULT);
19   g_object_set_data (G_OBJECT (listitem), "bind", bind);
20 }
21 
22 static void
23 unbind2_cb (GtkListItemFactory *factory, GtkListItem *listitem) {
24   GBinding *bind = G_BINDING (g_object_get_data (G_OBJECT (listitem), "bind"));
25 
26   if (bind)
27     g_binding_unbind(bind);
28   g_object_set_data (G_OBJECT (listitem), "bind", NULL);
29 }
~~~

- 1-6: `setup2_cb` — обработчик сигнала setup на GtkSignalListItemFactory.
Эта фабрика вставлена в свойство factory второго GtkColumnViewColumn.
Обработчик просто создаёт экземпляр GtkText и устанавливает child `listitem` на него.
Экземпляр будет уничтожен автоматически при уничтожении `listitem`.
Поэтому обработчик сигнала teardown не нужен.
- 8-20: `bind2_cb` — обработчик сигнала bind.
Он вызывается, когда `listitem` привязан к элементу в списке.
Элементы списка — это экземпляры LeData.
LeData определён в файле `listeditor.c` (исходный файл C редактора списка).
Это дочерний класс GObject и имеет строковые данные, которые являются содержимым строки.
  - 10-11: `text` — дочерний элемент `listitem`, и это экземпляр GtkText.
И `buffer` — это экземпляр GtkEntryBuffer для `text`.
  - 12: Экземпляр LeData `data` — это элемент, на который указывает `listitem`.
  - 15-16: Устанавливает текст `text` в `le_data_look_string (data)`.
le\_data\_look\_string возвращает строку `data`, и владение строкой по-прежнему принадлежит `data`.
Поэтому вызывающий не должен освобождать строку.
  - 18: `g_object_bind_property` привязывает свойство и свойство другого объекта.
Эта строка привязывает свойство "text" `buffer` (источник) и свойство "string" `data` (назначение).
Это однонаправленная привязка (`G_BINDING_DEFAULT`).
Когда пользователь изменяет текст GtkText, та же строка немедленно помещается в `data`.
Функция возвращает экземпляр GBinding.
Эта привязка отличается от привязок GtkExpression.
Эта привязка требует существования двух свойств.
  - 19: GObject имеет таблицу.
Ключ — это строка (или GQuark), а значение — gpointer (указатель на любой тип).
Функция `g_object_set_data` устанавливает ассоциацию от ключа к значению.
Эта строка устанавливает ассоциацию от "bind" к экземпляру `bind`.
Это делает возможным для обработчика "unbind" получить экземпляр `bind`.
- 22-29: `unbind2_cb` — обработчик сигнала unbind.
  - 24: Извлекает экземпляр `bind` из таблицы в экземпляре `listitem`.
  - 26-27: Отвязывает привязку.
  - 28: Удаляет значение, соответствующее ключу "bind".

Эта техника не так сложна.
Вы можете использовать её, когда создаёте редактируемое приложение с ячейками.

Если невозможно использовать `g_object_bind_property`, используйте сигнал notify на экземпляре GtkEntryBuffer.
Вы можете использовать сигналы "deleted-text" и "inserted-text" вместо этого.
Обработчик сигналов выше копирует текст в экземпляре GtkEntryBuffer в строку LeData.
Подключите обработчик сигнала notify в `bind2_cb` и отключите его в `unbind2_cb`.

## Динамическое изменение ячейки GtkColumnView

Следующая тема — динамическое изменение ячеек GtkColumnView (или GtkListView).
Пример изменяет цвет кнопок, которые являются дочерними элементами экземпляров GtkListItem, когда позиция текущей строки перемещается.

Редактор строк имеет текущую позицию списка.

- Сначала ни одна строка не является текущей.
- Когда строка добавляется или вставляется, строка становится текущей.
- Когда текущая строка удаляется, ни одна строка не будет текущей.
- Когда кнопка в первом столбце GtkColumnView нажата, строка станет текущей.
- Необходимо установить статус строки (текущая или нет), когда GtkListItem привязан к элементу в списке.
Это потому, что GtkListItem переиспользуется.
GtkListItem мог быть текущей строкой раньше, но не текущей после повторного использования.
Также может произойти обратное.

Кнопка текущей строки окрашена в красный цвет, а иначе в белый.

Текущая строка не имеет отношения к объекту GtkSingleSelection.
GtkSingleSelection выбирает строку на дисплее.
Текущая строка не обязательно находится на дисплее.
Возможно, она находится на строке вне окна (GtkScrolledWindow).
На самом деле, программа не использует GtkSingleSelection.

Экземпляр LeWindow имеет две переменные экземпляра для записи текущей строки.

- `win->position`: Переменная типа int. Это позиция текущей строки. Она начинается с нуля. Если текущей строки не существует, она равна -1.
- `win->current_button`: Переменная указывает на кнопку, расположенную в первом столбце, на текущей строке. Если текущей строки не существует, она равна NULL.

Если текущая строка перемещается, вызываются следующие две функции.
Они обновляют две переменные.

~~~C
 1 static void
 2 update_current_position (LeWindow *win, int new) {
 3   char *s;
 4 
 5   win->position = new;
 6   if (win->position >= 0)
 7     s = g_strdup_printf ("%d", win->position);
 8   else
 9     s = "";
10   gtk_label_set_text (GTK_LABEL (win->position_label), s);
11   if (*s) // s isn't an empty string
12     g_free (s);
13 }
14 
15 static void
16 update_current_button (LeWindow *win, GtkButton *new_button) {
17   const char *non_current[1] = {NULL};
18   const char *current[2] = {"current", NULL};
19 
20   if (win->current_button) {
21     gtk_widget_set_css_classes (GTK_WIDGET (win->current_button), non_current);
22     g_object_unref (win->current_button);
23   }
24   win->current_button = new_button;
25   if (win->current_button) {
26     g_object_ref (win->current_button);
27     gtk_widget_set_css_classes (GTK_WIDGET (win->current_button), current);
28   }
29 }
~~~

Переменная `win->position_label` указывает на экземпляр GtkLabel.
Метка показывает позицию текущей строки.

Текущая кнопка имеет CSS класс "current".
Кнопка окрашена в красный цвет через CSS "button.current {background: red;}".

Порядок вызова этих двух функций важен.
Первая функция, которая обновляет позицию, обычно вызывается первой.
После этого новая строка добавляется или вставляется.
Затем вызывается вторая функция.

Следующие функции вызывают две функции выше.
Будьте осторожны с порядком вызова.

~~~C
 1 void
 2 select_cb (GtkButton *btn, GtkListItem *listitem) {
 3   LeWindow *win = LE_WINDOW (gtk_widget_get_ancestor (GTK_WIDGET (btn), LE_TYPE_WINDOW));
 4 
 5   update_current_position (win, gtk_list_item_get_position (listitem));
 6   update_current_button (win, btn);
 7 }
 8 
 9 static void
10 setup1_cb (GtkListItemFactory *factory, GtkListItem *listitem) {
11   GtkWidget *button = gtk_button_new ();
12   gtk_list_item_set_child (listitem, button);
13   gtk_widget_set_focusable (GTK_WIDGET (button), FALSE);
14   g_signal_connect (button, "clicked", G_CALLBACK (select_cb), listitem);
15 }
16 
17 static void
18 bind1_cb (GtkListItemFactory *factory, GtkListItem *listitem, gpointer user_data) {
19   LeWindow *win = LE_WINDOW (user_data);
20   GtkWidget *button = gtk_list_item_get_child (listitem);
21 
22   if (win->position == gtk_list_item_get_position (listitem))
23     update_current_button (win, GTK_BUTTON (button));
24 }
~~~

- 1-7: `select_cb` — обработчик сигнала "clicked".
Обработчик просто вызывает две функции и обновляет позицию и кнопку.
- 9-15: `setup1_cb` — обработчик сигнала setup на GtkSignalListItemFactory.
Он устанавливает child `listitem` как экземпляр GtkButton.
Сигнал "clicked" на кнопке подключён к обработчику `select_cb`.
Когда listitem уничтожается, child (GtkButton) также уничтожается.
В то же время соединение сигнала и обработчика также уничтожается.
Поэтому вам не нужен обработчик сигнала teardown.
- 17-24: `bind1_cb` — обработчик сигнала bind.
Обычно позиция перемещается до вызова этого обработчика.
Если элемент находится на текущей строке, кнопка обновляется.
Обработчик unbind не нужен.

Когда добавляется строка, текущая позиция обновляется заранее.

~~~C
 1 static void
 2 app_cb (GtkButton *btn, LeWindow *win) {
 3   LeData *data = le_data_new_with_data ("");
 4 
 5   if (win->position >= 0) {
 6     update_current_position (win, win->position + 1);
 7     g_list_store_insert (win->liststore, win->position, data);
 8   } else {
 9     update_current_position (win, g_list_model_get_n_items (G_LIST_MODEL (win->liststore)));
10     g_list_store_append (win->liststore, data);
11   }
12   g_object_unref (data);
13 }
14 
15 static void
16 ins_cb (GtkButton *btn, LeWindow *win) {
17   LeData *data = le_data_new_with_data ("");
18 
19   if (win->position >= 0)
20     g_list_store_insert (win->liststore, win->position, data);
21   else {
22     update_current_position (win, 0);
23     g_list_store_insert (win->liststore, 0, data);
24   }
25   g_object_unref (data);
26 }
~~~

Когда строка удаляется, текущая позиция становится -1, и ни одна кнопка не является текущей.

~~~C
1 static void
2 rm_cb (GtkButton *btn, LeWindow *win) {
3   if (win->position >= 0) {
4     g_list_store_remove (win->liststore, win->position);
5     update_current_position (win, -1);
6     update_current_button (win, NULL);
7   }
8 }
~~~

Цвет кнопок определяется стилем CSS "background".
Следующий узел CSS немного сложен.
Узел CSS `column view` имеет дочерний узел `listview`.
Он охватывает строки в GtkColumnView.
Узел `listview` такой же, как для GtkListView.
Он имеет дочерний узел `row`, который для каждого дочернего виджета.
Следовательно, следующий узел соответствует кнопкам на виджете GtkColumnView.
Кроме того, он применяется к классу "current".

~~~css
columnview listview row button.current {background: red;}
~~~

## Предупреждение от GtkText

Если ваша программа имеет следующие два условия, может быть выдано предупреждающее сообщение.

- Список имеет много элементов, и его нужно прокручивать.
- Экземпляр GtkText является виджетом фокуса.

~~~
GtkText - unexpected blinking selection. Removing
~~~

У меня нет точного представления, почему это происходит.
Но если свойство "focusable" GtkText равно FALSE, предупреждение не появляется.
Поэтому это, вероятно, происходит из-за фокуса и прокрутки.

Вы можете избежать этого, отменив любой виджет фокуса под главным окном.
Когда начинается прокрутка, испускается сигнал "value-changed" на вертикальной настройке прокручиваемого окна.

Следующее извлечено из файла ui и исходного файла C.

~~~xml
... ... ...
<object class="GtkScrolledWindow">
  <property name="hexpand">TRUE</property>
  <property name="vexpand">TRUE</property>
  <property name="vadjustment">
    <object class="GtkAdjustment">
      <signal name="value-changed" handler="adjustment_value_changed_cb" swapped="no" object="LeWindow"/>
    </object>
  </property>
... ... ...  
~~~

~~~C
1 static void
2 adjustment_value_changed_cb (GtkAdjustment *adjustment, gpointer user_data) {
3   GtkWidget *win = GTK_WIDGET (user_data);
4 
5   gtk_window_set_focus (GTK_WINDOW (win), NULL);
6 }
~~~

Up: [README.md](../README.md),  Prev: [Section 32](sec32.md)
