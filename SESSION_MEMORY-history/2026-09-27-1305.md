# Память сессии — Comfy Agent Tools (2026-09-27, сессия 2)

> Покажи этот файл агенту, чтобы продолжить работу.
> Всегда сверяйся с `AGENTS.md`, `README.md` и `SPECIFICATION.md`.

---

## 1. Что делали в этой сессии (кратко)

Перевели панель `comfyui-agent-panel` на **бесплатный бэкенд Kilo Gateway**
(OpenAI-совместимый шлюз **без API-ключа**). OpenCode Zen исследован и **отклонён**
(free tier → 403 только изнутри OpenCode). Настройка доведена до живого прогона:
агент отвечает, 41 comfyui tool + 96 panel tools, `Tool mode: full`.
Создан скилл `comfyui-kilo-gateway` (×3), `README.md` проекта (обновление ComfyUI),
`SPECIFICATION.md` §7. **Репозиторий создан и запушен.**

## 2. Итоговое состояние

**Конфиг панели применён** (`user/default/comfy.settings.json`, читается с точками):

```json
"comfyui-mcp.defaultBackend": "custom",
"comfyui-mcp.custom.baseUrl": "https://api.kilo.ai/api/gateway",
"comfyui-mcp.defaultModel.custom": "nvidia/nemotron-3-super-120b-a12b:free"
```

**Живая верификация** (`~/.comfyui-mcp/launch-logs/connect-orchestrator.log`):

- `custom endpoint config → model=… host=https://api.kilo.ai/api/gateway`
- `tool surface recovered: 0 → 41 comfyui tools, 96 panel tools`
- `ready (openai-compatible @ …, Tool mode: full)` → `turn done`

**Ключевые места в коде** (npx-кэш `_npx\28eae33d00f3f7f1\...\dist\orchestrator\`):

- `index.js` ветка `backend === "custom"` — `OllamaBackend` + те же `mcpServers`, что у ollama
- `index.js` `customDeps()` — `{api:"openai", host, apiKey?}`; **ключ опционален**
- `ollama-backend.js:673` `authHeaders()` — без ключа `authorization` не шлётся;
  `:1146-1147` fetch на `${host}/chat/completions`
- `rate-limit.js` — ретраи **только на 429** (shared free-пул)
- `index.js:4606` — `modelsByBackend.delete("custom")`: **каталог моделей не обновляется
  сам**, перепроба только при смене настроек custom / ключа / падении пробы / рестарте
  оркестратора (F5 не помогает)
- `services/panel-settings.js` — `~/.comfyui-mcp/panel-settings.json`

## 3. Проблемы, которые встречались (и как решали)

- **`never-connected` / `0 tools` при старте** — исходит от **пробного** инстанса
  (у probe нет `mcpServers`); реальный поднимает тулы на первом ходе →
  `tool surface recovered`. Проверять **после** первого сообщения.
- **`/api/status` (порт 9196) не показывает `custom`** — там только CLI-провайдеры. Не поломка.
- **Пользователь не видел, какие модели бесплатные** — панель показывает все 394.
  Правило: суффикс **`:free`** (18 шт.). Без `tools` — `content-safety`; слабая — `lfm-2.6b`;
  `qwen3.8` / `inkling` отдают 429.
- **`/tmp/...` невидим для Windows-Python** — артефакты в
  `C:/Users/dggio/AppData/Local/Temp/opencode/` (`kilo_models.json`).
- **`Comfy_agent_tools` не находится `check.py`** — там только `.md` (нет `check.json`/кода),
  проектом не считается → гонять полный `python _process/check.py`.

## 4. Что важно не сломать при продолжении работы

- ⛔ **Только Git Bash**, никакого PowerShell.
- ⛔ **Не `git init` в корне бандла** (`.no_git`); память — только здесь.
- ⛔ **Не ставить OpenCode Zen** — 403 FreeTierError + авто-доливка $20.
- ⛔ **Не удалять панель** — единственный внутриComfyUI-агент (режим «рядом»).
- ⛔ **`comfy update all` — НЕ ДАВАТЬ** (смоет локальный патч панели); только `comfy update comfy`.
- ⛔ **Не `comfy install`** — только `comfy set-default`.
- ⛔ **Патч npx-кэша хрупкий** — сотрётся при автообновлении `comfyui-mcp@latest`.
- ⛔ **Скиллы правятся в трёх папках сразу** + строка в `AGENTS.md` §3.
- ⛔ **Ключи в `comfy.settings.json` — с точками**; артефакты с `_` (старая версия) не читаются.
- ⛔ **Не амендить коммиты**; перед «готово» — `python _process/check.py`.

## 5. Что сделано / что осталось

**Сделано 27.09.2026:**
- ✅ Kilo Gateway live: `GET /models` → 394 (18 free), `POST /chat/completions` без ключа → 200,
  tool calling 4/4.
- ✅ Три ключа панели записаны через `set_ui`, прочитаны обратно.
- ✅ Живой прогон: `turn done`, 41+96 тулов, `Tool mode: full` (120B ≥ 70B порога).
- ✅ Zen отклонён (`SPECIFICATION.md` §7).
- ✅ Скилл `comfyui-kilo-gateway` ×3 + строка в `AGENTS.md` §3.
- ✅ `Comfy_agent_tools/README.md` — обновление ComfyUI: что слетит/не слетит + 4 команды проверки
  (**все протестированы**); строка в корневом `README.md` обновлена.
- ✅ `SPECIFICATION.md` — шапка с ролями файлов + §7 Kilo Gateway.
- ✅ Git: `b87e9be` README + SPEC §7; память — отдельным коммитом.
- ✅ **Репозиторий создан и запушен:** `https://github.com/vycheslav-iv/Comfy_agent_tools`
  (ветка `master`, запись добавлена в `AGENTS.md` §6.1).

**Осталось (идеи):**
1. Сравнить 39 инструментов официального сервера с реально нужными.
2. Проверить автокомпакт Qwen Code в ACP-режиме панели.
3. Проверить, читает ли Kilo `kilo.json` или `kilo.jsonc` (правка только в `kilo.json`).
4. Положить копии скиллов внутрь репозитория (сейчас только в корне бандла, `.no_git`).
5. Kilo: при 429 переключать модель; пресет переключения free-моделей в Settings.
6. Обновить `SPECIFICATION.md`, если появится новый патч панели.

## 6. Связанные файлы

- `F:\AI_projects\Custom_node_ComfyUI\Comfy_agent_tools\README.md` — **начинать с него**
- `.../SPECIFICATION.md` (§7 — Kilo Gateway), `SESSION_MEMORY.md`, `SESSION_MEMORY-history/`
- `F:\AI_projects\Custom_node_ComfyUI\.opencode\skills\comfyui-kilo-gateway\SKILL.md` (+`.kilo`, `.agents`)
- `F:\AI_projects\Custom_node_ComfyUI\AGENTS.md` — §3 скиллы, §6.1 remote
- `D:\ComfyUI_windows_portable\ComfyUI\user\default\comfy.settings.json` — 3 ключа панели
- `D:\ComfyUI_windows_portable\ComfyUI\custom_nodes\comfyui-agent-panel\` — локальный патч `M web/js/…`
- `C:\Users\dggio\.comfyui-mcp\launch-logs\connect-orchestrator.log` — верификация
- `C:\Users\dggio\AppData\Local\Temp\opencode\kilo_models.json` — 394 модели
- npx-кэш: `_npx\28eae33d00f3f7f1\`, `8af3af54f44861ce\` → `dist/orchestrator/`
