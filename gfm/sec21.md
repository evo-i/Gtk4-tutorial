Up: [README.md](../README.md),  Prev: [Section 20](sec20.md), Next: [Section 22](sec22.md)

# GtkFontDialogButton и Gsettings

## Диалог настроек

Если пользователь нажимает на меню настроек, появляется диалог настроек.

![Preference dialog](../image/pref_dialog.png)

В нём есть только одна кнопка - виджет GtkFontDialogButton.
Вы можете добавить больше виджетов в этот диалог, но такой простой диалог неплох для первой примерной программы.

Если нажать на кнопку, появляется FontDialog вот так.

![Font dialog](../image/fontdialog.png)

Если пользователь выбирает шрифт и нажимает кнопку выбора, шрифт изменяется.

GtkFontDialogButton и GtkFontDialog доступны начиная с версии GTK 4.10.
Они заменяют GtkFontButton и GtkFontChooserDialog, которые устарели с версии 4.10.

## Составной виджет

Диалог настроек содержит GtkBox, GtkLabel и GtkFontButton и определён как составной виджет.
Ниже приведён файл шаблона ui для TfePref.

~~~xml
 1 <?xml version="1.0" encoding="UTF-8"?>
 2 <interface>
 3   <template class="TfePref" parent="GtkWindow">
 4     <property name="title">Preferences</property>
 5     <property name="resizable">FALSE</property>
 6     <property name="modal">TRUE</property>
 7     <child>
 8       <object class="GtkBox">
 9         <property name="orientation">GTK_ORIENTATION_HORIZONTAL</property>
10         <property name="spacing">12</property>
11         <property name="halign">GTK_ALIGN_CENTER</property>
12         <property name="margin-start">12</property>
13         <property name="margin-end">12</property>
14         <property name="margin-top">12</property>
15         <property name="margin-bottom">12</property>
16         <child>
17           <object class="GtkLabel">
18             <property name="label">Font:</property>
19             <property name="xalign">1</property>
20           </object>
21         </child>
22         <child>
23           <object class="GtkFontDialogButton" id="font_dialog_btn">
24             <property name="dialog">
25               <object class="GtkFontDialog"/>
26             </property>
27           </object>
28         </child>
29       </object>
30     </child>
31   </template>
32 </interface>
~~~

- Тег Template указывает составной виджет.
Атрибут class указывает имя класса, которое является "TfePref".
Атрибут parent - это `GtkWindow`.
Следовательно, `TfePref` является дочерним классом `GtkWindow`.
Атрибут parent необязателен, но рекомендуется указывать его явно.
Вы можете сделать TfePref дочерним классом `GtkDialog`, но `GtkDialog` устарел с версии 4.10.
- Есть три свойства: title, resizable и modal.
- TfePref имеет дочерний виджет GtkBox, который является горизонтальным.
Контейнер имеет два дочерних элемента: GtkLabel и GtkFontDialogButton.

## Заголовочный файл

Файл `tfepref.h` определяет типы и объявляет публичную функцию.

~~~C
1 #pragma once
2 
3 #include <gtk/gtk.h>
4 
5 #define TFE_TYPE_PREF tfe_pref_get_type ()
6 G_DECLARE_FINAL_TYPE (TfePref, tfe_pref, TFE, PREF, GtkWindow)
7 
8 GtkWidget *
9 tfe_pref_new (void);
~~~

- 5: Определяет тип `TFE_TYPE_PREF`, который является макросом, заменяемым на `tfe_pref_get_type ()`.
- 6: Макрос `G_DECLAER_FINAL_TYPE` раскрывается в:
  - Объявление функции `tfe_pref_get_type ()`.
  - Определение типа TfePrep как typedef для `struct _TfePrep`.
  - Определение типа TfePrepClass как typedef для `struct {GtkWindowClass *parent;}`.
  - Определение двух функций `TFE_PREF ()` и `TFE_IS_PREF ()`.
- 8-9: Объявление функции `tfe_pref_new`. Она создаёт новый экземпляр TfePref.

## C-файл для составного виджета

Следующие коды извлечены из файла `tfepref.c`.

```C
#include <gtk/gtk.h>
#include "tfepref.h"

struct _TfePref
{
  GtkWindow parent;
  GtkFontDialogButton *font_dialog_btn;
};

G_DEFINE_FINAL_TYPE (TfePref, tfe_pref, GTK_TYPE_WINDOW);

static void
tfe_pref_dispose (GObject *gobject) {
  TfePref *pref = TFE_PREF (gobject);
  gtk_widget_dispose_template (GTK_WIDGET (pref), TFE_TYPE_PREF);
  G_OBJECT_CLASS (tfe_pref_parent_class)->dispose (gobject);
}

static void
tfe_pref_init (TfePref *pref) {
  gtk_widget_init_template (GTK_WIDGET (pref));
}

static void
tfe_pref_class_init (TfePrefClass *class) {
  G_OBJECT_CLASS (class)->dispose = tfe_pref_dispose;
  gtk_widget_class_set_template_from_resource (GTK_WIDGET_CLASS (class), "/com/github/ToshioCP/tfe/tfepref.ui");
  gtk_widget_class_bind_template_child (GTK_WIDGET_CLASS (class), TfePref, font_dialog_btn);
}

GtkWidget *
tfe_pref_new (void) {
  return GTK_WIDGET (g_object_new (TFE_TYPE_PREF, NULL));
}
```

- Структура `_TfePref` имеет член `font_dialog_btn`.
Он указывает на объект GtkFontDialogButton, указанный в XML-файле "tfepref.ui".
Имя члена `font_dialog_btn` должно совпадать с атрибутом id GtkFontDialogButton в XML-файле.
- Макрос `G_DEFINE_FINAL_TYPE` раскрывается в:
  - Объявление функций `tfe_pref_init` и `tfe_pref_class_init`.
Они определены в следующей части программы.
  - Определение переменной `tfe_pref_parent_class`.
  - Определение функции `tfe_pref_get_type`.
- Функция `tfe_pref_class_init` инициализирует класс TfePref.
Функция `gtk_widget_class_set_template_from_resource` инициализирует шаблон составного виджета из XML-ресурса.
Функция `gtk_widget_class_bind_template_child` связывает член структуры TfePref `font_dialog_btn` с GtkFontDialogButton в XML.
Имя члена и значение атрибута id должны совпадать.
- Функция `tfe_pref_init` инициализирует вновь созданный экземпляр. Функция `gtk_widget_init_template` создаёт и инициализирует дочерние виджеты.
- Функция `tfe_pref_dispose` освобождает объекты. Функция `gtk_widget_dispose_template` освобождает дочерние виджеты.

## GtkFontDialogButton и Pango

Если нажать кнопку GtkFontDialogButton, появляется диалог GtkFontDialog.
Пользователь может выбрать шрифт в этом диалоге.
Если пользователь нажимает кнопку "select", диалог исчезает.
И информация о шрифте передаётся экземпляру GtkFontDialogButton.
Данные о шрифте получаются методом `gtk_font_dialog_button_get_font_desc`.
Он возвращает указатель на структуру PangoFontDescription.

Pango - это движок компоновки текста.
[Документация](https://docs.gtk.org/Pango/index.html) доступна в интернете.

PangoFontDescription - это структура C, и прямой доступ к ней не допускается.
Документация находится [здесь](https://docs.gtk.org/Pango/struct.FontDescription.html).
Если вы хотите получить информацию о шрифте, есть несколько функций.

- `pango_font_description_to_string` возвращает строку вроде "Jamrul Bold Italic Semi-Expanded 12".
- `pango_font_description_get_family` возвращает семейство шрифта вроде "Jamrul".
- `pango_font_description_get_weight` возвращает константу PangoWeight вроде `PANGO_WEIGHT_BOLD`.
- `pango_font_description_get_style` возвращает константу PangoStyle вроде `PANGO_STYLE_ITALIC`.
- `pango_font_description_get_stretch` возвращает константу PangoStretch вроде `PANGO_STRETCH_SEMI_EXPANDED`.
- `pango_font_description_get_size` возвращает целое число вроде `12`.
Единица измерения - пункт или пиксель (единица устройства).
Функция `pango_font_description_get_size_is_absolute` возвращает TRUE, если единица абсолютная, то есть единица устройства.
В противном случае единица - пункт.

## GSettings

Мы хотим сохранить данные о шрифте после завершения работы приложения.
Есть несколько способов реализации.

- Создать файл конфигурации.
Например, текстовый файл "~/.config/tfe/font_desc.cfg" хранит информацию о шрифте.
- Использовать объект GSettings.
Основная идея GSettings похожа на файл конфигурации.
Данные конфигурационной информации помещаются в файл базы данных.

GSettings прост и лёгок в использовании, но концепция немного сложна для понимания.
Этот подраздел сначала описывает концепцию, а затем способ программирования.

### Схема GSettings

Схема GSettings описывает набор ключей, типов значений и некоторой другой информации.
Объект GSettings использует эту схему и записывает/читает значение ключа в/из нужного места в базе данных.

- Схема имеет id.
Id должен быть уникальным.
Мы часто используем ту же строку, что и id приложения, но id схемы и id приложения - это разные вещи.
Вы можете использовать имя, отличное от id приложения.
Id схемы - это строка, разделённая точками.
Например, "com.github.ToshioCP.tfe" - это корректный id схемы.
- Схема обычно имеет путь.
Путь - это расположение в базе данных.
Каждый ключ хранится по этому пути.
Например, если ключ `font-desc` определён с путём `/com/github/ToshioCP/tfe/`,
расположение ключа в базе данных будет `/com/github/ToshioCP/tfe/font-desc`.
Путь - это строка, начинающаяся и заканчивающаяся косой чертой (`/`).
И разделённая косыми чертами.
- GSettings сохраняет информацию в стиле ключ-значение.
Ключ - это строка, начинающаяся со строчной буквы, за которой следуют строчные буквы, цифры или тире (`-`), и заканчивающаяся строчной буквой или цифрой.
Последовательные тире не допускаются.
Значения могут быть любого типа.
GSettings хранит значения как тип GVariant, который может быть, например, целым числом, числом с плавающей точкой, булевым значением, строкой или сложными типами, такими как массив.
Тип значений должен быть определён в схеме.
- Для каждого ключа должно быть установлено значение по умолчанию.
- Для каждого ключа можно опционально установить резюме и описание.

Схемы описываются в формате XML.
Например,

~~~xml
 1 <?xml version="1.0" encoding="UTF-8"?>
 2 <schemalist>
 3   <schema path="/com/github/ToshioCP/tfe/" id="com.github.ToshioCP.tfe">
 4     <key name="font-desc" type="s">
 5       <default>'Monospace 12'</default>
 6       <summary>Font</summary>
 7       <description>A font to be used for textview.</description>
 8     </key>
 9   </schema>
10 </schemalist>
~~~

- 4: Атрибут type имеет значение "s".
Это строковый тип GVariant.
Для строковых типов GVariant смотрите [GLib API Reference -- GVariant Type Strings](https://docs.gtk.org/glib/struct.VariantType.html#gvariant-type-strings).
Другие распространённые типы:
  - "b": gboolean
  - "i": gint32.
  - "d": double.

Дополнительная информация находится в:

- [GLib API Reference -- GVariant Format Strings](https://docs.gtk.org/glib/gvariant-format-strings.html)
- [GLib API Reference -- GVariant Text Format](https://docs.gtk.org/glib/gvariant-text.html)
- [GLib API Reference -- GVariant](https://docs.gtk.org/glib/struct.Variant.html)
- [GLib API Reference -- VariantType](https://docs.gtk.org/glib/struct.VariantType.html)

### Команда Gsettings

Сначала давайте попробуем приложение `gsettings`.
Это инструмент конфигурации для GSettings.

~~~
$ gsettings help
Usage:
  gsettings --version
  gsettings [--schemadir SCHEMADIR] COMMAND [ARGS?]

Commands:
  help                      Show this information
  list-schemas              List installed schemas
  list-relocatable-schemas  List relocatable schemas
  list-keys                 List keys in a schema
  list-children             List children of a schema
  list-recursively          List keys and values, recursively
  range                     Queries the range of a key
  describe                  Queries the description of a key
  get                       Get the value of a key
  set                       Set the value of a key
  reset                     Reset the value of a key
  reset-recursively         Reset all values in a given schema
  writable                  Check if a key is writable
  monitor                   Watch for changes

Use "gsettings help COMMAND" to get detailed help.
~~~

Список схем.

~~~
$ gsettings list-schemas
org.gnome.rhythmbox.podcast
ca.desrt.dconf-editor.Demo.Empty
org.gnome.gedit.preferences.ui
org.gnome.evolution-data-server.calendar
org.gnome.rhythmbox.plugins.generic-player

... ...

~~~

Каждая строка - это id схемы.
Каждая схема имеет данные конфигурации ключ-значение.
Вы можете увидеть их с помощью команды list-recursively.
Давайте посмотрим на ключи и значения схемы `org.gnome.calculator`.

~~~
$ gsettings list-recursively org.gnome.calculator
org.gnome.calculator accuracy 9
org.gnome.calculator angle-units 'degrees'
org.gnome.calculator base 10
org.gnome.calculator button-mode 'basic'
org.gnome.calculator number-format 'automatic'
org.gnome.calculator precision 2000
org.gnome.calculator refresh-interval 604800
org.gnome.calculator show-thousands false
org.gnome.calculator show-zeroes false
org.gnome.calculator source-currency ''
org.gnome.calculator source-units 'degree'
org.gnome.calculator target-currency ''
org.gnome.calculator target-units 'radian'
org.gnome.calculator window-position (-1, -1)
org.gnome.calculator word-size 64
~~~

Эта схема используется GNOME Калькулятором.
Запустите калькулятор и измените режим, затем снова проверьте схему.

~~~
$ gnome-calculator
~~~

![gnome-calculator basic mode](../image/gnome_calculator_basic.png)


Измените режим на расширенный и закройте.

![gnome-calculator advanced mode](../image/gnome_calculator_advanced.png)

Запустите gsettings и проверьте значение `button-mode`.

~~~
$ gsettings list-recursively org.gnome.calculator

... ...

org.gnome.calculator button-mode 'advanced'

... ...

~~~

Теперь мы знаем, что GNOME Калькулятор использовал gsettings и установил ключ `button-mode` в значение "advanced".
Значение сохраняется даже после закрытия калькулятора.
Так что когда калькулятор запустится снова, он появится в расширенном режиме.

### Утилита Glib-compile-schemas

Схемы GSettings определяются в формате XML.
Файлы XML-схем должны иметь расширение `.gschema.xml`.
Ниже приведён XML-файл схемы для приложения `tfe`.

~~~xml
 1 <?xml version="1.0" encoding="UTF-8"?>
 2 <schemalist>
 3   <schema path="/com/github/ToshioCP/tfe/" id="com.github.ToshioCP.tfe">
 4     <key name="font-desc" type="s">
 5       <default>'Monospace 12'</default>
 6       <summary>Font</summary>
 7       <description>A font to be used for textview.</description>
 8     </key>
 9   </schema>
10 </schemalist>
~~~

Имя файла - "com.github.ToshioCP.tfe.gschema.xml".
Имена XML-файлов схем обычно представляют собой id схемы с суффиксом ".gschema.xml".
Вы можете использовать имя, отличное от id схемы, но это не рекомендуется.

- 2: Элемент верхнего уровня - `<schemalist>`.
- 3: Тег schema имеет атрибуты `path` и `id`.
Путь определяет, где настройки хранятся в концептуальном глобальном дереве настроек.
Id идентифицирует схему.
- 4: Тег Key имеет два атрибута.
Name - это имя ключа.
Type - это тип значения ключа, и это строка формата GVariant.
- 5: Значение по умолчанию ключа `font-desc` - `Monospace 12`.
- 6: Элементы Summery и description описывают ключ.
Они необязательны, но рекомендуется добавлять их в XML-файл.

XML-файл компилируется с помощью glib-compile-schemas.
При компиляции `glib-compile-schemas` компилирует все XML-файлы с расширением ".gschema.xml" в каталоге, указанном в качестве аргумента.
Он преобразует XML-файл в бинарный файл `gschemas.compiled`.
Предположим, что XML-файл выше находится в каталоге `tfe6`.

~~~
$ glib-compile-schemas tfe6
~~~

Затем `gschemas.compiled` генерируется в каталоге `tfe6`.
При тестировании вашего приложения установите переменную окружения `GSETTINGS_SCHEMA_DIR`, чтобы объект GSettings мог найти `gschemas.compiled`.

~~~
$ GSETTINGS_SCHEMA_DIR=(the directory gschemas.compiled is located):$GSETTINGS_SCHEMA_DIR (your application name)
~~~

Объект GSettings ищет этот файл по следующему процессу.

- Он ищет подкаталоги `glib-2.0/schemas` во всех каталогах, указанных в переменной окружения `XDG_DATA_DIRS`.
Обычные каталоги - `/usr/share/glib-2.0/schemas` и `/usr/local/share/glib-2.0/schemas`.
- Если существует `$HOME/.local/share/glib-2.0/schemas`, он также ищется.
- Если определена переменная окружения `GSETTINGS_SCHEMA_DIR`, она ищет все каталоги, указанные в переменной.
`GSETTINGS_SCHEMA_DIR` может указывать несколько каталогов, разделённых двоеточием (:).

Каталоги выше содержат более одного файла `.gschema.xml`.
Поэтому при установке вашего приложения следуйте инструкции ниже для установки ваших схем.

1. Создайте файл `.gschema.xml`.
2. Скопируйте его в один из каталогов выше. Например, `$HOME/.local/share/glib-2.0/schemas`.
3. Запустите `glib-compile-schemas` на этом каталоге.
Это скомпилирует все файлы схем в каталоге и создаст или обновит файл базы данных `gschemas.compiled`.

### Объект GSettings и привязка

Теперь перейдём к следующей теме - как программировать GSettings.

Вам нужно заранее скомпилировать ваш файл схемы.

Предположим, что id, ключ, имя класса и имя свойства следующие:

- GSettings id: com.github.ToshioCP.sample
- GSettings key: sample_key
- Имя класса: Sample
- Свойство для привязки: sample_property

Пример ниже использует `g_settings_bind`.
Если вы используете это, ключ GSettings и свойство экземпляра должны иметь одинаковый тип.
В примере предполагается, что тип "sample_key" и "sample_property" одинаковый.

```C
GSettings *settings;
Sample *sample_object;

settings = g_settings_new ("com.github.ToshioCP.sample");
sample_object = sample_new ();
g_settings_bind (settings, "sample_key", sample_object, "sample_property", G_SETTINGS_BIND_DEFAULT);
```

Функция `g_settings_bind` связывает значение GSettings со свойством экземпляра.
Если значение свойства изменяется, значение GSettings также изменяется, и наоборот.
Эти два значения всегда одинаковы.

Функция `g_settings_bind` проста и удобна, но не всегда применима.
Типы GSettings ограничены типами, которые имеет GVariant.
Некоторые типы свойств не входят в GVariant.
Например, GtkFontDialogButton имеет свойство "font-desc", и его тип - PangoFontDescription.
PangoFontDescription - это структура C, и она обёрнута в boxed-тип GValue для хранения в свойстве.
GVariant не поддерживает boxed-типы.

В этом случае используется другая функция `g_settings_bind_with_mapping`.
Она связывает значение GVariant GSettings и свойство объекта через GValue с функциями отображения.

```C
void
g_settings_bind_with_mapping (
  GSettings* settings,
  const gchar* key,
  GObject* object,
  const gchar* property,
  GSettingsBindFlags flags, // G_SETTINGS_BIND_DEFAULT is commonly used
  GSettingsBindGetMapping get_mapping, // GSettings => property, See the example below
  GSettingsBindSetMapping set_mapping, // property => GSettings, See the example below
  gpointer user_data, // NULL if unnecessary
  GDestroyNotify destroy //NULL if unnecessary
)
```

Функции отображения определяются так:

```C
gboolean
(* GSettingsBindGetMapping) (
  GValue* value,
  GVariant* variant,
  gpointer user_data
)

GVariant*
(* GSettingsBindSetMapping) (
  const GValue* value,
  const GVariantType* expected_type,
  gpointer user_data
)
```

Следующие коды извлечены из `tfepref.c`.

~~~C
 1 static gboolean // GSettings => property
 2 get_mapping (GValue* value, GVariant* variant, gpointer user_data) {
 3   const char *s = g_variant_get_string (variant, NULL);
 4   PangoFontDescription *font_desc = pango_font_description_from_string (s);
 5   g_value_take_boxed (value, font_desc);
 6   return TRUE;
 7 }
 8 
 9 static GVariant* // Property => GSettings
10 set_mapping (const GValue* value, const GVariantType* expected_type, gpointer user_data) {
11   char*font_desc_string = pango_font_description_to_string (g_value_get_boxed (value));
12   return g_variant_new_take_string (font_desc_string);
13 }
14 
15 static void
16 tfe_pref_init (TfePref *pref) {
17   gtk_widget_init_template (GTK_WIDGET (pref));
18   pref->settings = g_settings_new ("com.github.ToshioCP.tfe");
19   g_settings_bind_with_mapping (pref->settings, "font-desc", pref->font_dialog_btn, "font-desc", G_SETTINGS_BIND_DEFAULT,
20       get_mapping, set_mapping, NULL, NULL);
21 }
~~~

- 15-21: Эта функция `tfe_pref_init` инициализирует новый экземпляр TfePref.
- 18: Создаёт новый экземпляр GSettings. Id - "com.github.ToshioCP.tfe".
- 19-20: Связывает GSettings "font-desc" и свойство GtkFontDialogButton "font-desc". Функции отображения - `get_mapping` и `set_mapping`.
- 1-7: Функция отображения из GSettings в свойство.
Первый аргумент `value` - это GValue для хранения в свойстве.
Второй аргумент `variant` - это структура GVariant, которая приходит из значения GSettings.
- 3: Извлекает строку из структуры GVariant.
- 4: Строит структуру PangoFontDescription из строки и присваивает её адрес `font_desc`.
- 5: Помещает `font_desc` в GValue `value`.
Владение `font_desc` переходит к `value`.
- 6: Возвращает TRUE, что означает успешное отображение.
- 9-13: Функция отображения из свойства в GSettings.
Первый аргумент `value` содержит данные свойства.
Второй аргумент `expected_type` - это тип GVariant, который имеет значение GSettings.
Он не используется в этой функции.
- 11: Получает структуру PangoFontDescription из `value` и преобразует её в строку.
- 12: Строка вставляется в структуру GVariant.
Владение строкой `font_desc_string` переходит к возвращаемому значению.

## C-файл

Ниже приведён полный код `tfepref.c`

~~~C
 1 #include <gtk/gtk.h>
 2 #include "tfepref.h"
 3 
 4 struct _TfePref
 5 {
 6   GtkWindow parent;
 7   GSettings *settings;
 8   GtkFontDialogButton *font_dialog_btn;
 9 };
10 
11 G_DEFINE_FINAL_TYPE (TfePref, tfe_pref, GTK_TYPE_WINDOW);
12 
13 static void
14 tfe_pref_dispose (GObject *gobject) {
15   TfePref *pref = TFE_PREF (gobject);
16 
17   /* GSetting bindings are automatically removed when the object is finalized, so it isn't necessary to unbind them explicitly.*/
18   g_clear_object (&pref->settings);
19   gtk_widget_dispose_template (GTK_WIDGET (pref), TFE_TYPE_PREF);
20   G_OBJECT_CLASS (tfe_pref_parent_class)->dispose (gobject);
21 }
22 
23 /* ---------- get_mapping/set_mapping ---------- */
24 static gboolean // GSettings => property
25 get_mapping (GValue* value, GVariant* variant, gpointer user_data) {
26   const char *s = g_variant_get_string (variant, NULL);
27   PangoFontDescription *font_desc = pango_font_description_from_string (s);
28   g_value_take_boxed (value, font_desc);
29   return TRUE;
30 }
31 
32 static GVariant* // Property => GSettings
33 set_mapping (const GValue* value, const GVariantType* expected_type, gpointer user_data) {
34   char*font_desc_string = pango_font_description_to_string (g_value_get_boxed (value));
35   return g_variant_new_take_string (font_desc_string);
36 }
37 
38 static void
39 tfe_pref_init (TfePref *pref) {
40   gtk_widget_init_template (GTK_WIDGET (pref));
41   pref->settings = g_settings_new ("com.github.ToshioCP.tfe");
42   g_settings_bind_with_mapping (pref->settings, "font-desc", pref->font_dialog_btn, "font-desc", G_SETTINGS_BIND_DEFAULT,
43       get_mapping, set_mapping, NULL, NULL);
44 }
45 
46 static void
47 tfe_pref_class_init (TfePrefClass *class) {
48   G_OBJECT_CLASS (class)->dispose = tfe_pref_dispose;
49   gtk_widget_class_set_template_from_resource (GTK_WIDGET_CLASS (class), "/com/github/ToshioCP/tfe/tfepref.ui");
50   gtk_widget_class_bind_template_child (GTK_WIDGET_CLASS (class), TfePref, font_dialog_btn);
51 }
52 
53 GtkWidget *
54 tfe_pref_new (void) {
55   return GTK_WIDGET (g_object_new (TFE_TYPE_PREF, NULL));
56 }
~~~

## Тестовая программа

Тестовая программа находится в каталоге `src/tfe6/test`.

~~~C
 1 #include <gtk/gtk.h>
 2 #include "../tfepref.h"
 3 
 4 GSettings *settings;
 5 
 6 // "changed::font-desc" signal handler
 7 static void
 8 changed_font_desc_cb (GSettings *settings, char *key, gpointer user_data) {
 9   char *s;
10   s = g_settings_get_string (settings, key);
11   g_print ("%s\n", s);
12   g_free (s);
13 }
14 
15 static void
16 app_shutdown (GApplication *application) {
17   g_object_unref (settings);
18 }
19 
20 static void
21 app_activate (GApplication *application) {
22   GtkWidget *pref = tfe_pref_new ();
23 
24   gtk_window_set_application (GTK_WINDOW (pref), GTK_APPLICATION (application));
25   gtk_window_present (GTK_WINDOW (pref));
26 }
27 
28 static void
29 app_startup (GApplication *application) {
30   settings = g_settings_new ("com.github.ToshioCP.tfe");
31   g_signal_connect (settings, "changed::font-desc", G_CALLBACK (changed_font_desc_cb), NULL);
32   g_print ("%s\n", "Change the font with the font button. Then the new font will be printed out.\n");
33 }
34 
35 #define APPLICATION_ID "com.github.ToshioCP.test_tfe_pref"
36 
37 int
38 main (int argc, char **argv) {
39   GtkApplication *app;
40   int stat;
41 
42   app = gtk_application_new (APPLICATION_ID, G_APPLICATION_DEFAULT_FLAGS);
43   g_signal_connect (app, "startup", G_CALLBACK (app_startup), NULL);
44   g_signal_connect (app, "activate", G_CALLBACK (app_activate), NULL);
45   g_signal_connect (app, "shutdown", G_CALLBACK (app_shutdown), NULL);
46   stat = g_application_run (G_APPLICATION (app), argc, argv);
47   g_object_unref (app);
48   return stat;
49 }
~~~

Эта программа устанавливает своё активное окно как экземпляр TfePref, который является дочерним объектом GtkWindow.

Она устанавливает обработчик сигнала "changed::font-desc" в функции startup.
Процесс от выбора шрифта пользователем до обработчика следующий:

- Пользователь нажал на GtkFontDialogButton и появился GtkFontDialog.
- Он/она выбирает новый шрифт.
- Свойство "font-desc" экземпляра GtkFontDialogButton изменяется.
- Значение ключа "font-desc" в базе данных GSettings изменяется, так как оно связано со свойством.
- Сигнал "changed::font-desc" на экземпляре GSettings испускается.
- Обработчик вызывается.

Сборка программы разделена на четыре шага.

- Компиляция файла схемы
- Компиляция XML-файла в ресурс (файл исходного кода C)
- Компиляция C-файлов
- Запуск исполняемого файла

Команды показаны в следующих четырёх подразделах.
Вам не нужно их пробовать.
Последний подраздел показывает способ meson-ninja, который самый простой.

### Компиляция файла схемы

```
$ cd src/tef6/test
$ cp ../com.github.ToshioCP.tfe.gschema.xml com.github.ToshioCP.tfe.gschema.xml
$ glib-compile-schemas .
```

Будьте внимательны. Команда `glib-compile-schemas` имеет аргумент ".", который означает текущий каталог.
Это приводит к созданию файла `gschemas.compiled`.

### Компиляция XML-файла

```
$ glib-compile-resources --sourcedir=.. --generate-source --target=resource.c ../tfe.gresource.xml
```

### Компиляция C-файла

```
$ gcc `pkg-config --cflags gtk4` test_pref.c ../tfepref.c resource.c `pkg-config --libs gtk4`
```

### Запуск исполняемого файла

```
$ GSETTINGS_SCHEMA_DIR=. ./a.out

Jamrul Italic Semi-Expanded 12 # <= select Jamrul Italic 12
Monospace 12 #<= select Monospace Regular 12
```

### Способ Meson-ninja

Meson объединяет команды выше.
Создайте следующий текст и сохраните его в `meson.build`.

Примечание: Репозиторий Gtk4-tutorial имеет файл meson.build, который определяет несколько тестов.
Поэтому вы можете попробовать его вместо следующего текста.

```
project('tfe_pref_test', 'c')

gtkdep = dependency('gtk4')

gnome=import('gnome')
resources = gnome.compile_resources('resources','../tfe.gresource.xml', source_dir: '..')
gnome.compile_schemas(build_by_default: true, depend_files: 'com.github.ToshioCP.tfe.gschema.xml')

executable('test_pref', ['test_pref.c', '../tfepref.c'], resources, dependencies: gtkdep, export_dynamic: true, install: false)
```

- Имя проекта - 'tfe\_pref\_test', и он написан на языке C.
- Он зависит от библиотеки GTK4.
- Он использует модуль GNOME. Модули подготовлены Meson.
- Модуль GNOME имеет метод `compile_resources`.
Когда вы вызываете этот метод, вам нужен префикс "gnome.".
  - Целевое имя файла - resources.
  - XML-файл определения - '../tfe.gresource.xml'.
  - Исходный каталог - '..'. Все ui-файлы находятся там.
- Модуль GNOME имеет метод `compile_schemas`.
Он компилирует файл схемы 'com.github.ToshioCP.tfe.gschema.xml'.
Вам нужно заранее скопировать '../com.github.ToshioCP.tfe.gschema.xml' в текущий каталог.
- Он создаёт исполняемый файл 'test_pref'.
Исходные файлы - 'test_pref.c', '../tfepref.c' и `resources`, который создаётся с помощью `gnome.compile_resources`.
Он зависит от `gtkdep`, который является библиотекой GTK4.
Символы экспортируются, и поддержка установки отсутствует.

Введите вот так для сборки и тестирования программы.

```
$ cd src/tef6/test
$ cp ../com.github.ToshioCP.tfe.gschema.xml com.github.ToshioCP.tfe.gschema.xml
$ meson setup _build
$ ninja -C _build
$ GSETTINGS_SCHEMA_DIR=_build _build/test_pref
```

Появляется окно, и вы можете выбрать шрифт через GtkFontDialog.
Если вы выбираете новый шрифт, строка шрифта выводится в стандартный вывод.

Up: [README.md](../README.md),  Prev: [Section 20](sec20.md), Next: [Section 22](sec22.md)
