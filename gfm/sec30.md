Up: [README.md](../README.md),  Prev: [Section 29](sec29.md), Next: [Section 31](sec31.md)

# GtkGridView и сигнал activate

GtkGridView похож на GtkListView.
Он отображает GListModel в виде сетки, что похоже на квадратную мозаику.

![Grid](../image/list4.png)

Это часто встречается при использовании файлового браузера, такого как GNOME Files (Nautilus).

В этом разделе давайте создадим очень простой файловый браузер `list4`.
Он просто показывает файлы в текущем каталоге.
И пользователь может выбрать список или сетку, нажав на кнопки на панели инструментов.
Каждый элемент в списке или сетке имеет значок и имя файла.
Кроме того, `list4` предоставляет способ открыть текстовый редактор `tfe` для показа текстового файла.
Пользователь может сделать это, дважды щёлкнув по элементу или нажав клавишу enter, когда элемент выбран.

## GtkDirectoryList

GtkDirectoryList реализует GListModel и содержит информацию о файлах в определённом каталоге.
Элементы списка — это объекты GFileInfo.

В исходных файлах `list4` GtkDirectoryList описан в ui-файле и построен с помощью GtkBuilder.
Экземпляр GtkDirectoryList присваивается свойству "model" экземпляра GtkSingleSelection.
И экземпляр GtkSingleSelection присваивается свойству "model" экземпляра GListView или GGridView.

~~~
GtkListView (model property) => GtkSingleSelection (model property) => GtkDirectoryList
GtkGridView (model property) => GtkSingleSelection (model property) => GtkDirectoryList
~~~

![DirectoryList](../image/directorylist.png)

Следующее — часть ui-файла `list4.ui`.

~~~xml
<object class="GtkListView" id="list">
  <property name="model">
    <object class="GtkSingleSelection" id="singleselection">
      <property name="model">
        <object class="GtkDirectoryList" id="directorylist">
          <property name="attributes">standard::name,standard::icon,standard::content-type</property>
        </object>
      </property>
    </object>
  </property>
</object>
<object class="GtkGridView" id="grid">
  <property name="model">singleselection</property>
</object>
~~~

GtkDirectoryList имеет свойство "attributes".
Это атрибуты GFileInfo, такие как "standard::name", "standard::icon" и "standard::content-type".

- standard::name — это имя файла.
- standard::icon — это значок файла. Это объект GIcon.
- standard::content-type — это тип содержимого.
Тип содержимого совпадает с mime-типом для интернета.
Например, "text/plain" — это текстовый файл, "text/x-csrc" — это исходный код C и так далее.
("text/x-csrc" не зарегистрирован в типах медиа IANA.
Такой подтип "x-" не является стандартным mime-типом.)
Тип содержимого также используется настольными системами.

GtkGridView использует тот же экземпляр GtkSingleSelection (`singleselection`).
Поэтому его свойство model установлено на него.

## Ui-файл окна

Окно построено с помощью следующего ui-файла.
(См. скриншот в начале этого раздела).

~~~xml
 1 <?xml version="1.0" encoding="UTF-8"?>
 2 <interface>
 3   <object class="GtkApplicationWindow" id="win">
 4     <property name="title">file list</property>
 5     <property name="default-width">600</property>
 6     <property name="default-height">400</property>
 7     <child>
 8       <object class="GtkBox" id="boxv">
 9         <property name="orientation">GTK_ORIENTATION_VERTICAL</property>
10         <child>
11           <object class="GtkBox" id="boxh">
12             <property name="orientation">GTK_ORIENTATION_HORIZONTAL</property>
13             <child>
14               <object class="GtkLabel" id="dmy1">
15                 <property name="hexpand">TRUE</property>
16               </object>
17             </child>
18             <child>
19               <object class="GtkButton" id="btnlist">
20                 <property name="name">btnlist</property>
21                 <property name="action-name">win.view</property>
22                 <property name="action-target">&apos;list&apos;</property>
23                 <child>
24                   <object class="GtkImage">
25                     <property name="resource">/com/github/ToshioCP/list4/list.png</property>
26                   </object>
27                 </child>
28               </object>
29             </child>
30             <child>
31               <object class="GtkButton" id="btngrid">
32                 <property name="name">btngrid</property>
33                 <property name="action-name">win.view</property>
34                 <property name="action-target">&apos;grid&apos;</property>
35                 <child>
36                   <object class="GtkImage">
37                     <property name="resource">/com/github/ToshioCP/list4/grid.png</property>
38                   </object>
39                 </child>
40               </object>
41             </child>
42             <child>
43               <object class="GtkLabel" id="dmy2">
44                 <property name="width-chars">10</property>
45               </object>
46             </child>
47           </object>
48         </child>
49         <child>
50           <object class="GtkScrolledWindow" id="scr">
51             <property name="hexpand">TRUE</property>
52             <property name="vexpand">TRUE</property>
53           </object>
54         </child>
55       </object>
56     </child>
57   </object>
58   <object class="GtkListView" id="list">
59     <property name="model">
60       <object class="GtkSingleSelection" id="singleselection">
61         <property name="model">
62           <object class="GtkDirectoryList" id="directory_list">
63             <property name="attributes">standard::name,standard::icon,standard::content-type</property>
64           </object>
65         </property>
66       </object>
67     </property>
68   </object>
69   <object class="GtkGridView" id="grid">
70     <property name="model">singleselection</property>
71   </object>
72 </interface>
73 
~~~

Файл состоит из двух частей.
Первая часть начинается на строке 3 и заканчивается на строке 57.
Эта часть — виджеты от окна верхнего уровня до окна с прокруткой.
Она также включает две кнопки.
Вторая часть начинается на строке 58 и заканчивается на строке 71.
Это часть GtkListView и GtkGridView.

- 13-17, 42-46: Две метки — это фиктивные метки.
Они просто работают как пространство для размещения двух кнопок в соответствующей позиции.
- 18-41: GtkButton `btnlist` и `btngrid`.
Эти две кнопки работают как кнопки выбора для переключения из списка в сетку и наоборот.
Эти две кнопки подключены к действию с состоянием `win.view`.
Это действие имеет параметр.
Такое действие состоит из префикса, имени действия и параметра.
Префикс действия — `win`, что означает, что действие принадлежит окну верхнего уровня.
Префикс даёт область действия.
Имя действия — `view`.
Параметры — `list` или `grid`, которые показывают состояние действия.
Параметр также называется целью (target), потому что это цель, на которую действие меняет своё состояние.
Мы часто пишем подробное действие как "win.view::list" или "win.view::grid".
- 21-22: Свойства "action-name" и "action-target" принадлежат интерфейсу GtkActionable.
GtkButton реализует GtkActionable.
Имя действия — "win.view", а цель — "list".
Как правило, цель — это GVariant, который может быть строкой, целым числом, числом с плавающей точкой и так далее.
Вам нужно использовать текстовый формат GVariant для записи значения GVariant в ui-файлах.
Если тип значения GVariant — строка, то значение в текстовом формате GVariant ограничивается одинарными или двойными кавычками.
Поскольку ui-файл — это текст в формате xml, одинарная кавычка не может быть записана без экранирования.
Её escape-последовательность — \&apos;.
Поэтому цель 'list' записывается как \&apos;list\&apos;.
Поскольку кнопка подключена к действию, обработчик сигнала "clicked" не нужен.
- 23-27: Дочерний виджет кнопки — GtkImage.
GtkImage имеет свойство "resource".
Это GResource, и GtkImage читает данные изображения из ресурса и устанавливает изображение.
Этот ресурс построен из данных изображения png размером 24x24, которое является оригинальным значком.
- 50-53: GtkScrolledWindow.
Его дочерний виджет будет установлен с помощью GtkListView или GtkGridView.

Действие `view` создаётся, подключается к обработчику сигнала "activate" и вставляется в окно (action map) следующим образом.

~~~C
act_view = g_simple_action_new_stateful ("view", g_variant_type_new("s"), g_variant_new_string ("list"));
g_signal_connect (act_view, "activate", G_CALLBACK (view_activated), NULL);
g_action_map_add_action (G_ACTION_MAP (win), G_ACTION (act_view));
~~~

Обработчик сигнала `view_activated` будет объяснён позже.

## Фабрики

Каждое представление (GtkListView и GtkGridView) имеет свою собственную фабрику, потому что его элементы имеют разную структуру виджетов.
Фабрики — это объекты GtkBuilderListItemFactory.
Их ui-файлы следующие.

factory_list.ui

~~~xml
 1 <?xml version="1.0" encoding="UTF-8"?>
 2 <interface>
 3   <template class="GtkListItem">
 4     <property name="child">
 5       <object class="GtkBox">
 6         <property name="orientation">GTK_ORIENTATION_HORIZONTAL</property>
 7         <property name="spacing">20</property>
 8         <child>
 9           <object class="GtkImage">
10             <binding name="gicon">
11               <closure type="GIcon" function="get_icon">
12                 <lookup name="item">GtkListItem</lookup>
13               </closure>
14             </binding>
15           </object>
16         </child>
17         <child>
18           <object class="GtkLabel">
19             <property name="hexpand">TRUE</property>
20             <property name="xalign">0</property>
21             <binding name="label">
22               <closure type="gchararray" function="get_file_name">
23                 <lookup name="item">GtkListItem</lookup>
24               </closure>
25             </binding>
26           </object>
27         </child>
28       </object>
29     </property>
30   </template>
31 </interface>
32 
~~~

factory_grid.ui

~~~xml
 1 <?xml version="1.0" encoding="UTF-8"?>
 2 <interface>
 3   <template class="GtkListItem">
 4     <property name="child">
 5       <object class="GtkBox">
 6         <property name="orientation">GTK_ORIENTATION_VERTICAL</property>
 7         <property name="spacing">20</property>
 8         <child>
 9           <object class="GtkImage">
10             <property name="icon-size">GTK_ICON_SIZE_LARGE</property>
11             <binding name="gicon">
12               <closure type="GIcon" function="get_icon">
13                 <lookup name="item">GtkListItem</lookup>
14               </closure>
15             </binding>
16           </object>
17         </child>
18         <child>
19           <object class="GtkLabel">
20             <property name="hexpand">TRUE</property>
21             <property name="xalign">0.5</property>
22             <binding name="label">
23               <closure type="gchararray" function="get_file_name">
24                 <lookup name="item">GtkListItem</lookup>
25               </closure>
26             </binding>
27           </object>
28         </child>
29       </object>
30     </property>
31   </template>
32 </interface>
33 
~~~

Два файла выше почти одинаковы.
Разница:

- Ориентация блока
- Размер значка
- Положение текста метки

~~~
$ cd list4; diff factory_list.ui factory_grid.ui
6c6
<         <property name="orientation">GTK_ORIENTATION_HORIZONTAL</property>
---
>         <property name="orientation">GTK_ORIENTATION_VERTICAL</property>
9a10
>             <property name="icon-size">GTK_ICON_SIZE_LARGE</property>
20c21
<             <property name="xalign">0</property>
---
>             <property name="xalign">0.5</property>
~~~

Два свойства "gicon" (свойство GtkImage) и "label" (свойство GtkLabel) находятся в ui-файлах выше.
Поскольку GFileInfo не имеет свойств, соответствующих значку или имени файла, фабрика использует тег closure для привязки свойств "gicon" и "label" к информации GFileInfo.
Функция `get_icon` получает GIcon из объекта GFileInfo.
И функция `get_file_name` получает имя файла из объекта GFileInfo.

~~~C
 1 GIcon *
 2 get_icon (GtkListItem *item, GFileInfo *info) {
 3   GIcon *icon;
 4 
 5    /* g_file_info_get_icon can return NULL */
 6   icon = G_IS_FILE_INFO (info) ? g_file_info_get_icon (info) : NULL;
 7   return icon ? g_object_ref (icon) : NULL;
 8 }
 9 
10 char *
11 get_file_name (GtkListItem *item, GFileInfo *info) {
12   return G_IS_FILE_INFO (info) ? g_strdup (g_file_info_get_name (info)) : NULL;
13 }
~~~

Одна важная вещь — это владение возвращаемыми значениями.
Возвращаемое значение принадлежит вызывающей стороне.
Поэтому необходимы `g_obect_ref` или `g_strdup`.

## Обработчик сигнала activate действия кнопки

Обработчик сигнала activate `view_activate` переключает представление.
Он делает две вещи.

- Изменяет дочерний виджет GtkScrolledWindow.
- Изменяет CSS кнопок, чтобы показать текущее состояние.

~~~C
 1 static void
 2 view_activated(GSimpleAction *action, GVariant *parameter) {
 3   const char *view = g_variant_get_string (parameter, NULL);
 4   const char *other;
 5   char *css;
 6 
 7   if (strcmp (view, "list") == 0) {
 8     other = "grid";
 9     gtk_scrolled_window_set_child (scr, GTK_WIDGET (list));
10   }else {
11     other = "list";
12     gtk_scrolled_window_set_child (scr, GTK_WIDGET (grid));
13   }
14   css = g_strdup_printf ("button#btn%s {background: silver;} button#btn%s {background: white;}", view, other);
15   gtk_css_provider_load_from_data (provider, css, -1);
16   g_free (css);
17   g_action_change_state (G_ACTION (action), parameter);
18 }
~~~

Второй параметр этого обработчика — это цель нажатой кнопки.
Его тип — GVariant.

- Если `btnlist` была нажата, то `parameter` — это GVariant строки "list".
- Если `btngrid` была нажата, то `parameter` — это GVariant строки "grid".

Третий параметр `user_data` указывает на NULL и здесь игнорируется.

- 3: `g_variant_get_string` получает строку из переменной GVariant.
- 7-13: Устанавливает дочерний элемент `scr`.
Функция `gtk_scrolled_window_set_child` уменьшает счётчик ссылок старого дочернего элемента на единицу.
И она увеличивает счётчик ссылок нового дочернего элемента на единицу.
- 14-16: Устанавливает CSS для кнопок.
Фон нажатой кнопки будет серебристого цвета, а другая кнопка будет белой.
- 17: Изменяет состояние действия.
 
## Сигнал activate на GtkListView и GtkGridView

Представления (GtkListView и GtkGridView) имеют сигнал "activate".
Он испускается при двойном щелчке по элементу в представлении или нажатии клавиши enter.
Вы можете делать всё, что вам нравится, подключив сигнал "activate" к обработчику.

Пример `list4` запускает текстовый редактор `tfe`, если элемент списка является текстовым файлом.

~~~C
static void
list_activate (GtkListView *list, int position, gpointer user_data) {
  GFileInfo *info = G_FILE_INFO (g_list_model_get_item (G_LIST_MODEL (gtk_list_view_get_model (list)), position));
  launch_tfe_with_file (info);
}

static void
grid_activate (GtkGridView *grid, int position, gpointer user_data) {
  GFileInfo *info = G_FILE_INFO (g_list_model_get_item (G_LIST_MODEL (gtk_grid_view_get_model (grid)), position));
  launch_tfe_with_file (info);
}

... ...
... ...

  g_signal_connect (GTK_LIST_VIEW (list), "activate", G_CALLBACK (list_activate), NULL);
  g_signal_connect (GTK_GRID_VIEW (grid), "activate", G_CALLBACK (grid_activate), NULL);
~~~

Второй параметр каждого обработчика — это позиция элемента (GFileInfo) GListModel.
Поэтому вы можете получить элемент с помощью функции `g_list_model_get_item`.

### Тип содержимого и запуск приложения

Функция `launch_tfe_with_file` получает файл из экземпляра GFileInfo.
Если файл является текстовым файлом, она запускает `tfe` с файлом.

GFileInfo содержит информацию о типе файла.
Тип файла похож на "text/plain", "text/x-csrc" и так далее.
Это называется типом содержимого.
Тип содержимого можно получить с помощью функции `g_file_info_get_content_type`.

~~~C
 1 static void
 2 launch_tfe_with_file (GFileInfo *info) {
 3   GError *err = NULL;
 4   GFile *file;
 5   GList *files = NULL;
 6   const char *content_type;
 7   const char *text_type = "text/";
 8   GAppInfo *appinfo;
 9   int i;
10 
11   if (! info)
12     return;
13   content_type = g_file_info_get_content_type (info);
14 #ifdef debug
15   g_print ("%s\n", content_type);
16 #endif
17   if (! content_type)
18     return;
19   for (i=0;i<5;++i) { /* compare the first 5 characters */
20     if (content_type[i] != text_type[i])
21       return;
22   }
23   appinfo = g_app_info_create_from_commandline ("tfe", "tfe", G_APP_INFO_CREATE_NONE, &err);
24   if (err) {
25     g_printerr ("%s\n", err->message);
26     g_error_free (err);
27     return;
28   }
29   err = NULL;
30   file = g_file_new_for_path (g_file_info_get_name (info));
31   files = g_list_append (files, file);
32   if (! (g_app_info_launch (appinfo, files, NULL, &err))) {
33     g_printerr ("%s\n", err->message);
34     g_error_free (err);
35   }
36   g_list_free_full (files, g_object_unref);
37   g_object_unref (appinfo);
38 }
~~~

- 13: Получает тип содержимого файла из GFileInfo.
- 14-16: Выводит тип содержимого, если "debug" определён.
Это полезно только для того, чтобы узнать тип содержимого файла.
Если вы не хотите этого, удалите или закомментируйте определение `#define debug 1` на строке 6 в исходном файле.
- 17-22: Если нет типа содержимого или тип содержимого не начинается с "text/", функция возвращается.
- 23: Создаёт объект GAppInfo приложения `tfe`.
GAppInfo — это интерфейс, и переменная `appinfo` указывает на экземпляр GDesktopAppInfo.
GAppInfo — это коллекция информации о приложениях.
- 32: Запускает приложение (`tfe`) с аргументом `file`.
`g_app_info_launch` имеет четыре параметра.
Первый параметр — объект GAppInfo.
Второй параметр — список объектов GFile.
В этой функции только один экземпляр GFile передаётся в `tfe`, но вы можете передать больше аргументов.
Третий параметр — GAppLaunchContext, но эта программа передаёт NULL вместо него.
Последний параметр — это указатель на указатель на GError.
- 36: `g_list_free_full` освобождает память, используемую списком и элементами.

Если ваш дистрибутив поддерживает GTK 4, использование `g_app_info_launch_default_for_uri` удобно.
Функция автоматически определяет приложение по умолчанию из файла и запускает его.
Например, если файл текстовый, то она запускает текстовый редактор GNOME с файлом.
Такая функция исходит от рабочего стола.

## Компиляция и выполнение

Исходные файлы расположены в каталоге [src/list4](../src/list4).
Для компиляции и выполнения list4 введите следующее.

~~~
$ cd list4 # or cd src/list4. It depends your current directory.
$ meson setup _build
$ ninja -C _build
$ _build/list4
~~~

Затем появляется список файлов в стиле списка.
Нажмите на кнопку на панели инструментов, чтобы вы могли изменить стиль на сетку или обратно на список.
Дважды щёлкните по элементу "list4.c", затем текстовый редактор `tfe` запустится с аргументом "list4.c".
Ниже приведён скриншот.

![Screenshot](../image/screenshot_list4.png)

## Свойство "gbytes" GtkBuilderListItemFactory

GtkBuilderListItemFactory имеет свойство "gbytes".
Свойство содержит последовательность байтов ui-данных.
Если вы используете это свойство, вы можете поместить содержимое `factory_list.ui` и `factory_grid.ui` в `list4.ui`.
Ниже показана часть нового ui-файла (`list5.ui`).

~~~xml
  <object class="GtkListView" id="list">
    <property name="model">
      <object class="GtkSingleSelection" id="singleselection">
        <property name="model">
          <object class="GtkDirectoryList" id="directory_list">
            <property name="attributes">standard::name,standard::icon,standard::content-type</property>
          </object>
        </property>
      </object>
    </property>
    <property name="factory">
      <object class="GtkBuilderListItemFactory">
        <property name="bytes"><![CDATA[
<?xml version="1.0" encoding="UTF-8"?>
<interface>
  <template class="GtkListItem">
    <property name="child">
      <object class="GtkBox">
        <property name="orientation">GTK_ORIENTATION_HORIZONTAL</property>
        <property name="spacing">20</property>
        <child>
          <object class="GtkImage">
            <binding name="gicon">
              <closure type="GIcon" function="get_icon">
                <lookup name="item">GtkListItem</lookup>
              </closure>
            </binding>
          </object>
        </child>
        <child>
          <object class="GtkLabel">
            <property name="hexpand">TRUE</property>
            <property name="xalign">0</property>
            <binding name="label">
              <closure type="gchararray" function="get_file_name">
                <lookup name="item">GtkListItem</lookup>
              </closure>
            </binding>
          </object>
        </child>
      </object>
    </property>
  </template>
</interface>
        ]]></property>
      </object>
    </property>
  </object>
~~~

Секция CDATA начинается с "<![CDATA[" и заканчивается "]]>".
Содержимое секции CDATA распознаётся как строка.
Любой символ, даже если это маркер ключевого синтаксиса, такой как '<' или '>', распознаётся буквально.
Поэтому текст между "<![CDATA[" и "]]>" вставляется в свойство "bytes" как есть.

Этот метод уменьшает количество ui-файлов.
Но новый ui-файл немного сложен, особенно для начинающих.
Если вы чувствуете некоторую сложность, лучше для вас разделить ui-файл.

Каталог [src/list5](../src/list5) включает ui-файл выше.


Up: [README.md](../README.md),  Prev: [Section 29](sec29.md), Next: [Section 31](sec31.md)
