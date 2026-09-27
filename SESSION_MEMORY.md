# Память сессии — Comfy Agent Tools (2026-09-27, сессия 3)

> Покажи этот файл агенту, чтобы продолжить работу.
> Всегда сверяйся с `AGENTS.md`, `README.md` и `SPECIFICATION.md`.

---

## 1. Что делали в этой сессии (кратко)

Сверили конфиги трёх агентов (opencode, Kilo, Freebuff) на одну пару MCP-серверов
(artokun + официальный Comfy-Org) — отвязали `@miller-joe` из `~/.agents/mcp.json`
(Freebuff), сделали `kilo.jsonc` единственным источником `mcp` в Kilo.
Документально закрепили в `README.md` (новые §4-§6: из чего собрана панель,
готовый промпт для агента после слета патчей) и `SPECIFICATION.md` (§1.3, §2).
Все проверки зелёные.

## 2. Итоговое состояние

**Конфиги MCP (одинаковы во всех агентах):**

| Агент | Файл | Ключи `mcp` |
|---|---|---|
| opencode | `~/.config/opencode/opencode.jsonc` | `comfyui` = npx `comfyui-mcp`, `comfy-mcp` = exe |
| Kilo | `~/.config/kilo/kilo.jsonc` | **только здесь** — `comfyui`, `comfy-mcp` |
| Freebuff/Cursor/Windsurf | `~/.agents/mcp.json` | `comfyui`, `comfy-mcp` (только artokun + official, без miller-joe) |

- `~/.config/kilo/kilo.json` — блок `mcp` удалён (27.09); остались `shell` (Git Bash) + `permission`.
- `~/.config/opencode/opencode.jsonc` — `provider.ollama.name = "Локальные модели (Ollama)"`.
- Бэкапы: `mcp.json.bak-20260927`, `kilo.jsonc.bak-20260927`, `kilo.json.bak-20260927`,
  `kilo.json.bak2-20260927`.

**Гит:** `Comfy_agent_tools/` — `M README.md`, `M SPECIFICATION.md` (commit — по запросу).

**Проверки (27.09.2026):**
- `kilo mcp list` → 2 сервера `connected`
- `opencode mcp list` → 2 сервера `connected`
- `python _process/check.py` → `ЗЕЛЁНОЕ: провалов 0`

**Состояние панели (не стёрлось):**

`comfy_agent_tools/SPECIFICATION.md` §1.1 | актуальное

- `D:/ComfyUI_windows_portable/ComfyUI/custom_nodes/comfyui-agent-panel/` — HEAD
  `v0.15.182-1-g8a4b885d`, патч `M web/js/comfyui-mcp-panel.js`
  (`BACKEND_HINTS.qwen` → `"Локальные модели (Ollama)"`).
- npx-кэш (2 копии `0.52.203`) — патч `qwen-backend.js` в обеих (`QWEN_MODELS.some`).
- `user/default/comfy.settings.json` — `defaultBackend = custom`,
  `custom.baseUrl = https://api.kilo.ai/api/gateway`,
  `defaultModel.custom = nvidia/nemotron-3-super-120b-a12b:free`.

## 3. Проблемы, которые встречались (и как решали)

- **Конфликт `comfyui` в двух kilo-конфигах** (`kilo.jsonc` = miller-joe vs `kilo.json` =
  artokun): Kilo схлопывает по имени ключа и светило 3 записи при 2 живых серверах.
  Решение: весь `mcp` сложить в один файл `kilo.jsonc`, из другого удалить.
  Подробности — `SPECIFICATION.md` §1.3.
- **Freebuff на miller-joe с 22.09** (файл `~/.agents/mcp.json` создан 21.09): заменил
  на artokun + official, остальные `run-state` с 22.09 в истории — нормально.

## 4. Что важно не сломать при продолжении работы

- ⛔ **Только Git Bash**, никакого PowerShell.
- ⛔ **`comfy update comfy`** — можно; **`comfy update all` — НЕ ДАВАТЬ** (смоет патч панели,
  см. README §2 + `comfyui-agent-model-labels`).
- Перед патч-меняй панели делать снапшот `git -C …/comfyui-agent-panel diff > patch.txt`.
- Патч npx-кэша сотрётся при `npx -y comfyui-mcp@latest` и при автоапде выше 60 мин —
  проверять через `comfyui-agent-model-labels` и `grep -c`.
- Скиллы правятся в трёх папках (`.opencode/.kilo/.agents`) + строка в `AGENTS.md` §3.
- Память — только в папке проекта с `.git` (сейчас здесь).

## 5. Следующие шаги (идеи, не сделано)

1. ✅ Сессия 3 завершена: конфиги выровнены, задокументированы, проверено.
2. (Опционально) Если понадобится `refine_image`/`generate_variations`/`upscale_image` —
   вернуть miller-joe как отдельный ключ `comfy-gen` (конфликта не будет).
3. Три ключа панели работают на бесплатном Kilo Gateway (см. сессию 2, SPEC §7).

## 6. Связанные файлы

- `Comfy_agent_tools/README.md` — начинать с §2/§4/§5
- `Comfy_agent_tools/SPECIFICATION.md` (§1.3 конфликты, §1.1 панель, §4 патчи, §7 Gateway)
- `Comfy_agent_tools/SESSION_MEMORY-history/2026-09-27-1208.md`, `-1305.md`
- `~/.config/opencode/opencode.jsonc`, `~/.config/kilo/kilo.jsonc` (источник `mcp`)
- `~/.agents/mcp.json` — Freebuff;
  `D:/ComfyUI_windows_portable/ComfyUI/custom_nodes/comfyui-agent-panel/web/js/comfyui-mcp-panel.js`
