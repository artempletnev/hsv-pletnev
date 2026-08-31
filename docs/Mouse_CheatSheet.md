# ZMK: настройка мыши (Pointing) и её ограничения с энкодерами

## 1. TL;DR — что нужно, чтобы мышь просто работала

1. В `.conf` файле (обычно `config/<keyboard>.conf`) добавить:
   ```
   CONFIG_ZMK_POINTING=y
   ```
2. В `.keymap` файле подключить заголовок:
   ```c
   #include <dt-bindings/zmk/pointing.h>
   ```
3. Привязать поведения к обычным клавишам (НЕ к энкодеру):
   ```c
   &mkp LCLK   // левый клик
   &mkp RCLK   // правый клик
   &mkp MCLK   // средний клик
   &mmv MOVE_UP
   &mmv MOVE_DOWN
   &mmv MOVE_LEFT
   &mmv MOVE_RIGHT
   &msc SCRL_UP
   &msc SCRL_DOWN
   ```
4. Пересобрать и прошить firmware на обе половины (если сплит).

Это включает базовый HID mouse report и три поведения: клики, движение курсора, скролл [web:230][web:232].

## 2. Как это работает "под капотом"

Когда `CONFIG_ZMK_POINTING=y` включён, ZMK добавляет в HID-дескриптор дополнительный mouse-report — без этого флага хост (Windows/macOS/Linux/Android) физически не получает данных о движении курсора или кликах, даже если behavior привязан к клавише [web:227][web:231].

### Доступные поведения

| Behavior | Назначение | Ссылка |
|---|---|---|
| `&mkp` | Клик кнопкой мыши (`LCLK`, `RCLK`, `MCLK`) | [web:232] |
| `&mmv` | Движение курсора (`MOVE_UP/DOWN/LEFT/RIGHT`, либо `MOVE_X(n)`/`MOVE_Y(n)` для кастомной скорости) | [web:230] |
| `&msc` | Скролл (`SCRL_UP/DOWN/LEFT/RIGHT`, либо `MOVE_X(n)`/`MOVE_Y(n)`) | [web:230] |

## 3. Все связанные Kconfig-параметры (в `.conf` файле)

| Параметр | Тип | По умолчанию | Что делает |
|---|---|---|---|
| `CONFIG_ZMK_POINTING` | bool | `n` | Главный флаг — включает mouse HID report и весь pointing-стек [web:227] |
| `CONFIG_ZMK_POINTING_SMOOTH_SCROLLING` | bool | `n` | Включает плавный скролл через HID Resolution Multipliers — без него скролл идёт "по шагам", с ним — плавнее, как у нативной мыши [web:227] |

Если ваша плата **аппаратно** содержит pointing-устройство (трекпад/трекбол на самой PCB), `CONFIG_ZMK_POINTING` может задаваться автоматически в `Kconfig.defconfig` самого shield-файла вместе с протоколом связи (SPI/I2C) — но для обычных клавиатур без встроенного сенсора этот флаг нужно ставить вручную в user-конфиге [web:237].

## 4. Настройка скорости движения и скролла

По умолчанию скорости довольно скромные. Их можно переопределить **до** `#include` заголовка pointing.h, прямо в `.keymap` файле:

```c
#define ZMK_POINTING_DEFAULT_MOVE_VAL 1500  // default: 600 — скорость курсора
#define ZMK_POINTING_DEFAULT_SCRL_VAL 20    // default: 10  — скорость скролла
#include <dt-bindings/zmk/pointing.h>
```

Дополнительно можно настроить время набора максимальной скорости и "кривую" ускорения через devicetree-переопределение самого behavior:

```c
&mmv {
    time-to-max-speed-ms = <400>;   // default: 300 — время до максимальной скорости
    acceleration-exponent = <1>;    // кривая ускорения (0 = линейно)
};
```

Аналогично можно переопределить `&msc` (или `&mwh` в некоторых форках) для скролла [web:230][web:240].

## 5. Ограничения энкодеров (sensor-rotate) — почему мышь через них работает нестабильно

### Архитектурная причина: issue #1494

ZMK разделяет поведения на два типа: "глобальные" и поведения с **"locality"** — то есть жёстко привязанные к конкретной физической половине сплит-клавиатуры (например, `&rgb_ug`, управляющий подсветкой именно той половины, где произошло нажатие). Официально подтверждённое архитектурное ограничение (issue #1494, упомянутое в ZMK State of the Firmware #6) гласит:

> "Currently behaviors that have 'locality' such as `&rgb_ug` do not work as expected via encoder rotation bindings in split keyboards, due to issue #1494" [web:250]

На практике это же ограничение по опыту сообщества затрагивает и mouse-behaviors (`&mmv`, `&msc`), когда они привязаны через `sensor-bindings` на периферийной (не-центральной) половине сплита — сенсорные события идут по отдельному пути обработки, чем обычные keypress-события, и не всегда корректно долетают до центральной половины, которая отвечает за отправку HID-отчётов хосту.

### Обходной путь

Привязывайте `&msc`/`&mmv` напрямую к **обычным клавишам** (`bindings`), а не к энкодеру (`sensor-bindings`). Через клавиши mouse-behaviors работают надёжно на любой половине сплита — проблема специфична именно для sensor-rotate.

## 6. Настройка самого энкодера (независимо от проблемы с мышью)

Чтобы энкодер в принципе работал (для любых behavior, не только мыши):

1. В `.conf` файле включить драйвер EC11:
   ```
   CONFIG_EC11=y
   CONFIG_EC11_TRIGGER_GLOBAL_THREAD=y
   ```
   [web:252]

2. В `.overlay`/`.dtsi` задать `steps` или `triggers-per-rotation` в соответствии с датчиком (обычно равно количеству физических щелчков-детентов на полный оборот, либо 4x от этого значения — уточняется по datasheet конкретного энкодера) [web:239].

3. Привязать поведение в `sensor-bindings`:
   ```c
   sensor-bindings = <&inc_dec_kp C_VOL_UP C_VOL_DN>;
   ```
   [web:252]

### Известная проблема: энкодер работает только если он на центральной половине

Если энкодер физически стоит на периферийной (не-центральной) половине сплита, а не на той, что подключена к компьютеру, есть отдельный документированный issue (#2301) с похожей природой — устройство должно быть на стороне, которая передаёт sensor-события правильно; workaround — сделать половину с энкодером центральной через `CONFIG_ZMK_SPLIT_BLE_ROLE_CENTRAL=y` в её `.conf` файле [web:248][web:243].

### Настройка задержки тапа для энкодера при использовании с мышью (если всё же пробуете)

Если решите рискнуть привязать `&msc` к энкодеру несмотря на ограничения, необходимо увеличить `tap-ms`, иначе хост не распознает событие как скролл:

```c
scroll_encoder: scroll_encoder {
    compatible = "zmk,behavior-sensor-rotate";
    #sensor-binding-cells = <0>;
    tap-ms = <20>;              // default слишком мал для распознавания хостом
    bindings = <&msc SCRL_DOWN>, <&msc SCRL_UP>;
};
```

Также рекомендуется увеличить `ZMK_POINTING_DEFAULT_SCRL_VAL` (например до 140), иначе одного тапа энкодера будет недостаточно для заметного скролла [web:245].

## 7. Сводная таблица всех параметров

| Файл | Параметр | Значение | Эффект |
|---|---|---|---|
| `.conf` | `CONFIG_ZMK_POINTING` | `y` | Включает mouse HID report — обязательно |
| `.conf` | `CONFIG_ZMK_POINTING_SMOOTH_SCROLLING` | `y` | Плавный скролл вместо шагового |
| `.conf` | `CONFIG_EC11` | `y` | Драйвер энкодера |
| `.conf` | `CONFIG_EC11_TRIGGER_GLOBAL_THREAD` | `y` | Требуется для работы EC11-драйвера |
| `.conf` | `CONFIG_ZMK_SPLIT_BLE_ROLE_CENTRAL` | `y` | Делает эту половину центральной (workaround для энкодера на периферии) |
| `.keymap` (define) | `ZMK_POINTING_DEFAULT_MOVE_VAL` | число (default 600) | Максимальная скорость курсора |
| `.keymap` (define) | `ZMK_POINTING_DEFAULT_SCRL_VAL` | число (default 10) | Максимальная скорость скролла |
| `.keymap` (`&mmv{}`) | `time-to-max-speed-ms` | мс (default 300) | Время набора максимальной скорости движения |
| `.keymap` (`&mmv{}`) | `acceleration-exponent` | 0/1/2... | Кривая ускорения (0 = линейно) |
| `.dtsi`/`.overlay` | `steps` / `triggers-per-rotation` | по datasheet | Точность распознавания щелчков энкодера |
| `.dtsi` (sensor-rotate) | `tap-ms` | мс (например 20) | Длительность "тапа" для событий энкодера — критично для scroll через sensor |

## Полезные ссылки

- Официальная документация Pointing Configuration: https://zmk.dev/docs/config/pointing
- Официальная документация Mouse Emulation Behaviors: https://zmk.dev/docs/keymaps/behaviors/mouse-emulation
- Issue #1494 (упомянут в ZMK SOTF #6, корень проблемы с locality-behaviors на энкодерах): https://github.com/zmkfirmware/zmk/issues/1494
- Issue #2944 (проблемы при добавлении mouse scroll encoder): https://github.com/zmkfirmware/zmk/issues/2944
- Issue #2301 (энкодер не работает, если только на периферийной половине): https://github.com/zmkfirmware/zmk/issues/2301
- ZMK State of the Firmware #6 (официальное упоминание ограничения): https://zmk.dev/blog/2023/10/05/zmk-sotf-6
