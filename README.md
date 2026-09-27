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
| `README.md` | этот файл: обновление ComfyUI + быстрая проверка + **§5 переустановка панели** + **§6 промпт для агента** |
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
| Локальный патч панели (1 строка — `BACKEND_HINTS.qwen` в `web/js/comfyui-mcp-panel.js`) | **`comfy update all`** — Manager делает `git pull` по панели, где есть незакоммиченное изменение | не давать `update all`; если слетело — **§5** (готовый промпт и команды) |
| Патч `qwen-backend.js` (`QWEN_MODELS`, `isQwenModel`) | автообновление npx-кэша `comfyui-mcp@latest` (к ComfyUI отношения не имеет) | **§5**; проверка — `grep -c QWEN_MODELS` в обеих копиях кэша |
| Сама нода-панель `comfyui-agent-panel` | чистая переустановка ComfyUI / `comfy node uninstall` | `comfy node install comfyui-agent-panel` — см. **§4** |
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

## 4. Панель агента и MCP-сервер — из чего собраны (для переустановки)

### 4.1. Сначала главное: нода и сервер — это РАЗНЫЕ вещи

Частый вопрос: «какая нода делает artokun MCP-сервер?» — **никакая**. Сервер — это не
нода, а отдельная npm-программа. Панель — отдельная кастом-нода. Вот полный список:

| Что это | Название (именно его вводить) | Где лежит | Как поставить заново |
|---|---|---|---|
| ⭐ **Кастомная нода «Панель агента»** (сайдбар в ComfyUI) | **`comfyui-agent-panel`** (Comfy Registry, publisher `artokun`, версия `0.15.182`) | `D:\ComfyUI_windows_portable\ComfyUI\custom_nodes\comfyui-agent-panel\` | `comfy node install comfyui-agent-panel` |
| **MCP-сервер artokun** (41 тул) + **оркестратор панели** | npm-пакет **`comfyui-mcp`** `0.52.203` | npx-кэш, скачивается сам | ничего ставить не надо: `npx -y comfyui-mcp` сам его скачает |
| **MCP-сервер официальный Comfy-Org** (39 тулов) | pip-пакет **`comfy-mcp`** `0.10.0` | `F:\Python\Python314\Scripts\comfy-mcp.exe` | `pip install -U comfy-mcp` |

**Других кастомных нод для агента/сервера нет.** Нода `comfyui-agent-panel` не содержит
Python-кода (`NODE_CLASS_MAPPINGS = {}`) — поэтому её **не будет в меню Add Node**,
она живёт в **сайдбаре** (правая панель ComfyUI). Оркестратор — это не нода, а команда
`npx -y comfyui-mcp --panel-orchestrator` (порт **9199**); нода его **не запускает**
(запрет реестра), а только проверяет, что он жив.

### 4.2. Поставить всё с нуля (после переустановки ComfyUI)

```bash
# 1) нода-панель
comfy node install comfyui-agent-panel

# 2) MCP-серверы — их конфиги в ~/.config НЕ пострадали, просто проверяем:
kilo mcp list        # должно быть: comfyui + comfy-mcp, оба connected
opencode mcp list    # то же самое

# 3) если сервера пропали — вернуть запись в ~/.config/kilo/kilo.jsonc:
#    "comfyui":    npx -y comfyui-mcp
#    "comfy-mcp":  F:/Python/Python314/Scripts/comfy-mcp.exe  (+ env COMFY_BIN)

# 4) обязательно вернуть наши ручные правки — см. §5
```

> ⛔ **После любого обновления панели наши две правки слетают** (§2). Поэтому шаг 4
> не пропускать.

---

## 5. Промпт для ИИ-агента: «слетели заплатки, восстанови»

**Скопируй и вставь агенту целиком** после `comfy update all`, после переустановки
ComfyUI или если панель ведёт себя странно. Язык простой, агент делает всё сам.

```text
Привет. Я обновил (или переустановил) ComfyUI. Возможно, слетели мои ручные правки
панели агента. Проведи проверку и восстановление.

Условия:
- Работай ТОЛЬКО через Git Bash (это Windows, PowerShell запрещён).
- Проект: F:\AI_projects\Custom_node_ComfyUI
- ComfyUI: D:\ComfyUI_windows_portable\ComfyUI
- Ничего не коммить и не пушить без моей просьбы. Не удаляй папки .git.
- ЗАПРЕЩЕНО: comfy update all (он смоет мои правки).

Шаги, по порядку:

1. Прочитай F:\AI_projects\Custom_node_ComfyUI\Comfy_agent_tools\README.md,
   разделы 4 и 5, и загрузи скиллы comfyui-agent-model-labels и comfyui-mcp-setup.

2. Проверь ноду-панель:
   ls "D:/ComfyUI_windows_portable/ComfyUI/custom_nodes/comfyui-agent-panel"
   Если папки нет — выполни `comfy node install comfyui-agent-panel`, скажи мне
   перезапустить ComfyUI и остановись, вернись после рестарта.

3. Проверь первую правку (подпись бэкенда в панели):
   grep -c 'Локальные модели (Ollama)' "D:/ComfyUI_windows_portable/ComfyUI/custom_nodes/comfyui-agent-panel/web/js/comfyui-mcp-panel.js"
   Если 0 — верни её по инструкции из README §5.2 (замена одной строки:
   "Qwen3 Coder · Coding Plan" -> "Локальные модели (Ollama)" в строке ~33699).
   Если замена не находит строку — обновился исходник, остановись и скажи мне,
   не придумывай похожие правки.

4. Проверь вторую правку (список моделей в npx-кэше), обе копии, должно быть 2 строки:
   grep -c "QWEN_MODELS.some" C:/Users/dggio/AppData/Local/npm-cache/_npx/*/node_modules/comfyui-mcp/dist/orchestrator/qwen-backend.js
   Если 0 или нашлась только одна копия — загрузи скилл comfyui-agent-model-labels
   и восстанови три точки в ОБЕИХ копиях: QWEN_MODELS (список только из моделей
   с capability tools), QWEN_DEFAULT_MODEL (satgeze/gemma4-12b-uncensored-1.5m:latest),
   isQwenModel (должен пускать и /^qwen/i, и любую id из QWEN_MODELS).
   После правки проверь синтаксис: node --check <файл> (без вывода = OK).

5. Проверь настройки панели (их обновление не трогает, но убедимся):
   python -c "import json,io; d=json.load(io.open(r'D:/ComfyUI_windows_portable/ComfyUI/user/default/comfy.settings.json',encoding='utf-8')); print({k:v for k,v in d.items() if 'comfyui-mcp' in k})"
   Должны быть ключи: defaultBackend, custom.baseUrl (https://api.kilo.ai/api/gateway),
   custom.model, bridgeUrl.claude. Если их нет — это из §7 SPECIFICATION.md (Kilo Gateway).

6. Прогони проверку проекта и покажи мне её вывод целиком:
   cd F:/AI_projects/Custom_node_ComfyUI && python _process/check.py

7. Скажи мне итог одной строкой: панель на месте / правки восстановлены /
   что-то не получилось и что именно. И напомни перезапустить ComfyUI, если
   трогал ноду.

Что НЕ нужно восстанавливать (оно не слетает): конфиги opencode и Kilo
(~/.config/...), ~/.agents/mcp.json (Freebuff), скиллы и память сессии.
```

### 5.2. Те же две правки — одной командой (если агент предпочитает сам)

```bash
# Правка 1: панель (одна строка, замена идемпотентна)
python - <<'EOF'
p = "D:/ComfyUI_windows_portable/ComfyUI/custom_nodes/comfyui-agent-panel/web/js/comfyui-mcp-panel.js"
s = open(p, encoding="utf-8").read()
old = 'qwen: "Qwen3 Coder · Coding Plan"'
new = 'qwen: "Локальные модели (Ollama)"'
if new in s:
    print("patch 1: already applied")
elif old in s:
    open(p, "w", encoding="utf-8").write(s.replace(old, new, 1))
    print("patch 1: applied")
else:
    print("patch 1: SOURCE CHANGED - manual fix needed, see skill")
EOF

# Правка 2: проверка npx-кэша (обе копии, ожидаем 2)
grep -c "QWEN_MODELS.some" C:/Users/dggio/AppData/Local/npm-cache/_npx/*/node_modules/comfyui-mcp/dist/orchestrator/qwen-backend.js
# 0 -> восстанавливать по скиллу comfyui-agent-model-labels
```

---

## 6. Связанные скиллы

| Скилл | Зачем |
|---|---|
| `comfyui-kilo-gateway` | бесплатный backend панели: эндпоинт, free-модели, ключи, верификация |
| `comfyui-agent-model-labels` | патч npx-кэша и подсказок панели — **восстановление после `update all`** |
| `comfyui-mcp-setup` | три MCP-пакета: какой за что отвечает |
