Up: [README.md](../README.md),  Prev: [Section 23](sec23.md), Next: [Section 25](sec25.md)

# GtkDrawingArea и Cairo

Если вы хотите рисовать фигуры или динамически отображать изображения на экране, используйте виджет GtkDrawingArea.

GtkDrawingArea предоставляет контекст рисования cairo.
Вы можете рисовать изображения с помощью функций библиотеки cairo.
Этот раздел описывает:

1. Cairo, но кратко
2. GtkDrawingArea, с очень простым примером.

## Cairo

Cairo — это библиотека рисования для двумерной графики.
На [веб-сайте Cairo](https://www.cairographics.org/) есть много документов.
Если вы не знакомы с Cairo, стоит прочитать [руководство](https://www.cairographics.org/tutorial/).

Ниже приведено введение в библиотеку Cairo.
Сначала вам нужно знать о поверхностях, источниках, масках, назначениях, контексте cairo и преобразованиях.

- Поверхность представляет изображение.
Это как холст.
Мы можем рисовать фигуры и изображения с разными цветами на поверхностях.
- Шаблон источника, или просто источник, подобен краске, которая будет перенесена на поверхность назначения функциями cairo.
- Маска описывает область, которая будет использоваться при копировании;
- Назначение — это целевая поверхность;
- Контекст cairo управляет переносом из источника в назначение через маску с помощью своих функций;
Например, `cairo_stroke` — это функция для рисования пути к назначению путём переноса.
- Преобразование может быть применено до завершения переноса.
Применяемое преобразование называется аффинным, что является математическим термином, означающим преобразования,
которые сохраняют прямые линии.
Масштабирование, поворот, отражение, сдвиг и перемещение — примеры аффинных преобразований.
Они математически представлены умножением матриц и сложением векторов.
В этом разделе мы не будем его использовать, вместо этого мы будем использовать только тождественное преобразование.
Это означает, что координаты в источнике и маске совпадают с координатами в назначении.

![Stroke a rectangle](../image/cairo.png)

Инструкция следующая:

1. Создать поверхность.
Это будет назначение.
2. Создать контекст cairo с поверхностью, поверхность будет назначением контекста.
3. Создать шаблон источника в контексте.
4. Создать пути, которые являются линиями, прямоугольниками, дугами, текстами или более сложными фигурами в маске.
5. Использовать оператор рисования, такой как `cairo_stroke`, для переноса краски из источника в назначение.
6. Сохранить поверхность назначения в файл, если необходимо.

Вот простой пример программы, которая рисует небольшой квадрат и сохраняет его как файл png.
Путь к файлу — `src/misc/cairo.c`.

~~~C
 1 #include <cairo.h>
 2 
 3 int
 4 main (int argc, char **argv)
 5 {
 6   cairo_surface_t *surface;
 7   cairo_t *cr;
 8   int width = 100;
 9   int height = 100;
10   int square_size = 40.0;
11 
12   /* Create surface and cairo */
13   surface = cairo_image_surface_create (CAIRO_FORMAT_RGB24, width, height);
14   cr = cairo_create (surface);
15 
16   /* Drawing starts here. */
17   /* Paint the background white */
18   cairo_set_source_rgb (cr, 1.0, 1.0, 1.0);
19   cairo_paint (cr);
20   /* Draw a black rectangle */
21   cairo_set_source_rgb (cr, 0.0, 0.0, 0.0);
22   cairo_set_line_width (cr, 2.0);
23   cairo_rectangle (cr,
24                    width/2.0 - square_size/2,
25                    height/2.0 - square_size/2,
26                    square_size,
27                    square_size);
28   cairo_stroke (cr);
29 
30   /* Write the surface to a png file and clean up cairo and surface. */
31   cairo_surface_write_to_png (surface, "rectangle.png");
32   cairo_destroy (cr);
33   cairo_surface_destroy (surface);
34 
35   return 0;
36 }
~~~

- 1: включает заголовочный файл Cairo.
- 6: `cairo_surface_t` — это тип поверхности.
- 7: `cairo_t` — это тип контекста cairo.
- 8-10: `width` и `height` — это размер `surface`.
`square_size` — это размер квадрата, который будет нарисован на поверхности.
- 13: `cairo_image_surface_create` создаёт поверхность изображения.
`CAIRO_FORMAT_RGB24` — это константа, которая означает, что каждый пиксель имеет красные, зелёные и синие данные,
и каждая точка данных является 8-битным числом (всего 24 бита).
Современные дисплеи имеют такую глубину цвета.
Ширина и высота в пикселях задаются целыми числами.
- 14: создаёт контекст cairo.
Поверхность, переданная в качестве аргумента, будет назначением контекста.
- 18: `cairo_set_source_rgb` создаёт шаблон источника, который является сплошной белой краской.
Второй-четвёртый аргументы — это значения красного, зелёного и синего цвета соответственно, и они имеют тип float.
Значения находятся между нулём (0.0) и единицей (1.0).
Чёрный — это (0.0,0.0,0.0), а белый — это (1.0,1.0,1.0).
- 19: `cairo_paint` копирует везде из источника в назначение.
Назначение заполняется белыми пикселями этой командой.
- 21: устанавливает цвет источника в чёрный.
- 22: `cairo_set_line_width` устанавливает ширину линий.
В этом случае ширина линии устанавливается в два пикселя и в итоге будет того же размера.
(Это потому, что преобразование является тождественным.
Если преобразование не является тождественным, например масштабирование с коэффициентом три, фактическая ширина в назначении будет шесть (2x3=6) пикселей.)
- 23: рисует прямоугольник (квадрат) на маске.
Квадрат расположен в центре.
- 24: `cairo_stroke` переносит источник в назначение через прямоугольник в маске.
- 31: выводит изображение в файл png `rectangle.png`.
- 32: уничтожает контекст. В то же время источник уничтожается.
- 33: уничтожает поверхность.

Для компиляции измените текущий каталог на `src/misc` и введите следующее.

```
$ gcc `pkg-config --cflags cairo` cairo.c `pkg-config --libs cairo`
```
s
![rectangle.png](../image/rectangle.png)

Для получения дополнительной информации см. [веб-сайт Cairo](https://www.cairographics.org/).

## GtkDrawingArea

Ниже приведён очень простой пример.

~~~C
 1 #include <gtk/gtk.h>
 2 
 3 static void
 4 draw_function (GtkDrawingArea *area, cairo_t *cr, int width, int height, gpointer user_data) {
 5   int square_size = 40.0;
 6 
 7   cairo_set_source_rgb (cr, 1.0, 1.0, 1.0); /* white */
 8   cairo_paint (cr);
 9   cairo_set_line_width (cr, 2.0);
10   cairo_set_source_rgb (cr, 0.0, 0.0, 0.0); /* black */
11   cairo_rectangle (cr,
12                    width/2.0 - square_size/2,
13                    height/2.0 - square_size/2,
14                    square_size,
15                    square_size);
16   cairo_stroke (cr);
17 }
18 
19 static void
20 app_activate (GApplication *app, gpointer user_data) {
21   GtkWidget *win = gtk_application_window_new (GTK_APPLICATION (app));
22   GtkWidget *area = gtk_drawing_area_new ();
23 
24   gtk_window_set_title (GTK_WINDOW (win), "da1");
25   gtk_drawing_area_set_draw_func (GTK_DRAWING_AREA (area), draw_function, NULL, NULL);
26   gtk_window_set_child (GTK_WINDOW (win), area);
27 
28   gtk_window_present (GTK_WINDOW (win));
29 }
30 
31 #define APPLICATION_ID "com.github.ToshioCP.da1"
32 
33 int
34 main (int argc, char **argv) {
35   GtkApplication *app;
36   int stat;
37 
38   app = gtk_application_new (APPLICATION_ID, G_APPLICATION_DEFAULT_FLAGS);
39   g_signal_connect (app, "activate", G_CALLBACK (app_activate), NULL);
40   stat = g_application_run (G_APPLICATION (app), argc, argv);
41   g_object_unref (app);
42   return stat;
43 }
~~~

Функция `main` почти такая же, как и раньше.
Две функции `app_activate` и `draw_function` важны в этом примере.

- 22: создаёт экземпляр GtkDrawingArea.
- 25: устанавливает функцию рисования виджета.
Виджет GtkDrawingArea использует функцию `draw_function` для рисования своего содержимого всякий раз, когда это необходимо.
Например, когда пользователь перетаскивает указатель мыши и изменяет размер окна верхнего уровня, GtkDrawingArea также изменяет размер.
Затем всё окно нужно перерисовать.
Для получения информации о `gtk_drawing_area_set_draw_func` см. [Gtk API Reference -- gtk\_drawing\_area\_set\_draw\_func](https://docs.gtk.org/gtk4/method.DrawingArea.set_draw_func.html).

Функция рисования имеет пять параметров.

~~~C
void drawing_function (GtkDrawingArea *drawing_area, cairo_t *cr, int width, int height,
                       gpointer user_data);
~~~

Первый параметр — это виджет GtkDrawingArea.
Вы не можете изменять какие-либо свойства, например `content-width` или `content-height`, в этой функции.
Второй параметр — это контекст cairo, предоставленный виджетом.
Поверхность назначения контекста подключена к содержимому виджета.
То, что вы рисуете на этой поверхности, появится в виджете на экране.
Третий и четвёртый параметры — это размер поверхности назначения.
Теперь снова посмотрите на программу.

- 3-17: функция рисования.
- 7-8: устанавливает источник в белый цвет и окрашивает назначение в белый.
- 9: устанавливает ширину линии в 2.
- 10: устанавливает источник в чёрный цвет.
- 11-15: добавляет прямоугольник к маске.
- 16: рисует прямоугольник чёрным цветом в назначение.

Программа находится в [src/misc/da1.c](../src/misc/da1.c).
Скомпилируйте и запустите её, тогда появится окно с чёрным прямоугольником (квадратом).
Попробуйте изменить размер окна.
Квадрат всегда появляется в центре окна, потому что функция рисования вызывается каждый раз при изменении размера окна.

![Square in the window](../image/da1.png)

Up: [README.md](../README.md),  Prev: [Section 23](sec23.md), Next: [Section 25](sec25.md)
