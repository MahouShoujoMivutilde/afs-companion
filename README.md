# Скрипт-компаньон для [Advanced FrameServer](http://advancedfs.sourceforge.net/)
Рассчитан на работу вместе __[AviSynth](http://avisynth.nl/index.php/Main_Page)__ и __[FFmpeg](http://ffmpeg.zeranoe.com/builds/)__ 32bit (из-за AviSynth только 32, увы), который, разумеется, должен быть добавлен в __PATH__, написан под __Python 3.3+__.

Создан для личного использования, ибо __libx264 [выигрывает по качеству](https://imgur.com/a/uLmJW) у mainconcept h.264__ в Adobe PP Pro при аналогичном битрейте. Последний же, более того, имеет весьма скудный набор опций, среди которых нет возможности задать RF, который может быть полезен, когда не знаешь, насколько хорошо картинка будет ужиматься при конкретном битрейте.

~~Разумеется, в сравнении выше недостаточный битрей был выбран намеренно - чтобы продемонстировать разницу более наглядно. 
На каком-нибудь 10М 2-pass всё было бы далеко не настолько печально. Но тем не менее~~...

..._Однако_, кодирует на ~30%+ дольше - и это не считая задержу в 12 секунд между кодированием файлов в очереди, от которой, увы, не избавиться (разве что в исходники AFS лезть, но пока как-то лениво). Увы, за хорошее сжатие приходится платить...

## Использование
_exe-версию можно найти [тут](https://github.com/MahouShoujoMivutilde/AFS-companion/releases)._

По-умолчанию __работает в фоне, открывая консоль только во время кодирования__ ради вывода прогресса.

В настройках AFS поставить: 

![stop serving when idle → idle timeout установить на 10 секунд - минимальное доступное время](https://i.imgur.com/J5eNY6F.png)

Добавить скрипт в автозапуск через планировщик заданий или ещё как-нибудь, и запускать так:
```
>>> ...path\to\frameserver.pyw -w d:\my\custom\watch_dir
```
...где `watch_dir` - папка, куда будут кидаться тобой, человек, временные avi-файлы из AFS (создастся, если отсутствует по заданному пути). Если её не задать - создастся по необходимости сразу при запуске скрипта и будет юзаться `%HOMEPATH%\AFS_OUTPUT\`, с мыслью о том, что уж юзер-директория то точно должна быть.

~~...Ну или запускать каждый раз ручками, idk. Но тогда разумно будет сменить расширение с `.pyw` на `.py` (или запускать *-cli.exe версию, соответственно) - чтобы была возможность вырубить скрипт не только через завершение процесса...~~


После запуска скрипт будет каждую секунду проверять, нет ли avi-файлика в заданной папке, и если есть - начинать кодирование с дефолтными или заданными опциями (о которых ниже), и если кодирование таки пошло - после будет таймаут 12 секунд ради того чтобы AFS _точно_ успел удалить отработанный avi-файл, если же и этого будет мало - то скрипт попробует выпилить его своими силами.

## Что еще умеет
__Поддерживает очередь__, собственно, ради этого и нужен таймаут.

__Некоторые настройки можно задавать через имя__ выходного __файла__ близким к FFmpeg синтаксисом:
* `my file name scale=w,h.avi` в итоге даст масштаб до ширины `w` и высоты `h` аналогично опции FFmpeg `-vf scale=w:h -sws_flags lanczos`:
* * так, например, `scale=-2,1080` отмасштабирует до 1080px по высоте, сохраняя при этом делимость на 2 ширины (нужно для кодека x264)
* * умеет только в числа, ~~в будущем, быть может, добавлю `iw` и `ih`, хотя, учитывая контекст использования, пока не вижу смысла~~...
* `my file name -an.avi` уберет аудио - _сделано специально для mp4-анимаций telegram_, ~~ради которого, собственно, это добро и написано~~. Без неё кодирует AAC 576k.
* `my file name -crf 20.avi` даст кодирование с RF = 20, принимает значение в пределах [0-51], при отсутствии/неверном значении кодируется с RF=18:
* * ![0 lossless >>> 18 better >>> 23 x264 default >>> 28 worse >>> 51 worst](https://i.imgur.com/oeuko1s.png)

...ну, а не перечисленные здесь - легко меняются буквально в первых строчках скрипта.

__Параметры можно комбинировать__, разумеется.

Протестировано и успешно работает на
* __Adobe Premiere Pro CC 2017__ и __Adobe Media Encoder CC 2017__
* __Python 3.6.3__
* __AviSynth 2.6.0.6__
* __FFmpeg 3.3.3 win32 shared__

## Возможные альтернативы
* [x264vfw](https://sourceforge.net/projects/x264vfw) - можно использовать, если вывод в контейнер avi - приемлем, и _вся_ очередь должна быть _с одинаковыми настройками_ кодека. Из-за того, что конфигурация кодека глобальная - отсутствует возможность сделать пресет с конкретными настойками. Также, увы, не умеет определять исходные характеристики файла - т.е. задавать разрешение и частоту кадров нужно _всегда_ вручную. Ну, и, разумеется, [скрипт](frameserver.pyw) можно при желании отредактировать под кодирование любого поддерживаемого FFmpeg кодека, когда как x264vfw, как очевидно даже из названия, - это только x264.
* * Использование: установить, в экспорте выбрать "формат - AVI" -> "видеокодек - x264vfw". 

---
_p.s._ Из-за AviSynth боится всяких кандзи и прочего юникода в названиях файлов. Увы.