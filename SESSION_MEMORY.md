# Память сессии — Comfy Agent Tools (2026-09-27, сессия 2)

> Покажи этот файл агенту, чтобы продолжить работу.
> Всегда сверяйся с `AGENTS.md` и `SPECIFICATION.md`.

---

## 1. Что делали в этой сессии (кратко)

Перевели панель `comfyui-agent-panel` на **бесплатный бэкенд Kilo Gateway** —
OpenAI-совместимый шлюз без API-ключа. OpenCode Zen исследован и **отклонён**
(free tier даёт 403 только изнутри OpenCode). Настройка доведена до живого прогона:
агент отвечает, тулы подключены (41 comfyui + 96 panel). Создан скилл
`comfyui-kilo-gateway`, обновлены `SPECIFICATION.md` §7 и таблица скиллов `AGENTS.md`.

## 2. Итоговое состояние

**Конфиг панели применён** (`comfy.settings.json`, читается кодом с точками):

```json
"comfyui-mcp.defaultBackend": "custom",
"comfyui-mcp.custom.baseUrl": "https://api.kilo.ai/api/gateway",
"comfyui-mcp.defaultModel.custom": "nvidia/nemotron-3-super-120b-a12b:free"
```

**Живая верификация** (`~/.comfyui-mcp/launch-logs/connect-orchestrator.log`):

- `custom endpoint config → model=… host=https://api.kilo.ai/api/gateway`
- `tool surface recovered: 0 → 41 comfyui tools, 96 panel tools`
- `ready (openai-compatible @ …, Tool mode: full)` → `turn done`

**Ключевые места в коде (npx-кэш `_npx\28eae33d00f3f7f1\...\dist\orchestrator\`):**

- `index.js` ветка `backend === "custom"` — `OllamaBackend` + `mcpServers` (те же stdio-тулы, что у ollama)
- `index.js` `customDeps()` — `{api:"openai", host, apiKey?}`; **ключ опционален**
- `ollama-backend.js:673` `authHeaders()` — заголовок не шлётся без ключа; `:1146-1147` fetch на `${host}/chat/completions`
- `rate-limit.js` — ретраи **только на 429** (shared free-пул)
- `services/panel-settings.js` — `~/.comfyui-mcp/panel-settings.json`

**Скилл `comfyui-kilo-gateway`** — три копии (`.opencode/`, `.kilo/`, `.agents/`),
строка добавлена в `AGENTS.md` §3. ⚠️ лежат в корне бандла (`.no_git`) — версионируются нигде.

## 3. Проблемы, которые встречались (и как решали)

- **`never-connected` / `0 tools` в логе при старте** — пугает, но это **пробный**
  инстанс бэкенда (у probe нет `mcpServers`). Реальный поднимает тулы на первом ходе →
  `tool surface recovered`. Проверять **после** отправки сообщения в панель.
- **`/api/status` (порт 9196) не показывает `custom`** — там только CLI-провайдеры.
  Не признак поломки.
- **Пользователь не понимал, какие модели бесплатные** — панель показывает все 394.
  Правило: суффикс **`:free`** (18 шт.); без `tools` — `content-safety`, слабая — `lfm-2.6b`.
- **Тест `/tmp/...` из Windows-Python не работает** — артефакты в
  `C:/Users/dggio/AppData/Local/Temp/opencode/` (`kilo_models.json`).
- **Закоммичено в сессии 1** — дерево чистое, `git status` пуст.

## 4. Что важно не сломать при продолжении работы

- ⛔ **Только Git Bash**, никакого PowerShell.
- ⛔ **Не `git init` в корне бандла** (`.no_git`); память — только в `Comfy_agent_tools/`.
- ⛔ **Не ставить OpenCode Zen** — 403 FreeTierError + авто-доливка $20.
- ⛔ **Не удалять панель** — она единственный внутриComfyUI-агент (режим «рядом»).
- ⛔ **Не `comfy install`** — только `comfy set-default`.
- ⛔ **Патч npx-кэша хрупкий** — сотрётся при автообновлении `comfyui-mcp@latest`.
- ⛔ **Скиллы правятся в трёх папках сразу** + строка в `AGENTS.md` §3.
- ⛔ **Ключи в `comfy.settings.json` — с точками**, артефакты с `_` (старая версия) не читаются.

## 5. Что сделано / что осталось

**Сделано 27.09.2026:**
- ✅ Kilo Gateway исследован live: `GET /models` → 394 (18 free); `POST /chat/completions`
  без ключа → 200; tool calling 4/4.
- ✅ Три ключа панели записаны через `set_ui`, прочитаны обратно из `comfy.settings.json`.
- ✅ Живой прогон: `turn done`, 41+96 тулов, `Tool mode: full` (модель 120B ≥ 70B порога).
- ✅ Zen отклонён (403/401, документировано в `SPECIFICATION.md` §7).
- ✅ Скилл `comfyui-kilo-gateway` ×3 + `AGENTS.md` §3 + `SPECIFICATION.md` §7.
- ✅ Память сессии 1 снапшотнута в `SESSION_MEMORY-history/2026-09-27.md`.

**Осталось (идеи):**
1. Сравнить 39 инструментов официального сервера с реально нужными.
2. Проверить автокомпакт Qwen Code в ACP-режиме панели.
3. Проверить, читает ли Kilo `kilo.json` или `kilo.jsonc` (правка в `kilo.json`).
4. Опционально: `gh repo create` для этого проекта (команды не было).
5. Опционально: копии скиллов внутрь репозитория (сейчас только в `.no_git`).
6. Возможные улучшения Kilo: при 429 переключать модель, добавить `set_ui`-пресет
   переключения free-моделей.

## 6. Связанные файлы

- `F:\AI_projects\Custom_node_ComfyUI\Comfy_agent_tools\SPECIFICATION.md` (§7 — Kilo)
- `F:\AI_projects\Custom_node_ComfyUI\.opencode\skills\comfyui-kilo-gateway\SKILL.md` (+`.kilo`, `.agents`)
- `F:\AI_projects\Custom_node_ComfyUI\AGENTS.md` — §3 таблица скиллов
- `D:\ComfyUI_windows_portable\ComfyUI\user\default\comfy.settings.json` — 3 ключа панели
- `C:\Users\dggio\.comfyui-mcp\launch-logs\connect-orchestrator.log` — верификация
- `C:\Users\dggio\AppData\Local\Temp\opencode\kilo_models.json` — 394 модели
- npx-кэш: `_npx\28eae33d00f3f7f1\`, `8af3af54f44861ce\` → `dist/orchestrator/`
