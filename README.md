# Comfy Agent Tools — README

> **Кому и зачем:** этот файл — для **вас** (что тут как устроено и что делать после
> обновления ComfyUI) и для **агентов** (чеклист до/после обновления).
> Полная карта инструментов — `SPECIFICATION.md`; состояние прошлой сессии — `SESSION_MEMORY.md`.

---

## 1. Что это за папка

Не нода, а инструменты агента **внутри** ComfyUI: панель `comfyui-agent-panel`,
MCP-серверы, бесплатные модели. Лежит в `Custom_node_ComfyUI/`, у самой свой `.git`.

| Файл | Что внутри |
|---|---|
| `README.md` | этот файл: обновление ComfyUI + быстрая проверка |
| `SPECIFICATION.md` | полная инвентаризация: 3 MCP-сервера, конфиги, патчи, §7 — Kilo Gateway |
| `SESSION_MEMORY.md` | что делали в последней сессии (читать первой) |
| `SESSION_MEMORY-history/` | снапшоты памяти по датам |

---

## 2. Обновление ComfyUI — что слетит, а что нет

### ✅ Не слетит никогда (не зависит от ComfyUI)

| Что | Где лежит |
|---|---|
| 3 ключа Kilo (`comfyui-mcp.defaultBackend`, `custom.baseUrl`, `defaultModel.custom`) | `ComfyUI/user/default/comfy.settings.json` — папка `user/` **не в git** |
| Состояние панели и логи | `~/.comfyui-mcp/` (`panel-settings.json`, `launch-logs/`) |
| MCP-конфиги | `~/.config/opencode/opencode.jsonc`, `~/.config/kilo/kilo.json` |
| Скиллы и память | `F:\AI_projects\Custom_node_ComfyUI\` |

**Сама панель** при обновлении **ядра** тоже цела: в `.gitignore` ComfyUI прописано
`/custom_nodes/`, поэтому `git pull` её не трогает.

### ⚠️ Что может слететь

| Риск | Когда | Как спастись |
|---|---|---|
| Локальный патч панели (1 строка — `BACKEND_HINTS.qwen` в `web/js/comfyui-mcp-panel.js`) | **`comfy update all`** — Manager делает `git pull` по панели, где есть незакоммиченное изменение | не давать `update all`; если слетело — вернуть по скиллу `comfyui-agent-model-labels` |
| Патч `qwen-backend.js` (`QWEN_MODELS`, `isQwenModel`) | автообновление npx-кэша `comfyui-mcp@latest` (к ComfyUI отношения не имеет) | тот же скилл; проверка — `grep -c QWEN_MODELS` в обеих копиях кэша |
| Панель ломается на новом фронтенде | обновление ядра меняет JS API | панель **EOL** (последний коммит `docs: archive notice — this project is no longer maintained`) — автора обновлений больше нет |

### ✅ Правило обновления

```bash
comfy update comfy      # только ядро — безопасно, делай
comfy update all        # ⛔ НЕ ДАВАТЬ — обновит и панель, смоет патч
```

**Агенту:** перед любым `update` сделай снапшот локальных правок панели:

```bash
git -C "D:/ComfyUI_windows_portable/ComfyUI/custom_nodes/comfyui-agent-panel" diff > /tmp/panel-patch.txt
```

---

## 3. Быстрая проверка после обновления (Git Bash)

```bash
# 1. ключи Kilo на месте
python -c "import json,io; d=json.load(io.open(r'D:/ComfyUI_windows_portable/ComfyUI/user/default/comfy.settings.json',encoding='utf-8')); print({k:v for k,v in d.items() if 'custom' in k or 'defaultBackend' in k})"

# 2. панель и её локальный патч
git -C "D:/ComfyUI_windows_portable/ComfyUI/custom_nodes/comfyui-agent-panel" status --short

# 3. патч npx-кэша (обе копии, должно быть 2 строки)
grep -c "QWEN_MODELS" C:/Users/dggio/AppData/Local/npm-cache/_npx/*/node_modules/comfyui-mcp/dist/orchestrator/qwen-backend.js

# 4. жив ли бэкенд Kilo
grep -E "tool surface recovered|ready \(openai-compatible" C:/Users/dggio/.comfyui-mcp/launch-logs/connect-orchestrator.log | tail -3
```

Желательный итог шага 4:

```
[ollama-backend] tool surface recovered: 0 → 41 comfyui tools, 96 panel tools
[ollama-backend] ready (openai-compatible @ https://api.kilo.ai/api/gateway, … Tool mode: full)
```

Если строк нет — отправьте в панели любое сообщение (тулы поднимаются на первом ходе,
`never-connected` на старте — ложная тревога, см. `SPECIFICATION.md` §7).

И напоследок — штатная проверка папки:

```bash
python _process/check.py
```

---

## 4. Связанные скиллы

| Скилл | Зачем |
|---|---|
| `comfyui-kilo-gateway` | бесплатный backend панели: эндпоинт, free-модели, ключи, верификация |
| `comfyui-agent-model-labels` | патч npx-кэша и подсказок панели — **восстановление после `update all`** |
| `comfyui-mcp-setup` | три MCP-пакета: какой за что отвечает |
