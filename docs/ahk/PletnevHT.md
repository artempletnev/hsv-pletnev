#Requires AutoHotkey v2.0

; Стартовое значение — поставь то что активно когда запускаешь скрипт
global CurrentLang := "EN"

; ========== ЯЗЫК ==========
F23:: {
    global CurrentLang
    if (CurrentLang != "RU") {
        Send "{LAlt down}{LShift down}{LShift up}{LAlt up}"
        Sleep(80)
        CurrentLang := "RU"
    }
    TrayTip("Язык", "RU", 1)
}

F24:: {
    global CurrentLang
    if (CurrentLang != "EN") {
        Send "{LAlt down}{LShift down}{LShift up}{LAlt up}"
        Sleep(80)
        CurrentLang := "EN"
    }
    TrayTip("Язык", "EN", 1)
}

; ========== СИМВОЛЫ ==========
F13::SendInput "{Text}@"
F14::SendInput "{Text}$"
F15::SendInput "{Text}^"
F16::SendInput "{Text}#"
F17::SendInput "{Text}|"
F18::SendInput "{Text}~"
F19::SendInput "{Text}["
F20::SendInput "{Text}]"
F21::SendInput "{Text}{"
F22::SendInput "{Text}}"


