# Миниатюрный интерпретатор графики черепашки

Программа `turtle` является примером комбинации объектов TfeTextView и GtkDrawingArea.
Это очень маленький интерпретатор, но с его помощью вы можете рисовать фрактальные кривые.
На следующей диаграмме показана кривая Коха, которая является одной из известных фрактальных кривых.

![Koch curve](turtle/image/turtle_koch.png){width=8cm height=5.11cm}

Следующая кривая имеет форму снежного кристалла.
Она состоит из шести кривых Коха.

![Snow](../image/turtle_snow.png){width=8cm height=5.11cm}

Эта программа использует flex и bison.
Flex — это лексический анализатор.
Bison — это генератор парсеров.
Эти две программы похожи на lex и yacc, которые являются проприетарным программным обеспечением, разработанным в Bell Laboratory.
Однако flex и bison являются программным обеспечением с открытым исходным кодом.
Этот раздел описывает их, но они не являются темами о GTK 4.
Поэтому читатели могут пропустить этот раздел.

## Как использовать turtle

@@@if gfm
The turtle document is [here](turtle/turtle_doc.src.md).
@@@elif html
The turtle document is [here](https://toshiocp.github.io/Gtk4-tutorial/turtle_doc.html).
@@@elif latex
The turtle document is in the appendix.
@@@end
Я покажу вам простой пример.

~~~
fc (1,0,0) # Foreground color is red, rgb = (1,0,0).
pd         # Pen down.
rp (4) {   # Repeat four times.
  fd 100   # Go forward by 100 pixels.
  tr 90    # Turn right by 90 degrees.
}
~~~

1. Скомпилируйте и установите `turtle` (см. документацию выше).
Затем запустите `turtle`.
2. Введите программу выше в редакторе (левая часть окна).
3. Нажмите на кнопку `Run`, затем в правой части окна появится красный квадрат.
Сторона квадрата имеет длину 100 пикселей.

Таким же образом вы можете рисовать другие кривые.
Документация turtle включает некоторые фрактальные кривые, такие как дерево, снежинка и квадрат-Коха.
Исходные коды находятся в каталоге [src/turtle/example](turtle/example).
Вы можете загрузить эти файлы в редактор `turtle`, нажав на кнопку `Open`.

## Комбинация объектов TfeTextView и GtkDrawingArea

Turtle использует TfeTextView и GtkDrawingArea.

1. Пользователь вводит/читает программу turtle в буфер экземпляра TfeTextView.
2. Пользователь нажимает на кнопку "Run".
3. Парсер читает программу и генерирует древовидные структурированные данные.
4. Интерпретатор читает данные и выполняет их шаг за шагом.
И он рисует фигуры на поверхности.
Поверхность не является той, что в виджете GtkDrawingArea.
5. Виджет добавляется в очередь.
Он будет перерисован функцией рисования, которая просто копирует поверхность в ту, что в GtkDrawingArea.

![Parser, interpreter and drawing function](../image/turtle.png)

Тело интерпретатора написано с использованием flex и bison.
Код не является потокобезопасным.
Поэтому функция обратного вызова `run_cb`, которая является обработчиком сигнала "clicked" на кнопке `Run`, предотвращает повторный вход.

@@@include
turtle/turtleapplication.c run_cb resize_cb
@@@

- 8, 13-15: Статическое значение `busy` содержит статус интерпретатора.
Если оно `TRUE`, интерпретатор работает и невозможно вызвать интерпретатор, потому что это не реентерабельная программа.
Если оно `FALSE`, безопасно вызвать интерпретатор и установить переменную `busy` в TRUE.
- 16-17: Получает содержимое `tb`.
- 18-30: Переменная `surface` является статической переменной.
Она указывает на экземпляр `cairo_surface_t`.
Она создается, когда экземпляр GtkDrawingArea реализован, и когда он изменяет размер.
Поэтому `surface` обычно не NULL.
Но если она NULL, интерпретатор не будет вызван.
- 18-24: Если `surface` указывает на экземпляр поверхности и строка `contents` не пустая, вызывается интерпретатор.
  - Инициализирует лексический анализатор.
  - Вызывает парсер. Парсер анализирует код программы синтаксически и генерирует древовидные структурированные данные.
  - Если парсер успешно проанализировал, он вызывает процедуру выполнения 'run'.
  - Завершает лексический анализатор.
- 25-29: Если `surface` указывает на экземпляр поверхности и строка `contents` пустая, она очищает поверхность `surface`.
- 31: Освобождает `contents`.
- 32: Добавляет виджет области рисования в очередь для рисования.
- 33: Устанавливает переменную `busy` в FALSE.
- 36-43: Обработчик сигнала "resized".
Если `surface` не NULL, она уничтожается.
Создается новая поверхность.
Ее размер такой же, как у поверхности экземпляра GtkDrawingArea.
Вызывается функция обратного вызова `run_cb` для перерисовки фигуры в области рисования.

Если нажата кнопка открытия и файл прочитан, имя файла будет показано в заголовке.

@@@include
turtle/turtleapplication.c show_filename
@@@

Эта функция является функцией обратного вызова для сигнала "change-file" на экземпляре TfeTextView.
Она вызывает `tfe_text_view_get_file`.

- Если возвращаемое значение является экземпляром GFile, заголовок будет "Turtle (имя файла)".
- В противном случае заголовок будет "Turtle".

Другая часть `turtleapplication.c` очень проста и похожа на код в предыдущих приложениях.
Код `turtleapplication.c` находится в [каталоге turtle](turtle).

## Что делает интерпретатор?

Предположим, что приложение turtle работает со следующей программой.

~~~
distance = 100
fd distance*2
~~~

Приложение распознает программу и работает следующим образом.

- Обычно программа состоит из токенов.
Токены в приведенном выше примере — это "distance", "=", "100", "fd", "*" и "2".
- Парсер вызывает функцию `yylex` для чтения токена в исходном файле.
`yylex` возвращает код, который называется "тип токена", и устанавливает глобальную переменную `yylval` в значение, которое называется семантическим значением.
Тип `yylval` — объединение. Типы `yylval.ID` и `yylval.NUM` — строка и double соответственно.
В программе семь токенов, поэтому `yylex` вызывается семь раз.

|   |token kind|yylval.ID|yylval.NUM|
|:-:|:--------:|:-------:|:--------:|
| 1 |    ID    |distance |          |
| 2 |    =     |         |          |
| 3 |   NUM    |         |   100    |
| 4 |    FD    |         |          |
| 5 |    ID    |distance |          |
| 6 |    *     |         |          |
| 7 |   NUM    |         |    2     |

- Функция `yylex` возвращает тип токена каждый раз, но она не устанавливает `yylval.ID` или `yylval.NUM` каждый раз.
Это потому, что ключевые слова (`FD`) и символы (`=` и `*`) не имеют семантических значений.
Функция `yylex` называется лексическим анализатором или сканером.
- Приложение `turtle` создает древовидные структурированные данные.
Эта часть `turtle` называется парсером.

![turtle parser tree](../image/turtle_parser_tree.png){width=12cm height=5.34cm}

- `Turtle` анализирует дерево и выполняет его.
Эта часть `turtle` называется процедурой выполнения или интерпретатором.
Дерево состоит из прямоугольников и линейных сегментов между прямоугольниками.
Прямоугольники называются узлами.
Например, N\_PROGRAM, N\_ASSIGN, N\_FD и N\_MUL — это узлы.
  1. Спускается от N\_PROGRAM к N\_ASSIGN.
  2. Узел N_ASSIGN имеет двух потомков, ID и NUM.
Этот узел происходит от "distance = 100", что синтаксически является "ID = NUM".
Сначала `turtle` проверяет, является ли первый потомок ID.
Если это ID, тогда `turtle` ищет переменную в таблице переменных.
Если она не существует, он регистрирует ID (`distance`) в таблице.
Затем возвращается к узлу N\_ASSIGN.
  3. `Turtle` вычисляет второго потомка.
В этом случае это число 100.
Сохраняет 100 в таблице переменных в записи `distance`.
  4. `Turtle` возвращается к N\_PROGRAM, затем переходит к следующему узлу N\_FD.
У него только один потомок.
Спускается к потомку N\_MUL.
  5. Первый потомок — ID (distance).
Ищет переменную `distance` в таблице переменных и получает значение 100.
Второй потомок — число 2.
Умножает 100 на 2 и получает 200.
Затем `turtle` возвращается к N_FD.
  6. Теперь `turtle` знает, что расстояние равно 200.
Он перемещает курсор вперед на 200 пикселей.
Сегмент рисуется на `surface`.
  8. Больше нет узлов для обработки.
Процедура выполнения возвращается к функции `run_cb`.

- Функция `run_cb` вызывает `gtk_widget_queue_draw` и помещает виджет GtkDrawingArea в очередь.
- Система перерисовывает виджет.
В это время вызывается функция рисования `draw_func`.
Функция копирует `surface` на поверхность в GtkDrawingArea.

Фактическая программа turtle более сложна, чем приведенный выше пример.
Однако то, что делает turtle, в основном то же самое.
Интерпретация состоит из трех частей.

- Лексический анализ
- Синтаксический разбор и генерация дерева
- Интерпретация и выполнение дерева.

## Поток компиляции

Исходные файлы:

- исходный файл flex => `turtle.lex`
- исходный файл bison => `turtle.y`
- заголовочный файл C => `turtle_lex.h`
- исходный файл C => `turtleapplication.c`
- другие файлы => `turtle.ui`, `turtle.gresources.xml` и `meson.build`

Процесс компиляции немного сложен.

1. glib-compile-resources компилирует `turtle.ui` в `resources.c` согласно `turtle.gresource.xml`.
Также генерирует `resources.h`.
2. bison компилирует `turtle.y` в `turtle_parser.c` и генерирует `turtle_parser.h`
3. flex компилирует `turtle.lex` в `turtle_lex.c`.
4. gcc компилирует `application.c`, `resources.c`, `turtle_parser.c` и `turtle_lex.c` с `turtle_lex.h`, `resources.h` и `turtle_parser.h`.
Генерирует исполняемый файл `turtle`.

![compile process](../image/turtle_compile_process.png){width=12cm height=9cm}

Meson управляет процессом.
Инструкция описана в `meson.build`.

@@@include
turtle/meson.build
@@@

- 1: Имя проекта — "turtle", а язык программирования — C.
- 3: Получает компилятор C.
Обычно это `gcc` в linux.
- 4: Получает математическую библиотеку.
Эта программа использует тригонометрические функции.
Они определены в математической библиотеке, но библиотека опциональна.
Поэтому необходимо включить ее с помощью `#include <math.h>` и также связать библиотеку с компоновщиком.
- 6: Получает библиотеку gtk4.
- 8: Получает модуль gnome. Для дополнительной информации см. [Meson build system website -- GNOME module](https://mesonbuild.com/Gnome-module.html#gnome-module).
- 9: Компилирует ui файл в исходный файл C согласно XML файлу `turtle.gresource.xml`.
- 11: Получает flex.
- 12: Получает bison.
- 13: Компилирует `turtle.y` в `turtle_parser.c` и `turtle_parser.h` с помощью bison.
Функция `custom_target` создает пользовательскую цель верхнего уровня.
Для дополнительной информации см. [Meson build system website -- custom target](https://mesonbuild.com/Reference-manual_functions.html#custom_target).
- 14: Компилирует `turtle.lex` в `turtle_lex.c` с помощью flex.
- 16: Переменная `sourcefiles` является файловым объектом, созданным с исходными файлами C.
- 18: Компилирует исходные файлы C, включая сгенерированные файлы glib-compile-resources, bison и flex.
Аргумент `turtleparser[1]` ссылается на `tirtle_parser.h`, который является вторым выходом в строке 13.

## Turtle.lex

### Что делает flex?

Flex создает лексический анализатор из исходного файла flex.
Исходный файл flex — это текстовый файл.
Его синтаксические правила будут объяснены позже.
Сгенерированный лексический анализатор — это исходный файл C.
Он также называется сканером.
Он читает текстовый файл, который является исходным файлом языка программирования, и получает имена переменных, числа и символы.
Предположим, вот исходный файл turtle.

~~~
fc (1,0,0) # Foreground color is red, rgb = (1,0,0).
pd         # Pen down.
distance = 100
angle = 90
fd distance    # Go forward by distance (100) pixels.
tr angle     # Turn right by angle (90) degrees.
~~~

Содержимое текстового файла разделяется на `fc`, `(`, `1` и так далее.
Слова `fc`, `pd`, `distance`, `angle`, `tr`, `1`, `0`, `100` и `90` называются токенами.
Символы '`(`' (левая скобка), '`,`' (запятая), '`)`' (правая скобка) и '`=`' (знак равенства) называются символами.
(Иногда эти символы также называют токенами.)

Flex читает `turtle.lex` и генерирует исходный файл C для сканера.
Файл `turtle.lex` определяет токены, символы и поведение, которое соответствует каждому токену или символу.
Turtle.lex — это не большая программа.

@@@include
turtle/turtle.lex
@@@

Файл состоит из трех разделов, которые разделены "%%" (строки 19 и 59).
Это разделы определений, правил и пользовательского кода.

### Раздел определений

- 1-12: Строки между "%top{" и "}" являются исходным кодом C.
Они будут скопированы в начало сгенерированного исходного файла C.
- 2-3: Эта программа использует две функции `strlen` (с.65) и `atof` (с.40).
Они определены в `string.h` и `stdlib.h` соответственно.
Эти два заголовочных файла включены здесь.
- 4: Эта программа использует некоторые функции и структуры GLib, такие как `g_strdup` и `GSList`.
Заголовочный файл GLib — это `glib.h`, и он включен здесь.
- 5: Заголовочный файл "turtle_parser.h" генерируется из "turtle.y" с помощью bison.
Он определяет некоторые константы и функции, такие как `PU` и `yylloc`.
Заголовочный файл включен здесь.
- 7-9: Текущая позиция ввода указывается `nline` и `ncolumn`.
Функция `get_location` объявлена здесь, чтобы ее можно было вызвать до того, как функция определена (с.61-65).
- 12: GSlist — это структура для односвязного списка.
Переменная `list` определена в `turtle.y`, поэтому ее класс — `extern`.
Это начальная точка списка.
Список используется для хранения выделенной памяти.
- 15: Эта опция `%option noyywrap` должна быть указана, когда у вас есть только один исходный файл для сканера. Обратитесь к "9 The Generated Scanner" в документации flex в вашем дистрибутиве.
(Документация не в Интернете.)
- 17-18: `REAL_NUMBER` и `IDENTIFIER` — это имена.
Имя начинается с буквы или подчеркивания, за которым следуют ноль или более букв, цифр, подчеркиваний (`_`) или тире (`-`).
За ними следуют регулярные выражения, которые являются их определениями.
Они будут использоваться в разделе правил и расширяться до определения.

### Раздел правил

Этот раздел является наиболее важной частью.
Правила состоят из шаблонов и действий.
Шаблоны — это регулярные выражения или имена, заключенные в фигурные скобки.
Имена должны быть определены в разделе определений.
Определение регулярного выражения написано в документации flex.

Например, строка 40 является правилом.

- `{REAL_NUMBER}` — это шаблон
- `get_location (yytext); yylval.NUM = atof (yytext); return NUM;` — это действие.

`{REAL_NUMBER}` определен в строке 17, поэтому он расширяется до `(0|[1-9][0-9]*)(\.[0-9]+)?`.
Это регулярное выражение соответствует числам, таким как `0`, `12` и `1.5`.
Если ввод является числом, он соответствует шаблону в строке 40.
Затем совпавший текст присваивается `yytext`, и выполняется соответствующее действие.
Функция `get_location` изменяет переменные местоположения на позицию в тексте.
Она присваивает `atof (yytext)`, которое является числом типа double, преобразованным из `yytext`, к `yylval.NUM` и возвращает `NUM`.
`NUM` — это тип токена, и он представляет числа (типа double).
Он определен в `turtle.y`.

Сканер, сгенерированный flex, имеет функцию `yylex`.
Если `yylex` вызывается и вводом является "123.4", то он работает следующим образом.

1. Строка "123.4" соответствует `{REAL_NUMBER}`.
2. Обновляет переменную местоположения `ncolumn`. Структура `yylloc` устанавливается `get_location`.
3. Функция `atof` преобразует строку "123.4" в число типа double 123.4.
4. Оно присваивается `yylval.NUM`.
5. `yylex` возвращает `NUM` вызывающему.

Затем вызывающий знает, что ввод является числом (`NUM`), и его значение равно 123.4.

- 20-58: Раздел правил.
- 21: Символ `.` (точка) соответствует любому символу, кроме новой строки.
Поэтому комментарий начинается с `#`, за которым следуют любые символы, кроме новой строки.
Никакого действия не происходит.
Это означает, что комментарии игнорируются.
- 22: Пробел просто увеличивает переменную `ncolumn` на единицу.
- 23: Предполагается, что табуляция равна восьми пробелам.
- 24: Новая строка увеличивает переменную `nline` на единицу и сбрасывает `ncolumn`.
- 26-38: Ключевые слова обновляют переменные местоположения `ncolumn` и `yylloc` и возвращают типы токенов ключевых слов.
- 40: Константа вещественного числа.
Действие преобразует текст `yytext` в число типа double, помещает его в `yylval.NUM` и возвращает `NUM`.
- 42: `IDENTIFIER` определен в строке 18.
Идентификатор — это имя переменной или процедуры.
Он начинается с буквы и за ним следуют буквы или цифры.
Переменные местоположения обновляются, и имя идентификатора присваивается `yylval.ID`.
Память для имени выделяется функцией `g_strdup`.
Память регистрируется в списке (список типа GSlist).
Память будет освобождена после завершения процедуры выполнения.
Возвращается тип токена `ID`.
- 46-57: Символы обновляют переменную местоположения и возвращают типы токенов.
Тип токена совпадает с самим символом.
- 58: Если ввод не соответствует шаблонам, то это ошибка.
Возвращается специальный тип токена `YYUNDEF`.

### Раздел пользовательского кода

Этот раздел просто копируется в исходный файл C.

- 61-66: Функция `get_location`.
Местоположение ввода записывается в `nline` и `ncolumn`.
Переменная `yylloc` используется парсером.
Это структура C, имеющая четыре члена: `first_line`, `first_column`, `last_line` и `last_column`.
Они указывают начало и конец текущего входного текста.
- 68: `YY_BUFFER_STATE` — это указатель, указывающий на входной буфер.
Flex создает определение `YY_BUFFER_STATE` в файле C (исходный файл сканера `turtle_lex.c`).
Для дополнительной информации см. вашу документацию flex, раздел 11 Multiple Input Buffers.
- 70-73: Функция `init_flex` вызывается `run_cb`, который является обработчиком сигнала "clicked" на кнопке `Run`.
Она имеет один параметр типа строка.
Вызывающий присваивает ему содержимое экземпляра GtkTextBuffer.
Функция `yy_scan_string` устанавливает входной буфер для сканера.
- 75-78: Функция `finalize_flex` вызывается после завершения процедуры выполнения.
Она удаляет входной буфер.

## Turtle.y

Turtle.y имеет более 800 строк, поэтому трудно объяснить весь исходный код.
Поэтому я объясню ключевые моменты и опущу другие менее важные части.

### Что делает bison?

Bison создает исходный файл C парсера из исходного файла bison.
Исходный файл bison — это текстовый файл.
Парсер анализирует исходный код программы в соответствии с его грамматикой.
Предположим, вот исходный файл turtle.

~~~
fc (1,0,0) # Foreground color is red, rgb = (1,0,0).
pd         # Pen down.
distance = 100
angle = 90
fd distance    # Go forward by distance (100) pixels.
tr angle     # Turn right by angle (90) degrees.
~~~

Парсер вызывает `yylex` для получения токена.
Токен состоит из его типа (тип токена) и значения (семантическое значение).
Итак, парсер получает элементы в следующей таблице при каждом вызове `yylex`.

|   |token kind|yylval.ID|yylval.NUM|
|:-:|:--------:|:-------:|:--------:|
| 1 |    FC    |         |          |
| 2 |    (     |         |          |
| 3 |   NUM    |         |   1.0    |
| 4 |    ,     |         |          |
| 5 |   NUM    |         |   0.0    |
| 6 |    ,     |         |          |
| 7 |   NUM    |         |   0.0    |
| 8 |    )     |         |          |
| 9 |    PD    |         |          |
|10 |    ID    |distance |          |
|11 |    =     |         |          |
|12 |   NUM    |         |  100.0   |
|13 |    ID    |  angle  |          |
|14 |    =     |         |          |
|15 |   NUM    |         |   90.0   |
|16 |    FD    |         |          |
|17 |    ID    |distance |          |
|18 |    TR    |         |          |
|19 |    ID    |  angle  |          |

Исходный код bison определяет грамматические правила языка turtle.
Например, `fc (1,0,0)` называется первичной процедурой.
Процедура похожа на функцию C типа void.
Она не возвращает никаких значений.
Программисты могут определять свои собственные процедуры.
С другой стороны, `fc` — это встроенная процедура.
Такие процедуры называются первичными процедурами.
Это описано в исходном коде bison следующим образом:

~~~
primary_procedure: FC '(' expression ',' expression ',' expression ')';
expression: ID | NUM;
~~~

Это означает:

- Первичная процедура — это FC, за которым следуют '(', expression, ',', expression, ',', expression и ')'.
- expression — это ID или NUM.

Описание выше называется BNF (форма Бэкуса-Наура).
Точнее говоря, это не совсем то же самое, что BNF.
Но разница невелика.

Первая строка:

~~~
FC '(' NUM ',' NUM ',' NUM ')';
~~~

Парсер анализирует исходный код turtle, и если ввод соответствует приведенному выше определению, парсер распознает его как первичную процедуру.

Грамматика turtle описана в [руководстве Turtle](https://toshiocp.github.io/Gtk4-tutorial/turtle_doc.html).
Ниже приведена выдержка из документа.

~~~
program:
  statement
| program statement
;

statement:
  primary_procedure
| procedure_definition
;

primary_procedure:
  PU
| PD
| PW expression
| FD expression
| TR expression
| TL expression
| BC '(' expression ',' expression ',' expression ')'
| FC '(' expression ',' expression ',' expression ')'
| ID '=' expression
| IF '(' expression ')' '{' primary_procedure_list '}'
| RT
| RS
| RP '(' expression ')' '{' primary_procedure_list '}'
| ID '(' ')'
| ID '(' argument_list ')'
;

procedure_definition:
  DP ID '('  ')' '{' primary_procedure_list '}'
| DP ID '(' parameter_list ')' '{' primary_procedure_list '}'
;

parameter_list:
  ID
| parameter_list ',' ID
;

argument_list:
  expression
| argument_list ',' expression
;

primary_procedure_list:
  primary_procedure
| primary_procedure_list primary_procedure
;

expression:
  expression '=' expression
| expression '>' expression
| expression '<' expression
| expression '+' expression
| expression '-' expression
| expression '*' expression
| expression '/' expression
| '-' expression %prec UMINUS
| '(' expression ')'
| ID
| NUM
;
~~~

Грамматическое правило определяет `program` первым.

- program — это statement или program, за которым следует statement.

Определение рекурсивно.

- `statement` — это program.
- `statement statement` — это `program statement`.
Следовательно, это program.
- `statement statement statement` — это `program statement`, потому что первые два statement являются `program`.
Следовательно, это program.

Вы можете обнаружить, что последовательность statements также является program.

Символы `program` и `statement` не являются токенами.
Они не появляются во вводе.
Они называются нетерминальными символами.
С другой стороны, токены называются терминальными символами.
Слово "токен", используемое здесь, имеет широкое значение, оно включает токены и символы, которые появляются во вводе.
Нетерминальные символы часто сокращаются до nterm.

Давайте проанализируем программу выше так, как это делает bison.

|   |token kind|yylval.ID|yylval.NUM|parse                               |S/R|
|:-:|:--------:|:-------:|:--------:|:-----------------------------------|:-:|
| 1 |    FC    |         |          |FC                                  | S |
| 2 |    (     |         |          |FC(                                 | S |
| 3 |   NUM    |         |   1.0    |FC(NUM                              | S |
|   |          |         |          |FC(expression                       | R |
| 4 |    ,     |         |          |FC(expression,                      | S |
| 5 |   NUM    |         |   0.0    |FC(expression,NUM                   | S |
|   |          |         |          |FC(expression,expression            | R |
| 6 |    ,     |         |          |FC(expression,expression,           | S |
| 7 |   NUM    |         |   0.0    |FC(expression,expression,NUM        | S |
|   |          |         |          |FC(expression,expression,expression | R |
| 8 |    )     |         |          |FC(expression,expression,expression)| S |
|   |          |         |          |primary_procedure                   | R |
|   |          |         |          |statement                           | R |
|   |          |         |          |program                             | R |
| 9 |    PD    |         |          |program PD                          | S |
|   |          |         |          |program primary_procedure           | R |
|   |          |         |          |program statement                   | R |
|   |          |         |          |program                             | R |
|10 |    ID    |distance |          |program ID                          | S |
|11 |    =     |         |          |program ID=                         | S |
|12 |   NUM    |         |  100.0   |program ID=NUM                      | S |
|   |          |         |          |program ID=expression               | R |
|   |          |         |          |program primary_procedure           | R |
|   |          |         |          |program statement                   | R |
|   |          |         |          |program                             | R |
|13 |    ID    |  angle  |          |program ID                          | S |
|14 |    =     |         |          |program ID=                         | S |
|15 |   NUM    |         |   90.0   |program ID=NUM                      | S |
|   |          |         |          |program ID=expression               | R |
|   |          |         |          |program primary_procedure           | R |
|   |          |         |          |program statement                   | R |
|   |          |         |          |program                             | R |
|16 |    FD    |         |          |program FD                          | S |
|17 |    ID    |distance |          |program FD ID                       | S |
|   |          |         |          |program FD expression               | R |
|   |          |         |          |program primary_procedure           | R |
|   |          |         |          |program statement                   | R |
|   |          |         |          |program                             | R |
|18 |    TR    |         |          |program TR                          | S |
|19 |    ID    |  angle  |          |program TR ID                       | S |
|   |          |         |          |program TR expression               | R |
|   |          |         |          |program primary_procedure           | R |
|   |          |         |          |program statement                   | R |
|   |          |         |          |program                             | R |

Самый правый столбец показывает сдвиг/свёртку.
Сдвиг — это добавление ввода в буфер.
Свёртка — это замена шаблона в буфере на нетерминал более высокого уровня.
Например, NUM заменяется на expression в четвертой строке.
Эта замена называется "свёртка".

Bison повторяет сдвиг и свёртку до конца ввода.
Если результат сворачивается к `program`, ввод синтаксически корректен.
Bison выполняет действие при каждой свёртке.
Действия строят дерево.
Дерево анализируется и выполняется процедурой выполнения позже.

Исходные файлы bison называются грамматическими файлами bison.
Грамматический файл bison состоит из четырех разделов: пролог, объявления, правила и эпилог.
Формат следующий:

~~~
%{
prologue
%}
declarations
%%
rules
%%
epilogue
~~~

### Пролог

Раздел пролога состоит из кода C, и код копируется в файл реализации парсера.
Вы можете использовать директивы `%code` для уточнения пролога и явного определения цели.
Ниже приведена выдержка из `turtle.y`.

@@@if gfm
~~~bison
@@@elif html
~~~{.bison}
@@@else
~~~
@@@end
%code top{
  #include <stdarg.h>
  #include <setjmp.h>
  #include <math.h>
  #include <glib.h>
  #include <cairo.h>
  #include "turtle_parser.h"

  /* The following line defines 'debug' so that debug information is printed out during the run time. */
  /* However it makes the program slow. */
  /* If you want to debug on, uncomment the line. */

  /* #define debug 1 */

  extern cairo_surface_t *surface;

  /* error reporting */
  static void yyerror (char const *s) { /* for syntax error */
    g_printerr ("%s from line %d, column %d to line %d, column %d\n",s, yylloc.first_line, yylloc.first_column, yylloc.last_line, yylloc.last_column);
  }
  /* Node type */
  enum {
    N_PU,
    N_PD,
    N_PW,
 ... ... ...
  };
}
~~~

Директива `%code top` копирует свое содержимое в начало файла реализации парсера.
Обычно она включает директивы `#include`, объявления функций и определения констант.
Функция `yyerror` сообщает о синтаксической ошибке и вызывается парсером.
Тип узла идентифицирует узел в дереве.

Другая директива `%code requires` копирует свое содержимое как в файл реализации парсера, так и в заголовочный файл.
Заголовочный файл читается исходным файлом C сканера и другими файлами.

@@@if gfm
```bison
%code requires {
  int yylex (void);
  int yyparse (void);
  void run (void);

  /* semantic value type */
  typedef struct _node_t node_t;
  struct _node_t {
    int type;
    union {
      struct {
        node_t *child1, *child2, *child3;
      } child;
      char *name;
      double value;
    } content;
  };
}
```
@@@elif html
```{.bison}
%code requires {
  int yylex (void);
  int yyparse (void);
  void run (void);

  /* semantic value type */
  typedef struct _node_t node_t;
  struct _node_t {
    int type;
    union {
      struct {
        node_t *child1, *child2, *child3;
      } child;
      char *name;
      double value;
    } content;
  };
}
```
@@@else
```
%code requires {
  int yylex (void);
  int yyparse (void);
  void run (void);

  /* semantic value type */
  typedef struct _node_t node_t;
  struct _node_t {
    int type;
    union {
      struct {
        node_t *child1, *child2, *child3;
      } child;
      char *name;
      double value;
    } content;
  };
}
```
@@@end

- `yylex` используется совместно файлом реализации парсера и файлом сканера.
- `yyparse` и `run` вызываются `run_cb` в `turtleapplication.c`.
- `node_t` — это тип семантического значения nterms.
Заголовочный файл определяет `YYSTYPE`, который является типом семантического значения, со всеми типами значений токенов и nterm.
Ниже приведена выдержка из заголовочного файла.

@@@if gfm
```C
/* Value type.  */
#if ! defined YYSTYPE && ! defined YYSTYPE_IS_DECLARED
union YYSTYPE
{
  char * ID;                               /* ID  */
  double NUM;                              /* NUM  */
  node_t * program;                        /* program  */
  node_t * statement;                      /* statement  */
  node_t * primary_procedure;              /* primary_procedure  */
  node_t * primary_procedure_list;         /* primary_procedure_list  */
  node_t * procedure_definition;           /* procedure_definition  */
  node_t * parameter_list;                 /* parameter_list  */
  node_t * argument_list;                  /* argument_list  */
  node_t * expression;                     /* expression  */
};
```
@@@else
```{.C}
/* Value type.  */
#if ! defined YYSTYPE && ! defined YYSTYPE_IS_DECLARED
union YYSTYPE
{
  char * ID;                               /* ID  */
  double NUM;                              /* NUM  */
  node_t * program;                        /* program  */
  node_t * statement;                      /* statement  */
  node_t * primary_procedure;              /* primary_procedure  */
  node_t * primary_procedure_list;         /* primary_procedure_list  */
  node_t * procedure_definition;           /* procedure_definition  */
  node_t * parameter_list;                 /* parameter_list  */
  node_t * argument_list;                  /* argument_list  */
  node_t * expression;                     /* expression  */
};
```
@@@end

Другие полезные макросы и объявления помещаются в директиву `%code`.

```
%code {
/* The following macro is convenient to get the member of the node. */
  #define child1(n) (n)->content.child.child1
  #define child2(n) (n)->content.child.child2
  #define child3(n) (n)->content.child.child3
  #define name(n) (n)->content.name
  #define value(n) (n)->content.value

  /* start of nodes */
  static node_t *node_top = NULL;
  /* functions to generate trees */
  static node_t *tree1 (int type, node_t *child1, node_t *child2, node_t *child3);
  static node_t *tree2 (int type, double value);
  static node_t *tree3 (int type, char *name);
}
```

### Объявления Bison

Объявления Bison определяют терминальные и нетерминальные символы.
Они также задают некоторые директивы.

```
%locations
%define api.value.type union /* YYSTYPE, the type of semantic values, is union of following types */
 /* key words */
%token PU
%token PD
%token PW
%token FD
%token TR
%token TL /* ver 0.5 */
%token BC
%token FC
%token DP
%token IF
%token RT
%token RS
%token RP /* ver 0.5 */
 /* constant */
%token <double> NUM
 /* identifier */
%token <char *> ID
 /* non terminal symbol */
%nterm <node_t *> program
%nterm <node_t *> statement
%nterm <node_t *> primary_procedure
%nterm <node_t *> primary_procedure_list
%nterm <node_t *> procedure_definition
%nterm <node_t *> parameter_list
%nterm <node_t *> argument_list
%nterm <node_t *> expression
 /* logical relation symbol */
%left '=' '<' '>'
 /* arithmetic symbol */
%left '+' '-'
%left '*' '/'
%precedence UMINUS /* unary minus */
```

Директива `%locations` вставляет структуру местоположения в заголовочный файл.
Она выглядит так.

@@@if gfm
```C
typedef struct YYLTYPE YYLTYPE;
struct YYLTYPE
{
  int first_line;
  int first_column;
  int last_line;
  int last_column;
};
```
@@@else
```{.C}
typedef struct YYLTYPE YYLTYPE;
struct YYLTYPE
{
  int first_line;
  int first_column;
  int last_line;
  int last_column;
};
```
@@@end

Этот тип используется совместно файлом сканера и файлом реализации парсера.
Функция сообщения об ошибках `yyerror` использует его, чтобы информировать о местоположении, где произошла ошибка.

`%define api.value.type union` генерирует тип семантического значения с токенами и nterms и вставляет его в заголовочный файл.
Вставленная часть показана в предыдущем подразделе как выдержки, показывающие тип значения (YYSTYPE).

Директивы `%token` и `%nterm` определяют токены и нетерминальные символы соответственно.

```
%token PU
... ...
%token <double> NUM
```

Эти директивы определяют токены `PU` и `NUM`.
Значения типов токенов `PU` и `NUM` определяются как перечислимая константа в заголовочном файле.

```
  enum yytokentype
  {
  ... ... ...
    PU = 258,                      /* PU  */
  ... ... ...
    NUM = 269,                     /* NUM  */
  ... ... ...
  };
  typedef enum yytokentype yytoken_kind_t;
```

Кроме того, тип семантического значения `NUM` определяется как double в заголовочном файле из-за тега `<double>`.

```
union YYSTYPE
{
  char * ID;                               /* ID  */
  double NUM;                              /* NUM  */
  ... ...
}
```

Все символы nterm имеют один и тот же тип `*node_t` семантического значения.

Директивы `%left` и `%precedence` определяют приоритет операционных символов.

```
 /* logical relation symbol */
%left '=' '<' '>'
 /* arithmetic symbol */
%left '+' '-'
%left '*' '/'
%precedence UMINUS /* unary minus */
```

Директива `%left` определяет следующие символы как левоассоциативные операторы.
Если оператор `+` левоассоциативный, то

```
A + B + C = (A + B) + C
```

То есть, вычисление выполняется сначала для левого оператора, затем для правого оператора.
Если оператор `*` правоассоциативный, то:

```
A * B * C = A * (B * C)
```

Приведенное выше определение определяет поведение парсера.
Сложение и умножение подчиняются ассоциативному закону, поэтому результат `(A+B)+C` и `A+(B+C)` равны с точки зрения математики.
Однако парсер будет сбит с толку, если не указана левая (или правая) ассоциативность.

Директивы `%left` и `%precedence` показывают приоритет операторов.
Операторы, объявленные позже, имеют более высокий приоритет, чем объявленные ранее.
Приведенное выше объявление говорит, например,

```
v=w+z*5+7 is the same as v=((w+(z*5))+7)
```

Будьте осторожны.
Оператор `=` выше — это присваивание.
Присваивание не является выражением в языке turtle.
Это primary_procedure.
Но если `=` появляется в выражении, это логический оператор, а не присваивание.
Логическое равенство '`=`' обычно используется в условном выражении, например, в операторе `if`.
(Язык Turtle использует '=' вместо '==' в языке C).

### Грамматические правила

Раздел грамматических правил определяет синтаксическую грамматику языка.
Она похожа на форму BNF.

~~~
result: components { action };
~~~

- result — это nterm.
- components — это список токенов или nterms.
- action — это код C. Он выполняется всякий раз, когда компоненты сворачиваются к результату.
Действие может быть опущено.

Ниже приведена часть грамматического правила в `turtle.y`.
Но она не совсем такая же.

~~~
program:
  statement { node_top = $$ = $1; }
;
statement:
  primary_procedure
;
primary_procedure:
  FD expression    { $$ = tree1 (N_FD, $2, NULL, NULL); }
;
expression:
  NUM   { $$ = tree2 (N_NUM, $1); }
;
~~~

- Первые две строки говорят, что `program` — это `statement`.
- Всякий раз, когда `statement` сворачивается к `program`, выполняется действие `node_top=$$=$1;`.
- `node_top` — это статическая переменная.
Она указывает на верхний узел дерева.
- Символ `$$` — это семантическое значение результата.
Например, `$$` в строке 2 — это семантическое значение `program`.
Это указатель на структуру типа `node_t`.
- Символ `$1` — это семантическое значение первого компонента.
Например, `$1` в строке 2 — это семантическое значение `statement`.
Это также указатель на `node_t`.
- Следующее правило заключается в том, что `statement` — это `primary_procedure`.
Действие не указано.
Тогда выполняется действие по умолчанию `$$ = $1`.
- Следующее правило заключается в том, что `primary_procedure` — это `FD`, за которым следует expression.
Действие вызывает `tree1` и присваивает его возвращаемое значение `$$`.
Функция `tree1` создает узел дерева.
Узел дерева имеет тип и объединение из трех указателей на дочерние узлы, строку или double.
~~~
node --+-- type
       +-- union contents
                    +---struct {node_t *child1, *child2, *child3;};
                    +---char *name
                    +---double value
~~~
- `tree1` присваивает четыре аргумента членам type, child1, child2 и child3.
- Последнее правило заключается в том, что `expression` — это `NUM`.
- `tree2` создает узел дерева. Параметры `tree2` — это тип и семантическое значение.

Предположим, парсер читает следующую программу.

~~~
fd 100
~~~

Что делает парсер?

1. Парсер распознает, что ввод — это `FD`.
Может быть, это начало `primary_procedure`, но парсеру нужно прочитать следующий токен.
2. `yylex` возвращает тип токена `NUM` и устанавливает `yylval.NUM` на 100.0 (тип — double). Парсер сворачивает `NUM` к `expression`.
Одновременно он устанавливает семантическое значение `expression` для указания на новый узел.
Узел имеет тип `N_NUM` и семантическое значение 100.0.
3. После свёртки в буфере есть `FD` и `expression`.
Парсер сворачивает его к `primary_procedure`.
И он устанавливает семантическое значение `primary_procedure` для указания на новый узел.
Узел имеет тип `N_FD`, и его член child1 указывает на узел `expression`, тип которого `N_NUM`.
4. Парсер сворачивает `primary_procedure` к `statement`.
Семантическое значение `statement` такое же, как у `primary_procedure`,
которое указывает на узел `N_FD`.
5. Парсер сворачивает `statement` к `program`.
Семантическое значение `statement` присваивается значению `program` и статической переменной `node_top`.
6. Наконец, `node_top` указывает на узел `N_FD`, а узел `N_FD` указывает на узел `N_NUM`.

![tree](../image/tree.png){width=6.51cm height=5.46cm}

Ниже приведено грамматическое правило, извлеченное из `turtle.y`.
Правила там основаны на той же идее, что и выше.
Я не хочу объяснять все правила ниже.
Пожалуйста, внимательно изучите каждую строку, чтобы понять все правила и действия.

@@@if gfm
```bison
program:
  statement { node_top = $$ = $1; }
| program statement {
        node_top = $$ = tree1 (N_program, $1, $2, NULL);
#ifdef debug
if (node_top == NULL) g_printerr ("program: node_top is NULL.\n"); else g_printerr ("program: node_top is NOT NULL.\n");
#endif
        }
;

statement:
  primary_procedure
| procedure_definition
;

primary_procedure:
  PU    { $$ = tree1 (N_PU, NULL, NULL, NULL); }
| PD    { $$ = tree1 (N_PD, NULL, NULL, NULL); }
| PW expression    { $$ = tree1 (N_PW, $2, NULL, NULL); }
| FD expression    { $$ = tree1 (N_FD, $2, NULL, NULL); }
| TR expression    { $$ = tree1 (N_TR, $2, NULL, NULL); }
| TL expression    { $$ = tree1 (N_TL, $2, NULL, NULL); } /* ver 0.5 */
| BC '(' expression ',' expression ',' expression ')' { $$ = tree1 (N_BC, $3, $5, $7); }
| FC '(' expression ',' expression ',' expression ')' { $$ = tree1 (N_FC, $3, $5, $7); }
 /* assignment */
| ID '=' expression   { $$ = tree1 (N_ASSIGN, tree3 (N_ID, $1), $3, NULL); }
 /* control flow */
| IF '(' expression ')' '{' primary_procedure_list '}' { $$ = tree1 (N_IF, $3, $6, NULL); }
| RT    { $$ = tree1 (N_RT, NULL, NULL, NULL); }
| RS    { $$ = tree1 (N_RS, NULL, NULL, NULL); }
| RP '(' expression ')' '{' primary_procedure_list '}'    { $$ = tree1 (N_RP, $3, $6, NULL); }
 /* user defined procedure call */
| ID '(' ')'  { $$ = tree1 (N_procedure_call, tree3 (N_ID, $1), NULL, NULL); }
| ID '(' argument_list ')'  { $$ = tree1 (N_procedure_call, tree3 (N_ID, $1), $3, NULL); }
;

procedure_definition:
  DP ID '('  ')' '{' primary_procedure_list '}'  {
         $$ = tree1 (N_procedure_definition, tree3 (N_ID, $2), NULL, $6);
        }
| DP ID '(' parameter_list ')' '{' primary_procedure_list '}'  {
         $$ = tree1 (N_procedure_definition, tree3 (N_ID, $2), $4, $7);
        }
;

parameter_list:
  ID { $$ = tree3 (N_ID, $1); }
| parameter_list ',' ID  { $$ = tree1 (N_parameter_list, $1, tree3 (N_ID, $3), NULL); }
;

argument_list:
  expression
| argument_list ',' expression { $$ = tree1 (N_argument_list, $1, $3, NULL); }
;

primary_procedure_list:
  primary_procedure
| primary_procedure_list primary_procedure {
         $$ = tree1 (N_primary_procedure_list, $1, $2, NULL);
        }
;

expression:
  expression '=' expression { $$ = tree1 (N_EQ, $1, $3, NULL); }
| expression '>' expression { $$ = tree1 (N_GT, $1, $3, NULL); }
| expression '<' expression { $$ = tree1 (N_LT, $1, $3, NULL); }
| expression '+' expression { $$ = tree1 (N_ADD, $1, $3, NULL); }
| expression '-' expression { $$ = tree1 (N_SUB, $1, $3, NULL); }
| expression '*' expression { $$ = tree1 (N_MUL, $1, $3, NULL); }
| expression '/' expression { $$ = tree1 (N_DIV, $1, $3, NULL); }
| '-' expression %prec UMINUS { $$ = tree1 (N_UMINUS, $2, NULL, NULL); }
| '(' expression ')' { $$ = $2; }
| ID    { $$ = tree3 (N_ID, $1); }
| NUM   { $$ = tree2 (N_NUM, $1); }
;
```
@@@else
```{.yacc}
program:
  statement { node_top = $$ = $1; }
| program statement {
        node_top = $$ = tree1 (N_program, $1, $2, NULL);
#ifdef debug
if (node_top == NULL) g_printerr ("program: node_top is NULL.\n"); else g_printerr ("program: node_top is NOT NULL.\n");
#endif
        }
;

statement:
  primary_procedure
| procedure_definition
;

primary_procedure:
  PU    { $$ = tree1 (N_PU, NULL, NULL, NULL); }
| PD    { $$ = tree1 (N_PD, NULL, NULL, NULL); }
| PW expression    { $$ = tree1 (N_PW, $2, NULL, NULL); }
| FD expression    { $$ = tree1 (N_FD, $2, NULL, NULL); }
| TR expression    { $$ = tree1 (N_TR, $2, NULL, NULL); }
| TL expression    { $$ = tree1 (N_TL, $2, NULL, NULL); } /* ver 0.5 */
| BC '(' expression ',' expression ',' expression ')' { $$ = tree1 (N_BC, $3, $5, $7); }
| FC '(' expression ',' expression ',' expression ')' { $$ = tree1 (N_FC, $3, $5, $7); }
 /* assignment */
| ID '=' expression   { $$ = tree1 (N_ASSIGN, tree3 (N_ID, $1), $3, NULL); }
 /* control flow */
| IF '(' expression ')' '{' primary_procedure_list '}' { $$ = tree1 (N_IF, $3, $6, NULL); }
| RT    { $$ = tree1 (N_RT, NULL, NULL, NULL); }
| RS    { $$ = tree1 (N_RS, NULL, NULL, NULL); }
| RP '(' expression ')' '{' primary_procedure_list '}'    { $$ = tree1 (N_RP, $3, $6, NULL); }
 /* user defined procedure call */
| ID '(' ')'  { $$ = tree1 (N_procedure_call, tree3 (N_ID, $1), NULL, NULL); }
| ID '(' argument_list ')'  { $$ = tree1 (N_procedure_call, tree3 (N_ID, $1), $3, NULL); }
;

procedure_definition:
  DP ID '('  ')' '{' primary_procedure_list '}'  {
         $$ = tree1 (N_procedure_definition, tree3 (N_ID, $2), NULL, $6);
        }
| DP ID '(' parameter_list ')' '{' primary_procedure_list '}'  {
         $$ = tree1 (N_procedure_definition, tree3 (N_ID, $2), $4, $7);
        }
;

parameter_list:
  ID { $$ = tree3 (N_ID, $1); }
| parameter_list ',' ID  { $$ = tree1 (N_parameter_list, $1, tree3 (N_ID, $3), NULL); }
;

argument_list:
  expression
| argument_list ',' expression { $$ = tree1 (N_argument_list, $1, $3, NULL); }
;

primary_procedure_list:
  primary_procedure
| primary_procedure_list primary_procedure {
         $$ = tree1 (N_primary_procedure_list, $1, $2, NULL);
        }
;

expression:
  expression '=' expression { $$ = tree1 (N_EQ, $1, $3, NULL); }
| expression '>' expression { $$ = tree1 (N_GT, $1, $3, NULL); }
| expression '<' expression { $$ = tree1 (N_LT, $1, $3, NULL); }
| expression '+' expression { $$ = tree1 (N_ADD, $1, $3, NULL); }
| expression '-' expression { $$ = tree1 (N_SUB, $1, $3, NULL); }
| expression '*' expression { $$ = tree1 (N_MUL, $1, $3, NULL); }
| expression '/' expression { $$ = tree1 (N_DIV, $1, $3, NULL); }
| '-' expression %prec UMINUS { $$ = tree1 (N_UMINUS, $2, NULL, NULL); }
| '(' expression ')' { $$ = $2; }
| ID    { $$ = tree3 (N_ID, $1); }
| NUM   { $$ = tree2 (N_NUM, $1); }
;
```
@@@end

### Эпилог

Эпилог написан на языке C и копируется в файл реализации парсера.
Обычно вы можете поместить что угодно в эпилог.
В случае интерпретатора turtle процедура выполнения и некоторые другие функции находятся в эпилоге.

#### Функции для создания узлов дерева

Есть три функции: `tree1`, `tree2` и `tree3`.

- `tree1` создает узел и устанавливает тип узла и указатели на его трех потомков (возможно NULL).
- `tree2` создает узел и устанавливает тип узла и значение (double).
- `tree3` создает узел и устанавливает тип узла и указатель на строку.

Каждая функция сначала получает память и строит на ней узел.
Памяти вставляются в список.
Они будут освобождены, когда процедура выполнения завершится.

Эти три функции вызываются в действиях в разделе правил.

@@@if gfm
```C
/* Dynamically allocated memories are added to the single list. They will be freed in the finalize function. */
GSList *list = NULL;

node_t *
tree1 (int type, node_t *child1, node_t *child2, node_t *child3) {
  node_t *new_node;

  list = g_slist_prepend (list, g_malloc (sizeof (node_t)));
  new_node = (node_t *) list->data;
  new_node->type = type;
  child1(new_node) = child1;
  child2(new_node) = child2;
  child3(new_node) = child3;
  return new_node;
}

node_t *
tree2 (int type, double value) {
  node_t *new_node;

  list = g_slist_prepend (list, g_malloc (sizeof (node_t)));
  new_node = (node_t *) list->data;
  new_node->type = type;
  value(new_node) = value;
  return new_node;
}

node_t *
tree3 (int type, char *name) {
  node_t *new_node;

  list = g_slist_prepend (list, g_malloc (sizeof (node_t)));
  new_node = (node_t *) list->data;
  new_node->type = type;
  name(new_node) = name;
  return new_node;
}
```
@@@else
```{.C}
/* Dynamically allocated memories are added to the single list. They will be freed in the finalize function. */
GSList *list = NULL;

node_t *
tree1 (int type, node_t *child1, node_t *child2, node_t *child3) {
  node_t *new_node;

  list = g_slist_prepend (list, g_malloc (sizeof (node_t)));
  new_node = (node_t *) list->data;
  new_node->type = type;
  child1(new_node) = child1;
  child2(new_node) = child2;
  child3(new_node) = child3;
  return new_node;
}

node_t *
tree2 (int type, double value) {
  node_t *new_node;

  list = g_slist_prepend (list, g_malloc (sizeof (node_t)));
  new_node = (node_t *) list->data;
  new_node->type = type;
  value(new_node) = value;
  return new_node;
}

node_t *
tree3 (int type, char *name) {
  node_t *new_node;

  list = g_slist_prepend (list, g_malloc (sizeof (node_t)));
  new_node = (node_t *) list->data;
  new_node->type = type;
  name(new_node) = name;
  return new_node;
}
```
@@@end

#### Таблица символов

Переменные и пользовательские процедуры регистрируются в таблице символов.
Эта таблица является массивом C.
В будущей версии она должна быть заменена лучшим алгоритмом и структурой данных, например, хешем.

- Переменные регистрируются с их именем и значением.
- Процедуры регистрируются с их именем и указателем на узел процедуры.

Следовательно, таблица имеет следующие поля.

- тип для идентификации переменной или процедуры
- имя
- значение или указатель на узел

@@@if gfm
```C
#define MAX_TABLE_SIZE 100
enum {
  PROC,
  VAR
};

struct {
  int type;
  char *name;
  union {
    node_t *node;
    double value;
  } object;
} table[MAX_TABLE_SIZE];
int tp;

void
init_table (void) {
  tp = 0;
}
```
@@@else
```{.C}
#define MAX_TABLE_SIZE 100
enum {
  PROC,
  VAR
};

struct {
  int type;
  char *name;
  union {
    node_t *node;
    double value;
  } object;
} table[MAX_TABLE_SIZE];
int tp;

void
init_table (void) {
  tp = 0;
}
```
@@@end

Функция `init_table` инициализирует таблицу.
Она должна быть вызвана перед регистрациями.

Есть пять функций для доступа к таблице,

- `proc_install` устанавливает процедуру.
- `var_install` устанавливает переменную.
- `proc_lookup` ищет процедуру. Если процедура найдена, она возвращает указатель на узел. В противном случае возвращает NULL.
- `var_lookup` ищет переменную. Если переменная найдена, она возвращает TRUE и устанавливает указатель (аргумент) для указания на значение. В противном случае возвращает FALSE.
- `var_replace` заменяет значение переменной. Если переменная еще не зарегистрирована, она устанавливает переменную.

@@@if gfm
```C
int
tbl_lookup (int type, char *name) {
  int i;

  if (tp == 0)
    return -1;
  for (i=0; i<tp; ++i)
    if (type == table[i].type && strcmp(name, table[i].name) == 0)
      return i;
  return -1;
}

void
tbl_install (int type, char *name, node_t *node, double value) {
  if (tp >= MAX_TABLE_SIZE)
    runtime_error ("Symbol table overflow.\n");
  else if (tbl_lookup (type, name) >= 0)
    runtime_error ("Name %s is already registered.\n", name);
  else {
    table[tp].type = type;
    table[tp].name = name;
    if (type == PROC)
      table[tp++].object.node = node;
    else
      table[tp++].object.value = value;
  }
}

void
proc_install (char *name, node_t *node) {
  tbl_install (PROC, name, node, 0.0);
}

void
var_install (char *name, double value) {
  tbl_install (VAR, name, NULL, value);
}

void
var_replace (char *name, double value) {
  int i;
  if ((i = tbl_lookup (VAR, name)) >= 0)
    table[i].object.value = value;
  else
    var_install (name, value);
}

node_t *
proc_lookup (char *name) {
  int i;
  if ((i = tbl_lookup (PROC, name)) < 0)
    return NULL;
  else
    return table[i].object.node;
}

gboolean
var_lookup (char *name, double *value) {
  int i;
  if ((i = tbl_lookup (VAR, name)) < 0)
    return FALSE;
  else {
    *value = table[i].object.value;
    return TRUE;
  }
}
```
@@@else
```{.C}
int
tbl_lookup (int type, char *name) {
  int i;

  if (tp == 0)
    return -1;
  for (i=0; i<tp; ++i)
    if (type == table[i].type && strcmp(name, table[i].name) == 0)
      return i;
  return -1;
}

void
tbl_install (int type, char *name, node_t *node, double value) {
  if (tp >= MAX_TABLE_SIZE)
    runtime_error ("Symbol table overflow.\n");
  else if (tbl_lookup (type, name) >= 0)
    runtime_error ("Name %s is already registered.\n", name);
  else {
    table[tp].type = type;
    table[tp].name = name;
    if (type == PROC)
      table[tp++].object.node = node;
    else
      table[tp++].object.value = value;
  }
}

void
proc_install (char *name, node_t *node) {
  tbl_install (PROC, name, node, 0.0);
}

void
var_install (char *name, double value) {
  tbl_install (VAR, name, NULL, value);
}

void
var_replace (char *name, double value) {
  int i;
  if ((i = tbl_lookup (VAR, name)) >= 0)
    table[i].object.value = value;
  else
    var_install (name, value);
}

node_t *
proc_lookup (char *name) {
  int i;
  if ((i = tbl_lookup (PROC, name)) < 0)
    return NULL;
  else
    return table[i].object.node;
}

gboolean
var_lookup (char *name, double *value) {
  int i;
  if ((i = tbl_lookup (VAR, name)) < 0)
    return FALSE;
  else {
    *value = table[i].object.value;
    return TRUE;
  }
}
```
@@@end

#### Стек для параметров и аргументов

Стек — это структура данных "последним пришел — первым вышел".
Сокращенно LIFO.
Turtle использует стек для хранения параметров и аргументов.
Они подобны переменным класса `auto` в языке C.
Они помещаются в стек всякий раз, когда вызывается процедура.
Структура LIFO полезна для рекурсивных вызовов.

Каждый элемент стека имеет имя и значение.

~~~C
#define MAX_STACK_SIZE 500
struct {
  char *name;
  double value;
} stack[MAX_STACK_SIZE];
int sp, sp_biggest;

void
init_stack (void) {
  sp = sp_biggest = 0;
}
~~~

`sp` — это указатель стека.
Это индекс массива `stack`, и он всегда указывает на элемент массива для сохранения следующих данных.
`sp_biggest` — это наибольшее число, присвоенное `sp`.
Мы можем узнать количество элементов, использованных в массиве во время выполнения.
Цель переменной — найти подходящий `MAX_STACK_SIZE`.
Она будет не нужна в будущей версии, если стек будет реализован с лучшей структурой данных и выделением памяти.

Процедура выполнения помещает данные в стек при выполнении узла вызова процедуры.
(Тип узла — `N_procedure_call`.)

~~~
dp drawline (angle, distance) { ... ... ... }
drawline (90, 100)
~~~

- Первая строка определяет процедуру `drawline`.
Процедура выполнения сохраняет имя `drawline` и узел процедуры в таблице символов.
- Вторая строка вызывает процедуру.
Сначала она ищет процедуру в таблице символов и получает ее узел.
Затем она ищет узел для параметров и получает `angle` и `distance`.
- Она помещает ("angle", 90.0) в стек.
- Она помещает ("distance", 100.0) в стек.
- Она помещает (NULL, 2.0) в стек.
Число 2.0 — это количество параметров (или аргументов).
Оно используется при возврате процедуры.

Следующая диаграмма показывает структуру стека.
Сначала вызывается `procedure 1`.
Процедура имеет два параметра.
В `procedure 1` вызывается другая процедура `procedure 2`.
У нее один параметр.
В `procedure 2` вызывается другая процедура `procedure 3`.
У нее три параметра.
Эти три процедуры вложены.

Программы помещают данные в стек от младшего адреса памяти к старшему адресу памяти.
На следующей диаграмме самый младший адрес находится вверху, а самый старший адрес — внизу.
Это порядок адресов.
Однако "вершина стека" — это последние помещенные данные, а "дно стека" — это первые помещенные данные.
Следовательно, "вершина стека" находится внизу прямоугольника на диаграмме, а "дно стека" — вверху прямоугольника.

![Stack](../image/stack.png){width=6.64cm height=8.05cm}

Есть четыре функции для доступа к стеку.

- `stack_push` помещает данные в стек.
- `stack_lookup` ищет в стеке переменную по ее имени, переданному в качестве аргумента.
Она ищет только параметры последней процедуры.
Она возвращает TRUE и устанавливает аргумент `value` для указания на значение, если переменная была найдена.
В противном случае возвращает FALSE.
- `stack_replace` заменяет значение переменной в стеке.
Если она успешна, она возвращает TRUE. В противном случае возвращает FALSE.
- `stack_return` отбрасывает последние параметры.
Указатель стека возвращается к точке перед последним вызовом процедуры, чтобы он указывал на параметры ранее вызванной процедуры.

@@@if gfm
```C
void
stack_push (char *name, double value) {
  if (sp >= MAX_STACK_SIZE)
    runtime_error ("Stack overflow.\n");
  else {
    stack[sp].name = name;
    stack[sp++].value = value;
    sp_biggest = sp > sp_biggest ? sp : sp_biggest;
  }
}

int
stack_search (char *name) {
  int depth, i;

  if (sp == 0)
    return -1;
  depth = (int) stack[sp-1].value;
  if (depth + 1 > sp) /* something strange */
    runtime_error ("Stack error.\n");
  for (i=0; i<depth; ++i)
    if (strcmp(name, stack[sp-(i+2)].name) == 0) {
      return sp-(i+2);
    }
  return -1;
}

gboolean
stack_lookup (char *name, double *value) {
  int i;

  if ((i = stack_search (name)) < 0)
    return FALSE;
  else {
    *value = stack[i].value;
    return TRUE;
  }
}

gboolean
stack_replace (char *name, double value) {
  int i;

  if ((i = stack_search (name)) < 0)
    return FALSE;
  else {
    stack[i].value = value;
    return TRUE;
  }
}

void
stack_return(void) {
  int depth;

  if (sp <= 0)
    return;
  depth = (int) stack[sp-1].value;
  if (depth + 1 > sp) /* something strange */
    runtime_error ("Stack error.\n");
  sp -= depth + 1;
}
```
@@@else
```C
void
stack_push (char *name, double value) {
  if (sp >= MAX_STACK_SIZE)
    runtime_error ("Stack overflow.\n");
  else {
    stack[sp].name = name;
    stack[sp++].value = value;
    sp_biggest = sp > sp_biggest ? sp : sp_biggest;
  }
}

int
stack_search (char *name) {
  int depth, i;

  if (sp == 0)
    return -1;
  depth = (int) stack[sp-1].value;
  if (depth + 1 > sp) /* something strange */
    runtime_error ("Stack error.\n");
  for (i=0; i<depth; ++i)
    if (strcmp(name, stack[sp-(i+2)].name) == 0) {
      return sp-(i+2);
    }
  return -1;
}

gboolean
stack_lookup (char *name, double *value) {
  int i;

  if ((i = stack_search (name)) < 0)
    return FALSE;
  else {
    *value = stack[i].value;
    return TRUE;
  }
}

gboolean
stack_replace (char *name, double value) {
  int i;

  if ((i = stack_search (name)) < 0)
    return FALSE;
  else {
    stack[i].value = value;
    return TRUE;
  }
}

void
stack_return(void) {
  int depth;

  if (sp <= 0)
    return;
  depth = (int) stack[sp-1].value;
  if (depth + 1 > sp) /* something strange */
    runtime_error ("Stack error.\n");
  sp -= depth + 1;
}
```
@@@end

#### Поверхность и cairo

Глобальная переменная `surface` используется совместно `turtleapplication.c` и `turtle.y`.
Она инициализируется в `turtleapplication.c`.

Процедура выполнения имеет свой собственный контекст cairo.
Он отличается от cairo в экземпляре GtkDrawingArea.
Процедура выполнения рисует фигуру на `surface` с контекстом cairo.
После того, как процедура выполнения возвращается к `run_cb`, `run_cb` добавляет виджет GtkDrawingArea в очередь для перерисовки.
Когда виджет перерисовывается, вызывается функция рисования `draw_func`.
Она копирует `surface` на поверхность в объекте GtkDrawingArea.

`turtle.y` имеет две функции `init_cairo` и `destroy_cairo`.

- `init_cairo` инициализирует статические переменные и контекст cairo.
Переменные хранят состояние пера (вверх или вниз), направление, начальное местоположение, ширину линии и цвет.
Размер `surface` изменяется в соответствии с размером окна.
Всякий раз, когда пользователь перетаскивает и изменяет размер окна, `surface` также изменяет размер.
`init_cairo` сначала получает размер и устанавливает начальное местоположение черепашки (центр поверхности) и матрицу преобразования.
- `destroy_cairo` просто уничтожает контекст cairo.

Turtle имеет свою собственную систему координат.
Начало координат находится в центре поверхности, а положительное направление осей x и y — вправо и вверх соответственно.
Но поверхности имеют свою собственную систему координат.
Ее начало координат находится в верхнем левом углу поверхности, а положительное направление x и y — вправо и вниз соответственно.
Плоскость с системой координат черепашки называется пользовательским пространством, что совпадает с пользовательским пространством cairo.
Плоскость с системой координат поверхности называется пространством устройства.

Cairo обеспечивает преобразование, которое является аффинным преобразованием.
Оно преобразует координату пользовательского пространства (x, y) в координату пространства устройства (z, w).

![transformation](../image/transformation.png){width=6.3cm height=2.04cm}

Функция `init_cairo` получает ширину и высоту `surface` (см. программу ниже).

- Центр поверхности — это (0,0) относительно координаты пользовательского пространства и (width/2, height/2) относительно координаты пространства устройства.
- Положительное направление оси x в двух пространствах одинаково. Итак, (1,0) преобразуется в (1+width/2,height/2).
- Положительное направление оси y в двух пространствах противоположно. Итак, (0,1) преобразуется в (width/2,-1+height/2).

Вы можете определить a, b, c, d, p и q, подставив числа выше вместо x, y, z и w в уравнение выше.
Решение системы уравнений:

~~~
a = 1, b = 0, c = 0, d = -1, p = width/2, q = height/2
~~~

Cairo предоставляет структуру `cairo_matrix_t`.
Функция `init_cairo` использует ее и устанавливает преобразование cairo (см. программу ниже).
После установки матрицы преобразование всегда выполняется при вызове функции `cairo_stroke`.

@@@if gfm
```C
/* status of the surface */
static gboolean pen = TRUE;
static double angle = 90.0; /* angle starts from x axis and measured counterclockwise */
                   /* Initially facing to the north */
static double cur_x = 0.0;
static double cur_y = 0.0;
static double line_width = 2.0;

struct color {
  double red;
  double green;
  double blue;
};
static struct color bc = {0.95, 0.95, 0.95}; /* white */
static struct color fc = {0.0, 0.0, 0.0}; /* black */

/* cairo */
static cairo_t *cr;
gboolean
init_cairo (void) {
  int width, height;
  cairo_matrix_t matrix;

  pen = TRUE;
  angle = 90.0;
  cur_x = 0.0;
  cur_y = 0.0;
  line_width = 2.0;
  bc.red = 0.95; bc.green = 0.95; bc.blue = 0.95;
  fc.red = 0.0; fc.green = 0.0; fc.blue = 0.0;

  if (surface) {
    width = cairo_image_surface_get_width (surface);
    height = cairo_image_surface_get_height (surface);
    matrix.xx = 1.0; matrix.xy = 0.0; matrix.x0 = (double) width / 2.0;
    matrix.yx = 0.0; matrix.yy = -1.0; matrix.y0 = (double) height / 2.0;

    cr = cairo_create (surface);
    cairo_transform (cr, &matrix);
    cairo_set_source_rgb (cr, bc.red, bc.green, bc.blue);
    cairo_paint (cr);
    cairo_set_source_rgb (cr, fc.red, fc.green, fc.blue);
    cairo_move_to (cr, cur_x, cur_y);
    return TRUE;
  } else
    return FALSE;
}

void
destroy_cairo () {
  cairo_destroy (cr);
}
```
@@@else
```{.C}
/* status of the surface */
static gboolean pen = TRUE;
static double angle = 90.0; /* angle starts from x axis and measured counterclockwise */
                   /* Initially facing to the north */
static double cur_x = 0.0;
static double cur_y = 0.0;
static double line_width = 2.0;

struct color {
  double red;
  double green;
  double blue;
};
static struct color bc = {0.95, 0.95, 0.95}; /* white */
static struct color fc = {0.0, 0.0, 0.0}; /* black */

/* cairo */
static cairo_t *cr;
gboolean
init_cairo (void) {
  int width, height;
  cairo_matrix_t matrix;

  pen = TRUE;
  angle = 90.0;
  cur_x = 0.0;
  cur_y = 0.0;
  line_width = 2.0;
  bc.red = 0.95; bc.green = 0.95; bc.blue = 0.95;
  fc.red = 0.0; fc.green = 0.0; fc.blue = 0.0;

  if (surface) {
    width = cairo_image_surface_get_width (surface);
    height = cairo_image_surface_get_height (surface);
    matrix.xx = 1.0; matrix.xy = 0.0; matrix.x0 = (double) width / 2.0;
    matrix.yx = 0.0; matrix.yy = -1.0; matrix.y0 = (double) height / 2.0;

    cr = cairo_create (surface);
    cairo_transform (cr, &matrix);
    cairo_set_source_rgb (cr, bc.red, bc.green, bc.blue);
    cairo_paint (cr);
    cairo_set_source_rgb (cr, fc.red, fc.green, fc.blue);
    cairo_move_to (cr, cur_x, cur_y);
    return TRUE;
  } else
    return FALSE;
}

void
destroy_cairo () {
  cairo_destroy (cr);
}
```
@@@end

#### Функция Eval

Функция `eval` вычисляет выражение и возвращает значение выражения.
Она вызывает себя рекурсивно.
Например, если узел — `N_ADD`, то:

1. Вызывает eval(child1(node)) и получает value1.
2. Вызывает eval(child2(node)) и получает value2.
3. Возвращает value1+value2.

Это выполняется макросом `calc`, определенным в шестой строке следующей программы.

@@@if gfm
```C
double
eval (node_t *node) {
double value = 0.0;
  if (node == NULL)
    runtime_error ("No expression to evaluate.\n");
#define calc(op) eval (child1(node)) op eval (child2(node))
  switch (node->type) {
    case N_EQ:
      value = (double) calc(==);
      break;
    case N_GT:
      value = (double) calc(>);
      break;
    case N_LT:
      value = (double) calc(<);
      break;
    case N_ADD:
      value = calc(+);
      break;
    case N_SUB:
      value = calc(-);
      break;
    case N_MUL:
      value = calc(*);
      break;
    case N_DIV:
      if (eval (child2(node)) == 0.0)
        runtime_error ("Division by zerp.\n");
      else
        value = calc(/);
      break;
    case N_UMINUS:
      value = -(eval (child1(node)));
      break;
    case N_ID:
      if (! (stack_lookup (name(node), &value)) && ! var_lookup (name(node), &value) )
        runtime_error ("Variable %s not defined.\n", name(node));
      break;
    case N_NUM:
      value = value(node);
      break;
    default:
      runtime_error ("Illegal expression.\n");
  }
  return value;
}
```
@@@else
```{.C}
double
eval (node_t *node) {
double value = 0.0;
  if (node == NULL)
    runtime_error ("No expression to evaluate.\n");
#define calc(op) eval (child1(node)) op eval (child2(node))
  switch (node->type) {
    case N_EQ:
      value = (double) calc(==);
      break;
    case N_GT:
      value = (double) calc(>);
      break;
    case N_LT:
      value = (double) calc(<);
      break;
    case N_ADD:
      value = calc(+);
      break;
    case N_SUB:
      value = calc(-);
      break;
    case N_MUL:
      value = calc(*);
      break;
    case N_DIV:
      if (eval (child2(node)) == 0.0)
        runtime_error ("Division by zerp.\n");
      else
        value = calc(/);
      break;
    case N_UMINUS:
      value = -(eval (child1(node)));
      break;
    case N_ID:
      if (! (stack_lookup (name(node), &value)) && ! var_lookup (name(node), &value) )
        runtime_error ("Variable %s not defined.\n", name(node));
      break;
    case N_NUM:
      value = value(node);
      break;
    default:
      runtime_error ("Illegal expression.\n");
  }
  return value;
}
```
@@@end

#### Функция Execute

Первичные процедуры и определения процедур анализируются и выполняются функцией `execute`.
Она не возвращает никаких значений.
Она вызывает себя рекурсивно.
Обработка `N_RT` и `N_procedure_call` сложна.
Она будет объяснена после следующей программы.
Другие части не так сложны.
Внимательно прочитайте программу ниже, чтобы понять процесс.

@@@if gfm
```C
/* procedure - return status */
static int proc_level = 0;
static int ret_level = 0;

void
execute (node_t *node) {
  double d, x, y;
  char *name;
  int n, i;

  if (node == NULL)
    runtime_error ("Node is NULL.\n");
  if (proc_level > ret_level)
    return;
  switch (node->type) {
    case N_program:
      execute (child1(node));
      execute (child2(node));
      break;
    case N_PU:
      pen = FALSE;
      break;
    case N_PD:
      pen = TRUE;
      break;
    case N_PW:
      line_width = eval (child1(node)); /* line width */
      break;
    case N_FD:
      d = eval (child1(node)); /* distance */
      x = d * cos (angle*M_PI/180);
      y = d * sin (angle*M_PI/180);
      /* initialize the current point = start point of the line */
      cairo_move_to (cr, cur_x, cur_y);
      cur_x += x;
      cur_y += y;
      cairo_set_line_width (cr, line_width);
      cairo_set_source_rgb (cr, fc.red, fc.green, fc.blue);
      if (pen)
        cairo_line_to (cr, cur_x, cur_y);
      else
        cairo_move_to (cr, cur_x, cur_y);
      cairo_stroke (cr);
      break;
    case N_TR:
      angle -= eval (child1(node));
      for (; angle < 0; angle += 360.0);
      for (; angle>360; angle -= 360.0);
      break;
    case N_BC:
      bc.red = eval (child1(node));
      bc.green = eval (child2(node));
      bc.blue = eval (child3(node));
#define fixcolor(c)  c = c < 0 ? 0 : (c > 1 ? 1 : c)
      fixcolor (bc.red);
      fixcolor (bc.green);
      fixcolor (bc.blue);
      /* clear the shapes and set the background color */
      cairo_set_source_rgb (cr, bc.red, bc.green, bc.blue);
      cairo_paint (cr);
      break;
    case N_FC:
      fc.red = eval (child1(node));
      fc.green = eval (child2(node));
      fc.blue = eval (child3(node));
      fixcolor (fc.red);
      fixcolor (fc.green);
      fixcolor (fc.blue);
      break;
    case N_ASSIGN:
      name = name(child1(node));
      d = eval (child2(node));
      if (! stack_replace (name, d)) /* First, tries to replace the value in the stack (parameter).*/
        var_replace (name, d); /* If the above fails, tries to replace the value in the table. If the variable isn't in the table, installs it, */
      break;
    case N_IF:
      if (eval (child1(node)))
        execute (child2(node));
      break;
    case N_RT:
      ret_level--;
      break;
    case N_RS:
      pen = TRUE;
      angle = 90.0;
      cur_x = 0.0;
      cur_y = 0.0;
      line_width = 2.0;
      fc.red = 0.0; fc.green = 0.0; fc.blue = 0.0;
      /* To change background color, use bc. */
      break;
    case N_procedure_call:
      name = name(child1(node));
node_t *proc = proc_lookup (name);
      if (! proc)
        runtime_error ("Procedure %s not defined.\n", name);
      if (strcmp (name, name(child1(proc))) != 0)
        runtime_error ("Unexpected error. Procedure %s is called, but invoked procedure is %s.\n", name, name(child1(proc)));
/* make tuples (parameter (name), argument (value)) and push them to the stack */
node_t *param_list;
node_t *arg_list;
      param_list = child2(proc);
      arg_list = child2(node);
      if (param_list == NULL) {
        if (arg_list == NULL) {
          stack_push (NULL, 0.0); /* number of argument == 0 */
        } else
          runtime_error ("Procedure %s has different number of argument and parameter.\n", name);
      }else {
/* Don't change the stack until finish evaluating the arguments. */
#define TEMP_STACK_SIZE 20
        char *temp_param[TEMP_STACK_SIZE];
        double temp_arg[TEMP_STACK_SIZE];
        n = 0;
        for (; param_list->type == N_parameter_list; param_list = child1(param_list)) {
          if (arg_list->type != N_argument_list)
            runtime_error ("Procedure %s has different number of argument and parameter.\n", name);
          if (n >= TEMP_STACK_SIZE)
            runtime_error ("Too many parameters. the number must be %d or less.\n", TEMP_STACK_SIZE);
          temp_param[n] = name(child2(param_list));
          temp_arg[n] = eval (child2(arg_list));
          arg_list = child1(arg_list);
          ++n;
        }
        if (param_list->type == N_ID && arg_list -> type != N_argument_list) {
          temp_param[n] = name(param_list);
          temp_arg[n] = eval (arg_list);
          if (++n >= TEMP_STACK_SIZE)
            runtime_error ("Too many parameters. the number must be %d or less.\n", TEMP_STACK_SIZE);
          temp_param[n] = NULL;
          temp_arg[n] = (double) n;
          ++n;
        } else
          runtime_error ("Unexpected error.\n");
        for (i = 0; i < n; ++i)
          stack_push (temp_param[i], temp_arg[i]);
      }
      ret_level = ++proc_level;
      execute (child3(proc));
      ret_level = --proc_level;
      stack_return ();
      break;
    case N_procedure_definition:
      name = name(child1(node));
      proc_install (name, node);
      break;
    case N_primary_procedure_list:
      execute (child1(node));
      execute (child2(node));
      break;
    default:
      runtime_error ("Unknown statement.\n");
  }
}
```
@@@else
```{.C}
/* procedure - return status */
static int proc_level = 0;
static int ret_level = 0;

void
execute (node_t *node) {
  double d, x, y;
  char *name;
  int n, i;

  if (node == NULL)
    runtime_error ("Node is NULL.\n");
  if (proc_level > ret_level)
    return;
  switch (node->type) {
    case N_program:
      execute (child1(node));
      execute (child2(node));
      break;
    case N_PU:
      pen = FALSE;
      break;
    case N_PD:
      pen = TRUE;
      break;
    case N_PW:
      line_width = eval (child1(node)); /* line width */
      break;
    case N_FD:
      d = eval (child1(node)); /* distance */
      x = d * cos (angle*M_PI/180);
      y = d * sin (angle*M_PI/180);
      /* initialize the current point = start point of the line */
      cairo_move_to (cr, cur_x, cur_y);
      cur_x += x;
      cur_y += y;
      cairo_set_line_width (cr, line_width);
      cairo_set_source_rgb (cr, fc.red, fc.green, fc.blue);
      if (pen)
        cairo_line_to (cr, cur_x, cur_y);
      else
        cairo_move_to (cr, cur_x, cur_y);
      cairo_stroke (cr);
      break;
    case N_TR:
      angle -= eval (child1(node));
      for (; angle < 0; angle += 360.0);
      for (; angle>360; angle -= 360.0);
      break;
    case N_BC:
      bc.red = eval (child1(node));
      bc.green = eval (child2(node));
      bc.blue = eval (child3(node));
#define fixcolor(c)  c = c < 0 ? 0 : (c > 1 ? 1 : c)
      fixcolor (bc.red);
      fixcolor (bc.green);
      fixcolor (bc.blue);
      /* clear the shapes and set the background color */
      cairo_set_source_rgb (cr, bc.red, bc.green, bc.blue);
      cairo_paint (cr);
      break;
    case N_FC:
      fc.red = eval (child1(node));
      fc.green = eval (child2(node));
      fc.blue = eval (child3(node));
      fixcolor (fc.red);
      fixcolor (fc.green);
      fixcolor (fc.blue);
      break;
    case N_ASSIGN:
      name = name(child1(node));
      d = eval (child2(node));
      if (! stack_replace (name, d)) /* First, tries to replace the value in the stack (parameter).*/
        var_replace (name, d); /* If the above fails, tries to replace the value in the table. If the variable isn't in the table, installs it, */
      break;
    case N_IF:
      if (eval (child1(node)))
        execute (child2(node));
      break;
    case N_RT:
      ret_level--;
      break;
    case N_RS:
      pen = TRUE;
      angle = 90.0;
      cur_x = 0.0;
      cur_y = 0.0;
      line_width = 2.0;
      fc.red = 0.0; fc.green = 0.0; fc.blue = 0.0;
      /* To change background color, use bc. */
      break;
    case N_procedure_call:
      name = name(child1(node));
node_t *proc = proc_lookup (name);
      if (! proc)
        runtime_error ("Procedure %s not defined.\n", name);
      if (strcmp (name, name(child1(proc))) != 0)
        runtime_error ("Unexpected error. Procedure %s is called, but invoked procedure is %s.\n", name, name(child1(proc)));
/* make tuples (parameter (name), argument (value)) and push them to the stack */
node_t *param_list;
node_t *arg_list;
      param_list = child2(proc);
      arg_list = child2(node);
      if (param_list == NULL) {
        if (arg_list == NULL) {
          stack_push (NULL, 0.0); /* number of argument == 0 */
        } else
          runtime_error ("Procedure %s has different number of argument and parameter.\n", name);
      }else {
/* Don't change the stack until finish evaluating the arguments. */
#define TEMP_STACK_SIZE 20
        char *temp_param[TEMP_STACK_SIZE];
        double temp_arg[TEMP_STACK_SIZE];
        n = 0;
        for (; param_list->type == N_parameter_list; param_list = child1(param_list)) {
          if (arg_list->type != N_argument_list)
            runtime_error ("Procedure %s has different number of argument and parameter.\n", name);
          if (n >= TEMP_STACK_SIZE)
            runtime_error ("Too many parameters. the number must be %d or less.\n", TEMP_STACK_SIZE);
          temp_param[n] = name(child2(param_list));
          temp_arg[n] = eval (child2(arg_list));
          arg_list = child1(arg_list);
          ++n;
        }
        if (param_list->type == N_ID && arg_list -> type != N_argument_list) {
          temp_param[n] = name(param_list);
          temp_arg[n] = eval (arg_list);
          if (++n >= TEMP_STACK_SIZE)
            runtime_error ("Too many parameters. the number must be %d or less.\n", TEMP_STACK_SIZE);
          temp_param[n] = NULL;
          temp_arg[n] = (double) n;
          ++n;
        } else
          runtime_error ("Unexpected error.\n");
        for (i = 0; i < n; ++i)
          stack_push (temp_param[i], temp_arg[i]);
      }
      ret_level = ++proc_level;
      execute (child3(proc));
      ret_level = --proc_level;
      stack_return ();
      break;
    case N_procedure_definition:
      name = name(child1(node));
      proc_install (name, node);
      break;
    case N_primary_procedure_list:
      execute (child1(node));
      execute (child2(node));
      break;
    default:
      runtime_error ("Unknown statement.\n");
  }
}
```
@@@end

Узел `N_procedure_call` создается парсером, когда он находит вызов пользовательской процедуры.
Процедура была определена в предыдущем операторе.
Предположим, парсер читает следующий пример кода.

```
dp drawline (angle, distance) {
  tr angle
  fd distance
}
drawline (90, 100)
drawline (90, 100)
drawline (90, 100)
drawline (90, 100)
```

Этот пример рисует квадрат.

Когда парсер читает строки с первой по четвертую, он создает узлы следующим образом:

![Nodes of drawline](../image/tree2.png){width=10cm height=6cm}

Процедура выполнения просто сохраняет процедуру в таблице символов с ее именем и узлом.

![Symbol table](../image/table.png){width=10cm height=5cm}

Когда парсер читает пятую строку в примере, он создает узлы следующим образом:

![Nodes of procedure call](../image/proc_call.png){width=10cm height=5cm}

Когда процедура выполнения встречает узел `N_procedure_call`, она ведет себя следующим образом:

1. Ищет процедуру в таблице символов по имени.
2. Получает указатели на узел параметров и узел тела.
3. Создает временный стек.
Создает кортеж каждого имени параметра и значения аргумента.
Помещает кортежи в стек и, наконец, (NULL, количество параметров).
Если ошибок не происходит, копирует их из временного стека в стек параметров.
4. Увеличивает `prc_level` на один.
Устанавливает `ret_level` равным `proc_level`.
`proc_level` равен нулю, когда процедура выполнения работает в основной процедуре.
Если она входит в процедуру, `proc_level` увеличивается на один.
Следовательно, `proc_level` — это глубина вызова процедуры.
`ret_level` — это уровень для возврата.
Если он равен `proc_level`, процедура выполнения выполняет команды в порядке команд в процедуре.
Если он меньше `proc_level`, процедура выполнения не выполняет команды, пока он не станет равным уровню `proc_level`.
`ret_level` используется для возврата из процедуры.
5. Выполняет узел тела процедуры.
6. Уменьшает `proc_level` на один.
Устанавливает `ret_level` равным `proc_level`.
Вызывает `stack_return`.

Когда процедура выполнения встречает узел `N_RT`, она уменьшает `ret_level` на один, чтобы следующие команды в процедуре игнорировались процедурой выполнения.

#### Функции входа в процедуру выполнения и обработки ошибок

Функция `run` является точкой входа в процедуру выполнения.
Функция `runtime_error` сообщает об ошибке, возникшей во время работы процедуры выполнения.
(Ошибки, возникающие во время синтаксического анализа, называются синтаксическими ошибками и сообщаются `yyerror`.)
После того, как `runtime_error` сообщает об ошибке, она останавливает выполнение команды и возвращается к `run` для выхода.

Используются функции Setjmp и longjmp.
Они объявлены в `<setjmp.h>`.
`setjmp (buf)` сохраняет информацию о состоянии в `buf` и возвращает ноль.
`longjmp(buf, 1)` восстанавливает информацию о состоянии из `buf` и возвращает `1` (второй аргумент).
Поскольку информация — это состояние в момент вызова `setjmp`, longjmp возобновляет выполнение после вызова функции setjmp.
В следующей программе longjmp возобновляет присваивание переменной `i`.
Когда вызывается setjmp, 0 присваивается `i` и вызывается `execute(node_top)`.
С другой стороны, когда вызывается longjmp, 1 присваивается `i` и `execute(node_top)` не вызывается.

`g_slist_free_full` освобождает всю выделенную память.

@@@if gfm
```C
static jmp_buf buf;

void
run (void) {
  int i;

  if (! init_cairo()) {
    g_print ("Cairo not initialized.\n");
    return;
  }
  init_table();
  init_stack();
  ret_level = proc_level = 1;
  i = setjmp (buf);
  if (i == 0)
    execute(node_top);
  /* else ... get here by calling longjmp */
  destroy_cairo ();
  g_slist_free_full (g_steal_pointer (&list), g_free);
}

/* format supports only %s, %f and %d */
static void
runtime_error (char *format, ...) {
  va_list args;
  char *f;
  char b[3];
  char *s;
  double v;
  int i;

  va_start (args, format);
  for (f = format; *f; f++) {
    if (*f != '%') {
      b[0] = *f;
      b[1] = '\0';
      g_print ("%s", b);
      continue;
    }
    switch (*++f) {
      case 's':
        s = va_arg(args, char *);
        g_print ("%s", s);
        break;
      case 'f':
        v = va_arg(args, double);
        g_print("%f", v);
        break;
      case 'd':
        i = va_arg(args, int);
        g_print("%d", i);
        break;
      default:
        b[0] = '%';
        b[1] = *f;
        b[2] = '\0';
        g_print ("%s", b);
        break;
    }
  }
  va_end (args);

  longjmp (buf, 1);
}
```
@@@else
```{.C}
static jmp_buf buf;

void
run (void) {
  int i;

  if (! init_cairo()) {
    g_print ("Cairo not initialized.\n");
    return;
  }
  init_table();
  init_stack();
  ret_level = proc_level = 1;
  i = setjmp (buf);
  if (i == 0)
    execute(node_top);
  /* else ... get here by calling longjmp */
  destroy_cairo ();
  g_slist_free_full (g_steal_pointer (&list), g_free);
}

/* format supports only %s, %f and %d */
static void
runtime_error (char *format, ...) {
  va_list args;
  char *f;
  char b[3];
  char *s;
  double v;
  int i;

  va_start (args, format);
  for (f = format; *f; f++) {
    if (*f != '%') {
      b[0] = *f;
      b[1] = '\0';
      g_print ("%s", b);
      continue;
    }
    switch (*++f) {
      case 's':
        s = va_arg(args, char *);
        g_print ("%s", s);
        break;
      case 'f':
        v = va_arg(args, double);
        g_print("%f", v);
        break;
      case 'd':
        i = va_arg(args, int);
        g_print("%d", i);
        break;
      default:
        b[0] = '%';
        b[1] = *f;
        b[2] = '\0';
        g_print ("%s", b);
        break;
    }
  }
  va_end (args);

  longjmp (buf, 1);
}
```
@@@end

Функция `runtime_error` имеет список аргументов переменной длины.

@@@if gfm
```C
void runtime_error (char *format, ...)
```
@@@else
```{.C}
void runtime_error (char *format, ...)
```
@@@end

Это реализовано с помощью заголовочного файла `<stdarg.h>`.
Переменная типа `va_list` `args` будет ссылаться на каждый аргумент по очереди.
Функция `va_start` инициализирует `args`.
Функция `va_arg` возвращает аргумент и перемещает ссылку `args` на следующий.
Функция `va_end` очищает все необходимое в конце.

Функция `runtime_error` имеет формат, похожий на стандартную функцию printf.
Но ее формат имеет только `%s`, `%f` и `%d`.

Функции, объявленные в `<setjmp.h>` и `<stdarg.h>`, объясняются в очень известной книге "Язык программирования C", написанной Брайаном Кернигеном и Деннисом Ритчи.
Я ссылался на книгу при написании программы выше.

Программа `turtle` неотшлифованна и неполированна.
Если вы хотите создать свой собственный язык, вам нужно знать больше и больше.
Я не знаю ни одного хорошего учебника по компиляторам и интерпретаторам.
Если вы знаете хорошую книгу, пожалуйста, дайте мне знать.

Однако следующая информация очень полезна (но старая).

- Документация Bison
- Документация Flex
- Software tools, написанная Brian W. Kernighan & P. J. Plauger (1976)
- Unix programming environment, написанная Brian W. Kernighan и Rob Pike (1984)
- Исходный код языка, например, ruby.

В последнее время много исходных кодов находится в Интернете.
Может быть, чтение исходных кодов наиболее полезно для программистов.
