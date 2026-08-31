# Nice!View Gem — Полное руководство (Dual Display)

> **Версия для клавиатур с дисплеями на обеих половинках**

## Содержание
1. [Архитектура dual-display](#архитектура-dual-display)
2. [Структура дисплея](#структура-дисплея)
3. [Где какие файлы править](#где-какие-файлы-править)
4. [Все параметры nice_view_gem](#все-параметры-nice_view_gem)
5. [Готовые пресеты для dual display](#готовые-пресеты-for-dual-display)
6. [Разные виджеты на разных экранах](#разные-виджеты-на-разных-экранах)
7. [Устранение проблем](#устранение-проблем)

---

## Архитектура dual-display

```
┌─────────────────┐          ┌─────────────────┐
│  LEFT (Central) │◄──BLE──►│ RIGHT (Peripheral)│
│                 │          │                 │
│ • nice!view ✅  │          │ • nice!view ✅  │
│ • Подключается   │          │ • Нет USB       │
│   к ПК/телефону  │          │ • Энкодер       │
│ • Энкодер       │          │ • Дисплей        │
└─────────────────┘          └─────────────────┘
```

| Половинка | Роль | Дисплей | Подключение к ПК | Файл конфига |
|-----------|------|---------|------------------|--------------|
| **Левая** | Central (hub) | ✅ Есть | USB или BLE | `hillside_view_enc_left.conf` |
| **Правая** | Peripheral | ✅ Есть | ❌ Нет (только BLE к левой) | `hillside_view_enc_right.conf` |

> **Важно:** Обе половинки имеют дисплеи. Левая подключается к устройству, правая — только к левой по BLE.

---

## Структура дисплея

Каждый nice!view экран (160x68 пикселей) разделён на 3 канваса:

```
┌─────────────────────────────────────────────┐
│ Top Canvas (child 0, y=0)                   │
│  🔋 Battery %   📡 USB/BLE icon            │
├─────────────────────────────────────────────┤
│ Middle Canvas (child 1, y=-44)              │
│  💎 Crystal / 📊 WPM / ⌨️ Mods            │  ← ЗДЕСЬ ВЫБИРАЕМ ВИДЖЕТ
├─────────────────────────────────────────────┤
│ Bottom Canvas (child 2, y=-112)             │
│ 🔵🔵🔵🔵🔵 Profile dots                   │
│ Layer: Default                              │
└─────────────────────────────────────────────┘
```

| Канвас | Что показывает | Настройка |
|--------|----------------|-----------|
| **Top** | Батарея этой половинки + статус подключения | Всегда включён в gem |
| **Middle** | **Основной виджет** (кристалл, WPM, модификаторы) | `CONFIG_NICE_VIEW_GEM_MODIFIERS` |
| **Bottom** | Точки BT-профилей + имя слоя | Всегда включён в gem |

> **На правой половине:** Top canvas показывает батарею правой половинки и статус BLE-связи с левой.

---

## Где какие файлы править

### Файлы конфигурации (`config/`)

| Файл | Для чего | Пример содержимого |
|------|----------|-------------------|
| `hillside_view_enc.conf` | **Общий** для left+right | Имя клавиатуры, BT мощность, энкодеры, Studio, общие настройки gem |
| `hillside_view_enc_left.conf` | **Только левая** (central) | Дисплей, battery, кастомный экран, модификаторы для левого экрана |
| `hillside_view_enc_right.conf` | **Только правая** (peripheral) | Дисплей, battery, кастомный экран, модификаторы для правого экрана |


### Файлы shield (`boards/shields/hillside_view_enc/`)

| Файл | Для чего |
|------|----------|
| `hillside_view_enc.zmk.yml` | Мета-данные shield (описание, siblings) |
| `hillside_view_enc_left.overlay` | Device tree для левой половины (пины, энкодер, display) |
| `hillside_view_enc_right.overlay` | Device tree для правой половины (пины, энкодер, display) |
| `Kconfig.defconfig` | Дефолтные Kconfig значения (split role, display, battery) |
| `Kconfig.shield` | Определение config-опций shield |

> **Оба overlay файла** содержат:
> ```dtsi
> &chosen {
>   zephyr,display = &nice_view;    // Дисплей подключён
> };
> &nice_view_spi {
>   status = "okay";                // SPI шина включена
> };
> &nice_view {
>   status = "okay";                // Дисплей активен
> };
> ```

---

## Все параметры nice_view_gem

### 🔧 Включение/выключение

| Параметр | Значения | Описание | По умолчанию |
|----------|----------|----------|--------------|
| `CONFIG_ZMK_DISPLAY_STATUS_SCREEN_CUSTOM` | `y` / `n` | Использовать кастомный экран (gem) вместо стандартного ZMK | `n` |
| `CONFIG_NICE_VIEW_GEM_MODIFIERS` | `y` / `n` | Показать модификаторы (⌘⌥⌃⇧) на middle canvas | `n` |

> **Если `STATUS_SCREEN_CUSTOM=n`** — все gem-параметры игнорируются, используется стандартный ZMK дисплей.

### 💎 Анимация кристалла

| Параметр | Значения | Описание | По умолчанию |
|----------|----------|----------|--------------|
| `CONFIG_NICE_VIEW_GEM_ANIMATION` | `y` / `n` | Включить/выключить анимацию кристалла | `y` |
| `CONFIG_NICE_VIEW_GEM_ANIMATION_FRAME` | `0` — `16` | Фиксированный кадр (0 = случайный) | `0` |
| `CONFIG_NICE_VIEW_GEM_ANIMATION_MS` | `1` — `999999` | Задержка между кадрами в мс (960 = 60fps, 96000 = ~1 кадр/1.6мин) | `960` |

### 📊 WPM Gauge (Words Per Minute)

| Параметр | Значения | Описание | По умолчанию |
|----------|----------|----------|--------------|
| `CONFIG_NICE_VIEW_GEM_WPM_FIXED_RANGE` | `y` / `n` | Фиксированный диапазон шкалы WPM | `y` |
| `CONFIG_NICE_VIEW_GEM_WPM_FIXED_RANGE_MAX` | `1` — `999` | Максимум шкалы WPM (для калибровки) | `100` |

> **WPM показывается только если `MODIFIERS=n` и `ANIMATION=n`** (или если animation выключена).

### 📌 Полный список параметров

```conf
# === Основные ===
CONFIG_ZMK_DISPLAY_STATUS_SCREEN_CUSTOM=y          # Включить кастомный gem-экран
CONFIG_NICE_VIEW_GEM_MODIFIERS=y                   # Модификаторы вместо WPM/кристалла

# === Анимация кристалла ===
CONFIG_NICE_VIEW_GEM_ANIMATION=y                   # Включить анимацию
CONFIG_NICE_VIEW_GEM_ANIMATION_FRAME=0             # Фиксированный кадр (0=random, 1-16=конкретный)
CONFIG_NICE_VIEW_GEM_ANIMATION_MS=960              # Скорость (мс между кадрами)

# === WPM Gauge ===
CONFIG_NICE_VIEW_GEM_WPM_FIXED_RANGE=y             # Фиксированная шкала WPM
CONFIG_NICE_VIEW_GEM_WPM_FIXED_RANGE_MAX=100       # Максимум WPM на шкале
```

---

## Готовые пресеты for dual display

### Пресет 1: Модификаторы на обоих экранах

**Левый экран (`hillside_view_enc_left.conf`):**
```conf
CONFIG_ZMK_DISPLAY=y
CONFIG_ZMK_DISPLAY_STATUS_SCREEN_CUSTOM=y
CONFIG_NICE_VIEW_GEM_MODIFIERS=y
CONFIG_NICE_VIEW_GEM_ANIMATION=n
CONFIG_ZMK_BATTERY=y
CONFIG_ZMK_BLE=y
```

**Правый экран (`hillside_view_enc_right.conf`):**
```conf
CONFIG_ZMK_DISPLAY=y
CONFIG_ZMK_DISPLAY_STATUS_SCREEN_CUSTOM=y
CONFIG_NICE_VIEW_GEM_MODIFIERS=y
CONFIG_NICE_VIEW_GEM_ANIMATION=n
CONFIG_ZMK_BATTERY=y
CONFIG_ZMK_BLE=y
```

**Результат:** Обе половинки показывают ⌘⌥⌃⇧ при нажатии модификаторов.

---

### Пресет 2: Модификаторы слева, кристалл справа

**Левый экран (`hillside_view_enc_left.conf`):**
```conf
CONFIG_ZMK_DISPLAY=y
CONFIG_ZMK_DISPLAY_STATUS_SCREEN_CUSTOM=y
CONFIG_NICE_VIEW_GEM_MODIFIERS=y
CONFIG_NICE_VIEW_GEM_ANIMATION=n
CONFIG_ZMK_BATTERY=y
CONFIG_ZMK_BLE=y
```

**Правый экран (`hillside_view_enc_right.conf`):**
```conf
CONFIG_ZMK_DISPLAY=y
CONFIG_ZMK_DISPLAY_STATUS_SCREEN_CUSTOM=y
CONFIG_NICE_VIEW_GEM_MODIFIERS=n
CONFIG_NICE_VIEW_GEM_ANIMATION=y
CONFIG_NICE_VIEW_GEM_ANIMATION_MS=960   # 60fps
CONFIG_ZMK_BATTERY=y
CONFIG_ZMK_BLE=y
```

**Результат:**
- Левый: ⌘⌥⌃⇧ модификаторы
- Правый: 💎 летающий кристалл

---

### Пресет 3: WPM слева, модификаторы справа

**Левый экран (`hillside_view_enc_left.conf`):**
```conf
CONFIG_ZMK_DISPLAY=y
CONFIG_ZMK_DISPLAY_STATUS_SCREEN_CUSTOM=y
CONFIG_NICE_VIEW_GEM_MODIFIERS=n
CONFIG_NICE_VIEW_GEM_ANIMATION=n
CONFIG_NICE_VIEW_GEM_WPM_FIXED_RANGE=y
CONFIG_NICE_VIEW_GEM_WPM_FIXED_RANGE_MAX=100
CONFIG_ZMK_BATTERY=y
CONFIG_ZMK_BLE=y
```

**Правый экран (`hillside_view_enc_right.conf`):**
```conf
CONFIG_ZMK_DISPLAY=y
CONFIG_ZMK_DISPLAY_STATUS_SCREEN_CUSTOM=y
CONFIG_NICE_VIEW_GEM_MODIFIERS=y
CONFIG_NICE_VIEW_GEM_ANIMATION=n
CONFIG_ZMK_BATTERY=y
CONFIG_ZMK_BLE=y
```

**Результат:**
- Левый: 📊 WPM счётчик
- Правый: ⌘⌥⌃⇧ модификаторы

---

### Пресет 4: Кристалл на обоих (синхронизированный)

**Оба экрана одинаково:**
```conf
# hillside_view_enc_left.conf и hillside_view_enc_right.conf:
CONFIG_ZMK_DISPLAY=y
CONFIG_ZMK_DISPLAY_STATUS_SCREEN_CUSTOM=y
CONFIG_NICE_VIEW_GEM_MODIFIERS=n
CONFIG_NICE_VIEW_GEM_ANIMATION=y
CONFIG_NICE_VIEW_GEM_ANIMATION_FRAME=5    # Один и тот же кадр
CONFIG_ZMK_BATTERY=y
CONFIG_ZMK_BLE=y
```

**Результат:** Оба экрана показывают кристалл на кадре 5 (статично, но одинаково).

---

### Пресет 5: Разные кадры кристалла (асинхронно)

**Левый экран (`hillside_view_enc_left.conf`):**
```conf
CONFIG_ZMK_DISPLAY=y
CONFIG_ZMK_DISPLAY_STATUS_SCREEN_CUSTOM=y
CONFIG_NICE_VIEW_GEM_MODIFIERS=n
CONFIG_NICE_VIEW_GEM_ANIMATION=n
CONFIG_NICE_VIEW_GEM_ANIMATION_FRAME=3
CONFIG_ZMK_BATTERY=y
CONFIG_ZMK_BLE=y
```

**Правый экран (`hillside_view_enc_right.conf`):**
```conf
CONFIG_ZMK_DISPLAY=y
CONFIG_ZMK_DISPLAY_STATUS_SCREEN_CUSTOM=y
CONFIG_NICE_VIEW_GEM_MODIFIERS=n
CONFIG_NICE_VIEW_GEM_ANIMATION=n
CONFIG_NICE_VIEW_GEM_ANIMATION_FRAME=12
CONFIG_ZMK_BATTERY=y
CONFIG_ZMK_BLE=y
```

**Результат:**
- Левый: кристалл кадр 3
- Правый: кристалл кадр 12

---

### Пресет 6: Стандартный ZMK на обоих (без gem)

**Оба экрана одинаково:**
```conf
CONFIG_ZMK_DISPLAY=y
CONFIG_ZMK_DISPLAY_STATUS_SCREEN_CUSTOM=n
CONFIG_ZMK_BATTERY=y
CONFIG_ZMK_BLE=y
```

**Результат:** Стандартный ZMK дисплей с WPM, батареей, слоем на обеих половинках.

---

## Разные виджеты на разных экранах

### Комбинированные варианты

| Левый экран (Central) | Правый экран (Peripheral) | Для чего подходит |
|----------------------|---------------------------|-------------------|
| ⌘⌥⌃⇧ Модификаторы | 💎 Кристалл | Прагматично + эстетика |
| 📊 WPM | ⌘⌥⌃⇧ Модификаторы | Слева продуктивность, справа контроль |
| 💎 Кристалл | 📊 WPM | Эстетика слева, продуктивность справа |
| ⌘⌥⌃⇧ | ⌘⌥⌃⇧ | Максимальная информативность |
| 💎 Кристалл | 💎 Кристалл | Максимальная эстетика |
| 📊 WPM | 📊 WPM | Дублирование WPM (редко нужно) |
| Стандартный ZMK | ⌘⌥⌃⇧ | Гибридный подход |

### Рекомендации по выбору:

1. **Если ты левша или правша:**
   - Доминантная рука → модификаторы (контроль)
   - Недоминантная → кристалл/WPM (инфо/эстетика)

2. **Если важна продуктивность:**
   - Левый (central) → WPM (основная работа)
   - Правый → модификаторы (контроль модов)

3. **Если важна эстетика:**
   - Оба → кристалл (разные кадры или анимация)

4. **Если нужна максимальная информативность:**
   - Оба → модификаторы (вижу все моды с любого угла)

---

## Устранение проблем

### Дисплей не показывает ничего на одной из половин

```conf
# Проверь в соответствующем _left.conf или _right.conf:
CONFIG_ZMK_DISPLAY=y
CONFIG_ZMK_DISPLAY_STATUS_SCREEN_CUSTOM=y   # или n для стандартного
```

### Модификаторы не показываются

```conf
# Проверь в соответствующем конфиге:
CONFIG_ZMK_DISPLAY_STATUS_SCREEN_CUSTOM=y
CONFIG_NICE_VIEW_GEM_MODIFIERS=y
```

### Кристалл слишком быстрый/медленный

```conf
CONFIG_NICE_VIEW_GEM_ANIMATION_MS=960     # Быстро (60fps)
CONFIG_NICE_VIEW_GEM_ANIMATION_MS=9600    # Средне (~10fps)
CONFIG_NICE_VIEW_GEM_ANIMATION_MS=96000   # Медленно (~1 кадр/1.6мин)
```

### WPM не обновляется

```conf
CONFIG_NICE_VIEW_GEM_MODIFIERS=n          # Модификаторы выключены
CONFIG_NICE_VIEW_GEM_WPM_FIXED_RANGE=y
```

### Дисплей мерцает или тормозит

```conf
CONFIG_ZMK_DISPLAY_WORK_QUEUE_DEDICATED=y  # Выделенный поток для дисплея
```

### Правая половина не показывает батарею

Убедись, что в `hillside_view_enc.conf` (общий конфиг) есть:
```conf
CONFIG_ZMK_SPLIT_BLE_CENTRAL_BATTERY_LEVEL_FETCHING=y
CONFIG_ZMK_SPLIT_BLE_CENTRAL_BATTERY_LEVEL_PROXY=y
```

И в `hillside_view_enc_right.conf`:
```conf
CONFIG_ZMK_BATTERY=y