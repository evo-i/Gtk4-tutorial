# GtkColumnView

## GtkColumnView

GtkColumnView похож на GtkListView, но имеет несколько столбцов.
Каждый столбец является GtkColumnViewColumn.

![Column View](../image/column_view.png){width=11.3cm height=9cm}

- GtkColumnView имеет свойство "model".
Свойство указывает на объект GtkSelectionModel.
- Каждый GtkColumnViewColumn имеет свойство "factory".
Свойство указывает на GtkListItemFactory (GtkSignalListItemFactory или GtkBuilderListItemFactory).
- Фабрика связывает GtkListItem и элементы GtkSelectionModel.
И фабрика создаёт виджеты-потомки GtkColumnView для отображения элемента на экране.
Этот процесс такой же, как и в GtkListView.

Следующая диаграмма показывает, как это работает.

![ColumnView](../image/column.png){width=12cm height=9cm}

Пример в этом разделе — это окно, которое отображает информацию о файлах в текущем каталоге.
Информация — это имя, размер и время последнего изменения файлов.
Таким образом, есть три столбца.

Кроме того, пример использует GtkSortListModel и GtkSorter для сортировки информации.

## column.ui

Файл Ui задаёт виджеты и шаблоны элементов списка.

@@@include
column/column.ui
@@@

- 3-12: GtkApplicationWindow имеет дочерний виджет GtkScrolledWindow.
GtkScrolledWindoww имеет дочерний виджет GtkColumnView.
- 12-18: GtkColumnView имеет свойство "model".
Оно указывает на интерфейс GtkSelectionModel.
Класс GtkNoSelection используется как GtkSelectionModel.
И снова, он имеет свойство "model".
Оно указывает на GtkSortListModel.
Эта модель списка поддерживает сортировку списка.
Это будет объяснено в следующем подразделе.
И он также имеет свойство "model".
Оно указывает на GtkDirectoryList.
Следовательно, цепочка такова: GtkColumnView => GtkNoSelection => GtkSortListModel => GtkDirectoryList.
- 18-20: GtkDirectoryList.
Это список GFileInfo, который хранит информацию о файлах в каталоге.
Он имеет свойство "attributes".
Оно указывает, какие атрибуты хранятся в каждом GFileInfo.
  - "standard::name" — это имя файла.
  - "standard::icon" — это объект GIcon файла
  - "standard::size" — это размер файла.
  - "time::modified" — это дата и время последнего изменения файла.
- 29-79: Первый объект GtkColumnViewColumn.
Есть четыре свойства: "title", "expand", factory" и "sorter".
- 31: устанавливает свойство "title" в "Name".
Это заголовок в заголовке столбца.
- 32: устанавливает свойство "expand" в TRUE, чтобы позволить столбцу расширяться насколько возможно.
(См. изображение выше).
- 33- 69: устанавливает свойство "factory" в GtkBuilderListItemFactory.
Фабрика имеет свойство "bytes", которое содержит строку ui для определения шаблона для расширения класса GtkListItem.
Раздел CDATA (строка 36-66) — это строка ui для помещения в свойство "bytes".
Содержимое то же самое, что и в файле ui `factory_list.ui` в разделе 30.
- 70-77: устанавливает свойство "sorter" в объект GtkStringSorter.
Этот объект предоставляет сортировщик, который сравнивает строки.
Он имеет свойство "expression".
Здесь используется тег closure с функцией типа string `get_file_name`.
Функция будет объяснена позже.
- 80-115: Второй объект GtkColumnViewColumn.
Его свойство sorter установлено в GtkNumericSorter.
- 116-151: Третий объект GtkColumnViewColumn.
Его свойство sorter установлено в GtkNumericSorter.

## GtkSortListModel и GtkSorter

GtkSortListModel — это модель списка, которая сортирует свои элементы согласно экземпляру GtkSorter, назначенному свойству "sorter".
Свойство привязано к свойству "sorter" GtkColumnView в строках 22-24.

~~~xml
<object class="GtkSortListModel" id="sortlist">
... ... ...
  <binding name="sorter">
    <lookup name="sorter">columnview</lookup>
  </binding>
~~~

Поэтому `columnview` определяет способ сортировки модели списка.
Свойство "sorter" GtkColumnView — это свойство только для чтения, и это особый сортировщик.
Он отражает выбор сортировки пользователя.
Если пользователь нажимает заголовок столбца, то сортировщик (свойство "sorter") столбца ссылается на свойство "sorter" GtkColumnView.
Если пользователь нажимает заголовок другого столбца, то свойство "sorter" GtkColumnView ссылается на свойство "sorter" вновь нажатого столбца.

Привязка выше создаёт косвенное соединение между свойством "sorter" GtkSortListModel и свойством "sorter" каждого столбца.

GtkSorter сравнивает два элемента (GObject или его потомок).
GtkSorter имеет несколько дочерних объектов.

- GtkStringSorter сравнивает строки, взятые из элементов.
- GtkNumericSorter сравнивает числа, взятые из элементов.
- GtkCustomSorter использует обратный вызов для сравнения.
- GtkMultiSorter объединяет несколько сортировщиков.

Пример использует GtkStringSorter и GtkNumericSorter.

GtkStringSorter использует GtkExpression для получения строк из элементов (объектов).
GtkExpression хранится в свойстве "expression" GtkStringSorter.
Когда GtkStringSorter сравнивает два элемента, он оценивает выражение, вызывая функцию `gtk_expression_evaluate`.
Он назначает каждый элемент второму аргументу (объект 'this') функции.

В файле ui выше GtkExpression находится в строках 71-76.

~~~xml
<object class="GtkStringSorter">
  <property name="expression">
    <closure type="gchararray" function="get_file_name">
    </closure>
  </property>
</object>
~~~

GtkExpression вызывает функцию `get_file_name` при её оценке.

@@@include
column/column.c get_file_name
@@@

Функции передаётся элемент (GFileInfo) GtkSortListModel в качестве аргумента (объект `this`).
Но нужно быть осторожным, что он может быть NULL во время переработки элемента списка.
Поэтому `G_IS_FILE_INFO (info)` всегда необходим в функциях обратного вызова.
Функция извлекает имя файла из `info`.
Строка принадлежит `info`, поэтому необходимо дублировать.
И она возвращает скопированную строку.

GtkNumericSorter сравнивает числа.
Он используется в строках 106-112 и строках 142-148.
Строки от 106 до 112:

~~~xml
<object class="GtkNumericSorter">
  <property name="expression">
    <closure type="gint64" function="get_file_size">
    </closure>
  </property>
  <property name="sort-order">GTK_SORT_ASCENDING</property>
</object>
~~~

Тег closure указывает функцию обратного вызова `get_file_size`.

@@@include
column/column.c get_file_size
@@@

Она просто возвращает размер `info`.
Тип размера — `goffset`.
Тип `goffset` такой же, как `gint64`.

Строки от 142 до 148:

~~~xml
<object class="GtkNumericSorter" id="sorter_datetime_modified">
  <property name="expression">
    <closure type="gint64" function="get_file_unixtime_modified">
    </closure>
  </property>
  <property name="sort-order">GTK_SORT_ASCENDING</property>
</object>
~~~

Тег closure указывает функцию обратного вызова `get_file_unixtime_modified`.

@@@include
column/column.c get_file_unixtime_modified
@@@

Она получает дату и время модификации (тип GDateTime) из `info`.
Затем она получает unix time из `dt`.
Unix time, иногда называемое unix epoch, — это количество секунд, прошедших с 00:00:00 UTC 1 января 1970 года.
Она возвращает unix time (тип gint64).

## column.c

`column.c` выглядит следующим образом.
Он простой и короткий благодаря `column.ui`.

@@@include
column/column.c
@@@


## Компиляция и выполнение.

Все исходные файлы находятся в каталоге [`src/column`](column).
Измените текущий каталог на этот каталог и введите следующее.

~~~
$ cd src/colomn
$ meson setup _build
$ ninja -C _build
$ _build/column
~~~

Затем появляется окно.

![Column View](../image/column_view.png){width=11.3cm height=9cm}

Если вы нажмёте заголовок столбца, то весь список будет отсортирован по этому столбцу.
Если вы нажмёте заголовок другого столбца, то весь список будет отсортирован по вновь выбранному столбцу.

GtkColumnView очень полезен, и он может управлять очень большим GListModel.
Можно использовать его для списка файлов, списка приложений, интерфейса базы данных и так далее.
