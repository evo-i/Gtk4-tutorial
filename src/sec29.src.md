# GtkListView

GTK 4 добавил новые объекты списков GtkListView, GtkGridView и GtkColumnView.
Новая функциональность описана в [Gtk API Reference -- List Widget Overview](https://docs.gtk.org/gtk4/section-list-widget.html).

GTK 4 имеет другие средства для реализации списков.
Это GtkListBox и GtkTreeView, которые перенесены из GTK 3.
Есть статья в [Gtk Development blog](https://blog.gtk.org/2020/06/07/scalable-lists-in-gtk-4/) о виджетах списков от Matthias Clasen.
Он описал, почему GtkListView разработан для замены GtkTreeView.
GtkTreeView устарел с версии 4.10.

GtkListView, GtkGridView, GtkColumnView и связанные объекты описаны в разделах 29-33.

## Обзор

Список — это последовательная структура данных.
Например, упорядоченная последовательность строк "one", "two", "three", "four" является списком.
Каждый элемент называется элементом (item).
Список похож на массив, но во многих случаях он реализован с помощью указателей, которые указывают на следующие элементы списка.
И у него есть начальная точка.
Таким образом, к каждому элементу можно обратиться по индексу элемента (первый элемент, второй элемент, ..., n-й элемент, ...).
Существует два случая.
Один — индекс начинается с единицы (one-based), другой — с нуля (zero-based).

Gio предоставляет интерфейс GListModel.
Это список с нулевой индексацией, и его элементы являются одним и тем же типом потомков GObject или объектами, реализующими один и тот же интерфейс.
Объект, реализующий GListModel, не является виджетом.
Поэтому список не отображается на экране напрямую.
Существует другой объект GtkListView, который является виджетом для отображения списка.
Элементы в списке должны быть связаны с элементами в GtkListView.
Экземпляр GtkListItemFactory сопоставляет элементы списка с GtkListView.

![List](../image/list.png){width=10cm height=7.5cm}

## GListModel и GtkStringList

Если вы хотите создать список строк с помощью GListModel, например, "one", "two", "three", "four", обратите внимание, что строки не могут быть элементами списка.
Потому что GListModel — это список объектов GObject, а строки не являются объектами GObject.
Слово "GObject" здесь означает "класс GObject или его класс-потомок".
Поэтому вам нужна обёртка, которая является GObject и содержит строку.
GtkStringObject — это объект-обёртка, а GtkStringList, реализующий GListModel, — это список объектов GtkStringObject.

@@@if gfm
~~~C
char *array[] = {"one", "two", "three", "four", NULL};
GtkStringList *stringlist = gtk_string_list_new ((const char * const *) array);
~~~
@@@elif html
~~~{.C}
char *array[] = {"one", "two", "three", "four", NULL};
GtkStringList *stringlist = gtk_string_list_new ((const char * const *) array);
~~~
@@@elif latex
~~~{.C}
char *array[] = {"one", "two", "three", "four", NULL};
GtkStringList *stringlist = gtk_string_list_new ((const char * const *) array);
~~~
@@@else
~~~
char *array[] = {"one", "two", "three", "four", NULL};
GtkStringList *stringlist = gtk_string_list_new ((const char * const *) array);
~~~
@@@end

Функция `gtk_string_list_new` создаёт объект GtkStringList.
Его элементами являются объекты GtkStringObject, которые содержат строки "one", "two", "three" и "four".
Существуют функции для добавления элементов в список или удаления элементов из списка.

- `gtk_string_list_append` добавляет элемент в список
- `gtk_string_list_remove` удаляет элемент из списка
- `gtk_string_list_get_string` получает строку из списка

Для дополнительной информации см. [GTK 4 API Reference -- GtkStringList](https://docs.gtk.org/gtk4/class.StringList.html).

Другие объекты списков будут объяснены позже.

## GtkSelectionModel

GtkSelectionModel — это интерфейс для поддержки выделения.
Благодаря этой модели пользователь может выбирать элементы, нажимая на них.
Он реализован объектами GtkMultiSelection, GtkNoSelection и GtkSingleSelection.
Этих трёх объектов обычно достаточно для создания приложения.
Они создаются с другим GListModel.
Вы также можете создать их отдельно и добавить GListModel позже.

- GtkMultiSelection поддерживает множественный выбор.
- GtkNoSelection поддерживает отсутствие выбора. Это обёртка для GListModel, когда требуется GtkSelectionModel.
- GtkSingleSelection поддерживает одиночный выбор.

## GtkListView

GtkListView — это виджет для отображения элементов GListModel.
GtkListItem используется GtkListView для представления элементов модели списка.
Но сам GtkListItem не является виджетом, поэтому пользователю нужно установить виджет, например GtkLabel, в качестве дочернего элемента GtkListItem для отображения элемента модели списка.
Свойство "item" GtkListItem указывает на объект, принадлежащий модели списка.

![GtkListItem](../image/gtklistitem.png){width=10cm height=7.5cm}

В случае, если количество элементов очень велико, например, более тысячи, GtkListItem переиспользуется и связывается с другим элементом, который только что отображается.
Это переиспользование делает количество объектов GtkListItem довольно малым, менее 200.
Это очень эффективно для сдерживания роста потребления памяти, так что GListModel может содержать множество элементов, например, более миллиона элементов.

## GtkListItemFactory

GtkListItemFactory создаёт или переиспользует GtkListItem и связывает его с элементом модели списка.
Существует два дочерних класса этой фабрики: GtkSignalListItemFactory и GtkBuilderListItemFactory.

### GtkSignalListItemFactory

GtkSignalListItemFactory предоставляет сигналы для пользователей для настройки объекта GtkListItem.
Есть четыре сигнала.

1. "setup" испускается для настройки объекта GtkListItem.
Пользователь устанавливает его дочерний виджет в обработчике.
Например, создаёт виджет GtkLabel и устанавливает свойство child GtkListItem на него.
Эта настройка сохраняется, даже если экземпляр GtkListItem переиспользуется (для связывания с другим элементом GListModel).
2. "bind" испускается для связывания элемента в модели списка с виджетом.
Например, пользователь получает элемент из свойства "item" экземпляра GtkListItem.
Затем получает строку элемента и устанавливает свойство label экземпляра GtkLabel этой строкой.
Этот сигнал испускается, когда GtkListItem только что создан, переиспользуется или произошли какие-то изменения с элементом списка.
3. "unbind" испускается для отвязывания элемента.
Пользователь отменяет всё, что было сделано на шаге 2, в обработчике сигнала.
Если какие-то объекты были созданы на шаге 2, они должны быть уничтожены.
4. "teardown" испускается для отмены всего, что было сделано на шаге 1.
Таким образом, виджет, созданный на шаге 1, должен быть уничтожен.
После этого сигнала элемент списка будет уничтожен.

Следующая программа `list1.c` показывает список строк "one", "two", "three" и "four".
Используется GtkNoSelection, поэтому пользователь не может выбрать ни один элемент.

@@@include
misc/list1.c
@@@

Файл `list1.c` расположен в каталоге [src/misc](misc).
Создайте shell-скрипт ниже и сохраните его в ваш каталог bin, например `$HOME/bin`.

@@@if gfm
~~~Shell
gcc `pkg-config --cflags gtk4` $1.c `pkg-config --libs gtk4`
~~~
@@@elif html
~~~{.bash}
gcc `pkg-config --cflags gtk4` $1.c `pkg-config --libs gtk4`
~~~
@@@elif latex
~~~{.bash}
gcc `pkg-config --cflags gtk4` $1.c `pkg-config --libs gtk4`
~~~
@@@else
~~~
gcc `pkg-config --cflags gtk4` $1.c `pkg-config --libs gtk4`
~~~
@@@end

Перейдите в каталог, содержащий `list1.c`, и введите следующее.

~~~
$ chmod 755 $HOME/bin/comp # or chmod 755 (your bin directory)/comp
$ comp list1
$ ./a.out
~~~

Затем появится следующее окно.

![list1](../image/list1.png){width=6.04cm height=4.40cm}

Программа не так сложна.
Если вы чувствуете некоторую сложность, прочитайте этот раздел снова, особенно подраздел GtkSignalListItemFactory.

### GtkBuilderListItemFactory

GtkBuilderListItemFactory — это ещё одна GtkListItemFactory.
Её поведение определяется с помощью ui-файла.

@@@if gfm
~~~xml
<interface>
  <template class="GtkListItem">
    <property name="child">
      <object class="GtkLabel">
        <binding name="label">
          <lookup name="string" type="GtkStringObject">
            <lookup name="item">GtkListItem</lookup>
          </lookup>
        </binding>
      </object>
    </property>
  </template>
</interface>
~~~
@@@else
~~~{.xml}
<interface>
  <template class="GtkListItem">
    <property name="child">
      <object class="GtkLabel">
        <binding name="label">
          <lookup name="string" type="GtkStringObject">
            <lookup name="item">GtkListItem</lookup>
          </lookup>
        </binding>
      </object>
    </property>
  </template>
</interface>
~~~
@@@end

Тег template используется для определения GtkListItem.
И его свойство child — это объект GtkLabel.
Фабрика видит этот шаблон и создаёт GtkLabel и устанавливает свойство child GtkListItem.
Это то же самое, что делал обработчик setup GtkSignalListItemFactory.

Затем свяжите свойство label GtkLabel со свойством string объекта GtkStringObject.
Строковый объект ссылается на свойство item GtkListItem.
Таким образом, тег lookup выглядит так:

~~~
label <- string <- GtkStringObject <- item <- GtkListItem
~~~

The last lookup tag has a content `GtkListItem`.
Usually, C type like `GtkListItem` doesn't appear in the content of tags.
This is a special case.
There is an explanation in the [GTK Development Blog](https://blog.gtk.org/2020/09/05/a-primer-on-gtklistview/) by Matthias Clasen.

> Remember that the classname (GtkListItem) in a ui template is used as the “this” pointer referring to the object that is being instantiated.

Therefore, GtkListItem instance is used as the `this` object of the lookup tag when it is evaluated.
`this` object will be explained in [section 31](sec31).

The C source code is as follows.
Its name is `list2.c` and located under [src/misc](misc) directory.

@@@include
misc/list2.c
@@@

Для GtkBulderListItemFactory не требуется обработчик сигналов.
Используется GtkSingleSelection, поэтому пользователь может выбрать один элемент за раз.

Поскольку это небольшая программа, ui-данные задаются как строка.

## GtkDirectoryList

GtkDirectoryList — это модель списка, содержащая объекты GFileInfo, которые являются информацией о файлах в определённом каталоге.
Она использует `g_file_enumerate_children_async()` для получения объектов GFileInfo.
Модель списка создаётся функцией `gtk_directory_list_new`.

@@@if gfm
~~~C
GtkDirectoryList *gtk_directory_list_new (const char *attributes, GFile *file);
~~~
@@@else
~~~{.C}
GtkDirectoryList *gtk_directory_list_new (const char *attributes, GFile *file);
~~~
@@@end

`attributes` — это список атрибутов файла, разделённый запятыми.
Атрибуты файла — это пары ключ-значение.
Ключ состоит из пространства имён и имени.
Например, ключ "standard::name" — это имя файла.
"standard" означает общую информацию о файле.
"name" означает имя файла.
В следующей таблице показаны некоторые примеры.

|ключ            |значение                                                                        |
|:---------------|:-------------------------------------------------------------------------------|
|standard::type  |тип файла. например, обычный файл, каталог, символическая ссылка и т.д.         |
|standard::name  |имя файла                                                                       |
|standard::size  |размер файла в байтах                                                          |
|access::can-read|право на чтение, если пользователь может читать файл                            |
|time::modified  |время последнего изменения файла в секундах с начала эпохи UNIX                 |

Текущий каталог — это ".".
Следующая программа создаёт GtkDirectoryList `dl`, и его содержимое — это объекты GFileInfo в текущем каталоге.

@@@if gfm
~~~C
GFile *file = g_file_new_for_path (".");
GtkDirectoryList *dl = gtk_directory_list_new ("standard::name", file);
g_object_unref (file);
~~~
@@@else
~~~{.C}
GFile *file = g_file_new_for_path (".");
GtkDirectoryList *dl = gtk_directory_list_new ("standard::name", file);
g_object_unref (file);
~~~
@@@end

Не так сложно создать программу для вывода списка файлов, изменив `list2.c` из предыдущего подраздела.
Одна проблема в том, что GInfoFile не имеет свойств.
Тег lookup ищет свойство, поэтому он бесполезен для поиска имени файла из объекта GFileInfo.
Вместо этого в данном случае подходит тег closure.
Тег closure указывает функцию и тип возвращаемого значения функции.

@@@if gfm
~~~C
char *
get_file_name (GtkListItem *item, GFileInfo *info) {
  return G_IS_FILE_INFO (info) ? g_strdup (g_file_info_get_name (info)) : NULL;
}
... ...
... ...

"<interface>"
  "<template class=\"GtkListItem\">"
    "<property name=\"child\">"
      "<object class=\"GtkLabel\">"
        "<binding name=\"label\">"
          "<closure type=\"gchararray\" function=\"get_file_name\">"
            "<lookup name=\"item\">GtkListItem</lookup>"
          "</closure>"
        "</binding>"
      "</object>"
    "</property>"
  "</template>"
"</interface>"
~~~
@@@else
~~~{.C}
char *
get_file_name (GtkListItem *item, GFileInfo *info) {
  return G_IS_FILE_INFO (info) ? g_strdup (g_file_info_get_name (info)) : NULL;
}
... ...
... ...

"<interface>"
  "<template class=\"GtkListItem\">"
    "<property name=\"child\">"
      "<object class=\"GtkLabel\">"
        "<binding name=\"label\">"
          "<closure type=\"gchararray\" function=\"get_file_name\">"
            "<lookup name=\"item\">GtkListItem</lookup>"
          "</closure>"
        "</binding>"
      "</object>"
    "</property>"
  "</template>"
"</interface>"
~~~
@@@end

- Строка "gchararray" — это имя типа.
Тип "gchar" — это имя типа, и он совпадает с типом C "char".
Поэтому "gchararray" — это "массив типа char", что совпадает с типом строки.
Он используется для получения типа объекта GValue.
GValue — это обобщённое значение, и оно может содержать различные типы значений.
Например, имя типа может быть gboolean, gchar (char), gint (int), gfloat (float), gdouble (double), gchararray (char *) и так далее.
Эти имена типов — это имена фундаментальных типов, зарегистрированных в системе типов.
См. [GObject tutorial](https://github.com/ToshioCP/Gobject-tutorial/blob/main/gfm/sec5.md#gvalue).
- Тег closure имеет атрибут type и атрибут function.
Атрибут function указывает имя функции, а атрибут type указывает тип возвращаемого значения функции.
Содержимое тега closure (между \<closure...\> и\</closure\>) — это параметры функции.
`<lookup name="item">GtkListItem</lookup>` даёт значение свойства item GtkListItem.
Это будет второй аргумент функции.
Первый параметр всегда является экземпляром GListItem, который является объектом 'this'.
Объект 'this' объясняется в разделе 31.
- Функция `gtk_file_name` является функцией обратного вызова для тега closure.
Сначала она проверяет параметр `info`.
Потому что он может быть NULL, когда GListItem `item` не связан.
Если это GFileInfo, она возвращает скопированное имя файла.
Потому что возвращаемое значение (имя файла) `g_file_info_get_name` принадлежит объекту GFileInfo.
Поэтому строку нужно дублировать, чтобы передать владение вызывающей стороне.
Тег binding связывает свойство "label" GtkLabel с тегом closure.

Вся программа (`list3.c`) следующая.
Программа расположена в каталоге [src/misc](misc).

@@@include
misc/list3.c
@@@

Ui-данные (xml-данные выше) используются для построения шаблона GListItem во время выполнения.
GtkBuilder обращается к таблице символов для поиска функции `get_file_name`.

Как правило, таблица символов используется компоновщиком для связывания объектов в исполняемый файл.
Она включает имена функций и их местоположение.
Компоновщик обычно не помещает таблицу символов в созданный исполняемый файл.
Но если задана опция `--export-dynamic`, компоновщик добавляет таблицу символов в исполняемый файл.

Для этого компилятору C передаётся опция `-Wl,--export-dynamic`.

- `-Wl` — это опция компилятора C, которая передаёт следующую опцию компоновщику.
- `--export-dynamic` — это опция компоновщика.
Следующее цитируется из документации компоновщика.
"При создании динамически связанного исполняемого файла добавить все символы в динамическую таблицу символов.
Динамическая таблица символов — это набор символов, которые видны из динамических объектов во время выполнения."

Скомпилируйте и выполните её.

~~~
$ gcc -Wl,--export-dynamic `pkg-config --cflags gtk4` list3.c `pkg-config --libs gtk4`
~~~

Вы также можете создать shell-скрипт для компиляции `list3.c`

@@@if gfm
~~~bash
gcc -Wl,--export-dynamic `pkg-config --cflags gtk4` $1.c `pkg-config --libs gtk4`
~~~
@@@else
~~~{.bash}
gcc -Wl,--export-dynamic `pkg-config --cflags gtk4` $1.c `pkg-config --libs gtk4`
~~~
@@@end

Сохраните эту одну строку в файл `comp`.
Затем скопируйте его в `$HOME/bin` и дайте ему права на выполнение.

~~~
$ cp comp $HOME/bin/comp
$ chmod +x $HOME/bin/comp
~~~

Вы можете скомпилировать `list3.c` и выполнить его так:

~~~
$ comp list3
$ ./a.out
~~~

![screenshot list3](../image/list3.png){width=10cm height=7.3cm}
