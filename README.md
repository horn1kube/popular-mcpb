# popular-mcpb

Коллекция готовых [MCPB](https://github.com/modelcontextprotocol/mcpb)-расширений (`.mcpb`) для Claude Desktop —
по одному пакету на сервис. Расширение одним кликом ставит MCP-сервер в Claude
Desktop, включая форму для ввода кредов пользователя.

## Пакеты

| Папка | Сервис | MCP-сервер | Как запускается |
|---|---|---|---|
| [`atlassian/`](atlassian) | Jira + Confluence | [`mcp-atlassian`](https://github.com/sooperset/mcp-atlassian) | `uvx mcp-atlassian` |
| [`clickhouse/`](clickhouse) | ClickHouse | [`mcp-clickhouse`](https://github.com/ClickHouse/mcp-clickhouse) | `uv run --with mcp-clickhouse ... mcp-clickhouse` |
| [`postgres/`](postgres) | PostgreSQL | [`postgres-mcp`](https://github.com/crystaldba/postgres-mcp) | `uvx --python=3.12 --with "mcp<2" postgres-mcp --access-mode=restricted` |

Каждая папка — самодостаточный пакет: сейчас в ней лежит только `manifest.json`
(без исходников и зависимостей), потому что сам MCP-сервер каждый раз
подтягивается и запускается на лету через `uv`/`uvx`. Поэтому у всех пакетов
на машине, где расширение *работает*, должен быть установлен **uv**:
https://docs.astral.sh/uv/


## Как собрать `.mcpb` для конкретного пакета

`.mcpb` — это обычный ZIP-архив с `manifest.json` внутри (плюс, опционально,
иконка и файлы сервера). Собирать нужно из папки нужного пакета — это даёт
файл `<name>-<version>.mcpb`.

Разово, без установки, для конкретного пакета:

```powershell
cd atlassian    # или cd clickhouse / cd postgres
npx --yes @anthropic-ai/mcpb pack .
```

Команда:
1. валидирует `manifest.json` по MCPB-схеме;
2. валидирует иконку, если она есть (рекомендуемый размер 512×512, PNG);
3. упаковывает содержимое папки пакета в `.mcpb`.

Если хочется не тянуть CLI через `npx` каждый раз, ставится глобально:

```powershell
npm install -g @anthropic-ai/mcpb
mcpb pack .
```

Если в папке уже лежит старый `.mcpb`, стоит удалить его перед пересборкой,
чтобы не перепутать версии:

```powershell
Remove-Item .\*.mcpb -ErrorAction SilentlyContinue
npx --yes @anthropic-ai/mcpb pack .
```

`npm`/`npx` (Node.js) нужны **только на машине, где собирается пакет** — на
машине конечного пользователя Node.js не требуется.

## Установка в Claude Desktop

Готовый `.mcpb`-файл (собранный любым из двух способов выше) ставится любым из
трёх способов:

1. двойной клик по файлу `.mcpb`;
2. перетащить `.mcpb`-файл прямо в окно Claude Desktop;
3. Settings → Extensions → Advanced settings → Install Extension… → выбрать
   файл.

Во всех трёх случаях Claude Desktop сам построит форму настроек по
`user_config` из манифеста — там нужно будет заполнить URL/креды (Jira,
Confluence, ClickHouse, PostgreSQL — в зависимости от пакета). Поля с
`"sensitive": true` маскируются в UI и хранятся через нативное хранилище ОС
(Keychain / Credential Manager / keyring), а не в открытом виде.

Подробности per-пакетных `user_config` — прямо в соответствующем
`manifest.json` (`atlassian/manifest.json`, `clickhouse/manifest.json`,
`postgres/manifest.json`).

## Как добавить новый пакет

1. создать папку `<name>/`;
2. положить туда `manifest.json` по спецификации
   https://github.com/modelcontextprotocol/mcpb/blob/main/MANIFEST.md;
3. опционально — иконку `icon.png` (512×512, PNG);
4. собрать (см. выше) и проверить установку в Claude Desktop.

## Версионирование

При каждом значимом изменении манифеста пакета стоит поднимать его `version`
(semver) — Claude Desktop показывает версию расширения и использует её для
апдейтов при переустановке `.mcpb`.

## Лицензия

[Apache-2.0](LICENSE). Лицензии самих MCP-серверов — свои для каждого
проекта (см. `license` в соответствующем `manifest.json`).
