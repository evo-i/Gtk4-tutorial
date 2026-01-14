# Пользовательское рисование

Пользовательское рисование - это динамическое рисование фигур.
В этом разделе показан пример пользовательского рисования.
Вы можете рисовать прямоугольники, перетаскивая мышь.

Нажатие кнопки.

![нажатие кнопки](../image/cd0.png){width=6cm height=4.83cm}

Перемещение мыши

![перемещение мыши](../image/cd1.png){width=6cm height=4.83cm}

Отпускание кнопки.

![отпускание кнопки](../image/cd2.png){width=6cm height=4.83cm}

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

@@@include
custom_drawing/rect.gresource.xml
@@@

Префикс - `/com/github/ToshioCP/rect`, а файл - `rect.ui`.
Следовательно, GtkBuilder читает ресурс из `/com/github/ToshioCP/rect/rect.ui`.

## rect.ui

Ниже приведен ui-файл, который определяет виджеты.
Есть два виджета: GtkApplicationWindow и GtkDrawingArea.
Их идентификаторы - `win` и `da` соответственно.

@@@include
custom_drawing/rect.ui
@@@

## rect.c

### GtkApplication

Эта программа использует GtkApplication.
Идентификатор приложения - `com.github.ToshioCP.rect`.

```c
#define APPLICATION_ID "com.github.ToshioCP.rect"
```

Смотрите [документацию разработчика GNOME](https://developer.gnome.org/documentation/tutorials/application-id.html) для получения дополнительной информации.

Функция `main` вызывается в начале работы приложения.

@@@include
custom_drawing/rect.c main
@@@

Она подключает три сигнала и обработчика.

- startup: Испускается после регистрации приложения в системе.
- activate: Испускается при активации приложения.
- shutdown: Испускается непосредственно перед завершением работы приложения.

@@@include
custom_drawing/rect.c app_startup
@@@

Обработчик startup выполняет три действия.

- Создает виджеты.
- Инициализирует экземпляр GtkDrawingArea.
  - Устанавливает функцию рисования
  - Подключает сигнал "resize" и обработчик.
- Создает и инициализирует экземпляр GtkGestureDrag.
Жесты будут объяснены далее в этом разделе.

@@@include
custom_drawing/rect.c app_activate
@@@

Обработчик activate просто показывает окно.

### GtkDrawingArea

Программа имеет две cairo-поверхности, на которые указывают глобальные переменные.

@@@if gfm
```C
static cairo_surface_t *surface = NULL;
static cairo_surface_t *surface_save = NULL;
```
@@@else
```{.C}
static cairo_surface_t *surface = NULL;
static cairo_surface_t *surface_save = NULL;
```
@@@end

Процесс рисования выглядит следующим образом.

- Создает изображение на `surface`.
- Копирует `surface` на cairo-поверхность GtkDrawingArea.
- Вызывает ` gtk_widget_queue_draw (da)` для отрисовки при необходимости.

Они создаются в обработчике сигнала "resize".

@@@include
custom_drawing/rect.c resize_cb
@@@

Этот обратный вызов вызывается, когда GtkDrawingArea отображается.
Это единственный вызов, потому что окно не изменяет размер.

Он создает поверхности изображений для `surface` и `surface_save`.
Поверхность `surface` окрашивается в белый цвет, который является цветом фона.

Функция рисования копирует `surface` на поверхность GtkDrawingArea.

@@@include
custom_drawing/rect.c draw_cb
@@@

Эта функция вызывается системой, когда необходимо перерисовать область рисования.

Две поверхности `surface` и `surface_save` уничтожаются перед завершением работы приложения.

@@@include
custom_drawing/rect.c app_shutdown
@@@

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

@@@include
custom_drawing/rect.c app_startup
@@@

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

@@@if gfm
```C
static double start_x;
static double start_y;
```
@@@else
```{.C}
static double start_x;
static double start_y;
```
@@@end

Ниже приведен обработчик для сигнала "drag-begin".

@@@include
custom_drawing/rect.c copy_surface drag_begin
@@@

- Копирует `surface` в `surface_save`, которая представляет изображение непосредственно перед перетаскиванием.
- Сохраняет точки в `start_x` и `start_y`.

@@@include
custom_drawing/rect.c drag_update
@@@

- Восстанавливает `surface` из `surface_save`.
- Рисует прямоугольник тонкими линиями.
- Вызывает `gtk_widget_queue_draw`, чтобы добавить GtkDrawingArea в очередь для перерисовки.

@@@include
custom_drawing/rect.c drag_end
@@@

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

![Экран программы rect](../image/rect.png){width=12.4cm height=10cm}
