# Управление документацией Voronoi Meshwork

Этот шаблон рассчитан на две параллельные языковые версии документации:

```text
docs/
├─ en/    # английская версия; production/public
├─ ru/    # русская рабочая версия; private/local
└─ readme.md
```

Имена Markdown-файлов в `en` и `ru` одинаковые. Это важно: одинаковая структура упрощает перевод, перекрёстные ссылки и переключение языка на локальном сайте.

В корне проекта находятся два конфигурационных файла:

```text
mkdocs.yml          # production: только английская документация
mkdocs.local.yml    # local preview: English + Русский с переключателем языка
requirements-docs.txt
```

`mkdocs.local.yml` наследует общую структуру, тему и `nav` из `mkdocs.yml`. Поэтому **порядок глав задаётся только один раз — в `mkdocs.yml`**.

---

## 1. Установка окружения и пакетов

Рекомендуется отдельное virtual environment (виртуальное окружение).

Windows CMD / PowerShell:

```powershell
py -m venv .venv
.venv\Scripts\activate
py -m pip install -r requirements-docs.txt
```

В шаблоне используются:

```text
MkDocs 1.x
Material for MkDocs
mkdocs-nav-numbering-plugin
mkdocs-static-i18n
```

Версия MkDocs ограничена `<2`, чтобы шаблон не переключился автоматически на будущую несовместимую ветку MkDocs 2.x.

Если раньше был установлен `mkdocs-enumerate-headings-plugin`, удалять его необязательно: новая конфигурация его просто не использует. Автонумерацию теперь выполняет `mkdocs-nav-numbering-plugin`, потому что он умеет нумеровать одновременно навигацию и заголовки страниц.

---

## 2. Локальный предпросмотр всех языков

Основной рабочий режим для редактирования документации:

```powershell
mkdocs serve -f mkdocs.local.yml -a 127.0.0.1:8001
```

Открыть:

```text
http://127.0.0.1:8001/
```

Английский язык является default locale и открывается в корне сайта:

```text
/
```

Русская версия создаётся по адресу:

```text
/ru/
```

Material показывает переключатель языков в верхней панели. При одинаковых именах страниц переключение языка сохраняет текущую страницу, например:

```text
/coordinate-space/
        ↕
/ru/coordinate-space/
```

В `mkdocs.local.yml` установлено:

```yaml
fallback_to_default: false
```

Поэтому если русская страница отсутствует, вместо незаметной подстановки английского текста будет видно, что перевод действительно отсутствует.

Важно: не включать `navigation.instant` в Material для этого локального многоязычного режима — `mkdocs-static-i18n` указывает его как несовместимый с language switcher.

---

## 3. Предпросмотр production-версии

Чтобы увидеть ровно тот сайт, который планируется публиковать:

```powershell
mkdocs serve -f mkdocs.yml -a 127.0.0.1:8001
```

Этот режим использует только:

```text
docs/en/
```

Русские файлы при такой сборке вообще не входят в `docs_dir`.

---

## 4. Если порт занят или Windows выдаёт WinError 10013

Порт можно заменить на любой свободный, например:

```powershell
mkdocs serve -f mkdocs.local.yml -a 127.0.0.1:8080
```

или:

```powershell
mkdocs serve -f mkdocs.local.yml -a 127.0.0.1:8888
```

Проверить, используется ли порт:

```bat
netstat -ano | findstr :8000
```

Проверить зарезервированные Windows диапазоны портов:

```bat
netsh interface ipv4 show excludedportrange protocol=tcp
```

Права администратора для обычного `mkdocs serve` не требуются.

---

## 5. Порядок страниц: никакой алфавитной сортировки

Порядок документации задаётся вручную в разделе `nav:` файла:

```text
mkdocs.yml
```

Пример:

```yaml
nav:
  - Getting Started:
      - "Introduction": index.md
      - "Key Features": key-features.md
      - "Installation": installation.md
      - "Quick Start": quick-start.md
```

MkDocs не должен самостоятельно выводить страницы в алфавитном порядке, потому что все опубликованные страницы явно перечислены в `nav`.

`mkdocs.local.yml` наследует этот `nav`, поэтому дублировать порядок для локальной версии не нужно.

---

## 6. Автоматическая нумерация глав и разделов

Используется пакет:

```text
mkdocs-nav-numbering-plugin
```

Он уже включён и в production, и в local-конфигурацию:

```yaml
- nav-numbering:
    nav_depth: 4
    heading_depth: 4
    number_h1: true
    number_nav: true
    number_headings: true
    preserve_anchor_ids: true
    separator: "."
```

Нумерация генерируется автоматически и **не записывается в Markdown-файлы**.

При текущей иерархии `nav` результат должен выглядеть примерно так:

```text
1. Getting Started
   1.1 Introduction
   1.2 Key Features
   1.3 Installation
   1.4 Quick Start

2. Core Concepts and Controls
   2.1 Basic Concepts
   2.2 Voronoi Modes
   ...
```

Внутри страницы заголовки также получают иерархические номера, например:

```text
2.5 Coordinate Space
2.5.1 World
2.5.2 Local
2.5.3 None
```

`preserve_anchor_ids: true` сохраняет URL-якоря без числового префикса. Поэтому перекрёстная ссылка остаётся устойчивой после вставки новых глав:

```text
coordinate-space.md#world
```

а не привязывается к текущему номеру главы.

Для `preserve_anchor_ids` уже включено Markdown-расширение:

```yaml
- attr_list
```

---

## 7. Как добавить новую главу

Например, появляется новая глава `Site Groups`.

### Шаг 1. Создать одинаковые файлы в обоих языках

```text
docs/en/site-groups.md
docs/ru/site-groups.md
```

### Шаг 2. Начать каждый файл с H1

English:

```markdown
# Site Groups
```

Русский:

```markdown
# Группы Sites
```

### Шаг 3. Добавить страницу в нужное место `nav` только в `mkdocs.yml`

Например:

```yaml
- Core Concepts and Controls:
    - "Filters": filters.md
    - "Site Groups": site-groups.md
    - "Patterns, Symmetry & Moiré": patterns-symmetry-moire.md
```

### Шаг 4. Добавить русский перевод пункта меню в `mkdocs.local.yml`

В `nav_translations`:

```yaml
Site Groups: Группы Sites
```

На этом всё. Перенумеровывать другие главы не нужно.

---

## 8. Как добавить новый язык

Например, немецкий.

### Создать зеркальное дерево

```text
docs/de/
```

Файлы желательно сохранять с теми же английскими именами:

```text
docs/en/coordinate-space.md
docs/ru/coordinate-space.md
docs/de/coordinate-space.md
```

### Добавить язык в `mkdocs.local.yml`

```yaml
- locale: de
  name: Deutsch
  build: true
  nav_translations:
    Getting Started: Erste Schritte
    Introduction: Einführung
```

После перезапуска локального сервера новый язык появится в language switcher.

Пока публично нужен только английский, `mkdocs.yml` менять не требуется.

---

## 9. Перекрёстные ссылки между страницами

Ссылаться нужно по имени файла, а не по номеру главы.

English:

```markdown
See [Coordinate Space](coordinate-space.md).
```

Русский:

```markdown
См. [Система координат](coordinate-space.md).
```

Ссылка на раздел:

```markdown
See [World mode](coordinate-space.md#world).
```

Номера глав в ссылках использовать нельзя: они автоматически изменяются при перестановке или добавлении страниц.

Для особенно важных разделов можно явно закрепить anchor:

```markdown
## Coordinate Space {#coordinate-space}
```

---

## 10. Изображения и видео

Для каждого языка в шаблоне предусмотрены собственные каталоги:

```text
docs/en/assets/images/
docs/en/assets/video/
docs/ru/assets/images/
docs/ru/assets/video/
```

Так языковые деревья остаются полностью самодостаточными. Если изображение одинаково для двух языков, его можно временно копировать в оба каталога; если позже появятся локализованные подписи, структура уже готова.

Пример изображения:

```markdown
![Voronoi Fracture](assets/images/voronoi-fracture.webp)
```

Пример видео:

```html
<video controls autoplay loop muted>
    <source src="assets/video/moire.webm" type="video/webm">
</video>
```

Для коротких демонстраций обычно предпочтительнее WebM/MP4, чем большой GIF.

---

## 11. Production и local build

Проверить английскую production-сборку:

```powershell
mkdocs build -f mkdocs.yml --strict
```

Результат:

```text
site/
```

Проверить локальную многоязычную сборку:

```powershell
mkdocs build -f mkdocs.local.yml --strict
```

Результат:

```text
site-local/
```

`--strict` полезен перед релизом: предупреждения MkDocs превращаются в ошибки и не дают случайно выпустить документацию с проблемными ссылками или конфигурацией.

---

## 12. Private и public repositories

Рекомендуемая схема проекта:

```text
PRIVATE VM REPOSITORY
├─ source code
├─ docs/
│  ├─ en/
│  ├─ ru/
│  └─ readme.md
├─ mkdocs.yml
├─ mkdocs.local.yml
└─ requirements-docs.txt

             release
                ↓

PUBLIC DOCUMENTATION REPOSITORY
├─ docs/
│  └─ en/
├─ mkdocs.yml
└─ requirements-docs.txt
```

В публичный репозиторий копируется английское дерево **без изменения его структуры**:

```text
private: docs/en/...
public:  docs/en/...
```

`docs/ru/`, `docs/readme.md` и `mkdocs.local.yml` наружу копировать не нужно.

Публичный репозиторий поэтому вообще не содержит русскую документацию, а GitHub Pages строит только английскую release-версию.

---

## 13. Публикация английской версии через GitHub Pages

В публичном docs-репозитории можно выполнить:

```powershell
mkdocs gh-deploy --force
```

Команда использует `mkdocs.yml`, собирает английский сайт и публикует результат в ветку `gh-pages`.

В GitHub Pages для простого варианта выбирается:

```text
Settings
→ Pages
→ Deploy from a branch
→ gh-pages
→ / (root)
```

Позже публикацию можно автоматизировать GitHub Actions, но для первого релиза ручная публикация проще контролируется.

---

## 14. Рекомендуемый release workflow

```text
1. Меняется Voronoi Meshwork.
2. Обновляется docs/en и при необходимости docs/ru.
3. Локально запускается mkdocs.local.yml и проверяются оба языка.
4. Отдельно проверяется mkdocs.yml — ровно тот английский сайт, который уйдёт наружу.
5. Выполняется mkdocs build -f mkdocs.yml --strict.
6. docs/en + mkdocs.yml + requirements-docs.txt копируются в public docs repo.
7. Публичный GitHub Pages пересобирается.
```

---

## 15. Полезные правила проекта

- Не добавлять номера в имена файлов: `coordinate-space.md`, а не `09-coordinate-space.md`.
- Не добавлять номера вручную в `#` / `##` заголовки.
- Не ссылаться на номера глав.
- Порядок страниц менять только в `nav` файла `mkdocs.yml`.
- Для новой страницы создавать одинаковое имя файла во всех поддерживаемых языковых каталогах.
- Для нового языка добавлять новый каталог `docs/<locale>/` и запись в `mkdocs.local.yml`.
- Английскую версию считать публичной release-документацией.
- Русскую версию можно использовать как рабочую/личную и не копировать в public repo.
- Перед релизом запускать английскую сборку с `--strict`.
- Не включать `navigation.instant` в многоязычный local preview.
- Большие GIF по возможности заменять WebM/MP4.

---

## 16. Что находится в этом шаблоне

Шаблон содержит 27 страниц на английском и 27 зеркальных страниц на русском языке. В начале каждой страницы уже есть H1 с названием темы, а ниже — базовые подзаголовки-заглушки, чтобы сразу были видны:

- иерархия навигации;
- автоматическая нумерация;
- правое оглавление текущей страницы;
- переключение English / Русский;
- работа перекрёстных ссылок.

По мере написания документации строки:

```text
_Content to be added._
```

и:

```text
_Содержимое будет добавлено._
```

заменяются реальным содержанием.
