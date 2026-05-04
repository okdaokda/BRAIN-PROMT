# 100% исполнительный prompt для Codex: локальная AI Knowledge Base без Obsidian

Скопируй блок ниже в Codex в пустой папке. Это НЕ обучающий prompt. Он заставляет Codex **самому всё создать, установить, прогнать, проверить и отдать только готовый результат**.

---

```text
ТЫ НЕ КОНСУЛЬТАНТ. ТЫ ИСПОЛНИТЕЛЬ.

Твоя задача — НЕ объяснить мне, как сделать локальную AI Knowledge Base, а СДЕЛАТЬ ЕЁ ПОД КЛЮЧ в текущей папке.

ЗАПРЕЩЕНО:
- писать мне “выполни команду ...” вместо выполнения;
- давать инструкцию до того, как всё создано и проверено;
- останавливаться после создания структуры без smoke-тестов;
- делать placeholder-файлы вместо рабочего CLI;
- спрашивать подтверждение, если можно выбрать безопасный дефолт;
- завершать работу, пока базовый сценарий не прошёл.

ОБЯЗАН:
1. Сам создать проект `AI_Knowledge_Base`.
2. Сам создать все папки и файлы.
3. Сам написать рабочий `scripts/kb.py`.
4. Сам создать `.venv`.
5. Сам поставить зависимости.
6. Сам запустить smoke-тесты.
7. Если что-то упало — сам исправить и повторить.
8. Сам проверить, что файлы реально появились.
9. В конце отдать только короткий отчёт: путь, что готово, что проверено, как пользоваться дальше.

Если среда не даёт поставить зависимости — не останавливайся. Сделай fallback-режим на стандартной библиотеке Python, зафиксируй это в `projects/reports/setup-report.md` и всё равно прогони smoke-тесты.

Цель системы:
Создать локальную AI Knowledge Base без Obsidian, куда пользователь сможет добавлять YouTube-ссылки, PDF, HTML, TXT, Markdown, DOCX, видео/аудио и папки курсов. Система должна сохранять оригиналы, извлекать текст, делать summary, обновлять wiki/indexes и позволять Codex отвечать по базе.

Работай в текущей папке.

Создай структуру:

AI_Knowledge_Base/
  README.md
  AGENTS.md
  index.md
  requirements.txt
  .gitignore

  inbox/
    README.md

  sources/
    youtube/
    pdf/
    html/
    text/
    media/
    courses/
    other/

  extracts/
    transcripts/
    pdf/
    html/
    text/
    media/
    docx/

  knowledge/
    wiki/
    summaries/
    playbooks/
    concepts/
    people/
    glossary.md

  indexes/
    root.md
    sources.md
    topics.md
    personas.md

  personas/
    default.md
    strict-source.md
    teacher.md
    strategist.md

  projects/
    imports/
    reports/
    manifests/

  scripts/
    kb.py

Смысл папок:
- `inbox/` — временный вход для новых материалов.
- `sources/` — оригиналы, которые нельзя удалять без прямой команды.
- `extracts/` — извлечённый текст.
- `knowledge/` — обработанные знания: summaries, wiki, concepts, playbooks.
- `indexes/` — навигация для агента.
- `personas/` — режимы ответа.
- `projects/imports/` — отчёты по импорту.
- `projects/reports/` — отчёты установки/проверок.

Главный файл: `AI_Knowledge_Base/scripts/kb.py`.
Он должен быть РЕАЛЬНО РАБОЧИМ CLI, а не заглушкой.

Команды CLI:

`python scripts/kb.py init`
- проверяет и создаёт недостающую структуру.

`python scripts/kb.py add <path_or_url>`
- добавляет файл, папку или URL в базу.
- YouTube URL → `sources/youtube/` + `extracts/transcripts/` или fallback note.
- PDF → `sources/pdf/` + `extracts/pdf/`.
- HTML → `sources/html/` + `extracts/html/`.
- TXT/MD → `sources/text/` + `extracts/text/`.
- DOCX → `sources/other/` + `extracts/docx/`.
- MP4/MP3/M4A/WAV/MOV → `sources/media/` + `extracts/media/` fallback/transcript note.
- папка курса → `sources/courses/` + manifest + обработка поддерживаемых файлов.

`python scripts/kb.py process`
- обрабатывает всё из `inbox/`.

`python scripts/kb.py status`
- показывает количество sources, extracts, summaries, wiki, manifests, errors.

`python scripts/kb.py rebuild-index`
- пересобирает `indexes/root.md`, `indexes/sources.md`, `indexes/topics.md`, `index.md`.

`python scripts/kb.py search "запрос"`
- простой локальный поиск по `.md`, `.txt`, `.json`.

Требования к `kb.py`:
1. Python 3.
2. Основа — стандартная библиотека, чтобы fallback работал всегда.
3. Делай sha256 для файлов.
4. Не импортируй дубликаты повторно.
5. Для каждого импорта создавай JSON manifest в `projects/imports/`.
6. Для каждого источника создавай markdown summary в `knowledge/summaries/`.
7. Summary должен содержать:
   - title
   - source path/url
   - type
   - imported_at
   - краткое содержание
   - ключевые идеи
   - практические выводы
   - теги
   - ссылки на source/extract
8. Создай `knowledge/wiki/getting-started.md`.
9. Индексы должны быть markdown и пригодны для чтения агентом.
10. Ошибки не должны валить весь импорт — пиши их в manifest.
11. Все пути в markdown делай относительными к `AI_Knowledge_Base`.

YouTube:
- Определи YouTube URL по `youtube.com` или `youtu.be`.
- Если есть `yt-dlp`, используй его через subprocess:
  - получить metadata JSON;
  - попытаться получить subtitles/auto-subtitles;
  - НЕ скачивать тяжёлое видео по умолчанию.
- Если `yt-dlp` нет или субтитров нет — создай source note и extract note со статусом `needs transcript`.

PDF:
- Если есть `pypdf` — извлеки текст по страницам.
- Если нет — скопируй source и создай fallback extract с текстом, что нужен `pypdf`.

HTML:
- Если есть `beautifulsoup4`/`markdownify` — извлеки title, основной текст, ссылки, iframe/video src.
- Если нет — сделай простой regex fallback без падения.

DOCX:
- Если есть `python-docx` — извлеки параграфы.
- Если нет — fallback extract.

Media:
- MP4/MP3/M4A/WAV/MOV сохраняй в `sources/media/`.
- Если есть `ffmpeg` и whisper/faster-whisper — можешь транскрибировать.
- Если нет — создай extract note: `media saved, transcription needed`.

Folders/courses:
- Рекурсивно просканируй папку.
- Создай manifest со списком файлов, типов, размеров, sha256, статусом.
- Обработай поддерживаемые файлы.
- Неподдерживаемые сохрани как references в manifest.
- Не падай из-за одного плохого файла.

Создай `requirements.txt`:
- yt-dlp
- pypdf
- beautifulsoup4
- python-docx
- markdownify

Создай `.gitignore`:
- .venv/
- __pycache__/
- *.pyc
- .DS_Store
- tmp/
- *.wav
- *.mp3

Создай `README.md` для обычного человека:
1. Что это такое.
2. Что уже настроено автоматически.
3. Куда кидать материалы.
4. Как добавить YouTube.
5. Как добавить PDF.
6. Как добавить папку курса.
7. Как спросить Codex по базе.
8. Как обновить индексы.
9. Что делать, если не работает транскрибация.

ВАЖНО: README может содержать команды на будущее, но первичную настройку ты обязан выполнить сам.

Создай `AGENTS.md`:
- агент всегда сначала читает `index.md` и `indexes/root.md`;
- не удаляет `sources/` без прямой команды;
- если отвечает по базе — указывает источники;
- если данных нет — честно говорит, что в базе нет подтверждения;
- новые материалы сначала сохраняет как source, потом делает extract, потом summary/wiki, потом обновляет indexes;
- не смешивает personas без команды пользователя.

Создай personas:

`personas/default.md`
- коротко, практично, без воды.

`personas/strict-source.md`
- отвечать только по материалам базы;
- не додумывать;
- если нет источника — сказать, что нет подтверждения.

`personas/teacher.md`
- объяснять как преподаватель;
- давать примеры и упражнения.

`personas/strategist.md`
- превращать знания в план действий, продукт, оффер, контент, систему.

АВТОМАТИЧЕСКАЯ УСТАНОВКА:
1. Сам перейди в `AI_Knowledge_Base`.
2. Сам создай `.venv`: `python3 -m venv .venv`.
3. Сам поставь зависимости: `.venv/bin/python -m pip install -r requirements.txt`.
4. Если установка не удалась — продолжай fallback, но запиши причину в `projects/reports/setup-report.md`.
5. Все проверки запускай через `.venv/bin/python`, если venv создан. Иначе через `python3`.

ОБЯЗАТЕЛЬНЫЕ SMOKE-ТЕСТЫ, которые ты должен выполнить сам:

1. `python scripts/kb.py init`
2. `python scripts/kb.py status`
3. Создай файл `inbox/test-note.txt` с текстом:
   `This is a test note about AI knowledge bases and freelance automation.`
4. Создай файл `inbox/test-page.html` с HTML-заголовком, абзацем и ссылкой.
5. `python scripts/kb.py process`
6. `python scripts/kb.py rebuild-index`
7. `python scripts/kb.py search "freelance"`
8. `python scripts/kb.py search "knowledge"`
9. Проверь через shell/find/test, что существуют:
   - `sources/text/`
   - `sources/html/`
   - `extracts/text/`
   - `extracts/html/`
   - `knowledge/summaries/`
   - хотя бы один `projects/imports/*.json`
   - `indexes/root.md`
   - `index.md`
   - `knowledge/wiki/getting-started.md`

Если любой пункт не прошёл — исправь код и повтори smoke-тесты. Не завершай работу до прохождения.

Финальный ответ после успешной проверки:
- максимум 8 строк;
- без длинной теории;
- без “теперь вам нужно установить”;
- только готовый результат.

Формат финального ответа:

Готово.
База: `<absolute_path_to_AI_Knowledge_Base>`
Проверки: `<кратко что прошло>`
Работает: `add / process / status / rebuild-index / search`
Зависимости: `<установлены или fallback + что не встало>`
Дальше: `кидай материалы в inbox или используй kb.py add`
```
