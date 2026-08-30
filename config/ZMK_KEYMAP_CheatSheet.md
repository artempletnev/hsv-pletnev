# ZMK Keymap — Шпаргалка

## 1. Поведения (Behaviors)

| Поведение | Синтаксис | Описание |
|-----------|-----------|----------|
| `&kp` | `&kp KEY` | Key Press — обычное нажатие клавиши |
| `&mo` | `&mo N` | Momentary Layer — пока держишь, активен слой N |
| `&to` | `&to N` | To Layer — переключиться на слой N (постоянно) |
| `&sk` | `&sk MOD` | Sticky Key — липкий модификатор (отпускается после след. клавиши) |
| `&mt` | `&mt MOD KEY` | Mod Tap — tap=KEY, hold=MOD (встроенный hold-tap с модом) |
| `&lt` | `&lt N KEY` | Layer Tap — tap=KEY, hold=слой N (встроенный hold-tap со слоем) |
| `&gt` | `&gt MOD N` | Gimballed Tap — tap=слой N, hold=MOD |
| `&trans` | — | Transparent — пропускает нажатие на слой ниже |
| `&none` | — | Нет действия (глушит клавишу) |
| `&bt` | `&bt BT_SEL N` | Bluetooth: выбрать профиль N |
| `&bt` | `&bt BT_CLR` | Bluetooth: очистить все профили |
| `&out` | `&out OUT_TOG` | Переключить вывод USB ↔ Bluetooth |
| `&ext_power` | `&ext_power EP_TOG` | Переключить внешнее питание |
| `&bootloader` | — | Перезагрузка в bootloader |
| `&sys_reset` | — | Системный сброс |
| `&studio_unlock` | — | Разблокировать ZMK Studio |
| `&caps_word` | — | Caps Word — авто-Caps на текущее слово |
| `&rgb_ug` | `&rgb_ug RGB_TOG` | RGB: Toggle |
| `&rgb_ug` | `&rgb_ug RGB_ON` | RGB: Включить |
| `&rgb_ug` | `&rgb_ug RGB_OFF` | RGB: Выключить |
| `&rgb_ug` | `&rgb_ug RGB_HUI` | RGB: Hue Up |
| `&rgb_ug` | `&rgb_ug RGB_HUD` | RGB: Hue Down |
| `&rgb_ug` | `&rgb_ug RGB_SAI` | RGB: Saturation Up |
| `&rgb_ug` | `&rgb_ug RGB_SAD` | RGB: Saturation Down |
| `&rgb_ug` | `&rgb_ug RGB_VAI` | RGB: Brightness Up |
| `&rgb_ug` | `&rgb_ug RGB_VAD` | RGB: Brightness Down |
| `&rgb_ug` | `&rgb_ug RGB_SPI` | RGB: Speed Up |
| `&rgb_ug` | `&rgb_ug RGB_SPD` | RGB: Speed Down |
| `&rgb_ug` | `&rgb_ug RGB_EFF` | RGB: Следующий эффект |
| `&rgb_ug` | `&rgb_ug RGB_EFR` | RGB: Предыдущий эффект |

## 2. Все клавиши (Keys)

### Буквы
```
A B C D E F G H I J K L M N O P Q R S T U V W X Y Z
```

### Цифры
```
N1 N2 N3 N4 N5 N6 N7 N8 N9 N0
```

### Модификаторы
| Код | Значение |
|-----|----------|
| `LSHFT` / `RSHFT` | Left / Right Shift |
| `LCTRL` / `RCTRL` | Left / Right Ctrl |
| `LALT` / `RALT` | Left / Right Alt (RALT = AltGr) |
| `LGUI` / `RGUI` | Left / Right Windows / Cmd (Super) |

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
| `PRINTSCREEN` | Print Screen |
| `SCROLLLOCK` | Scroll Lock |
| `PAUSEBREAK` | Pause/Break |

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
F1 F2 F3 F4 F5 F6 F7 F8 F9 F10 F11 F12 F13 F14 F15 F16 F17 F18 F19 F20 F21 F22 F23 F24
```

### Знаки препинания и символы
| Код | Символ | Код | Символ |
|-----|--------|-----|--------|
| `SEMI` | ; | `COLON` | : |
| `SQT` | ' | `DQT` | " |
| `COMMA` | , | `DOT` | . |
| `FSLH` | / | `BSLH` | \ |
| `PIPE` | | | `GRAVE` | ` |
| `EQUAL` | = | `MINUS` | - |
| `LBKT` | ( | `RBKT` | ) |
| `LBRC` | [ | `RBRC` | ] |
| `LBRACE` | { | `RBRACE` | } |
| `PLUS` | + | `HASH` | # |
| `DOLLAR` / `DLLR` | $ | `PERCENT` / `PRCNT` | % |
| `AMPERSAND` / `AMPS` | & | `ASTERISK` / `KP_MULTIPLY` | * |
| `AT` | @ | `CARET` | ^ |
| `TILDE` | ~ | `EXCLAMATION` / `EXCL` | ! |
| `QUESTION` | ? | `UNDERSCORE` / `UNDER` | _ |

### Медиа-контроль
| Код | Значение |
|-----|----------|
| `C_PP` | Play / Pause |
| `C_NEXT` | Следующий трек |
| `C_PREV` | Предыдущий трек |
| `C_VOL_UP` | Громкость + |
| `C_VOL_DN` | Громкость − |
| `C_MUTE` | Без звука |
| `C_STOP` | Остановить |
| `C_PLAY` | Воспроизвести |
| `C_RECORD` | Запись |

### Яркость (macOS)
| Код | Значение |
|-----|----------|
| `F14` | Яркость − |
| `F15` | Яркость + |

## 3. Комбинации клавиш

### Через `&kp` с модификаторами
```dtsi
&kp C+A             /* Ctrl+A */
&kp S+A             /* Shift+A */
&kp C+S+A           /* Ctrl+Shift+A */
&kp LGUI+A          /* Win/Cmd+A */
&kp LALT+F4         /* Alt+F4 */
&kp C+S+FSLH        /* Ctrl+Shift+/ */
```

Аббревиатуры модификаторов:
- `C` = Ctrl
- `S` = Shift
- `A` = Alt
- `G` = GUI (Win/Cmd)

### Через hold-tap (home-row mods)
```dtsi
/* Определяем hold-tap в behaviors: */
ht: hold_tap {
    compatible = "zmk,behavior-hold-tap";
    #binding-cells = <2>;
    flavor = "tap-preferred";
    tapping-term-ms = <200>;
    bindings = <&kp>, <&kp>;
};

/* В keymap: &ht HOLD_KEY TAP_KEY */
&ht LSHIFT J        /* tap = J, hold = Left Shift */
&ht LCTRL D         /* tap = D, hold = Left Ctrl */
&ht LALT F          /* tap = F, hold = Left Alt */
&ht LGUI G          /* tap = G, hold = Left GUI */
&ht N8 I            /* tap = I, hold = N8 (цифра 8) */
```

## 4. Hold-Tap Flavors

| Flavor | Поведение |
|--------|-----------|
| `"tap-preferred"` | При двойном нажатии → всегда tap. Максимальный приоритет на tap. |
| `"hold-preferred"` | Приоритет на hold. |
| `"balanced"` | Баланс: если за tap-ом последовало быстрое нажатие → tap, иначе hold. |
| `"tap-unless-otherwise"` | Tap, если за ним последовало другое нажатие в рамках tapping-term. |

## 5. Определение пользовательских поведений

```dtsi
behaviors {
    /* Алиас: имя_алиаса: имя_блока { ... }; */

    /* Пример 1: Layer Toggle Quick — tap=клавиша, hold=моментарный слой */
    lq: layer_toggle_quick {
        compatible = "zmk,behavior-hold-tap";
        #binding-cells = <2>;          /* 2 параметра: <слой клавиша> */
        flavor = "tap-preferred";
        tapping-term-ms = <150>;
        quick-tap-ms = <175>;
        bindings = <&mo>, <&kp>;       /* hold=&mo(слой), tap=&kp(клавиша) */
    };
    /* Использование: &lq SYM BSPC → tap=Bspc, hold=слой SYM */

    /* Пример 2: Hold-Tap для двух клавиш */
    ht: hold_tap {
        compatible = "zmk,behavior-hold-tap";
        #binding-cells = <2>;          /* 2 параметра: <hold_key tap_key> */
        flavor = "tap-preferred";
        tapping-term-ms = <200>;
        quick-tap-ms = <175>;
        bindings = <&kp>, <&kp>;       /* оба = Key Press */
    };
    /* Использование: &ht LSHIFT J → tap=J, hold=LSHIFT */

    /* Пример 3: Кастомное поведение с задержкой */
    my_delay {
        compatible = "zmk,behavior";
        #binding-cells = <0>;
        bindings = <&kp>;
    };
};
```

## 6. Определение слоёв

```dtsi
#define DEF 0     /* Базовый слой */
#define SYM 1     /* Слой символов */
#define NUM 2     /* Слой цифр */
#define ADJ 3     /* Слой настроек */
```

Максимум 64 слоя (0–63).

## 7. Комбо (Combos)

```dtsi
combos {
    compatible = "zmk,combos";

    caps {
        timeout-ms = <50>;             /* Время на нажатие всех клавиш комбо */
        key-positions = <17 18>;       /* Физические позиции клавиш */
        bindings = <&caps_word>;       /* Действие */
        /* layers = <DEF>; */          /* (опционально) только на слое DEF */
    };
};
```

## 8. Условные слои (Conditional Layers)

```dtsi
conditional_layers {
    compatible = "zmk,conditional-layers";

    rule_name {
        if-layers = <SYM NUM>;         /* Если активен SYM ИЛИ NUM */
        then-layer = <ADJ>;            /* То включить ADJ */
    };
};
```

## 9. Sticky Key настройки

```dtsi
&sk {
    release-after-ms = <600>;   /* Авто-отпускание через 600 мс */
    quick-release;              /* Отпускание при нажатии любой другой клавиши */
};
```

## 10. Sensor bindings (трекпад/колесо)

```dtsi
sensor-bindings = <
    &inc_dec_kp UP DOWN       /* Вверх/вниз = Up/Down */
    &inc_dec_kp LEFT RIGHT    /* Влево/вправо = Left/Right */
>;
```

## 11. Быстрые примеры

```dtsi
/* Обычная клавиша */
&kp A

/* Модификатор + клавиша */
&kp C+A

/* Home-row mod (hold-tap) */
&ht LSHIFT J

/* Моментарный слой */
&mo SYM

/* Sticky Shift */
&sk LSHFT

/* Переключение слоя */
&to DEF

/* Прозрачная */
&trans

/* Bluetooth профиль */
&bt BT_SEL 0

/* Медиа */
&kp C_PP

/* Bootloader */
&bootloader

/* Кастомный hold-tap: tap=слово, hold=слой */
&lq NUM DEL
```

# Шпаргалка: коды управления RGB Underglow в ZMK

## Подключение
Добавить в начало keymap-файла (или .dtsi, который он инклудит): `#include <dt-bindings/zmk/rgb.h>`

## Требования в Kconfig
В .conf-файле платы должно быть включено `CONFIG_ZMK_RGB_UNDERGLOW=y`, и в devicetree платы должна быть описана LED-strip нода (адресуемые светодиоды).

## Полный список кодов &rgb_ug
| Макрос | Действие | Комментарий |
|---|---|---|
| `RGB_TOG` | Toggle | Включить/выключить подсветку |
| `RGB_ON` | On | Принудительно включить |
| `RGB_OFF` | Off | Принудительно выключить |
| `RGB_HUI` | Hue Increase | Увеличить оттенок цвета |
| `RGB_HUD` | Hue Decrease | Уменьшить оттенок цвета |
| `RGB_SAI` | Saturation Increase | Увеличить насыщенность |
| `RGB_SAD` | Saturation Decrease | Уменьшить насыщенность |
| `RGB_BRI` | Brightness Increase | Увеличить яркость |
| `RGB_BRD` | Brightness Decrease | Уменьшить яркость |
| `RGB_SPI` | Effect Speed Increase | Ускорить анимацию эффекта |
| `RGB_SPD` | Effect Speed Decrease | Замедлить анимацию эффекта |
| `RGB_EFF` | Effect Forward | Следующий эффект по списку |
| `RGB_EFR` | Effect Reverse | Предыдущий эффект по списку |

Важно: макроса `RGB_EFS` не существует.

## Синтаксис в keymap
`&rgb_ug RGB_TOG`, `&rgb_ug RGB_EFF`, `&rgb_ug RGB_BRI` — пример рабочей строки: `&bootloader &rgb_ug RGB_TOG &rgb_ug RGB_EFF &rgb_ug RGB_EFR &rgb_ug RGB_SPI &trans &out OUT_TOG &trans &trans &trans &trans &bootloader`

## Статические цвета через HSB
`&rgb_ug RGB_COLOR_HSB(<hue> <sat> <bri>)` — hue: 0-359, saturation: 0-100, brightness: 0-100 (требует поддержки расширенного синтаксиса в используемой версии ZMK).

## Частые ошибки
- Код без префикса `RGB_` (например `TOG` вместо `RGB_TOG`) — ошибка парсера DTC "expected number or parenthesized expression".
- Опечатка в названии макроса (например `RGB_EFS` вместо `RGB_SPI`) — та же ошибка, несуществующий макрос не разворачивается в число.
- Забытый `#include <dt-bindings/zmk/rgb.h>` — тогда даже корректные коды не резолвятся.

## 12. Полезные ссылки

- Официальная документация: https://zmk.dev/docs/configuring/keymaps
- Список всех клавиш: `dt-bindings/zmk/keys.h`
- Список всех поведений: `dt-bindings/zmk/`