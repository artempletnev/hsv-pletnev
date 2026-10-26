# ZMK Keymap — Шпаргалка

Примеры сверены с официальной документацией ZMK ([zmk.dev](https://zmk.dev/docs/keymaps)); сборка для конкретной платы не выполнялась. Фрагменты ниже — справочный материал: число записей в слое, позиции клавиш и комбо зависят от конкретной клавиатуры. Выводимые символы (буквы, знаки) зависят от раскладки операционной системы.

## 1. Каркас keymap-файла

```dtsi
#include <behaviors.dtsi>               /* стандартные поведения */
#include <dt-bindings/zmk/keys.h>      /* коды клавиш */

/ {
    keymap {
        compatible = "zmk,keymap";
        default_layer {                /* слой 0 — базовый */
            bindings = < ... >;      /* многоточие — заполнитель, не валидный код */
        };
        sym_layer {                     /* слой 1 */
            bindings = < ... >;
        };
    };
};
```

Три места для кода в файле:

1. Узлы и настройки — внутри `/ { ... };`: новые поведения в `behaviors { ... }`, узлы `combos`, `conditional_layers`, а также слои keymap.
2. Привязки клавиш — внутри `bindings = < ... >;` конкретного слоя.
3. Изменение существующего поведения — отдельная запись вида `&sk { release-after-ms = <600>; };` на верхнем уровне файла.

Блоки `behaviors { ... }`, `combos { ... }` и `conditional_layers { ... }` из разделов ниже вставляются внутрь корневого `/ { ... };`, а `sensor-bindings` — свойство узла слоя, рядом с `bindings`.

Опциональные заголовки по мере использования: `#include <dt-bindings/zmk/bt.h>` (Bluetooth), `#include <dt-bindings/zmk/outputs.h>` (выбор вывода), `#include <dt-bindings/zmk/ext_power.h>` (внешнее питание), `#include <dt-bindings/zmk/rgb.h>` (RGB).

## 2. Встроенные поведения

| Поведение | Синтаксис | Описание |
|-----------|-----------|----------|
| `&kp` | `&kp KEY` | Key Press — обычное нажатие клавиши |
| `&mo` | `&mo N` | Momentary Layer — пока держишь, активен слой N |
| `&to` | `&to N` | To Layer — активирует слой N и деактивирует все остальные, кроме базового; не зависит от удержания клавиши |
| `&sk` | `&sk MOD` | Sticky Key — «липкий» модификатор: зажат, пока не будет нажата и отпущена другая клавиша |
| `&sl` | `&sl N` | Sticky Layer — «липкий» слой: активен до следующего нажатия |
| `&mt` | `&mt MOD KEY` | Mod Tap — tap = KEY, hold = MOD |
| `&lt` | `&lt N KEY` | Layer Tap — tap = KEY, hold = слой N |
| `&trans` | — | Transparent — пропускает нажатие на слой ниже |
| `&none` | — | Глушит клавишу: код не отправляется и ниже не передаётся |
| `&caps_word` | — | Caps Word — авто-Shift до конца текущего слова |
| `&bootloader` | — | Перезагрузка в bootloader |
| `&sys_reset` | — | Перезагрузка контроллера |
| `&studio_unlock` | — | Разблокировать ZMK Studio (требует `CONFIG_ZMK_STUDIO=y`) |
| `&out` | `&out OUT_TOG` | Переключить вывод USB ↔ BLE (также `OUT_USB`, `OUT_BLE`) |
| `&ext_power` | `&ext_power EP_TOG` | Переключить внешнее питание (требует `CONFIG_ZMK_EXT_POWER=y`) |
| `&rgb_ug` | `&rgb_ug RGB_TOG` | RGB Underglow — полный список кодов в разделе 12 |

Поведений с меткой `&gt` среди встроенных в ZMK нет: если такой символ встречается в чужом конфиге, это метка пользовательского поведения из модуля или keymap — ищите её определение.

### Bluetooth

Требуется `#include <dt-bindings/zmk/bt.h>` в начале файла. Подробнее — [в документации Bluetooth-поведения](https://zmk.dev/docs/keymaps/behaviors/bluetooth).

| Синтаксис | Действие |
|-----------|----------|
| `&bt BT_SEL N` | Выбрать профиль N |
| `&bt BT_PRV` / `&bt BT_NXT` | Предыдущий / следующий профиль |
| `&bt BT_CLR` | Очистить сопряжение выбранного профиля |
| `&bt BT_CLR_ALL` | Очистить сопряжения всех профилей |

## 3. Основные коды клавиш

Полный список — в [официальном справочнике кодов](https://zmk.dev/docs/keymaps/list-of-keycodes) (`dt-bindings/zmk/keys.h`).

### Буквы и цифры

```
A B C D E F G H I J K L M N O P Q R S T U V W X Y Z
N1 N2 N3 N4 N5 N6 N7 N8 N9 N0
```

### Модификаторы

| Код | Значение |
|-----|----------|
| `LSHFT` / `RSHFT` | Left / Right Shift |
| `LCTRL` / `RCTRL` | Left / Right Ctrl |
| `LALT` / `RALT` | Left / Right Alt (`RALT` = AltGr) |
| `LGUI` / `RGUI` | Left / Right Win / Cmd (Super) |

Для комбинаций с клавишами используйте функции модификаторов (раздел 4), а не сами эти коды.

### Спецклавиши

| Код | Значение |
|-----|----------|
| `SPACE` | Пробел |
| `TAB` | Tab |
| `RET` | Enter |
| `ESC` | Escape |
| `BSPC` | Backspace |
| `DEL` | Delete |
| `INS` | Insert |
| `CAPSLOCK` | Caps Lock |
| `PRINTSCREEN` (`PSCRN`) | Print Screen |
| `SCROLLLOCK` (`SLCK`) | Scroll Lock |
| `PAUSE_BREAK` | Pause/Break |

### Навигация

| Код | Значение |
|-----|----------|
| `UP` / `DOWN` | Стрелка вверх / вниз |
| `LEFT` / `RIGHT` | Стрелка влево / вправо |
| `HOME` | Home |
| `END` | End |
| `PG_UP` | Page Up |
| `PG_DN` | Page Down |

### F-клавиши

```
F1 ... F12 (далее до F24)
```

### Знаки препинания и символы

В первой колонке — каноническое имя, в скобках — алиасы.

| Код | Символ | Код | Символ |
|-----|--------|-----|--------|
| `SEMICOLON` (`SEMI`) | ; | `COLON` | : |
| `SINGLE_QUOTE` (`SQT`) | ' | `DOUBLE_QUOTES` (`DQT`) | " |
| `COMMA` | , | `PERIOD` (`DOT`) | . |
| `SLASH` (`FSLH`) | / | `BACKSLASH` (`BSLH`) | \\ |
| `PIPE` | \| | `GRAVE` | ` |
| `EQUAL` | = | `MINUS` | - |
| `LEFT_PARENTHESIS` (`LPAR`) | ( | `RIGHT_PARENTHESIS` (`RPAR`) | ) |
| `LEFT_BRACKET` (`LBKT`) | [ | `RIGHT_BRACKET` (`RBKT`) | ] |
| `LEFT_BRACE` (`LBRC`) | { | `RIGHT_BRACE` (`RBRC`) | } |
| `PLUS` | + | `HASH` (`POUND`) | # |
| `DOLLAR` (`DLLR`) | $ | `PERCENT` (`PRCNT`) | % |
| `AMPERSAND` (`AMPS`) | & | `ASTERISK` (`ASTRK`) | * |
| `AT_SIGN` (`AT`) | @ | `CARET` | ^ |
| `TILDE` | ~ | `EXCLAMATION` (`EXCL`) | ! |
| `QUESTION` (`QMARK`) | ? | `UNDERSCORE` (`UNDER`) | _ |

Примечания:

- `KP_MULTIPLY` — это клавиша «умножить» на нумпаде, отдельный код, а не алиас `ASTERISK`.
- Для не-US-раскладок есть `NON_US_BACKSLASH` (`NUBS`), `NON_US_HASH` (`NUHS`), а также `PIPE2` и `TILDE2`.

### Медиа и яркость

| Код | Значение |
|-----|----------|
| `C_PP` | Play / Pause |
| `C_PLAY` | Воспроизвести |
| `C_STOP` | Остановить |
| `C_RECORD` | Запись |
| `C_NEXT` / `C_PREV` | Следующий / предыдущий трек |
| `C_VOL_UP` / `C_VOL_DN` | Громкость + / − |
| `C_MUTE` | Без звука |
| `C_BRIGHTNESS_INC` (`C_BRI_UP`) | Яркость экрана + |
| `C_BRIGHTNESS_DEC` (`C_BRI_DN`) | Яркость экрана − |

## 4. Модификаторы и комбинации

Сочетания записываются функциями модификаторов, а не через `+`:

```dtsi
&kp LC(A)          /* Ctrl+A */
&kp LS(A)          /* Shift+A */
&kp LC(LS(A))      /* Ctrl+Shift+A */
&kp LG(A)          /* Win/Cmd+A */
&kp LA(F4)         /* Alt+F4 */
&kp LC(LS(FSLH))   /* Ctrl+Shift+/ */
```

| Функция | Модификатор |
|---------|-------------|
| `LC` / `RC` | Left / Right Ctrl |
| `LS` / `RS` | Left / Right Shift |
| `LA` / `RA` | Left / Right Alt |
| `LG` / `RG` | Left / Right GUI (Win/Cmd) |

Функции вкладываются друг в друга и работают также с `&sk`, `&mt` и другими поведениями, например `&sk LG(LS(LA(LCTRL)))`.

## 5. Hold-Tap: flavors и параметры

Hold-tap — поведение с двумя действиями: tap (короткое нажатие) и hold (удержание). Встроенные варианты: `&mt` (hold = модификатор) и `&lt` (hold = слой). Примеры home-row mods:

```dtsi
&mt LSHIFT J     /* tap = J, hold = Left Shift */
&mt LCTRL D      /* tap = D, hold = Left Ctrl */
&lt NUM SPC      /* tap = Space, hold = слой NUM */
```

### Flavors

| Flavor | Когда выбирается hold |
|--------|----------------------|
| `hold-preferred` | Когда истёк `tapping-term-ms` либо нажата другая клавиша, пока hold-tap удерживается. Значение по умолчанию для `&mt` |
| `balanced` | Когда истёк `tapping-term-ms` либо другая клавиша нажата и отпущена, пока hold-tap удерживается |
| `tap-preferred` | Когда истёк `tapping-term-ms`; нажатия других клавиш внутри срока на решение не влияют. Значение по умолчанию для `&lt` |
| `tap-unless-interrupted` | Только если другая клавиша нажата до истечения `tapping-term-ms`; во всех остальных случаях — tap |

### Параметры

- `tapping-term-ms` — порог «нажатие или удержание», по умолчанию `200`.
- `quick-tap-ms` — если после tap клавиша нажата повторно в течение этого интервала от первого нажатия, всегда срабатывает tap. По умолчанию отключено.

Подробнее — [в документации hold-tap](https://zmk.dev/docs/keymaps/behaviors/hold-tap).

## 6. Пользовательские поведения

Метка узла — это `имя_метки:` перед именем узла; на неё ссылаются через `&имя_метки`. Каждое поведение определяется один раз, затем используется во всех слоях. Все три примера ниже живут в одном узле `behaviors` — новые определения добавляйте в него, не создавая повторный узел.

```dtsi
behaviors {
    /* Свой hold-tap (свой flavor и timing) */
    hmods: home_row_mods {
        compatible = "zmk,behavior-hold-tap";
        #binding-cells = <2>;        /* два параметра: <hold tap> */
        flavor = "tap-preferred";
        tapping-term-ms = <200>;
        quick-tap-ms = <175>;
        bindings = <&kp>, <&kp>;    /* hold = &kp, tap = &kp */
    };
    /* Использование: &hmods LSHIFT J → tap = J, hold = Left Shift */

    /* Моментарный слой + клавиша */
    lq: layer_tap_quick {
        compatible = "zmk,behavior-hold-tap";
        #binding-cells = <2>;        /* два параметра: <слой клавиша> */
        flavor = "tap-preferred";
        tapping-term-ms = <150>;
        bindings = <&mo>, <&kp>;    /* hold = &mo(слой), tap = &kp(клавиша) */
    };
    /* Использование: &lq NUM DEL → tap = Delete, hold = слой NUM */

    /* Последовательность действий с интервалом (макрос) */
    seq: macro_sequence {
        compatible = "zmk,behavior-macro";
        #binding-cells = <0>;
        wait-ms = <100>;            /* пауза 100 мс между действиями */
        bindings = <&kp A>, <&kp B>;
    };
    /* Использование: &seq */
};
```

`wait-ms` задаёт интервал между действиями внутри макроса, а не задержку перед его запуском. Для одиночных сочетаний вроде Ctrl+Tab макрос не нужен — достаточно `&kp LC(TAB)`. Подробнее — [в документации макросов](https://zmk.dev/docs/keymaps/behaviors/macros).

## 7. Слои и их нумерация

```dtsi
#define DEF 0     /* Базовый слой */
#define SYM 1     /* Слой символов */
#define NUM 2     /* Слой цифр */
#define ADJ 3     /* Слой настроек */
```

- `#define` лишь даёт имена числам — сами слои создаются дочерними узлами `keymap`. Определения размещаются на верхнем уровне файла, до первого использования имён.
- Номер слоя — это его позиция по порядку определения: первый узел — слой 0, второй — 1 и т.д.
- Базовый слой (0) всегда активен. Несколько слоёв могут быть активны одновременно; при нажатии клавиши срабатывает самый верхний активный слой, откуда событие может уйти ниже через `&trans`.

## 8. Комбо

```dtsi
combos {
    compatible = "zmk,combos";

    caps {
        timeout-ms = <50>;          /* все клавиши должны быть нажаты за это время */
        key-positions = <17 18>;    /* физические индексы позиций — зависят от платы */
        bindings = <&caps_word>;    /* действие */
        layers = <DEF>;             /* (опционально) только на слое DEF */
    };
};
```

Дополнительные свойства: `require-prior-idle-ms` — комбо сработает, только если перед этим была пауза в наборе (защита от ложных срабатываний); `slow-release` — отпускание комбо при отпускании всех клавиш, а не любой из них. Подробнее — [в документации комбо](https://zmk.dev/docs/config/combos).

## 9. Условные слои

```dtsi
conditional_layers {
    compatible = "zmk,conditional-layers";

    both_layers {
        if-layers = <SYM NUM>;   /* когда активны SYM И NUM одновременно */
        then-layer = <ADJ>;       /* включить ADJ */
    };
};
```

Условие проверяется по логическому И: `then-layer` активен, только пока активны все слои из `if-layers`.

## 10. Sticky Key: настройки

```dtsi
&sk {
    release-after-ms = <600>;   /* отпустить модификатор через 600 мс, если не нажата другая клавиша (по умолчанию 1000) */
    quick-release;             /* отпускать при нажатии следующей клавиши, а не при её отпускании */
};
```

Подробнее — [в документации sticky key](https://zmk.dev/docs/keymaps/behaviors/sticky-key).

- По умолчанию sticky key зажата, пока другая клавиша не будет нажата и отпущена; `quick-release` сдвигает этот момент на нажатие — полезно при быстром наборе, чтобы не получить две заглавные подряд.
- `ignore-modifiers` (включён по умолчанию) — нажатие другого модификатора не снимает sticky key, что позволяет комбинировать `&sk LCTRL` + `&sk LSHIFT` + `&kp A` → Ctrl+Shift+A. Вложенные функции вида `&sk LS(LALT)` требуют этого флага.

## 11. Энкодеры (sensor bindings)

`sensor-bindings` указывается в узле слоя, рядом с `bindings`. Подробнее — [в документации энкодеров](https://zmk.dev/docs/features/encoders).

```dtsi
sensor-bindings = <&inc_dec_kp C_VOL_UP C_VOL_DN>;
/* первый параметр — по часовой, второй — против часовой */
```

## 12. RGB Underglow

Требования: в `.conf` платы включено `CONFIG_ZMK_RGB_UNDERGLOW=y`, а в devicetree описана LED-strip нода с адресуемыми светодиодами.

В начало keymap-файла: `#include <dt-bindings/zmk/rgb.h>`

### Коды

| Макрос | Действие |
|--------|----------|
| `RGB_TOG` | Включить/выключить подсветку |
| `RGB_ON` / `RGB_OFF` | Принудительно включить / выключить |
| `RGB_HUI` / `RGB_HUD` | Оттенок: больше / меньше |
| `RGB_SAI` / `RGB_SAD` | Насыщенность: больше / меньше |
| `RGB_BRI` / `RGB_BRD` | Яркость: больше / меньше |
| `RGB_SPI` / `RGB_SPD` | Скорость эффекта: быстрее / медленнее |
| `RGB_EFF` / `RGB_EFR` | Следующий / предыдущий эффект |

Макроса `RGB_EFS` не существует.

Подробнее — [в документации RGB Underglow](https://zmk.dev/docs/keymaps/behaviors/underglow).

### Статический цвет

```dtsi
&rgb_ug RGB_COLOR_HSB(128, 100, 100)
/* hue: 0–359, saturation: 0–100, brightness: 0–100; аргументы через запятую */
```

### Частые ошибки

- Забыт `#include <dt-bindings/zmk/rgb.h>` — даже корректные коды не резолвятся.
- Код без префикса `RGB_` (например, `TOG`) или опечатка в названии — ошибка парсера devicetree вида «expected number or parenthesized expression».

## 13. Быстрые примеры

```dtsi
/* Обычная клавиша */
&kp A

/* Модификатор + клавиша */
&kp LC(A)

/* Home-row mod */
&mt LSHIFT J

/* Моментарный слой */
&mo SYM

/* Sticky Shift */
&sk LSHFT

/* Активировать слой и выключить остальные */
&to DEF

/* Прозрачная клавиша */
&trans

/* Bluetooth: выбрать профиль 0 */
&bt BT_SEL 0

/* Медиа */
&kp C_PP

/* Bootloader */
&bootloader

/* Слой + клавиша: tap = Delete, hold = слой NUM */
&lt NUM DEL
```

## 14. Полезные ссылки

- Кеймпы и поведения: https://zmk.dev/docs/keymaps
- Справочник кодов клавиш: https://zmk.dev/docs/keymaps/list-of-keycodes
- Модификаторы: https://zmk.dev/docs/keymaps/modifiers
- Hold-tap: https://zmk.dev/docs/keymaps/behaviors/hold-tap
- Sticky key: https://zmk.dev/docs/keymaps/behaviors/sticky-key
- Макросы: https://zmk.dev/docs/keymaps/behaviors/macros
- Комбо: https://zmk.dev/docs/config/combos
- Bluetooth: https://zmk.dev/docs/keymaps/behaviors/bluetooth
- RGB Underglow: https://zmk.dev/docs/keymaps/behaviors/underglow
- Энкодеры: https://zmk.dev/docs/features/encoders
- Конфигурация keymap: https://zmk.dev/docs/config/keymap
