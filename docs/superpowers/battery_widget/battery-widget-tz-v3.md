# ТЗ: Виджет батареи для nice-view-gem (middle canvas, только левая половина)

## Контекст проекта

- Репозиторий: https://github.com/artempletnev/hsv-pletnev
- Модуль отображения: nice-view-gem (ZMK). У каждой половины клавиатуры свой физический экран Nice!View 160x68 пикселей.
- Каждый экран разделен на 3 канваса по 68x68 пикселей: top, middle, bottom.
- ОГРАНИЧЕНИЕ (по документации Display_CheatSheet.md): top canvas (батарея этой половины, статус USB/BLE) и bottom canvas (точки BT-профилей, имя слоя) - фиксированная часть nice-view-gem, их менять не нужно. Разработчику доступен ТОЛЬКО middle canvas, размер 68x68 пикселей.
- Клавиатура - split: левая половина - central (подключается к хосту по USB/BLE), правая - peripheral (связь только с central по BLE).
- Референсы: nice-view-gem-widget-guide.md, 2026-03-22-modifier-widget-2.md (шаблон паттерна виджета), Display_CheatSheet.md (структура dual-display, параметры, файлы конфигурации).

## Ключевое решение: виджет только на левой половине (central)

По умолчанию ZMK передает на хост только заряд central-половины. Peer-заряд (правой половины) central получает исключительно через отдельные опции CONFIG_ZMK_SPLIT_BLE_CENTRAL_BATTERY_LEVEL_FETCHING и CONFIG_ZMK_SPLIT_BLE_CENTRAL_BATTERY_LEVEL_PROXY. Peripheral физически не может видеть данные central ни при каких условиях.

Виджет battery detail реализуется и включается ТОЛЬКО в конфигурации левой половины (hillside_view_enc_left.conf). Правая половина (hillside_view_enc_right.conf) не изменяется.

## Ограничения точности исходных данных

Контроллер (nRF52 на nice!nano и подобных платах) НЕ измеряет ток потребления. Нет отдельного fuel gauge чипа с кулоновским счетчиком. ZMK оценивает уровень заряда исключительно по напряжению батареи через один из двух драйверов:

- zmk,battery-voltage-divider - чтение напряжения через резистивный делитель на ADC-пин.
- zmk,battery-nrf-vddh - чтение напряжения питания напрямую с пина VDDH чипа nRF52.

Напряжение переводится в проценты по зашитой линейной аппроксимации (4200мВ = 100%, 3450мВ = 0%) - грубое приближение. Известны случаи, когда полностью заряженная батарея показывает около 92% из-за разброса резисторов делителя (плюс-минус 1-5%).

Вывод: прогноз времени работы - оценка по тренду изменения процента, не измерение реального тока. Точность прогноза не превышает точность исходных данных о заряде.

## Опциональная калибровка параметров батареи

### Калибровка делителя напряжения (рекомендуется)

```
&vbatt {
    compatible = "zmk,battery-voltage-divider";
    io-channels = <&adc 7>;
    output-ohms = <474300>;
    full-ohms = <(1000000 + 474300)>;
};
```

Подгонка output-ohms/full-ohms под фактически замеренное мультиметром напряжение полностью заряженной батареи. Разовая калибровка на конкретный экземпляр платы.

### OCV-кривая под химию батареи (опционально, вторая очередь)

```
fuel_gauge: fuel_gauge {
    compatible = "zephyr,fuel-gauge-composite";
    source-primary = <&vbatt>;
    device-chemistry = "lithium-ion-polymer";
    ocv-capacity-table-0 = <BATTERY_OCV_CURVE_LITHIUM_ION_POLYMER_DEFAULT>;
    charge-full-design-microamp-hours = <1350000>;
};
```

Требует замены драйвера батареи. Не входит в MVP.

### Рекомендация

Перед реализацией сверить % заряда с реальным напряжением на полной/разряженной батарее. При расхождении более 5 п.п. - откалибровать output-ohms/full-ohms.

## Цель

Виджет Battery Detail для middle canvas (68x68px) ЛЕВОЙ половины: прогноз времени работы, прогноз до полной зарядки, скорость разряда, статус peer (правой половины), индикация потери BLE-связи.

## Сценарии использования (Definition of Done)

1. Краткий прогноз собственного времени работы (например "~10д") в верхней зоне canvas.
2. Компактная строка статуса правой половины (процент, прогноз или предупреждение) в нижней зоне.
3. Прогноз на основе скорости разряда за скользящее окно (24-72ч), не фиксированная линейная экстраполяция.
4. Прогноз "когда нужно зарядить" - момент падения ниже порога (по умолчанию 20%).
5. При зарядке - прогноз времени до полной зарядки вместо прогноза разряда.
6. Индикация низкого заряда (рамка, восклицательный знак) для своего и peer значения.
7. Индикация потери BLE-связи с peer - явный маркер "нет данных" вместо устаревшего процента.
8. Не более 2-3 текстовых строк и одна иконка на canvas, крупный шрифт для главного числа.
9. Переключение через Kconfig, взаимоисключающее с MODIFIERS и WPM-режимом.
10. Корректная деградация при отключенном BLE_CENTRAL_BATTERY_LEVEL_FETCHING.
11. Правая половина не изменяется.

## Технические предпосылки (ZMK)

Общий конфиг (hillside_view_enc.conf):
```
CONFIG_ZMK_BATTERY_REPORTING=y
CONFIG_ZMK_SPLIT_BLE_CENTRAL_BATTERY_LEVEL_FETCHING=y
CONFIG_ZMK_SPLIT_BLE_CENTRAL_BATTERY_LEVEL_PROXY=y
```

hillside_view_enc_left.conf:
```
CONFIG_ZMK_BATTERY=y
CONFIG_ZMK_DISPLAY=y
CONFIG_ZMK_DISPLAY_STATUS_SCREEN_CUSTOM=y
```

События для подписки: zmk_battery_state_changed, zmk_usb_conn_state_changed, zmk_split_peripheral_status_changed, получение battery level периферии через API split fetching.

## Архитектура состояния (status_state, util.h)

```c
struct status_state {
    uint8_t battery;
    bool charging;

#if !IS_ENABLED(CONFIG_ZMK_SPLIT) || IS_ENABLED(CONFIG_ZMK_SPLIT_ROLE_CENTRAL)
    uint8_t peer_battery;
    bool peer_connected;
    int16_t own_minutes_remaining;
    int16_t own_minutes_to_full;
    int16_t peer_minutes_remaining;
    uint8_t own_discharge_rate_pct_per_day;
    uint8_t peer_discharge_rate_pct_per_day;
#endif
};
```

## Требуемые файлы

| Действие | Файл | Назначение |
|---|---|---|
| Создать | widgets/battery_detail.h/.c | отрисовка middle canvas: верх - свой прогноз, низ - peer статус |
| Создать | widgets/battery_history.h/.c | кольцевой буфер, расчет скорости, экстраполяция |
| Изменить | widgets/util.h | новые поля в status_state (central-only) |
| Изменить | widgets/screen.c | подписки, get_state/set_status, draw_middle, init |
| Изменить | CMakeLists.txt (левая сборка) | conditional-блок выбора виджета |
| Изменить | Kconfig.defconfig | CONFIG_NICE_VIEW_GEM_BATTERY_DETAIL |
| Не изменять | конфиги/overlay правой половины | вне scope |

## Расчет прогноза (алгоритм)

1. При zmk_battery_state_changed - сохранять (timestamp, level) в буфер на 10-20 точек, агрегировать по часам.
2. Скорость разряда = (level_старое - level_новое) / время в часах, окно 24-72ч (по умолчанию 48).
3. own_minutes_remaining = current_level / discharge_rate_per_minute. Если rate <= 0 - "нет данных" (-1).
4. При charging - minutes_to_full = (100 - current_level) / charge_rate_per_minute.
5. Для peer - окно усреднения шире (по умолчанию 72ч), т.к. точки поступают реже.
6. Буфер только в RAM, не писать во flash на каждое обновление.
7. Формулировки - "примерно", не создавать ложное ощущение точности.

## Отрисовка (LVGL, 68x68, только левая половина)

- Верх (y=2-32): крупная строка своего прогноза ("10д") + подпись сверху.
- Низ (y=34-64): иконка батареи peer, процент, короткая метка. При peer_connected=false - перечеркнутая иконка + "no link".
- Разделительная линия между зонами.
- При charging - иконка bolt (5x9) + прогноз до полной зарядки вместо прогноза разряда.

## Kconfig-опции

```
config NICE_VIEW_GEM_BATTERY_DETAIL
    bool "Show battery runtime/forecast detail on left half middle canvas"
    default n

config NICE_VIEW_GEM_BATTERY_LOW_THRESHOLD
    int "Battery percentage considered low"
    default 20
    depends on NICE_VIEW_GEM_BATTERY_DETAIL

config NICE_VIEW_GEM_BATTERY_HISTORY_WINDOW_HOURS
    int "Rolling window (hours) for discharge rate averaging"
    default 48
    depends on NICE_VIEW_GEM_BATTERY_DETAIL
```

При BATTERY_DETAIL=y - select ZMK_SPLIT_BLE_CENTRAL_BATTERY_LEVEL_FETCHING/PROXY. Взаимное исключение с MODIFIERS и WPM.

## Критерии приемки

- [ ] Виджет только на левой половине; правая не изменена.
- [ ] Помещается в 68x68px.
- [ ] Виден прогноз (верх) и peer-статус (низ) одновременно.
- [ ] Прогнозы без отрицательных чисел, деления на ноль - вместо этого "нет данных".
- [ ] При обрыве BLE с peer - маркер "нет связи" в течение 1-2 интервалов.
- [ ] Индикатор low-battery для своего и peer значения.
- [ ] BATTERY_DETAIL взаимоисключающий с MODIFIERS/WPM.
- [ ] Сборка make hsv/left в обоих режимах без ошибок; make hsv/right не затронута.
- [ ] Буфер истории только в RAM.
- [ ] Проверена/задокументирована калибровка % заряда против напряжения.
- [ ] Документация обновлена с разделом об ограничениях точности.

## Порядок работы для агента

1. (Опционально) Сверить % заряда с напряжением, откалибровать output-ohms/full-ohms при расхождении >5 п.п.
2. Изучить существующую реализацию battery-индикатора на top canvas.
3. Добавить поля в util.h (central-only секция).
4. Реализовать battery_history.c (чистая логика, без UI).
5. Реализовать battery_detail.c (две зоны отрисовки).
6. Прописать подписки/get_state в screen.c.
7. Обновить CMakeLists.txt (левая сборка) и Kconfig.defconfig.
8. Собрать make hsv/left в обоих режимах, make hsv/right без изменений.
9. Обновить README/Display_CheatSheet.md.
10. Закоммитить пошагово.

## Визуализация макета

Средний канвас 68x68px левой половины: верх - крупный собственный прогноз, низ - статус правой половины (процент, прогноз/предупреждение, либо маркер отсутствия связи). См. приложенный файл battery_widget_left_mockup.png.

Правая половина не изменяется и не визуализирована - вне scope задачи.
