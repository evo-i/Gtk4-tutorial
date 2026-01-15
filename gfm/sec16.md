Up: [README.md](../README.md),  Prev: [Section 15](sec15.md), Next: [Section 17](sec17.md)

# Как собрать tfe (текстовый редактор файлов)

## Как скомпилировать и запустить текстовый редактор 'tfe'.

Сначала, исходные файлы находятся в [репозитории Gtk4-tutorial](https://github.com/ToshioCP/Gtk4-tutorial).
Как их загрузить, написано в конце [предыдущего раздела](sec15.md).

Ниже приведена инструкция по компиляции и выполнению.

- Вам нужны meson и ninja.
- Если вы установили gtk4 из исходного кода, вам нужно установить переменные окружения в соответствии с вашей установкой.
- Измените текущий каталог на каталог `src/tfe5`.
- Введите `meson setup _build` для конфигурации.
- Введите `ninja -C _build` для компиляции.
Затем приложение `tfe` будет собрано в каталоге `_build`.
- Введите `_build/tfe` для его выполнения.

Затем появится окно.
Есть четыре кнопки: `New`, `Open`, `Save` и `Close`.

- Нажмите кнопку `Open`, затем появится диалог выбора файла.
Выберите файл в списке и нажмите кнопку `Open`.
Затем файл будет прочитан, и появится новая страница Notebook.
- Отредактируйте файл и нажмите кнопку `Save`, тогда текст будет сохранён в исходный файл.
- Нажмите `Close`, тогда страница Notebook исчезнет.
- Нажмите `Close` снова, тогда страница Notebook `Untitled` исчезнет, и одновременно приложение завершит работу.

Это очень простой редактор.
Это хорошая практика для вас, чтобы добавить больше функций.

## Общее количество строк, слов и символов

~~~
$ LANG=C wc tfe5/meson.build tfe5/tfeapplication.c tfe5/tfe.gresource.xml tfe5/tfenotebook.c tfe5/tfenotebook.h tfetextview/tfetextview.c tfetextview/tfetextview.h tfe5/tfe.ui
   10    17   294 tfe5/meson.build
  110   335  3602 tfe5/tfeapplication.c
    6     9   153 tfe5/tfe.gresource.xml
  144   390  3668 tfe5/tfenotebook.c
   15    21   241 tfe5/tfenotebook.h
  235   821  8473 tfetextview/tfetextview.c
   32    54   624 tfetextview/tfetextview.h
   61   100  2073 tfe5/tfe.ui
  613  1747 19128 total
~~~

Up: [README.md](../README.md),  Prev: [Section 15](sec15.md), Next: [Section 17](sec17.md)
