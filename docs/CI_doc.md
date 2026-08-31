# CI/CD: сборка и релизы прошивки ZMK

Документ описывает, как в этом репозитории устроена автоматическая сборка прошивки и публикация стабильных релизов через GitHub Actions.

## Обзор процессов

В репозитории два независимых workflow:

| Workflow | Файл | Когда запускается | Что делает |
|---|---|---|---|
| Build ZMK firmware | `.github/workflows/build.yml` | push в любую ветку, pull request, вручную | Собирает прошивку ZMK и сохраняет результат как временный Actions artifact |
| Publish stable ZMK firmware | `.github/workflows/release-stable.yml` | push тега `stable` | Собирает прошивку и публикует её как GitHub Release с тегом `stable` |

Оба workflow используют официальный reusable workflow `zmkfirmware/zmk/.github/workflows/build-user-config.yml@main` для компиляции.

## 1. Build ZMK firmware (проверочная сборка)

Файл: `.github/workflows/build.yml`

Триггеры:

```yaml
on:
  push:
    branches:
      - "**"
    tags-ignore:
      - "**"
    paths-ignore:
      - "**.md"
      - "docs/**"
  pull_request:
    paths-ignore:
      - "**.md"
      - "docs/**"
  workflow_dispatch:
```

Особенности:

- `tags-ignore: ["**"]` — исключает пуш тегов из этого workflow, чтобы он не запускался повторно при выпуске stable-релиза (иначе на один и тот же коммит запускались бы два параллельных build).
- `paths-ignore` — пропускает сборку, если изменены только документация/README, т.к. компилировать прошивку в этом случае не нужно.
- `workflow_dispatch` — позволяет запустить сборку вручную из вкладки Actions, независимо от фильтров путей (paths-ignore на ручной запуск не действует).

Результат сборки сохраняется только как Actions artifact и автоматически удаляется через некоторое время (стандартная политика хранения GitHub). Он предназначен для проверки, а не для постоянного хранения.

## 2. Publish stable ZMK firmware (релиз)

Файл: `.github/workflows/release-stable.yml`

Триггер:

```yaml
on:
  push:
    tags:
      - stable
```

Запускается только при пуше тега с именем `stable` — то есть не на каждый коммит, а только когда мы явно решаем "продвинуть" текущую версию в стабильную.

Job состоит из двух частей:

1. `build` — вызывает reusable workflow ZMK и собирает прошивку с именем архива `hsv-stable`.
2. `release` — скачивает собранный артефакт и публикует/обновляет GitHub Release с тегом `stable`.

Ключевые шаги job `release`:

```yaml
steps:
  - name: Checkout repository
    uses: actions/checkout@v4

  - name: Download firmware archive
    uses: actions/download-artifact@v4
    with:
      name: hsv-stable
      path: release

  - name: Create or update the stable GitHub Release
    env:
      GH_TOKEN: ${{ github.token }}
      GH_REPO: ${{ github.repository }}
      TAG: ${{ github.ref_name }}
    run: |
      gh release create "$TAG" \
        --title "Stable firmware" \
        --notes "Stable firmware built from commit ${{ github.sha }}" \
        release/* \
      || gh release upload "$TAG" release/* --clobber
```

Важные детали:

- `actions/checkout@v4` обязателен: без него `gh` не может определить репозиторий и падает с ошибкой `not a git repository`.
- `gh release create ... || gh release upload ... --clobber` — если релиз `stable` уже существует, скрипт не завершается с ошибкой, а обновляет файлы в существующем релизе (`--clobber` заменяет одноимённые ассеты).
- `permissions: contents: write` на уровне workflow обязателен, иначе встроенный `GITHUB_TOKEN` не сможет создавать релизы.

## Как выпустить новую стабильную версию

Тег `stable` — "плавающий": он всегда должен указывать на последний проверенный коммит, поэтому его нужно принудительно передвигать (`--force`).

```powershell
git switch main
git pull --ff-only

# Передвинуть тег stable на текущий коммит
git tag -fa stable -m "Promote current firmware to stable"

# Отправить тег на GitHub (force обязателен, т.к. тег уже существует)
git push origin refs/tags/stable --force
```

После пуша тега:

1. GitHub запускает `Publish stable ZMK firmware`.
2. ZMK собирает прошивку.
3. Workflow публикует/обновляет Release `stable` с приложенным архивом сборки.
4. Файл доступен на странице `Releases -> stable`.

Если нужно выпустить релиз с конкретного (не последнего) коммита:

```powershell
git tag -fa stable <commit-sha> -m "Promote <commit-sha> to stable"
git push origin refs/tags/stable --force
```

## Частые проблемы

| Симптом | Причина | Решение |
|---|---|---|
| В релизе только Source code (zip/tar.gz), нет прошивки | Job `release` упал с ошибкой `not a git repository` | Добавить шаг `actions/checkout@v4` перед `gh release create` |
| При пуше тега `stable` запускаются два workflow одновременно | `build.yml` слушает `push` без ограничений и реагирует на теги тоже | В `build.yml` добавить `tags-ignore: ["**"]` |
| Сборка запускается на правки README | Нет фильтра путей | Добавить `paths-ignore` с `**.md`, `docs/**` |
| `gh release create` возвращает 403 | Недостаточно прав токена | Проверить `permissions: contents: write` в workflow и Settings -> Actions -> General -> Workflow permissions |

## Структура файлов

```text
.github/workflows/
  build.yml            # проверочная сборка на каждый push/PR
  release-stable.yml   # публикация стабильного релиза по тегу stable
```
