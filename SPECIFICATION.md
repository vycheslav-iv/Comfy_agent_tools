# Comfy Agent Tools — спецификация

Инвентаризация инструментов агентов для работы с ComfyUI: MCP-серверы, панель агента,
локальные модели и конфиги. Обновляется при изменении архитектуры.

Состояние на **2026-09-27**.

---

## 1. MCP-ландшафт

Установлено **три** MCP-сервера для ComfyUI. Это разные пакеты разных авторов — не путать.

| Ключ в конфиге | Пакет | Тип | Автор | Статус | Локация |
|---|---|---|---|---|---|
| `comfyui` | `comfyui-mcp` (npm) | нода + MCP | artokun | **EOL** с 09.10.2026 | npx-кэш |
| `comfy-mcp` | `comfy-mcp` (pip) 0.10.0 + `comfy-cli` 1.21.0 | только MCP | Comfy-Org | **beta**, активен (пуш 20.09.2026) | `F:\Python\Python314` |
| `comfyui` (в kilo.jsonc) | `@miller-joe/comfyui-mcp` 0.3.0 | stdio MCP | Joe Miller | — | npx-кэш `ac63b27a31aa6dfd` |

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

### 1.3. `@miller-joe/comfyui-mcp` — не путать

- Автор Joe Miller, «text-to-image generation via MCP».
- Прописан **только** в `~/.config/kilo/kilo.jsonc`: `npx -y @miller-joe/comfyui-mcp --stdio --comfyui-url http://127.0.0.1:8188`.
- npx-кэш `ac63b27a31aa6dfd` создан 17.09.2026 09:56.
- К панели artokun отношения **не имеет** — более поздняя независимая установка.

---

## 2. Конфиги

| Файл | Ключи `mcp` |
|---|---|
| `C:\Users\dggio\.config\opencode\opencode.jsonc` | `comfyui` (npx artokun), `comfy-mcp` (pip Comfy-Org) |
| `C:\Users\dggio\.config\kilo\kilo.json` | `comfyui` (npx artokun), **`comfy-mcp` (pip Comfy-Org)** + `permission.comfyui_*: allow` |
| `C:\Users\dggio\.config\kilo\kilo.jsonc` | `comfyui` (npx @miller-joe) |

> `kilo.json` (2026-09-20) новее `kilo.jsonc` (2026-09-17) → считается активным; официальный сервер
> добавлен **только** туда (решение 27.09.2026). Бэкап: `kilo.json.bak-20260927`.
> `permission.comfy-mcp_*` **не** добавлял: часть инструментов официального сервера
> (`partner_generate`, `run_template`, `emit_partner_workflow` + `run_workflow`) тратит кредиты —
> пусть Kilo спрашивает. Схема `https://app.kilo.ai/config.json` — 0 ошибок.

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
