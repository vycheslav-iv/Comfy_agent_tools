# Comfy Agent Tools — спецификация

Инвентаризация инструментов агентов для работы с ComfyUI: MCP-серверы, панель агента,
локальные модели и конфиги. Обновляется при изменении архитектуры.

| Файл | Роль |
|---|---|
| `SPECIFICATION.md` | это — полная карта (архитектура, конфиги, патчи) |
| `README.md` | **сначала сюда**: обновление ComfyUI — что слетит/не слетит, быстрая проверка |
| `SESSION_MEMORY.md` | состояние последней сессии |

Состояние на **2026-09-27** (сессия 2 — бэкенд Kilo Gateway, см. §7;
сессия 3 — приведение конфигов в порядок, см. §1.3 и §2).

---

## 1. MCP-ландшафт

Подключено **два** MCP-сервера — одинаковых во всех трёх агентах (opencode, Kilo, Freebuff).
Третий пакет (`@miller-joe`) **отвязан от конфигов 27.09.2026** (см. §1.3), сам пакет в npx-кэше остался.

| Ключ в конфиге | Пакет | Тип | Автор | Статус | Локация |
|---|---|---|---|---|---|
| `comfyui` | `comfyui-mcp` (npm) | нода + MCP | artokun | **EOL** с 09.10.2026 | npx-кэш |
| `comfy-mcp` | `comfy-mcp` (pip) 0.10.0 + `comfy-cli` 1.21.0 | только MCP | Comfy-Org | **beta**, активен (пуш 20.09.2026) | `F:\Python\Python314` |
| ~~`comfyui`~~ (был в kilo.jsonc и `~/.agents/mcp.json`) | `@miller-joe/comfyui-mcp` 0.3.0 | stdio MCP | Joe Miller | **отвязан везде 27.09.2026** | npx-кэш `ac63b27a31aa6dfd` |

### 1.1. `comfyui-mcp` (artokun) — поставщик панели

- Репозиторий: `github.com/artokun/comfyui-mcp-panel`
- Состоит из двух частей:
  - **npm `comfyui-mcp`** — сам MCP-сервер (control plane + stdio), `npx -y comfyui-mcp`
  - **кастом-нода `comfyui-agent-panel`** — сайдбар-интерфейс в ComfyUI, `pyproject.toml`:
    `PublisherId = "artokun"`, версия `0.15.182`, «UI-only pack: no Python deps»,
    «The sidebar UI for comfyui-mcp»
- **Установлена 13.09.2026 04:09:46** через ComfyUI-Manager (в `.git/.cnr-id` = `comfyui-agent-panel`).
- **Archive-notice стоит с 09.09.2026** (коммит `8a4b885d docs: archive notice — this project is no longer maintained (#2300)`)
  — проект был помечен мёртвым **за 4 дня до установки**. После клона коммитов не было.
- Текущий HEAD: `v0.15.182-1-g8a4b885d`, панель **активна** в `D:\ComfyUI_windows_portable\ComfyUI\custom_nodes\`.
- Единственный интерфейсный хук для **внутриComfyUI-агента** — у Comfy-Org такого нет
  (`Comfy In-App Agent` = private alpha).

### 1.2. `comfy-mcp` (Comfy-Org) — официальный

- Репозиторий: `github.com/Comfy-Org/comfy-mcp`, создан **01.07.2026**, 237★, лицензия AGPL-3.0 / Commercial.
- Документация: `docs.comfy.org/agent-tools` (разделы `local`, `mcp`, `cli`).
- **39 инструментов**; работает как stdio-сервер, каждая команда шеллит `comfy --where local --json`.
- Обязательные зависимости (устанавливаются **вместе**, не включены друг в друга):
  ```bash
  pip install comfy-mcp "comfy-cli>=1.14.0"
  ```
- Двухшаговая привязка к уже существующей ComfyUI (**не** `comfy install` — он создаёт новый workspace):
  ```bash
  comfy set-default "D:/ComfyUI_windows_portable/ComfyUI"
  comfy which   # должен вернуть workspace_path = D:\ComfyUI_windows_portable\ComfyUI
  ```
- Env: `COMFY_BIN` — путь к `comfy.exe` (MCP-клиент запускает сервер со своим PATH и `comfy` не находит).
- Все инструменты живут в разделе README «Tools»; ключевые: `server_info`, `system_stats`,
  `free_memory`, `run_workflow`, `run_template`, `generate_image`, `fetch_outputs`,
  `search_nodes`/`search_models`/`search_templates`, `validate_workflow`.

### 1.3. `@miller-joe/comfyui-mcp` — не путать (отвязан 27.09.2026)

- Автор Joe Miller, «text-to-image generation via MCP», версия 0.3.0.
- **Откуда взялся:** строку записала сама Kilo 17.09.2026 09:54 (формат файла — output
  мастер-а `kilo mcp add`: pretty-print, многострочные массивы, `timeout: 30000`,
  флаги `--stdio --comfyui-url`). npx-кэш `ac63b27a31aa6dfd` создан 17.09 09:56 —
  то есть сервер реально запускался. Истории изменений нет: `~/.config/kilo` — git-репо
  **без единого коммита**.
- **Почему это был конфликт:** ключ `comfyui` существовал и в `kilo.json` (artokun).
  Kilo читает цепочку `["kilo.jsonc","kilo.json","opencode.jsonc","opencode.json","config.json"]`
  (источник: `dist/extension.js` расширения Kilo) и **схлопывает по имени ключа**,
  `kilo.jsonc` приоритетнее → artokun-строка в Kilo мертва, а в UI светились **3** записи.
- **Удалён из `kilo.jsonc` 27.09.2026** → теперь в Kilo ровно 2 сервера, как в opencode.
  К панели artokun отношения никогда не имел — независимая установка.
- **Второй дом — `~/.agents/mcp.json`** (общий конфиг для Freebuff / Cursor / Windsurf):
  туда его тоже прописали, в сессиях Freebuff с **22.09.2026** он инжектится во все
  agent-шаблоны (`base-chat`, `base2`, … — до 50 штук). Файл создан 21.09 06:29.
  **Переведён на ту же пару 27.09.2026** (бэкап `mcp.json.bak-20260927`).
- **Замер `tools/list` (27.09.2026, живые серверы):**
  | Сервер | Тулов | Последний релиз |
  |---|---|---|
  | artokun `comfyui-mcp` 0.52.203 | **41** | 10.09.2026 (EOL 09.10.2026) |
  | miller-joe 0.3.0 | **15** | **25.04.2026** (5 мес. без релизов) |
  | официальный `comfy-mcp` 0.10.0 | **39** | активен (пуш 20.09.2026) |
  Уникальное у miller-joe: `refine_image`, `generate_variations`, `upscale_image`,
  `list_models`, `list_workflows`, реестр шаблонов. **Решение: не возвращать** —
  генерация покрыта официальным сервером, панели нужен artokun; если `refine_image`
  понадобится, добавить отдельным ключом `comfy-gen` (конфликта не будет).

---

## 2. Конфиги

| Файл | Ключи `mcp` |
|---|---|
| `C:\Users\dggio\.config\opencode\opencode.jsonc` | `comfyui` (npx artokun), `comfy-mcp` (pip Comfy-Org) |
| `C:\Users\dggio\.config\kilo\kilo.jsonc` | **`comfyui` (npx artokun), `comfy-mcp` (pip Comfy-Org)** — единственный источник |
| `C:\Users\dggio\.config\kilo\kilo.json` | **`mcp` убран** (27.09.2026); остались `shell` (Git Bash) и `permission.comfyui_*: allow` |
| `C:\Users\dggio\.agents\mcp.json` | **`comfyui` (npx artokun), `comfy-mcp` (exe)** — Freebuff / Cursor / Windsurf; бэкап `mcp.json.bak-20260927` |

> **Все три агента (opencode, Kilo, Freebuff) теперь на одной паре серверов.**
> Freebuff (npm `freebuff` → бинарь `~/.config/manicode/freebuff.exe`, продукт «manicode»)
> читает именно `~/.agents/mcp.json`; проектных `mcp.json` в репозиториях нет.

> **Правило (27.09.2026): один ключ = один файл.** У Kilo два конфига, и раньше в обоих был
> ключ `comfyui` (в `kilo.jsonc` — miller-joe, в `kilo.json` — artokun). Kilo читает
> `["kilo.jsonc","kilo.json", …]` и схлопывает по имени, поэтому `kilo.jsonc` вытеснял
> artokun, а в списке MCP горели **3** строки при 2 работающих серверах.
> Теперь весь блок `mcp` живёт **только в `kilo.jsonc`** (туда же пишет `kilo mcp add`),
> `kilo.json` от него избавлен — дубль невозможен.
> Бэкапы: `kilo.jsonc.bak-20260927`, `kilo.json.bak2-20260927`, `kilo.json.bak-20260927`.
> `permission.comfy-mcp_*` **не** добавлял: часть инструментов официального сервера
> (`partner_generate`, `run_template`, `emit_partner_workflow` + `run_workflow`) тратит кредиты —
> пусть Kilo спрашивает. Схема `https://app.kilo.ai/config.json` — 0 ошибок.
> Проверка после правки: `kilo mcp list` → `comfyui` (npx comfyui-mcp) + `comfy-mcp` (exe),
> оба `connected`, `2 server(s)`.

Запись официального сервера в `opencode.jsonc`:

```jsonc
"comfy-mcp": {
  "type": "local",
  "command": ["F:/Python/Python314/Scripts/comfy-mcp.exe"],
  "environment": { "COMFY_BIN": "F:/Python/Python314/Scripts/comfy.exe" },
  "enabled": true
}
```

**Схема**: `https://opencode.ai/config.json` — валидировать перед коммитом.
`command` — только массив строк; `environment` — объект; `type` обязателен.
Единственная известная ошибка валидации — предсуществующая: `model` (Ollama-модель вне enum схемы).

---

## 3. Панель: происхождение (разбор 27.09.2026)

Вопрос пользователя: «откуда взялась панель?» — ответ собран по хронологии:

| Дата (UTC) | Событие |
|---|---|
| 01.07.2026 | создан `Comfy-Org/comfy-mcp` |
| 09.09.2026 20:33 | artokun ставит archive-notice |
| **13.09.2026 01:09:05** | opencode-сеанс `run=09416059`, dir `F:\AI_projects\Custom_node_ComfyUI` |
| 13.09.2026 01:09:39 | лог: `server unavailable key=comfyui type=local status=failed` |
| **13.09.2026 01:09:46** | `git clone https://github.com/artokun/comfyui-mcp-panel` → `custom_nodes/comfyui-agent-panel` |
| 13.09.2026 01:11:34 / 01:11:41 | `npm view comfyui-mcp version`, `npx -y comfyui-mcp@latest --help` |
| 17.09.2026 09:54 / 09:56 | записан `kilo.jsonc` (miller-joe), создан npx-кэш `ac63b27a31aa6dfd` |
| 21.09.2026 03:29 | выбран `npx -y @miller-joe/comfy-mcp (Рекомендую)` |

**Вывод:** да, панель появилась из запроса «установи MCP сервер для комфи» — агент поставил
артокун-пакет (MCP + UI-нода). Но **не проверил официальную альтернативу**: в логе сеанса
нет ни одного `websearch`/`webfetch` про `comfy-mcp` / `Comfy-Org` / `Comfy Agent`.

---

## 4. Патч npx-кэша (подписи моделей)

Проблема: панель показывала «Qwen3 Coder Plus» — фантомное имя.

Реальность: Ollama на `127.0.0.1:11434`, шесть моделей; панель фильтровала их барьером
`isQwenModel(id) = /^qwen/i`, поэтому из списка оставался только фантом.

Патч применён в **обеих** копиях npx-кэша `dist/orchestrator/qwen-backend.js`
(версии 0.52.203, `node --check` → SYNTAX OK):

- `QWEN_MODELS` — 5 локальных моделей с эмодзи-лейблами;
- `QWEN_DEFAULT_MODEL = "satgeze/gemma4-12b-uncensored-1.5m:latest"`;
- `isQwenModel(id)` = `/^qwen/i.test(id) || QWEN_MODELS.some(m => m.id === id)`;
- `fluffy/magnum-v4-9b` исключён — у него capabilities = `['completion']`, **нет `tools`**.

Плюс `comfyui-mcp-panel.js:33699`: `BACKEND_HINTS.qwen` → `"Локальные модели (Ollama)"`.

Кадр моделей имеет форму `{value, displayName}` (не `{id,label}`).
`applyModelCatalog` сам чинит устаревший выбор: `prefs.model` отсутствует в каталоге →
`prefs.modelAuto = true` → Auto → `QWEN_DEFAULT_MODEL`.

**Хрупкость:** патч стирается при `npx -y comfyui-mcp@latest` / автообновлении
(`self-restart ... checking npm for updates every 60m`).

---

## 5. Окружение

- Windows 11 portable; ComfyUI **0.37.0**, Python **3.13.12**, Node **v24.16.0**.
- GPU: RTX 5060 Ti, **16 ГБ VRAM** (comfy-cli видит `vram_bytes = 17103323136`), RAM 64 ГБ.
- Python для MCP: `F:\Python\Python314` (3.14.5) — **не** `python_embeded` ComfyUI.
  `F:\Python\Python314\Scripts` **не** в PATH → в конфиге используются абсолютные пути.
- Оркестратор: порт **9199**, логи `/tmp/mcp-bridge.log`,
  `C:\Users\dggio\.comfyui-mcp\launch-logs\connect-orchestrator.log`.
- Логи: `C:\Users\dggio\.local\share\opencode\log\opencode.log`.
- **Только Git Bash**, не PowerShell (root `AGENTS.md`).

---

## 6. Как проверить, что всё живо

```bash
# 1. ComfyUI отвечает
curl -s -m 5 http://127.0.0.1:8188/system_stats

# 2. comfy-cli привязан
F:/Python/Python314/Scripts/comfy.exe which
#    -> workspace_path = D:\ComfyUI_windows_portable\ComfyUI

# 3. официальный MCP отвечает рукопожатием (нужен sleep, иначе stdin закроется раньше ответа)
export COMFY_BIN="F:/Python/Python314/Scripts/comfy.exe"
{ printf '%s\n' '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2024-11-05","capabilities":{},"clientInfo":{"name":"probe","version":"1"}}}' \
  '{"jsonrpc":"2.0","method":"notifications/initialized"}' \
  '{"jsonrpc":"2.0","id":2,"method":"tools/call","params":{"name":"server_info","arguments":{}}}'; sleep 25; } \
  | F:/Python/Python314/Scripts/comfy-mcp.exe

# 4. валидация конфига opencode
curl -s https://opencode.ai/config.json -o /tmp/oc-schema.json
# + jsonschema.Validator против ~/.config/opencode/opencode.jsonc
```

Ожидаемо: `serverInfo = comfy-mcp`, 39 tools, `server_info` → `isError: false`.

## 7. Бэкенд панели: Kilo Gateway (бесплатные модели, 27.09.2026)

Панель работает на бэкенде `custom` (OpenAI-совместимый шлюз **без API-ключа**):

| Ключ в `comfy.settings.json` | Значение |
|---|---|
| `comfyui-mcp.defaultBackend` | `custom` |
| `comfyui-mcp.custom.baseUrl` | `https://api.kilo.ai/api/gateway` |
| `comfyui-mcp.defaultModel.custom` | `nvidia/nemotron-3-super-120b-a12b:free` |

Альтернатива (env, приоритет выше json): `COMFYUI_MCP_CUSTOM_BASE_URL/_MODEL/_API_KEY`;
хранение — `~/.comfyui-mcp/panel-settings.json`, dotenv `~/.comfyui-mcp/.env`.
Бэкенд применяется на **F5**, без перезапуска ComfyUI.

**Каталог:** `GET …/models` → 394 модели, **18 free** (суффикс `:free`).
Tool calling подтверждён 4/4 (`nemotron-3-super-120b`, `north-mini-code`,
`laguna-xs-2.1`, `step-3.7-flash`). Без `tools` — `nemotron-3.5-content-safety`.
⚠️ все free: `mayTrainOnYourPrompts: True`; 429 на общем пуле ретраит `rate-limit.js`.

**Исследовано и отклонено — OpenCode Zen:** free-модель без ключа →
**403 `FreeTierError`** «free tier can only be used from within OpenCode»;
платная без ключа → 401. Плюс auto-reload баланса <$5 → +$20.

**Верификация** (`~/.comfyui-mcp/launch-logs/connect-orchestrator.log`):

```
[panel-orchestrator] custom endpoint config → model=… host=https://api.kilo.ai/api/gateway
[ollama-backend] tool surface recovered: 0 → 41 comfyui tools, 96 panel tools
[ollama-backend] ready (openai-compatible @ https://api.kilo.ai/api/gateway, … Tool mode: full)
[panel-agent orchestr] turn done
```

⚠️ **Ложная тревога:** на старте сессии `never-connected` / `0 tools` — исходит от
**пробного** инстанса бэкенда (без `mcpServers`); агентный получает те же stdio-тулы,
что и ollama (`index.js`, ветка `backend === "custom"`), и поднимает их на первом ходе.

Подробнее — скилл `comfyui-kilo-gateway` (корень бандла, `.opencode|.kilo|.agents`).
