Up: [README.md](../README.md),  Prev: [Section 2](sec2.md), Next: [Section 4](sec4.md)

# GtkApplication и GtkApplicationWindow

## GtkApplication

### GtkApplication и g\_application\_run

Люди пишут программный код для создания приложения.
Что такое приложения?
Приложения — это программное обеспечение, которое работает с использованием библиотек, таких как ОС, фреймворки и так далее.
В программировании GTK 4, GtkApplication — это программа (или исполняемый файл), которая работает с использованием библиотек Gtk.

Базовый способ написания GtkApplication выглядит следующим образом.

- Создать экземпляр GtkApplication.
- Запустить приложение.

Вот и всё.
Очень просто.
Ниже приведен код на C, представляющий вышеописанный сценарий.

~~~C
 1 #include <gtk/gtk.h>
 2 
 3 int
 4 main (int argc, char **argv) {
 5   GtkApplication *app;
 6   int stat;
 7 
 8   app = gtk_application_new ("com.github.ToshioCP.pr1", G_APPLICATION_DEFAULT_FLAGS);
 9   stat = g_application_run (G_APPLICATION (app), argc, argv);
10   g_object_unref (app);
11   return stat;
12 }
13 
~~~

Первая строка говорит, что эта программа включает заголовочные файлы библиотек Gtk.
Функция `main` является стартовой функцией в языке C.
Переменная `app` определена как указатель на экземпляр GtkApplication.
Функция `gtk_application_new` создает экземпляр GtkApplication и возвращает указатель на экземпляр.
Экземпляр GtkApplication — это данные структуры C, в которой хранится информация о приложении.
Аргументы будут объяснены позже.
Функция `g_application_run` запускает приложение, определенное экземпляром.
(Мы часто говорим, что функция запускает `app`.
На самом деле, `app` — это не приложение, а указатель на экземпляр приложения.
Однако это просто и коротко, и, вероятно, не вызывает путаницы.)

Здесь я использовал слово `экземпляр` (instance).
Экземпляр, класс и объект — это терминология объектно-ориентированного программирования.
Я использую эти слова одинаково.
Но я часто буду использовать "объект" вместо "экземпляр" в этом учебнике.
Это означает, что "объект" и "экземпляр" — это одно и то же.
Объект — это несколько неоднозначное слово.
В широком смысле объект имеет более широкое значение, чем экземпляр.
Поэтому читатели должны быть внимательны к контексту, чтобы понять значение "объекта".
Во многих случаях объект и экземпляр взаимозаменяемы.

Функция `gtk_application_new` имеет два параметра.

- ID приложения (com.github.ToshioCP.pr1).
Он используется для различения приложений системой.
Формат — обратный DNS.
Дополнительную информацию см. в [GNOME Developer Documentation -- Application ID](https://developer.gnome.org/documentation/tutorials/application-id.html).

- Флаг приложения (G\_APPLICATION\_DEFAULT\_FLAGS).
Если приложение запускается без аргументов, флагом является G\_APPLICATION\_DEFAULT\_FLAGS.
В противном случае вам нужны другие флаги.
Дополнительную информацию см. в [GIO API reference](https://docs.gtk.org/gio/flags.ApplicationFlags.html).

Примечание: Если ваша версия GLib-2.0 старше 2.74, используйте `G_APPLICATION_FLAGS_NONE` вместо `G_APPLICATION_DEFAULT_FLAGS`.
Это старый флаг, замененный `G_APPLICATION_DEFAULT_FLAGS` и объявленный устаревшим с версии 2.74.

Чтобы скомпилировать это, выполните следующую команду.
Строка `pr1.c` — это имя файла исходного кода C выше.

Если вы загрузили этот репозиторий, вам не нужно создавать файл.
Тот же файл находится по адресу `src/misc/pr1.c` в вашем локальном репозитории.
Все примеры кода также находятся в директории `src`.

~~~
$ gcc `pkg-config --cflags gtk4` pr1.c `pkg-config --libs gtk4`
~~~

Компилятор C gcc генерирует исполняемый файл `a.out`.
Давайте запустим его.

~~~
$ ./a.out

(a.out:5084): GLib-GIO-WARNING **: 09:52:04.236: Your application does not implement
g_application_activate() and has no handlers connected to the 'activate' signal.
It should do one of these.
$
~~~

О, он просто выдает сообщение об ошибке.
Это сообщение об ошибке показывает, что объект GtkApplication выполнился, без сомнения.
Теперь давайте подумаем, что означает это сообщение.

### Сигналы

Сообщение говорит нам, что:

1. Приложение не реализует `g_application_activate()`,
2. У него нет обработчиков, подключенных к сигналу "activate", и
3. Вам нужно решить хотя бы одну из этих проблем.

Эти две проблемы связаны с сигналами.
Поэтому сначала я объясню это.

Сигнал испускается, когда что-то происходит.
Например, создается окно, уничтожается окно и так далее.
Сигнал "activate" испускается, когда приложение активируется.
(Активировано немного отличается от запущено, но пока можно считать их почти одинаковыми.)
Если сигнал подключен к функции, которая называется обработчиком сигнала или просто обработчиком,
то функция вызывается, когда испускается сигнал.

Поток выглядит так:

1. Что-то происходит.
2. Если это связано с определенным сигналом, то сигнал испускается.
3. Если сигнал был заранее подключен к обработчику, то обработчик вызывается.

Сигналы определены в объектах.
Например, сигнал "activate" принадлежит объекту GApplication,
который является родительским объектом объекта GtkApplication.

Объект GApplication является дочерним объектом объекта GObject.
GObject — это верхний объект в иерархии всех объектов.

~~~
GObject -- GApplication -- GtkApplication
<---родитель                      --->потомок
~~~

Дочерний объект наследует сигналы, функции, свойства и так далее от своего родительского объекта.
Таким образом, GtkApplication также имеет сигнал "activate".

Теперь мы можем решить проблему в `pr1.c`.
Нам нужно подключить сигнал "activate" к обработчику.
Мы используем функцию `g_signal_connect`, которая подключает сигнал к обработчику.

~~~C
 1 #include <gtk/gtk.h>
 2 
 3 static void
 4 app_activate (GApplication *app, gpointer *user_data) {
 5   g_print ("GtkApplication is activated.\n");
 6 }
 7 
 8 int
 9 main (int argc, char **argv) {
10   GtkApplication *app;
11   int stat;
12 
13   app = gtk_application_new ("com.github.ToshioCP.pr2", G_APPLICATION_DEFAULT_FLAGS);
14   g_signal_connect (app, "activate", G_CALLBACK (app_activate), NULL);
15   stat = g_application_run (G_APPLICATION (app), argc, argv);
16   g_object_unref (app);
17   return stat;
18 }
19 
~~~

Сначала мы определяем обработчик `app_activate`, который просто отображает сообщение.
Функция `g_print` определена в GLib и похожа на printf из стандартной библиотеки C.
В функции `main` мы добавляем `g_signal_connect` перед `g_application_run`.
Функция `g_signal_connect` имеет четыре аргумента.

1. Экземпляр, которому принадлежит сигнал.
2. Имя сигнала.
3. Функция-обработчик (также называемая callback), которая должна быть приведена к типу с помощью `G_CALLBACK`.
4. Данные для передачи обработчику. Если данные не нужны, передается NULL.

Это описано в [GObject API Reference](https://docs.gtk.org/gobject/func.signal_connect.html).
Точнее, `g_signal_connect` — это макрос (а не функция C).

~~~c
#define g_signal_connect (
  instance,
  detailed_signal,
  c_handler,
  data
)
~~~

Вы можете найти описание каждого сигнала в справочном руководстве по API.
Например, сигнал "activate" находится в [разделе GApplication](https://docs.gtk.org/gio/signal.Application.activate.html) в справочнике GIO API.

~~~c
void
activate (
  GApplication* self,
  gpointer user_data
)
~~~

Это объявление обработчика сигнала "activate".
Вы можете использовать любое имя вместо "activate" в объявлении выше.
Параметры:

- self — это экземпляр, которому принадлежит сигнал.
- user\_data — это данные, определенные в четвертом аргументе функции `g_signal_connect`.
Если это NULL, то вы можете игнорировать и опустить второй параметр.

Справочное руководство по API очень важно.
Вы должны смотреть и понимать его.

Давайте скомпилируем исходный файл выше (`pr2.c`) и запустим его.

~~~
$ gcc `pkg-config --cflags gtk4` pr2.c `pkg-config --libs gtk4`
$ ./a.out
GtkApplication is activated.
$
~~~

Хорошо, отлично сделано.
Однако вы могли заметить, что набирать такую длинную строку для компиляции утомительно.
Хорошая идея — использовать shell-скрипт для решения этой проблемы.
Создайте текстовый файл, содержащий следующую строку.

~~~
gcc `pkg-config --cflags gtk4` $1.c `pkg-config --libs gtk4`
~~~

Затем сохраните его в директории $HOME/bin, которая обычно /home/(имя_пользователя)/bin.
(Если ваше имя пользователя James, то директория — /home/james/bin).
И включите бит выполнения файла.
Если имя файла `comp`, сделайте так:

~~~
$ chmod 755 $HOME/bin/comp
$ ls -log $HOME/bin
    ...  ...  ...
-rwxr-xr-x 1   62 May 23 08:21 comp
    ...  ...  ...
~~~

Если это первый раз, когда вы создаете директорию $HOME/bin и сохраняете в ней файл, вам нужно выйти и снова войти в систему.

~~~
$ comp pr2
$ ./a.out
GtkApplication is activated.
$
~~~

## GtkWindow и GtkApplicationWindow

### GtkWindow

В предыдущем подразделе было выведено сообщение "GtkApplication is activated.".
Это было хорошо с точки зрения теста GtkApplication.
Однако этого недостаточно, потому что Gtk — это фреймворк для графического пользовательского интерфейса (GUI).
Теперь давайте пойдем дальше и добавим окно в эту программу.
Что нам нужно сделать:

1. Создать GtkWindow.
2. Подключить его к GtkApplication.
3. Показать окно.

Теперь перепишем функцию `app_activate`.

#### Создание GtkWindow

~~~C
1 static void
2 app_activate (GApplication *app, gpointer user_data) {
3   GtkWidget *win;
4 
5   win = gtk_window_new ();
6   gtk_window_set_application (GTK_WINDOW (win), GTK_APPLICATION (app));
7   gtk_window_present (GTK_WINDOW (win));
8 }
~~~

Виджет — это абстрактная концепция, которая включает все интерфейсы GUI, такие как окна, метки, кнопки, многострочный текст, блоки и так далее.
И GtkWidget — это базовый объект, от которого происходят все объекты GUI.

~~~
родитель <-----> потомок
GtkWidget -- GtkWindow
~~~

GtkWindow включает GtkWidget в начале своего объекта.

![GtkWindow и GtkWidget](../image/window_widget.png)

Функция `gtk_window_new` определена следующим образом.

~~~C
GtkWidget *
gtk_window_new (void);
~~~

По этому определению она возвращает указатель на GtkWidget, а не GtkWindow.
Она фактически создает новый экземпляр GtkWindow (а не GtkWidget), но возвращает указатель на GtkWidget.
Однако указатель указывает на GtkWidget и в то же время также указывает на GtkWindow, который содержит GtkWidget в себе.

Если вы хотите использовать `win` как указатель на экземпляр типа GtkWindow, вам нужно привести его к типу.

~~~C
(GtkWindow *) win
~~~

Это работает, но обычно не используется.
Вместо этого используется макрос `GTK_WINDOW`.

~~~C
GTK_WINDOW (win)
~~~

Макрос рекомендуется, потому что он не только приводит указатель к типу, но и проверяет тип.

#### Подключение к GtkApplication.

Функция `gtk_window_set_application` используется для подключения GtkWindow к GtkApplication.

~~~C
gtk_window_set_application (GTK_WINDOW (win), GTK_APPLICATION (app));
~~~

Вам нужно привести `win` к GtkWindow и `app` к GtkApplication с помощью макросов `GTK_WINDOW` и `GTK_APPLICATION`.

GtkApplication продолжает работать до тех пор, пока связанное окно не будет уничтожено.
Если вы не подключили GtkWindow и GtkApplication, GtkApplication уничтожает себя немедленно.
Поскольку никакое окно не подключено к GtkApplication, GtkApplication не нужно ничего ждать.
Когда оно уничтожает себя, GtkWindow также уничтожается.

#### Показ окна.

Функция `gtk_window_present` представляет окно пользователю (показывает его пользователю).

GTK 4 изменяет видимость виджетов по умолчанию на включенную, поэтому каждому виджету не нужно изменять ее на включенную.
Но есть исключение.
Окно верхнего уровня (этот термин будет объяснен позже) не видно, когда оно создается.
Поэтому вам нужно использовать функцию выше, чтобы показать окно.

Вы можете использовать `gtk_widget_set_visible (win, true)` вместо `gtk_window_present`.
Но поведение этих двух функций отличается.
Предположим, что на экране есть два окна win1 и win2, и win1 находится позади win2.
Оба окна видимы.
Функция `gtk_widget_set_visible (win1, true)` ничего не делает, потому что win1 уже видно.
Таким образом, win1 все еще находится позади win2.
Другая функция `gtk_window_present (win1)` перемещает win1 в верхнюю часть стека окон.
Поэтому, если вы хотите представить окно, вы должны использовать `gtk_window_present`.

Две функции `gtk_widget_show` и `gtk_widget_hide` устарели с GTK 4.10.
Вместо них следует использовать `gtk_widget_set_visible`.

Сохраните программу как `pr3.c`, затем скомпилируйте и запустите её.

~~~
$ comp pr3
$ ./a.out
~~~

Появится небольшое окно.

![Скриншот окна](../image/screenshot_pr3.png)

Нажмите на кнопку закрытия, и окно исчезнет, а программа завершится.

### GtkApplicationWindow

GtkApplicationWindow является дочерним объектом GtkWindow.
Он имеет некоторые дополнительные функции для лучшей интеграции с GtkApplication.
Рекомендуется использовать его в качестве окна верхнего уровня приложения вместо GtkWindow.

Теперь перепишем программу и используем GtkApplicationWindow.

~~~C
1 static void
2 app_activate (GApplication *app, gpointer user_data) {
3   GtkWidget *win;
4 
5   win = gtk_application_window_new (GTK_APPLICATION (app));
6   gtk_window_set_title (GTK_WINDOW (win), "pr4");
7   gtk_window_set_default_size (GTK_WINDOW (win), 400, 300);
8   gtk_window_present (GTK_WINDOW (win));
9 }
~~~

Когда вы создаете GtkApplicationWindow, вам нужно передать экземпляр GtkApplication в качестве аргумента.
Тогда он автоматически подключает эти два экземпляра.
Поэтому вам больше не нужно вызывать `gtk_window_set_application`.

Программа устанавливает заголовок и размер окна по умолчанию.
Скомпилируйте её и запустите `a.out`, и вы увидите большее окно с заголовком "pr4".

![Скриншот окна](../image/screenshot_pr4.png)

Up: [README.md](../README.md),  Prev: [Section 2](sec2.md), Next: [Section 4](sec4.md)
