# Память сессии — Comfy Agent Tools (2026-09-27)

> Покажи этот файл агенту, чтобы продолжить работу.
> Всегда сверяйся с `AGENTS.md` и `SPECIFICATION.md`.

---

## 1. Что делали в этой сессии (кратко)

Выяснили происхождение панели `comfyui-agent-panel` (её установил агент 13.09.2026 по запросу
«установи MCP сервер для комфи» вместо официального `Comfy-Org/comfy-mcp`), затем поставили
официальный сервер **рядом** и вынесли всё в отдельную папку проекта.

## 2. Итоговое состояние

- `SPECIFICATION.md` — полная инвентаризация: 3 MCP-сервера, хронология установки панели,
  патч npx-кэша, окружение, команды проверки. **Начинать с него.**
- `SESSION_MEMORY.md` — этот файл.
- git-репозиторий создан (корень проекта = папка с `.git`, root `AGENTS.md` §9).

### Установлено в этой сессии

- `F:\Python\Python314`: `comfy-mcp 0.10.0` + `comfy-cli 1.21.0`
  (побочный эффект: `typing_extensions 4.15.0 → 4.16.0`)
- `comfy set-default "D:/ComfyUI_windows_portable/ComfyUI"` — привязка к существующей ComfyUI,
  `comfy install` **не** запускался
- `~/.config/opencode/opencode.jsonc` → добавлен `mcp."comfy-mcp"` рядом с `"comfyui"`
  (схема: `https://opencode.ai/config.json`, валидация чистая)
- `~/.config/kilo/kilo.json` → добавлен `mcp."comfy-mcp"` (бэкап `kilo.json.bak-20260927`,
  схема `https://app.kilo.ai/config.json` — 0 ошибок)
- скиллы `comfyui-mcp-setup` и `comfyui-agent-model-labels` — по правилу совместимости в
  **трёх** местах: `.opencode/skills/`, `.kilo/skills/`, `.agents/skills/` (6 файлов, валидны).
  ⚠️ **Они лежат в корне бандла, который `.no_git` — версионируются нигде.**
  Память и `SPECIFICATION.md` — в этом репозитории; скиллы — только на диске.
- корневой `README.md` — `Comfy_agent_tools` добавлен в карту «Что где лежит»

## 3. Проблемы, которые встречались (и как решали)

- **Панель показывала «Qwen3 Coder Plus»** — барьер `isQwenModel(/^qwen/i)` отсекал локальные
  Ollama-модели → патч `qwen-backend.js` в обеих копиях npx-кэша.
- **`comfy-mcp --version` не печатает ничего** — это stdio-сервер; проверять только через
  рукопожатие MCP, и **держать stdin открытым** (`sleep 25`), иначе ответ не придёт.
- **`/tmp/...` невидим для Windows-Python** — сохранять артефакты в
  `C:/Users/dggio/AppData/Local/Temp/opencode/`.
- **Валидация конфига даёт 1 ошибку** — предсуществующая: `model` вне enum схемы. К правкам не относится.

## 4. Что важно не сломать при продолжении работы

- ⛔ **Только Git Bash** (root `AGENTS.md`), никакого PowerShell.
- ⛔ **Не `git init` в корне бандла** — там `.no_git`; память только в папке с `.git`.
- ⛔ **Не удалять панель** `comfyui-agent-panel` без отдельного разрешения — пользователь выбрал
  режим «рядом»; панель = единственный внутриComfyUI-агент.
- ⛔ **Не `comfy install`** — он создаст второй workspace. Только `comfy set-default`.
- ⛔ **Патч npx-кэша хрупкий** — сотрётся при `comfyui-mcp@latest`; после автообновления перепроверить
  `dist/orchestrator/qwen-backend.js` (`QWEN_MODELS`, `QWEN_DEFAULT_MODEL`, `isQwenModel`).
- ⛔ **Не путать три MCP-пакета**: artokun `comfyui-mcp` ≠ Comfy-Org `comfy-mcp` ≠ miller-joe `@miller-joe/comfyui-mcp`.
- ⛔ **`F:\Python\Python314\Scripts` не в PATH** — в конфигах только абсолютные пути + `COMFY_BIN`.
- ⛔ **Скиллы правятся во всех трёх папках сразу** (`.opencode/`, `.kilo/`, `.agents/`) — иначе
  Kilo Code / Cursor не увидят изменение (правило совместимости, root `AGENTS.md`).
- ⛔ **Перед правкой sizing/CSS — читать исходники фронтенда** (скилл `comfyui-frontend-sources`).

## 5. Что сделано / что осталось

**Сделано 27.09.2026 (после перезапуска opencode):**
- ✅ Перезапуск opencode → **оба сервера подключены**: `comfyui` (artokun) + `comfy-mcp` (официальный).
  `server_info` отвечает: workspace `D:\ComfyUI_windows_portable\ComfyUI`, comfy-cli 1.21.0,
  envelope/1, видит RTX 5060 Ti 16 ГБ. Оба новых скилла загружены.
- ✅ `kilo.json` → добавлен `mcp."comfy-mcp"` (решение пользователя: **в kilo.json**, не в `.jsonc`).
  Бэкап `kilo.json.bak-20260927`; схема `https://app.kilo.ai/config.json` — 0 ошибок.
  `permission.comfy-mcp_*` **не** ставил (см. `SPECIFICATION.md` §2).
- ✅ Панель artokun — решение: **пока оставить** (EOL 09.10.2026, но это единственный
  внутриComfyUI-агент; у официального сервера `In-App Agent` = private alpha).

**Осталось (идеи, не сделано):**
1. Сравнить 39 инструментов официального сервера с тем, что реально нужно.
2. Проверить автокомпакт Qwen Code в ACP-режиме панели.
3. Проверить, читает ли Kilo `kilo.json` или `kilo.jsonc` (официальный сервер добавлен только в
   `kilo.json` — если активен `.jsonc`, правку надо перенести).
4. ⚠️ `server_info` показал `running: false` и `freshness.error` (таймаут `comfy outdated` 15с) —
   ComfyUI сейчас не запущен; проверить, когда понадобится.
5. Опционально: `gh repo create` для этого проекта (команда пользователя не было).
6. Опционально: положить копии скиллов внутрь репозитория — сейчас они только в корне бандла (`.no_git`).

## 6. Связанные файлы

- `F:\AI_projects\Custom_node_ComfyUI\Comfy_agent_tools\SPECIFICATION.md`
- `C:\Users\dggio\.config\opencode\opencode.jsonc` — `mcp.comfyui`, `mcp.comfy-mcp`
- `C:\Users\dggio\.config\kilo\kilo.json`, `kilo.jsonc`
- `F:\Python\Python314\Scripts\{comfy-mcp.exe,comfy.exe}`
- `D:\ComfyUI_windows_portable\ComfyUI\custom_nodes\comfyui-agent-panel\`
- npx-кэш: `...\_npx\28eae33d00f3f7f1\` и `8af3af54f44861ce\` → `dist/orchestrator/qwen-backend.js`
- `C:\Users\dggio\.local\share\opencode\log\opencode.log` — сеанс `run=09416059` (установка панели)
