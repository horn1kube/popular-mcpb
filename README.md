# ClickHouse MCPB-расширение для Claude

Расширение (`.mcpb`) для Claude Desktop, которое поднимает MCP-сервер `mcp-clickhouse`
для подключения к ClickHouse.

## Состав папки

```
mcp-clickhouse-mcpb/
├── manifest.json     # описание расширения, статические env и поля креденшиалов
├── icon.png           # иконка расширения (512×512, сгенерирована из clickhouse.svg)
├── clickhouse.svg      # исходник иконки (в .mcpb не попадает)
├── .mcpbignore        # список файлов, которые не нужно паковать (clickhouse.svg)
└── mcp-clickhouse-mcpb.mcpb  # собранный пакет — то, что устанавливается в Claude
```

## Требования

- **Node.js/npm** — только на машине, где собираете пакет (нужен для CLI `mcpb`).
- **uv** — на машине, где расширение будет *работать* (устанавливает и запускает
  `mcp-clickhouse` командой `uv run --with mcp-clickhouse --python 3.10 mcp-clickhouse`).
  Ставится отсюда: https://docs.astral.sh/uv/

## Как пересобрать `.mcpb`

Из этой папки:

```powershell
npx --yes @anthropic-ai/mcpb pack .
```

Команда:
1. валидирует `manifest.json` по MCPB-схеме;
2. валидирует иконку (рекомендуемый размер 512×512, PNG);
3. упаковывает всё (кроме файлов из `.mcpbignore`) в `mcp-clickhouse-1.0.0.mcpb` /
   `mcp-clickhouse-mcpb.mcpb` (имя файла CLI берёт из `name`/`version` в манифесте
   либо из имени папки, если паковали через `mcpb pack <dir>` без имени пакета).

Если уже есть старый `.mcpb`-файл, его стоит удалить перед пересборкой, чтобы не
перепутать версии:

```powershell
Remove-Item .\mcp-clickhouse-mcpb.mcpb -ErrorAction SilentlyContinue
npx --yes @anthropic-ai/mcpb pack .
```

Разово CLI можно поставить глобально, чтобы не тянуть его через `npx` каждый раз:

```powershell
npm install -g @anthropic-ai/mcpb
mcpb pack .
```

## Что менять и где

### Статические переменные окружения (зашиты в манифест, пользователь их не видит)

Лежат в `manifest.json → server.mcp_config.env` как обычные строки:

```json
"CLICKHOUSE_SECURE": "true",
"CLICKHOUSE_VERIFY": "true",
"CLICKHOUSE_CONNECT_TIMEOUT": "30",
"CLICKHOUSE_SEND_RECEIVE_TIMEOUT": "30",
"FASTMCP_DISABLE_DOCKET": "true",
"DOCKET_URL": ""
```

Чтобы добавить/поменять статическую переменную — правишь значение прямо тут и
пересобираешь пакет.

### Креды (пользователь вводит их сам в интерфейсе Claude при установке)

Описаны в `manifest.json → user_config` и подставляются в `env` через
`${user_config.<ключ>}`:

| env-переменная         | ключ в `user_config`   | поле в форме Claude |
|-------------------------|--------------------------|----------------------|
| `CLICKHOUSE_HOST`       | `clickhouse_host`        | ClickHouse Host      |
| `CLICKHOUSE_USER`       | `clickhouse_user`        | ClickHouse User      |
| `CLICKHOUSE_PASSWORD`   | `clickhouse_password`    | ClickHouse Password (sensitive) |
| `CLICKHOUSE_PORT`       | `clickhouse_port`        | ClickHouse Port (default 8443) |

`sensitive: true` у пароля — значение маскируется в UI и хранится через нативное
хранилище ОС (Keychain / Credential Manager / keyring), а не в открытом виде.

Чтобы добавить новый вводимый пользователем параметр:
1. добавить поле в `user_config` (`type`, `title`, `description`, `required`/`sensitive`/`default`);
2. сослаться на него в `server.mcp_config.env` как `"VAR": "${user_config.новый_ключ}"`;
3. пересобрать пакет.

### Иконка

Файл `icon.png`, путь на него — `manifest.json → icon`. Рекомендуемый размер —
512×512, формат PNG (SVG для поля `icon` не поддерживается спецификацией).

Перегенерировать из SVG (если меняли `clickhouse.svg`):

```powershell
npx --yes resvg-cli --fit-width 512 clickhouse.svg icon.png
```

Если получившееся изображение не квадратное (зависит от `viewBox` исходного SVG),
нужно доцентрировать его на прозрачном квадратном холсте — например через Pillow:

```python
from PIL import Image
im = Image.open("icon.png").convert("RGBA")
size = 512
canvas = Image.new("RGBA", (size, size), (0, 0, 0, 0))
canvas.paste(im, ((size - im.width) // 2, (size - im.height) // 2), im)
canvas.save("icon.png")
```

## Установка в Claude Desktop

1. Открыть `mcp-clickhouse-mcpb.mcpb` двойным кликом (или через Claude Desktop →
   Settings → Extensions → Install from file).
2. Claude Desktop сам построит форму настроек по `user_config` — заполнить
   Host / User / Password / Port.
3. Сохранить — сервер запустится командой `uv run --with mcp-clickhouse --python 3.10 mcp-clickhouse`
   с заполненными кредами и статическими env-переменными из манифеста.

## Версионирование

При каждом значимом изменении манифеста стоит поднимать `version` в
`manifest.json` (semver) — Claude Desktop показывает версию расширения и
использует её для апдейтов при переустановке `.mcpb`.
