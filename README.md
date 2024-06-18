# Конструктор ботов для проведения игровых тренировок в игре Counter-Strike
 
<img height="400" src="gif.gif">

## Содержание

1. [Code Overview](#code-overview)
2. [Trained Models](#trained-models)
3. [Datasets](#datasets)
4. [Requirements](#requirements)
5. [License](#license)
6. [Disclaimer](#disclaimer)
7. [Maintenance](#maintenance)
8. [Troubleshooting](#troubleshooting)


## Обзор модели передвижения

1) Используйте dm_record_data.py для захвата данных как зритель, или dm_record_data_me_wasd.py для записи при активной игре. Это создает .npy файлы со скриншотами и метаданными.
2) Запустите dm_infer_actions.py на .npy файлах из шага 1 для инференции действий из метаданных ("модель обратной динамики"). Эти данные добавляются к тем же .npy файлам. Визуализации и статистика, полученные с помощью этого скрипта, могут быть использованы для очистки данных -- например, если метаданные из GSI и RAM расходятся по некоторым переменным, это может означать, что данные ненадежны, если есть длинные периоды неподвижности (vel_static), это может означать, что вы отслеживали неподвижного игрока. Мы рекомендуем удалять такие файлы.
3) Запустите dm_pretrain_process.py на .npy файлах из шага 2, это создаст новые .hdf5 файлы, содержащие скриншоты и onehot цели.
4) (Необязательно.) Запустите tools_extract_metaadata.py, чтобы извлечь метаданные из .npy файлов, сохранит как новый .npy файл.
5) Запустите dm_train_model.py для обучения модели.
6) Используйте tools_create_stateful.py для создания "состоящей из состояний" версии модели.
7) Скрипт dm_run_agent.py запускает модель в окружении CSGO.


Краткий обзор назначения каждого скрипта:

config.py

Содержит ключевые глобальные гиперпараметры и настройки. Также несколько функций, используемых во многих скриптах.
screen_input.py

Получает скриншоты окна CSGO, выполняет обрезку и уменьшение масштаба. Использует win32gui, win32ui, win32con, OpenCV.
key_output.py

Содержит функции для отправки кликов мыши, установки положения мыши и нажатий клавиш. Использует ctypes.
key_input.py

Содержит функции для проверки положения мыши, кликов мыши и нажатий клавиш. Использует win32api и ctypes.
meta_utils.py

Содержит функции для настройки чтения из оперативной памяти (RAM). Также содержит класс сервера для подключения к инструменту GSI CSGO — переменную MYTOKENHERE следует обновить при настройке GSI.
dm_hazedumper_offsets.py

Содержит смещения для чтения переменных напрямую из оперативной памяти (RAM). Они обновляются функцией в meta_utils.py.
dm_record_data.py

Записывает скриншоты и метаданные в режиме наблюдателя. (Учтите, что эти скрипты записи могут быть капризными и зависят от актуальности смещений hazedumper.)
dm_record_data_me_wasd.py

Записывает скриншоты и метаданные при активной игре на локальной машине.
dm_infer_actions.py

Для записанного набора данных, инферирует действия из метаданных и добавляет их в тот же файл. Также вычисляет статистику, указывающую на качество данных, и предоставляет функциональность для последовательного переименования файлов.
dm_pretrain_process.py

Берет .npy файлы из dm_infer_actions.py и создает соответствующие .hdf5 файлы для быстрой загрузки данных с входами и метками, как ожидается моделью. Также содержит функцию для удаления изображений из .npy файлов.
dm_train_model.py

Обучает модель нейронной сети. Либо с нуля, либо из предыдущей контрольной точки.
dm_run_agent.py

Запускает обученную модель в CSGO. Есть возможность собирать метаданные (например, счет) с помощью IS_GSI=True.
tools_dataset_inspect.py

Минимальный код для открытия различных наборов данных и сопутствующих файлов метаданных.
tools_extract_metadata.py

Вырезает метаданные (currvars, infer_a и helper_arr) из оригинальных .npy файлов и сохраняет как новый .npy файл без изображений. Объединяет 100 файлов вместе.
tools_dataset_stats.py

Составляет метаданные из большого онлайн-набора данных и производит сводную статистику и фигуры, как в приложении A.
tools_create_stateful.py

Превращает обученную "несостоящую" модель в "состоящую" версию, которую следует использовать во время тестирования (иначе состояния LSTM сбрасываются между каждым прямым проходом). Необходимо из-за особенностей keras.
tools_map_coverage_analysis.py

Визуализирует покрытие карты различными агентами и наборами данных, вычисляет расстояние Земельного перемещения (Earth mover's distance).
tools_view_save_egs.py

Открывает и отображает последовательности изображений .hdf5 и метаданных .npy. Возможность наложения инференцированных действий. Возможность сохранения как .png.
console_csgo_setup.txt

Команды консоли для настройки уровней CSGO.


## Датасет

All datasets are available at: https://1drv.ms/u/s!AjG1JlThUkPgh1JEIxETxvaphzgC?e=2AJfA3 . (Please email ```tim dot pearce at microsoft dot com``` with any issues.) A brief description of dataset and directory structure is given below. 

- ```dataset_dm_aim```
    - total files: 22
    - approx size: 3,5 GB
    - map: aim_map
    - gamemode: deathmatch

- ```dataset_yolo```
    - total files: 10224
    - approx size: 5,38 GB
    - map: dust2, aim, inferno, mirage
    - gamemode: deathmatch

### Рекомендуемые технические характеристики

Запуск данного конструктора требует наличия CUDA ядер в видеокарте компьютера. Данный конструктор тестировался на видеокарте RTX 3080. Таким образом, рекомендуемая видеопамять - 10Гб, количество CUDA ядер - 8000.

## Лицензия
Этот репозиторий может быть использован для личных проектов и открытых исследований. 

## Предупреждение
Хотя код не предназначен для читерства/взлома, существует вероятность того, что Valve может обнаружить использование некоторых из этих скриптов в игре (например, имитация движений мыши и анализ оперативной памяти), что, в свою очередь, может вызвать подозрения в читерстве. Мы не несем ответственности за подобные проблемы. Используйте его на свой страх и риск!

## Настройки Counter Strike
Несколько советов, которые могут помочь запустить агента на вашей локальной системе:

    - Game resolution: Normal 4:3, 1024×768, windowed
    - Mouse sensitivity: 2.50
    - Mouse raw input: Off
    - Reverse mouse: Off
    - Mouse acceleration: Off
    - Crosshair settings: Classic static, green – RGB: (46, 250, 42), length 4.3, thickness 1.8, gap 2.0, no outline, no centre dot. (Length 2.8 also used in some training data and demos.)
    - (Or use crosshair code: CSGO-UKcZG-QN8eW-WQMvd-NX6xr-RPqRP)
    - All graphics options: Lowest quality setting
    - Boost Player Contrast: Enabled
    - Multisampling AA Mode: 2x MSAA
    - HUD edge positions: as large as possible
    - HUD scale: 0.56
    - HUD color: Default
    - Radar HUD size: 0.80
    - Clear decals is bound to ‘n’ key (https://www.skinwallet.com/csgo/clear-decals-csgo/)
    - cl_righthand 1
    - viewmodel_offset_x 1
    - viewmodel_offset_y 1
    - viewmodel_offset_z  -1
    - viewmodel_fov 60
    - cl_bobamt_lat 0.33
    - cl_bobamt_vert 0.14
    - cl_bobcycle 0.98
    - cl_viewmodel_shift_left_amt 1.5
    - cl_viewmodel_shift_right_amt 0.75



