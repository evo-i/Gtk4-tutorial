# GtkBuilder и UI-файл

## Кнопки New, Open, Save и Close

В предыдущем разделе мы создали очень простой редактор.
Он читает файлы в начале и записывает их в конце программы.
Он работает, но не очень хорошо.
Было бы лучше, если бы у нас были кнопки "New", "Open", "Save" и "Close".
Этот раздел описывает, как поместить эти кнопки в окно.

![Screenshot of the file editor](../image/screenshot_tfe2.png){width=9.3cm height=6.825cm}

Снимок экрана выше показывает макет.
Функция `app_open` в исходном коде `tfe2.c` выглядит следующим образом.

@@@include
tfe/tfe2.c app_open
@@@

Функция `app_open` строит виджеты в главном окне приложения.

- 26-28: создаёт экземпляр GtkApplicationWindow и устанавливает заголовок и размер по умолчанию.
- 30-31: создаёт экземпляр GtkBox `boxv`.
Это вертикальный контейнер и дочерний элемент GtkApplicationWindow.
У него два дочерних элемента.
Первый дочерний элемент — это горизонтальный контейнер.
Второй дочерний элемент — это GtkNotebook.
- 33-34: создаёт экземпляр GtkBox `boxh` и добавляет его к `boxv` как первый дочерний элемент.
- 36-41: создаёт три фиктивные метки.
Метки `dmy1` и `dmy3` имеют ширину в десять символов.
Другая метка `dmy2` имеет свойство hexpand, установленное в TRUE.
Это заставляет метку расширяться по горизонтали настолько, насколько это возможно.
- 42-45: создаёт четыре кнопки.
- 47-53: добавляет эти GtkLabel и GtkButton к `boxh`.
- 55-58: создаёт экземпляр GtkNotebook и устанавливает свойства hexpand и vexpand в TRUE.
Это заставляет его расширяться по горизонтали и вертикали, чтобы быть настолько большим, насколько это возможно.
Он добавляется к `boxv` как второй дочерний элемент.

Количество строк построения виджетов составляет 33(=58-26+1).
Нам также понадобилось много переменных (`boxv`, `boxh`, `dmy1`, ...), и большинство из них используются только для построения виджетов.
Есть ли какое-нибудь хорошее решение для сокращения этой работы?

Gtk предоставляет GtkBuilder.
Он читает данные пользовательского интерфейса (UI) и строит окно.
Это сокращает эту обременительную работу.

## UI-файл

Посмотрите на UI-файл `tfe3.ui`, который определяет структуру виджетов.

@@@include
tfe/tfe3.ui
@@@

Это XML-файл.
Теги начинаются с `<` и заканчиваются на `>`.
Существует два типа тегов: открывающий тег и закрывающий тег.
Например, `<interface>` — это открывающий тег, а `</interface>` — это закрывающий тег.
UI-файл начинается и заканчивается тегами interface.
Некоторые теги, например, теги object, могут иметь атрибуты class и id в своём открывающем теге.

- 1: объявление XML.
Оно указывает, что версия XML — 1.0, а кодировка — UTF-8.
- 3-6: тег object с классом `GtkApplicationWindow` и идентификатором `win`.
Это окно верхнего уровня.
Он определяет три свойства:
свойство `title` — "file editor", свойство `default-width` — 600, а свойство `default-height` — 400.
- 7: тег child означает дочерний виджет.
Например, строка 7 говорит нам, что объект GtkBox является дочерним виджетом `win`.

Сравните этот ui-файл и строки 26-58 в функции `app_open` файла `tfe2.c`.
Оба строят одно и то же окно с его дочерними виджетами.

Вы можете проверить ui-файл с помощью `gtk4-builder-tool`.

- `gtk4-builder-tool validate <имя ui-файла>` проверяет ui-файл.
Если ui-файл содержит какую-либо синтаксическую ошибку, `gtk4-builder-tool` выводит ошибку.
- `gtk4-builder-tool simplify <имя ui-файла>` упрощает ui-файл и выводит результат.
Если задана опция `--replace`, он заменяет ui-файл упрощённым.
Если ui-файл указывает значение свойства по умолчанию, это свойство будет удалено.
Например, ориентация по умолчанию — горизонтальная, поэтому упрощение удаляет строку 12.
Некоторые значения также упрощаются.
Например, "TRUE" и "FALSE" становятся "1" и "0" соответственно.
Однако "TRUE" и "FALSE" лучше для поддержки.

Хорошая идея — проверить ваш ui-файл перед компиляцией.

## GtkBuilder

GtkBuilder строит виджеты на основе ui-файла.

~~~C
GtkBuilder *build;

build = gtk_builder_new_from_file ("tfe3.ui");
win = GTK_WIDGET (gtk_builder_get_object (build, "win"));
gtk_window_set_application (GTK_WINDOW (win), GTK_APPLICATION (app));
nb = GTK_WIDGET (gtk_builder_get_object (build, "nb"));
g_object_unref(build);
~~~

Функция `gtk_builder_new_from_file` читает файл `tfe3.ui`.
Затем она строит виджеты и создаёт объект GtkBuilder.
Все виджеты соединяются на основе отношения родитель-дети, описанного в ui-файле.
Мы можем извлечь объекты из объекта builder с помощью функции `gtk_builder_get_object`.
Окно верхнего уровня, которое имеет идентификатор "win" в ui-файле, извлекается и присваивается переменной `win`.
Свойство application окна устанавливается в `app` с помощью функции `gtk_window_set_application`.
GtkNotebook, который имеет идентификатор "nb" в ui-файле, также извлекается и присваивается переменной `nb`.
После того, как окно и приложение соединены, нам больше не нужен экземпляр GtkBuilder.
Он освобождается с помощью функции `g_object_unref`.

UI-файл сокращает строки в исходном файле C.

@@@shell
cd tfe; diff tfe2.c tfe3.c
@@@

`61,104c62,66` означает, что 44 (=104-61+1) строки изменены на 5 (=66-62+1) строк.
Следовательно, 39 строк сокращены.
Использование ui-файла не только сокращает исходные файлы C, но и делает структуру виджетов более ясной.

Теперь я покажу вам функцию `app_open` в файле C `tfe3.c`.

@@@include
tfe/tfe3.c app_open
@@@

Полный исходный код `tfe3.c` хранится в директории [src/tfe](tfe).

### Использование ui-строки

GtkBuilder может строить виджеты с помощью строки.
Используйте `gtk_builder_new_from_string` вместо `gtk_builder_new_from_file`.

~~~C
char *uistring;

uistring =
"<interface>"
  "<object class=\"GtkApplicationWindow\" id=\"win\">"
    "<property name=\"title\">file editor</property>"
    "<property name=\"default-width\">600</property>"
    "<property name=\"default-height\">400</property>"
    "<child>"
      "<object class=\"GtkBox\">"
        "<property name=\"orientation\">GTK_ORIENTATION_VERTICAL</property>"
... ... ...
... ... ...
"</interface>";

build = gtk_builder_new_from_string (uistring, -1);
~~~

Этот метод имеет преимущество и недостаток.
Преимущество в том, что ui-строка записана в исходном коде.
Таким образом, ui-файл не нужен во время выполнения.
Недостаток в том, что написание строки C немного обременительно,
поскольку xml требует кавычек, а специальные символы требуют экранирования.
Если вы хотите использовать этот метод, вам следует написать скрипт, который преобразует ui-файлы в C-строки.

- Замените обратные слэши двумя обратными слэшами.
- Добавьте обратный слэш перед каждой двойной кавычкой.
- Добавьте двойные кавычки слева и справа от строки в каждой строке.

Или, если у вас установлен `jq`, вы можете использовать `jq -R < tfe3.ui` для выполнения кавычек и экранирования.

### Gresource

Gresource похож на строку, за исключением того, что Gresource — это сжатые двоичные данные, а не текстовые данные.
Программа `glib-compile-resources` компилирует ui-файлы в Gresources.
Она может компилировать не только текстовые файлы, но и двоичные файлы, такие как изображения, звуки и так далее.
После компиляции она объединяет их в один объект Gresource.

XML-файл необходим для компилятора ресурсов `glib-compile-resources`.
Он описывает файлы ресурсов.

@@@include
tfe/tfe3.gresource.xml
@@@

- 2: тег `gresources` может включать несколько gresources (теги gresource).
Однако этот xml имеет только один gresource.
- 3: gresource имеет префикс `/com/github/ToshioCP/tfe3`.
- 4: имя gresource — `tfe3.ui`.
На ресурс будет указывать GtkBuilder с помощью `/com/github/ToshioCP/tfe3/tfe3.ui`.
Шаблон — "префикс" + "имя".
Если вы хотите добавить больше файлов, вставьте их между строками 4 и 5.

Сохраните этот xml-текст в `tfe3.gresource.xml`.
Компилятор gresource `glib-compile-resources` показывает своё использование с аргументом `--help`.

```
$ glib-compile-resources --help
Usage:
  glib-compile-resources [OPTION..] FILE

Compile a resource specification into a resource file.
Resource specification files have the extension .gresource.xml,
and the resource file have the extension called .gresource.

Help Options:
  -h, --help                   Show help options

Application Options:
  --version                    Show program version and exit
  --target=FILE                Name of the output file
  --sourcedir=DIRECTORY        The directories to load files referenced in FILE from (default: current directory)
  --generate                   Generate output in the format selected for by the target filename extension
  --generate-header            Generate source header
  --generate-source            Generate source code used to link in the resource file into your code
  --generate-dependencies      Generate dependency list
  --dependency-file=FILE       Name of the dependency file to generate
  --generate-phony-targets     Include phony targets in the generated dependency file
  --manual-register            Don't automatically create and register resource
  --internal                   Don't export functions; declare them G_GNUC_INTERNAL
  --external-data              Don't embed resource data in the C file; assume it's linked externally instead
  --c-name                     C identifier name used for the generated source code
  -C, --compiler               The target C compiler (default: the CC environment variable)
  ```

Теперь запустите компилятор.

~~~
$ glib-compile-resources tfe3.gresource.xml --target=resources.c --generate-source
~~~

Затем генерируется исходный файл C `resources.c`.
Измените `tfe3.c` и сохраните его как `tfe3_r.c`.

~~~C
#include "resources.c"
... ... ...
... ... ...
build = gtk_builder_new_from_resource ("/com/github/ToshioCP/tfe3/tfe3.ui");
... ... ...
... ... ...
~~~

Функция `gtk_builder_new_from_resource` строит виджеты из ресурса.

Затем скомпилируйте и запустите.

~~~
$ comp tfe3_r
$ ./a.out tfe2.c
~~~

Появляется окно, и оно такое же, как снимок экрана в начале этой страницы.

Вообще говоря, ресурсы лучше всего подходят для программ на C.
Если вы используете другие языки, такие как Ruby, строки лучше, чем ресурсы.
