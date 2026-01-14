Up: [README.md](../README.md),  Prev: [Section 25](sec25.md), Next: [Section 27](sec27.md)

# Пользовательское рисование

Пользовательское рисование - это динамическое рисование фигур.
В этом разделе показан пример пользовательского рисования.
Вы можете рисовать прямоугольники, перетаскивая мышь.

Нажатие кнопки.

![нажатие кнопки](../image/cd0.png)

Перемещение мыши

![перемещение мыши](../image/cd1.png)

Отпускание кнопки.

![отпускание кнопки](../image/cd2.png)

Программы находятся в каталоге `src/custom_drawing`.
Скачайте [репозиторий](https://github.com/ToshioCP/Gtk4-tutorial) и посмотрите этот каталог.
Там четыре файла.

- meson.build
- rect.c
- rect.gresource.xml
- rect.ui

## rect.gresource.xml

Этот файл описывает ui-файл для компиляции.
Его использует компилятор glib-compile-resources.

~~~xml
1 <?xml version="1.0" encoding="UTF-8"?>
2 <gresources>
3   <gresource prefix="/com/github/ToshioCP/rect">
4     <file>rect.ui</file>
5   </gresource>
6 </gresources>
~~~

Префикс - `/com/github/ToshioCP/rect`, а файл - `rect.ui`.
Следовательно, GtkBuilder читает ресурс из `/com/github/ToshioCP/rect/rect.ui`.

## rect.ui

Ниже приведен ui-файл, который определяет виджеты.
Есть два виджета: GtkApplicationWindow и GtkDrawingArea.
Их идентификаторы - `win` и `da` соответственно.

~~~xml
 1 <?xml version="1.0" encoding="UTF-8"?>
 2 <interface>
 3   <object class="GtkApplicationWindow" id="win">
 4     <property name="default-width">800</property>
 5     <property name="default-height">600</property>
 6     <property name="resizable">FALSE</property>
 7     <property name="title">Custom drawing</property>
 8     <child>
 9       <object class="GtkDrawingArea" id="da">
10         <property name="hexpand">TRUE</property>
11         <property name="vexpand">TRUE</property>
12       </object>
13     </child>
14   </object>
15 </interface>
16 
~~~

## rect.c

### GtkApplication

Эта программа использует GtkApplication.
Идентификатор приложения - `com.github.ToshioCP.rect`.

```c
#define APPLICATION_ID "com.github.ToshioCP.rect"
```

Смотрите [документацию разработчика GNOME](https://developer.gnome.org/documentation/tutorials/application-id.html) для получения дополнительной информации.

Функция `main` вызывается в начале работы приложения.

~~~C
 1 int
 2 main (int argc, char **argv) {
 3   GtkApplication *app;
 4   int stat;
 5 
 6   app = gtk_application_new (APPLICATION_ID, G_APPLICATION_HANDLES_OPEN);
 7   g_signal_connect (app, "startup", G_CALLBACK (app_startup), NULL);
 8   g_signal_connect (app, "activate", G_CALLBACK (app_activate), NULL);
 9   g_signal_connect (app, "shutdown", G_CALLBACK (app_shutdown), NULL);
10   stat = g_application_run (G_APPLICATION (app), argc, argv);
11   g_object_unref (app);
12   return stat;
13 }
~~~

Она подключает три сигнала и обработчика.

- startup: Испускается после регистрации приложения в системе.
- activate: Испускается при активации приложения.
- shutdown: Испускается непосредственно перед завершением работы приложения.

~~~C
 1 static void
 2 app_startup (GApplication *application) {
 3   GtkApplication *app = GTK_APPLICATION (application);
 4   GtkBuilder *build;
 5   GtkWindow *win;
 6   GtkDrawingArea *da;
 7   GtkGesture *drag;
 8 
 9   build = gtk_builder_new_from_resource ("/com/github/ToshioCP/rect/rect.ui");
10   win = GTK_WINDOW (gtk_builder_get_object (build, "win"));
11   da = GTK_DRAWING_AREA (gtk_builder_get_object (build, "da"));
12   gtk_window_set_application (win, app);
13   g_object_unref (build);
14 
15   gtk_drawing_area_set_draw_func (da, draw_cb, NULL, NULL);
16   g_signal_connect_after (da, "resize", G_CALLBACK (resize_cb), NULL);
17 
18   drag = gtk_gesture_drag_new ();
19   gtk_gesture_single_set_button (GTK_GESTURE_SINGLE (drag), GDK_BUTTON_PRIMARY);
20   gtk_widget_add_controller (GTK_WIDGET (da), GTK_EVENT_CONTROLLER (drag));
21   g_signal_connect (drag, "drag-begin", G_CALLBACK (drag_begin), NULL);
22   g_signal_connect (drag, "drag-update", G_CALLBACK (drag_update), da);
23   g_signal_connect (drag, "drag-end", G_CALLBACK (drag_end), da);
24 }
~~~

Обработчик startup выполняет три действия.

- Создает виджеты.
- Инициализирует экземпляр GtkDrawingArea.
  - Устанавливает функцию рисования
  - Подключает сигнал "resize" и обработчик.
- Создает и инициализирует экземпляр GtkGestureDrag.
Жесты будут объяснены далее в этом разделе.

~~~C
1 static void
2 app_activate (GApplication *application) {
3   GtkApplication *app = GTK_APPLICATION (application);
4   GtkWindow *win;
5 
6   win = gtk_application_get_active_window (app);
7   gtk_window_present (win);
8 }
~~~

Обработчик activate просто показывает окно.

### GtkDrawingArea

Программа имеет две cairo-поверхности, на которые указывают глобальные переменные.

```C
static cairo_surface_t *surface = NULL;
static cairo_surface_t *surface_save = NULL;
```

Процесс рисования выглядит следующим образом.

- Создает изображение на `surface`.
- Копирует `surface` на cairo-поверхность GtkDrawingArea.
- Вызывает ` gtk_widget_queue_draw (da)` для отрисовки при необходимости.

Они создаются в обработчике сигнала "resize".

~~~C
 1 static void
 2 resize_cb (GtkWidget *widget, int width, int height, gpointer user_data) {
 3   cairo_t *cr;
 4 
 5   if (surface)
 6     cairo_surface_destroy (surface);
 7   surface = cairo_image_surface_create (CAIRO_FORMAT_RGB24, width, height);
 8   if (surface_save)
 9     cairo_surface_destroy (surface_save);
10   surface_save = cairo_image_surface_create (CAIRO_FORMAT_RGB24, width, height);
11   /* Paint the surface white. It is the background color. */
12   cr = cairo_create (surface);
13   cairo_set_source_rgb (cr, 1.0, 1.0, 1.0);
14   cairo_paint (cr);
15   cairo_destroy (cr);
16 }
~~~

Этот обратный вызов вызывается, когда GtkDrawingArea отображается.
Это единственный вызов, потому что окно не изменяет размер.

Он создает поверхности изображений для `surface` и `surface_save`.
Поверхность `surface` окрашивается в белый цвет, который является цветом фона.

Функция рисования копирует `surface` на поверхность GtkDrawingArea.

~~~C
1 static void
2 draw_cb (GtkDrawingArea *da, cairo_t *cr, int width, int height, gpointer user_data) {
3   if (surface) {
4     cairo_set_source_surface (cr, surface, 0.0, 0.0);
5     cairo_paint (cr);
6   }
7 }
~~~

Эта функция вызывается системой, когда необходимо перерисовать область рисования.

Две поверхности `surface` и `surface_save` уничтожаются перед завершением работы приложения.

~~~C
1 static void
2 app_shutdown (GApplication *application) {
3   if (surface)
4     cairo_surface_destroy (surface);
5   if (surface_save)
6     cairo_surface_destroy (surface_save);
7 }
~~~

### GtkGestureDrag

Класс Gesture используется для распознавания жестов пользователя, таких как щелчок, перетаскивание, панорамирование, проведение и т.д.
Это подкласс GtkEventController.
Класс GtkGesture является абстрактным и имеет несколько реализаций.

- GtkGestureClick
- GtkGestureDrag
- GtkGesturePan
- GtkGestureSwipe
- другие реализации

Программа `rect.c` использует GtkGestureDrag.
Это реализация для перетаскивания.
Отношение родитель-потомок следующее.

```
GObject -- GtkEventController -- GtkGesture -- GtkGestureSingle -- GtkGestureDrag
```

GtkGestureSingle - это подкласс GtkGesture, оптимизированный для жестов с одним касанием и жестов мыши.

Экземпляр GtkGestureDrag создается и инициализируется в обработчике сигнала startup в `rect.c`.
См. строки с 18 по 23 ниже.

~~~C
 1 static void
 2 app_startup (GApplication *application) {
 3   GtkApplication *app = GTK_APPLICATION (application);
 4   GtkBuilder *build;
 5   GtkWindow *win;
 6   GtkDrawingArea *da;
 7   GtkGesture *drag;
 8 
 9   build = gtk_builder_new_from_resource ("/com/github/ToshioCP/rect/rect.ui");
10   win = GTK_WINDOW (gtk_builder_get_object (build, "win"));
11   da = GTK_DRAWING_AREA (gtk_builder_get_object (build, "da"));
12   gtk_window_set_application (win, app);
13   g_object_unref (build);
14 
15   gtk_drawing_area_set_draw_func (da, draw_cb, NULL, NULL);
16   g_signal_connect_after (da, "resize", G_CALLBACK (resize_cb), NULL);
17 
18   drag = gtk_gesture_drag_new ();
19   gtk_gesture_single_set_button (GTK_GESTURE_SINGLE (drag), GDK_BUTTON_PRIMARY);
20   gtk_widget_add_controller (GTK_WIDGET (da), GTK_EVENT_CONTROLLER (drag));
21   g_signal_connect (drag, "drag-begin", G_CALLBACK (drag_begin), NULL);
22   g_signal_connect (drag, "drag-update", G_CALLBACK (drag_update), da);
23   g_signal_connect (drag, "drag-end", G_CALLBACK (drag_end), da);
24 }
~~~

- Функция `gtk_gesture_drag_new` создает новый экземпляр GtkGestureDrag.
- Функция `gtk_gesture_single_set_button` устанавливает номер кнопки для прослушивания.
Константа `GDK_BUTTON_PRIMARY` - это левая кнопка мыши.
- Функция `gtk_widget_add_controller` добавляет контроллер событий (жесты являются потомками контроллера событий) к виджету.
- Подключаются три сигнала и обработчика.
  - drag-begin: Испускается при начале перетаскивания.
  - drag-update: Испускается при перемещении точки перетаскивания.
  - drag-end: Испускается при окончании перетаскивания.

Процесс во время перетаскивания следующий.

- начало: сохранить поверхность и начальные точки
- обновление: восстановить поверхность и нарисовать тонкий прямоугольник между начальной точкой и текущей точкой мыши
- конец: восстановить поверхность и нарисовать толстый прямоугольник между начальной и конечной точками.

Нам нужны две глобальные переменные для начальной точки.

```C
static double start_x;
static double start_y;
```

Ниже приведен обработчик для сигнала "drag-begin".

~~~C
 1 static void
 2 copy_surface (cairo_surface_t *src, cairo_surface_t *dst) {
 3   if (!src || !dst)
 4     return;
 5   cairo_t *cr = cairo_create (dst);
 6   cairo_set_source_surface (cr, src, 0.0, 0.0);
 7   cairo_paint (cr);
 8   cairo_destroy (cr);
 9 }
10 
11 static void
12 drag_begin (GtkGestureDrag *gesture, double x, double y, gpointer user_data) {
13   // save the surface and record (x, y)
14   copy_surface (surface, surface_save);
15   start_x = x;
16   start_y = y;
17 }
~~~

- Копирует `surface` в `surface_save`, которая представляет изображение непосредственно перед перетаскиванием.
- Сохраняет точки в `start_x` и `start_y`.

~~~C
 1 static void
 2 drag_update  (GtkGestureDrag *gesture, double offset_x, double offset_y, gpointer user_data) {
 3   GtkWidget *da = GTK_WIDGET (user_data);
 4   cairo_t *cr;
 5   
 6   copy_surface (surface_save, surface);
 7   cr = cairo_create (surface);
 8   cairo_rectangle (cr, start_x, start_y, offset_x, offset_y);
 9   cairo_set_line_width (cr, 1.0);
10   cairo_stroke (cr);
11   cairo_destroy (cr);
12   gtk_widget_queue_draw (da);
13 }
~~~

- Восстанавливает `surface` из `surface_save`.
- Рисует прямоугольник тонкими линиями.
- Вызывает `gtk_widget_queue_draw`, чтобы добавить GtkDrawingArea в очередь для перерисовки.

~~~C
 1 static void
 2 drag_end  (GtkGestureDrag *gesture, double offset_x, double offset_y, gpointer user_data) {
 3   GtkWidget *da = GTK_WIDGET (user_data);
 4   cairo_t *cr;
 5   
 6   copy_surface (surface_save, surface);
 7   cr = cairo_create (surface);
 8   cairo_rectangle (cr, start_x, start_y, offset_x, offset_y);
 9   cairo_set_line_width (cr, 6.0);
10   cairo_stroke (cr);
11   cairo_destroy (cr);
12   gtk_widget_queue_draw (da);
13 }
~~~

- Восстанавливает `surface` из `surface_save`.
- Рисует прямоугольник толстыми линиями.
- Вызывает `gtk_widget_queue_draw`, чтобы добавить GtkDrawingArea в очередь для перерисовки.

## Сборка и запуск

Скачайте [репозиторий](https://github.com/ToshioCP/Gtk4-tutorial).
Измените текущий каталог на `src/custom_drawing`.
Запустите meson и ninja для сборки программы.
Введите `_build/rect` для запуска программы.
Попробуйте нарисовать прямоугольники.

```
$ cd src/custom_drawing
$ meson setup _build
$ ninja -C _build
$ _build/rect
```

![Экран программы rect](../image/rect.png)

Up: [README.md](../README.md),  Prev: [Section 25](sec25.md), Next: [Section 27](sec27.md)
